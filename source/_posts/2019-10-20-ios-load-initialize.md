---
title: load和initialize方法
tags: [iOS]
date: 2019-10-20 22:38:09
updated: 2019-10-20 22:38:09
categories: iOS
---

我们都知道，iOS的类中，有两个方法`load`和`initialize`，load方法在程序启动的时候就会执行，而initialize方法在类第一次调用的时候执行，这里我们从源码的角度探究一下这两个方法的原理和实现

<!-- more -->

我们下载[objc4](https://opensource.apple.com/tarballs/objc4/)运行时源码，下载最新的版本`objc4-779.1.tar.gz`，在`objc-os.mm`找到`_objc_init`方法，这个函数是OC运行时的入口函数，整个OC运行时从这里开始加载

```objc

/***********************************************************************
* _objc_init
* Bootstrap initialization. Registers our image notifier with dyld.
* Called by libSystem BEFORE library initialization time
**********************************************************************/

void _objc_init(void)
{
    static bool initialized = false;
    if (initialized) return;
    initialized = true;

    // fixme defer initialization until an objc-using image is found?
    environ_init();
    tls_init();
    static_init();
    runtime_init();
    exception_init();
    cache_init();
    _imp_implementationWithBlock_init();

    _dyld_objc_notify_register(&map_images, load_images, unmap_image);
}
```

## load

前面是一些初始化方法，最后一个`_dyld_objc_notify_register`是向`dyld`注册三个事件，我们这里关注`load_images`事件

```objc
void
load_images(const char *path __unused, const struct mach_header *mh)
{
    // Return without taking locks if there are no +load methods here.
    if (!hasLoadMethods((const headerType *)mh)) return;

    recursive_mutex_locker_t lock(loadMethodLock);

    // Discover load methods
    {
        mutex_locker_t lock2(runtimeLock);
        prepare_load_methods((const headerType *)mh);
    }

    // Call +load methods (without runtimeLock - re-entrant)
    call_load_methods();
}

void call_load_methods(void)
{
    static bool loading = NO;
    bool more_categories;

    loadMethodLock.assertLocked();

    // Re-entrant calls do nothing; the outermost call will finish the job.
    if (loading) return;
    loading = YES;

    void *pool = objc_autoreleasePoolPush();

    do {
        // 1. Repeatedly call class +loads until there aren't any more
        while (loadable_classes_used > 0) {
            call_class_loads();
        }

        // 2. Call category +loads ONCE
        more_categories = call_category_loads();

        // 3. Run more +loads if there are classes OR more untried categories
    } while (loadable_classes_used > 0  ||  more_categories);

    objc_autoreleasePoolPop(pool);

    loading = NO;
}
```

可以看出，load方法的加载顺序，先加载类的load方法，再加载Category里面的load方法

进入`call_class_loads`方法

```objc
static void call_class_loads(void)
{
    int i;

    // Detach current loadable list.
    struct loadable_class *classes = loadable_classes;
    int used = loadable_classes_used;
    loadable_classes = nil;
    loadable_classes_allocated = 0;
    loadable_classes_used = 0;

    // Call all +loads for the detached list.
    for (i = 0; i < used; i++) {
        Class cls = classes[i].cls;
        load_method_t load_method = (load_method_t)classes[i].method;
        if (!cls) continue;

        if (PrintLoading) {
            _objc_inform("LOAD: +[%s load]\n", cls->nameForLogging());
        }
        (*load_method)(cls, @selector(load));
    }

    // Destroy the detached list.
    if (classes) free(classes);
}
```

这里拿到了`loadable_classes`这个是带有`load`方法的类列表，并且可以直接拿到load方法的`imp`直接调用，**这里没有使用objc_msgSend机制调用方法，而是直接拿到IMP直接调用**，签名如下

```c
typedef void(*load_method_t)(id, SEL);
```

在`prepare_load_methods`方法会初始化所有的类的加载方法，遍历所有的类，如果类实现了`load`方法，就会构造`loadable_class`添加到`loadable_classes`中

```objc

void prepare_load_methods(const headerType *mhdr)
{
    Module mods;
    unsigned int midx;

    header_info *hi;
    for (hi = FirstHeader; hi; hi = hi->getNext()) {
        if (mhdr == hi->mhdr()) break;
    }
    if (!hi) return;

    if (hi->info()->isReplacement()) {
        // Ignore any classes in this image
        return;
    }

    // Major loop - process all modules in the image
    mods = hi->mod_ptr;
    for (midx = 0; midx < hi->mod_count; midx += 1)
    {
        unsigned int index;

        // Skip module containing no classes
        if (mods[midx].symtab == nil)
            continue;

        // Minor loop - process all the classes in given module
        for (index = 0; index < mods[midx].symtab->cls_def_cnt; index += 1)
        {
            // Locate the class description pointer
            Class cls = (Class)mods[midx].symtab->defs[index];
            if (cls->info & CLS_CONNECTED) {
                schedule_class_load(cls);
            }
        }
    }

    // Major loop - process all modules in the header
    mods = hi->mod_ptr;

    // NOTE: The module and category lists are traversed backwards 
    // to preserve the pre-10.4 processing order. Changing the order 
    // would have a small chance of introducing binary compatibility bugs.
    midx = (unsigned int)hi->mod_count;
    while (midx-- > 0) {
        unsigned int index;
        unsigned int total;
        Symtab symtab = mods[midx].symtab;

        // Nothing to do for a module without a symbol table
        if (mods[midx].symtab == nil)
            continue;
        // Total entries in symbol table (class entries followed
        // by category entries)
        total = mods[midx].symtab->cls_def_cnt +
            mods[midx].symtab->cat_def_cnt;

        // Minor loop - register all categories from given module
        index = total;
        while (index-- > mods[midx].symtab->cls_def_cnt) {
            old_category *cat = (old_category *)symtab->defs[index];
            add_category_to_loadable_list((Category)cat);
        }
    }
}

static void schedule_class_load(Class cls)
{
    if (!cls) return;
    ASSERT(cls->isRealized());  // _read_images should realize

    if (cls->data()->flags & RW_LOADED) return;

    // Ensure superclass-first ordering
    schedule_class_load(cls->superclass);

    add_class_to_loadable_list(cls);
    cls->setInfo(RW_LOADED);
}

void add_class_to_loadable_list(Class cls)
{
    IMP method;

    loadMethodLock.assertLocked();

    method = cls->getLoadMethod();
    if (!method) return;  // Don't bother if cls has no +load method

    if (PrintLoading) {
        _objc_inform("LOAD: class '%s' scheduled for +load", 
                     cls->nameForLogging());
    }

    if (loadable_classes_used == loadable_classes_allocated) {
        loadable_classes_allocated = loadable_classes_allocated*2 + 16;
        loadable_classes = (struct loadable_class *)
            realloc(loadable_classes,
                              loadable_classes_allocated *
                              sizeof(struct loadable_class));
    }

    loadable_classes[loadable_classes_used].cls = cls;
    loadable_classes[loadable_classes_used].method = method;
    loadable_classes_used++;
}
```

我们在`schedule_class_load`方法看到，这里面做了一次递归，也就是如果存在父类，会先添加父类，所以**父类的load方法会比子类先执行**

可以看出，load方法的加载顺序

1. 父类的load方法
2. 再加载子类的load方法（加载顺序为编译顺序）
3. 再加载category里面的load方法（加载顺序为编译顺序）

## initialize

由于`initialize`是在调用的时候才执行的，我们调试一下看看

```objc
@interface ABC : NSObject

@end

@implementation ABC

+ (void)initialize {
    // 添加断点
    NSLog(@"%@", @"ass");
}

@end


@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [ABC alloc];
}

@end

```

在initialize添加断点，查看调用栈

```sh
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x0000000104911f20 Test`+[ABC initialize](self=ABC, _cmd="initialize") at ViewController.m:19:5
    frame #1: 0x00000001818bb6f0 libobjc.A.dylib`CALLING_SOME_+initialize_METHOD + 20
    frame #2: 0x00000001818c1074 libobjc.A.dylib`initializeNonMetaClass + 640
    frame #3: 0x00000001818c2318 libobjc.A.dylib`initializeAndMaybeRelock(objc_class*, objc_object*, mutex_tt<false>&, bool) + 284
    frame #4: 0x00000001818cf074 libobjc.A.dylib`lookUpImpOrForward + 732
    frame #5: 0x00000001818bbe9c libobjc.A.dylib`_objc_msgSend_uncached + 60
    frame #6: 0x0000000104911f9c Test`-[ViewController viewDidLoad](self=0x0000000104e0cf90, _cmd="viewDidLoad") at ViewController.m:33:5
    ...
```

可以看到，在调用`init`方法的时候，调用了`objc_msgSend_uncached`，然后，走到了`lookUpImpOrForward`方法，我们在objc4源码中查看该方法

![_](/images/post/objc_msgSend_uncached.png)

在`_objc_msgSend_uncached`方法调用了`MethodTableLookup`

![_](/images/post/MethodTableLookup.png)

在`MethodTableLookup`方法调用了`lookUpImpOrForward`，并且behavior传了`LOOKUP_INITIALIZE | LOOKUP_RESOLVER`

```objc
IMP lookUpImpOrForward(id inst, SEL sel, Class cls, int behavior)
{
    const IMP forward_imp = (IMP)_objc_msgForward_impcache;
    IMP imp = nil;
    Class curClass;

    runtimeLock.assertUnlocked();

    // Optimistic cache lookup
    if (fastpath(behavior & LOOKUP_CACHE)) {
        imp = cache_getImp(cls, sel);
        if (imp) goto done_nolock;
    }

    // runtimeLock is held during isRealized and isInitialized checking
    // to prevent races against concurrent realization.

    // runtimeLock is held during method search to make
    // method-lookup + cache-fill atomic with respect to method addition.
    // Otherwise, a category could be added but ignored indefinitely because
    // the cache was re-filled with the old value after the cache flush on
    // behalf of the category.

    runtimeLock.lock();

    // We don't want people to be able to craft a binary blob that looks like
    // a class but really isn't one and do a CFI attack.
    //
    // To make these harder we want to make sure this is a class that was
    // either built into the binary or legitimately registered through
    // objc_duplicateClass, objc_initializeClassPair or objc_allocateClassPair.
    //
    // TODO: this check is quite costly during process startup.
    checkIsKnownClass(cls);

    if (slowpath(!cls->isRealized())) {
        cls = realizeClassMaybeSwiftAndLeaveLocked(cls, runtimeLock);
        // runtimeLock may have been dropped but is now locked again
    }

    if (slowpath((behavior & LOOKUP_INITIALIZE) && !cls->isInitialized())) {
        cls = initializeAndLeaveLocked(cls, inst, runtimeLock);
        // runtimeLock may have been dropped but is now locked again

        // If sel == initialize, class_initialize will send +initialize and
        // then the messenger will send +initialize again after this
        // procedure finishes. Of course, if this is not being called
        // from the messenger then it won't happen. 2778172
    }

    runtimeLock.assertLocked();
    curClass = cls;

    // The code used to lookpu the class's cache again right after
    // we take the lock but for the vast majority of the cases
    // evidence shows this is a miss most of the time, hence a time loss.
    //
    // The only codepath calling into this without having performed some
    // kind of cache lookup is class_getInstanceMethod().

    for (unsigned attempts = unreasonableClassCount();;) {
        // curClass method list.
        Method meth = getMethodNoSuper_nolock(curClass, sel);
        if (meth) {
            imp = meth->imp;
            goto done;
        }

        if (slowpath((curClass = curClass->superclass) == nil)) {
            // No implementation found, and method resolver didn't help.
            // Use forwarding.
            imp = forward_imp;
            break;
        }

        // Halt if there is a cycle in the superclass chain.
        if (slowpath(--attempts == 0)) {
            _objc_fatal("Memory corruption in class list.");
        }

        // Superclass cache.
        imp = cache_getImp(curClass, sel);
        if (slowpath(imp == forward_imp)) {
            // Found a forward:: entry in a superclass.
            // Stop searching, but don't cache yet; call method
            // resolver for this class first.
            break;
        }
        if (fastpath(imp)) {
            // Found the method in a superclass. Cache it in this class.
            goto done;
        }
    }

    // No implementation found. Try method resolver once.

    if (slowpath(behavior & LOOKUP_RESOLVER)) {
        behavior ^= LOOKUP_RESOLVER;
        return resolveMethod_locked(inst, sel, cls, behavior);
    }

 done:
    log_and_fill_cache(cls, imp, sel, inst, curClass);
    runtimeLock.unlock();
 done_nolock:
    if (slowpath((behavior & LOOKUP_NIL) && imp == forward_imp)) {
        return nil;
    }
    return imp;
}
```

如果传入的behavior是`LOOKUP_INITIALIZE`并且没有被初始化，就会进入`initializeAndLeaveLocked`

```objc
// initializeAndLeaveLocked -> initializeAndMaybeRelock -> initializeNonMetaClass -> callInitialize

void initializeNonMetaClass(Class cls)
{
    ASSERT(!cls->isMetaClass());

    Class supercls;
    bool reallyInitialize = NO;

    // Make sure super is done initializing BEFORE beginning to initialize cls.
    // See note about deadlock above.
    supercls = cls->superclass;
    if (supercls  &&  !supercls->isInitialized()) {
        initializeNonMetaClass(supercls);
    }

    ...
}

void callInitialize(Class cls)
{
    ((void(*)(Class, SEL))objc_msgSend)(cls, @selector(initialize));
    asm("");
}
```

这里`initializeNonMetaClass`对`superclass`进行了递归，如果存在父类，则先调用父类的方法
而`callInitialize`方法用到了消息发送`objc_msgSend`，根据消息发送机制，如果子类没有找到实现的方法，会到父类里面查找，找到方法，则调用，所以，如果多个子类没有实现initialize，而父类实现了，在不同的子类初始化的时候都会调用父类的`initialize`方法，导致`initialize`被调用多次

1. 第一次使用的时候（objc_msgSend）调用
2. 先调用父类`initialize`，再调用子类的
3. 调用方法使用`objc_msgSend`，如果子类没有实现`initialize`，会调用父类的`initialize`（存在多次调用的问题）
4. 由于使用`objc_msgSend`调用，与普通方法一样，分类的`initialize`方法会先找到，相当于覆盖类的`initialize`方法，顺序与编译顺序相反（后编译的分类方法会被加到前面）
