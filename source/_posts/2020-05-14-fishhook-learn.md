---
title: fishhook源码学习
tags: [iOS]
date: 2020-05-14 19:20:18
updated: 2020-05-14 19:20:18
categories: iOS
---

首先是知识储备

* Mach-O文件格式，用于定位懒绑定的函数地址
* Mach-O懒加载机制，运行时绑定自定义地址

<!-- more -->

先看个例子

```objc
int main(int argc, char * argv[]) {
    printf("111");
    printf("222");
    return 1;
}
```

调试查看汇编为

```s
Test`main:
    0x10486bf24 <+0>:  sub    sp, sp, #0x30             ; =0x30
    0x10486bf28 <+4>:  stp    x29, x30, [sp, #0x20]
    0x10486bf2c <+8>:  add    x29, sp, #0x20            ; =0x20
    0x10486bf30 <+12>: stur   wzr, [x29, #-0x4]
    0x10486bf34 <+16>: stur   w0, [x29, #-0x8]
    0x10486bf38 <+20>: str    x1, [sp, #0x10]
    0x10486bf3c <+24>: adrp   x0, 0
    0x10486bf40 <+28>: add    x0, x0, #0xfa4            ; =0xfa4
->  0x10486bf44 <+32>: bl     0x10486bf74               ; symbol stub for: printf
    0x10486bf48 <+36>: adrp   x8, 0
    0x10486bf4c <+40>: add    x8, x8, #0xfad            ; =0xfad
    0x10486bf50 <+44>: str    w0, [sp, #0xc]
    0x10486bf54 <+48>: mov    x0, x8
    0x10486bf58 <+52>: bl     0x10486bf74               ; symbol stub for: printf
    0x10486bf5c <+56>: mov    w9, #0x1
    0x10486bf60 <+60>: str    w0, [sp, #0x8]
    0x10486bf64 <+64>: mov    x0, x9
    0x10486bf68 <+68>: ldp    x29, x30, [sp, #0x20]
    0x10486bf6c <+72>: add    sp, sp, #0x30             ; =0x30
    0x10486bf70 <+76>: ret
```

进入 `0x10486bf74` 方法（用`si`命令）

```s
Test`printf:
    0x10486bf74 <+0>: nop
    0x10486bf78 <+4>: ldr    x16, #0x4088              ; (void *)0x000000010486bf98
->  0x10486bf7c <+8>: br     x16
```

* 第一次会跳到`0x000000010486bf98`这个地址，这个值存放在对应`__DATA`段的`__la_symbol_ptr`
* 这段代码对应`__TEXT`段`__stub_helper`里面，第一次会执行这里进行符号绑定
* 绑定完成后，会修改`__DATA`段的`__la_symbol_ptr`里面的值

下面是第二次执行`printf`，x16的值以及是`0x000000019d06df5c`了，这个就是真实的`printf`地址

```s
Test`printf:
->  0x10486bf74 <+0>: nop
    0x10486bf78 <+4>: ldr    x16, #0x4088              ; (void *)0x000000019d06df5c: printf
    0x10486bf7c <+8>: br     x16
```

## fishhook源码分析

fishhook就是利用了MachO对于符号（如：printf）在运行时才做真实地址的绑定（`__DATA`段`__la_symbol_ptr`或`__DATA_CONST`段`__got`），找到对应的符号占位地址修改为我们自己的函数地址，改地址存放在DATA段

`fishhook.c`源码如下

```c
// 用于重绑定符号的链表项
struct rebindings_entry {
  struct rebinding *rebindings;
  size_t rebindings_nel;
  struct rebindings_entry *next;
};

// 用于存放所有要重绑定符号的列表，链表结构
static struct rebindings_entry *_rebindings_head;

// 重绑定符号
int rebind_symbols(struct rebinding rebindings[], size_t rebindings_nel) {
  // 把rebindings放进链表_rebindings_head的头部，原来的head会被放到next
  int retval = prepend_rebindings(&_rebindings_head, rebindings, rebindings_nel);
  if (retval < 0) {
    return retval;
  }

  // 判断是否是第一次调用（第一次的话next为空）
  if (!_rebindings_head->next) {
    // 第一次调用的话，注册加载动态库的回调，加载完后会回调_rebind_symbols_for_image
    _dyld_register_func_for_add_image(_rebind_symbols_for_image);
  } else {
    // 已经加载完成动态库，遍历所有的动态库
    uint32_t c = _dyld_image_count();
    for (uint32_t i = 0; i < c; i++) {
      // 符号绑定，传入动态库header和ASLR偏移量
      _rebind_symbols_for_image(_dyld_get_image_header(i), _dyld_get_image_vmaddr_slide(i));
    }
  }
  return retval;
}

// 把rebindings放到链表rebindings_head的头部
static int prepend_rebindings(struct rebindings_entry **rebindings_head,
                              struct rebinding rebindings[],
                              size_t nel) {
  struct rebindings_entry *new_entry = malloc(sizeof(struct rebindings_entry));
  if (!new_entry) {
    return -1;
  }
  new_entry->rebindings = malloc(sizeof(struct rebinding) * nel);
  if (!new_entry->rebindings) {
    free(new_entry);
    return -1;
  }
  memcpy(new_entry->rebindings, rebindings, sizeof(struct rebinding) * nel);
  new_entry->rebindings_nel = nel;
  new_entry->next = *rebindings_head;
  *rebindings_head = new_entry;
  return 0;
}

// 绑定动态库符号
static void _rebind_symbols_for_image(const struct mach_header *header,
                                      intptr_t slide) {
    rebind_symbols_for_image(_rebindings_head, header, slide);
}

// 绑定动态库符号，主要方法
static void rebind_symbols_for_image(struct rebindings_entry *rebindings,
                                     const struct mach_header *header,
                                     intptr_t slide) {
  Dl_info info;
  // 获取 mach_header 这个符号的信息，将信息放到 info 中
  if (dladdr(header, &info) == 0) {
    return;
  }

  segment_command_t *cur_seg_cmd;
  segment_command_t *linkedit_segment = NULL;
  struct symtab_command* symtab_cmd = NULL;
  struct dysymtab_command* dysymtab_cmd = NULL;

  // 遍历loadCommand，找到__LINKEDIT，LC_SYMTAB, LC_DYSYMTAB
  uintptr_t cur = (uintptr_t)header + sizeof(mach_header_t);
  for (uint i = 0; i < header->ncmds; i++, cur += cur_seg_cmd->cmdsize) {
    cur_seg_cmd = (segment_command_t *)cur;
    if (cur_seg_cmd->cmd == LC_SEGMENT_ARCH_DEPENDENT) {
      if (strcmp(cur_seg_cmd->segname, SEG_LINKEDIT) == 0) {
        linkedit_segment = cur_seg_cmd;
      }
    } else if (cur_seg_cmd->cmd == LC_SYMTAB) {
      symtab_cmd = (struct symtab_command*)cur_seg_cmd;
    } else if (cur_seg_cmd->cmd == LC_DYSYMTAB) {
      dysymtab_cmd = (struct dysymtab_command*)cur_seg_cmd;
    }
  }

  // 判空
  if (!symtab_cmd || !dysymtab_cmd || !linkedit_segment ||
      !dysymtab_cmd->nindirectsyms) {
    return;
  }

  // 定位 “符号表”，“字符串表”，“重定向表” 实际内存地址

  // 代码段在内存的起始位置（ASLR偏移+PAGEZERO）(PAGEZERO = vmaddr - fileoff)
  uintptr_t linkedit_base = (uintptr_t)slide + linkedit_segment->vmaddr - linkedit_segment->fileoff;
  // 符号表的地址 = 基址 + 符号表偏移量
  nlist_t *symtab = (nlist_t *)(linkedit_base + symtab_cmd->symoff);
  // 字符串表的地址 = 基址 + 字符串表偏移量
  char *strtab = (char *)(linkedit_base + symtab_cmd->stroff);
  // 动态符号表地址 = 基址 + 动态符号表偏移量
  uint32_t *indirect_symtab = (uint32_t *)(linkedit_base + dysymtab_cmd->indirectsymoff);

  // 为了重新遍历loadCommand，重新设置cur
  cur = (uintptr_t)header + sizeof(mach_header_t);

  // 再次遍历LoadCommand
  for (uint i = 0; i < header->ncmds; i++, cur += cur_seg_cmd->cmdsize) {
    cur_seg_cmd = (segment_command_t *)cur;
    if (cur_seg_cmd->cmd == LC_SEGMENT_ARCH_DEPENDENT) {
      // 找到__DATA段和__DATA_CONST段
      if (strcmp(cur_seg_cmd->segname, SEG_DATA) != 0 &&
          strcmp(cur_seg_cmd->segname, SEG_DATA_CONST) != 0) {
        continue;
      }

      for (uint j = 0; j < cur_seg_cmd->nsects; j++) {
        section_t *sect =
          (section_t *)(cur + sizeof(segment_command_t)) + j;

        // 对__nl_symbol_ptr以及__la_symbol_ptr进行rebind
        if ((sect->flags & SECTION_TYPE) == S_LAZY_SYMBOL_POINTERS) {
          perform_rebinding_with_section(rebindings, sect, slide, symtab, strtab, indirect_symtab);
        }
        if ((sect->flags & SECTION_TYPE) == S_NON_LAZY_SYMBOL_POINTERS) {
          perform_rebinding_with_section(rebindings, sect, slide, symtab, strtab, indirect_symtab);
        }
      }
    }
  }
}

// 重新绑定符号
static void perform_rebinding_with_section(struct rebindings_entry *rebindings,
                                           section_t *section,
                                           intptr_t slide,
                                           nlist_t *symtab,
                                           char *strtab,
                                           uint32_t *indirect_symtab) {
  // 是否是__DATA_CONST常量区
  const bool isDataConst = strcmp(section->segname, SEG_DATA_CONST) == 0;

  // `nl_symbol_ptr`和`la_symbol_ptr`section中的`reserved1`字段指明对应的`indirect symbol table`起始的index
  //动态符号表中第一个解析的符号的起始地址
  uint32_t *indirect_symbol_indices = indirect_symtab + section->reserved1;

  // indirect_symbol_bindings是 `__la_symbol_ptr` or `__nl_symbol_ptr` 表
  // 它的首地址 = slide(基础偏移地址) + Section的内存相对地址 (memory address of this section)
  void **indirect_symbol_bindings = (void **)((uintptr_t)slide + section->addr);

  // 内存保护设置
  vm_prot_t oldProtection = VM_PROT_READ;
  if (isDataConst) {
    oldProtection = get_protection(rebindings);
    // 修改内存属性为可读写（__DATA_CONST段默认是只读的），便于我们修改函数地址等数据
    mprotect(indirect_symbol_bindings, section->size, PROT_READ | PROT_WRITE);
  }

  // 遍历表
  for (uint i = 0; i < section->size / sizeof(void *); i++) {
    // 符号表Index：根据动态符号表获取
    uint32_t symtab_index = indirect_symbol_indices[i];

    // 如果是 abs 或者 是 本地 则跳过 (因为不是动态库的外部符号)
    if (symtab_index == INDIRECT_SYMBOL_ABS || symtab_index == INDIRECT_SYMBOL_LOCAL ||
        symtab_index == (INDIRECT_SYMBOL_LOCAL   | INDIRECT_SYMBOL_ABS)) {
      continue;
    }

    // 字符串表偏移量
    uint32_t strtab_offset = symtab[symtab_index].n_un.n_strx;

    // 通过字符串表偏移量获取符号对应的字符串（符号的名字）
    char *symbol_name = strtab + strtab_offset;
    bool symbol_name_longer_than_1 = symbol_name[0] && symbol_name[1];

    // 获取第一个元素
    struct rebindings_entry *cur = rebindings;

    // 遍历所有需要hook的符号表
    while (cur) {
      for (uint j = 0; j < cur->rebindings_nel; j++) {
        // 判断符号名是否相同
        if (symbol_name_longer_than_1 &&
            strcmp(&symbol_name[1], cur->rebindings[j].name) == 0) {
          // 备份原来的符号地址
          if (cur->rebindings[j].replaced != NULL &&
              indirect_symbol_bindings[i] != cur->rebindings[j].replacement) {
            *(cur->rebindings[j].replaced) = indirect_symbol_bindings[i];
          }
          // 并且将hook函数的新函数地址 更新到 `__nl_symbol_ptr`或者`__la_symbol_ptr` 中
          indirect_symbol_bindings[i] = cur->rebindings[j].replacement;

          // 结束该内层的遍历, 查找下一个符号
          goto symbol_loop;
        }
      }
      cur = cur->next;
    }
  symbol_loop:;
  }

  // 恢复原来的内存权限改写
  if (isDataConst) {
    int protection = 0;
    if (oldProtection & VM_PROT_READ) {
      protection |= PROT_READ;
    }
    if (oldProtection & VM_PROT_WRITE) {
      protection |= PROT_WRITE;
    }
    if (oldProtection & VM_PROT_EXECUTE) {
      protection |= PROT_EXEC;
    }
    mprotect(indirect_symbol_bindings, section->size, protection);
  }
}
```

整个过程

* 读取MachO文件的LoadCommand的地址和数量
* 遍历LoadCommand，获取到`__LINKEDIT`，`LC_SYMTAB`, `LC_DYSYMTAB`信息
* 根据LoadCommand的段偏移量和ASLR偏移量，算出`符号表`，`动态符号表`，`字符串表`的内存地址
* 遍历LoadCommand，找出`__DATA`段和`__DATA_CONST`段，遍历`S_LAZY_SYMBOL_POINTERS`和`S_NON_LAZY_SYMBOL_POINTERS`段信息
* 如果是`__DATA_CONST`，则需要修改内存权限为可读写
* 遍历上面两个段，过滤`abs`和`本地符号`，找到匹配符号的，保存原来的地址，替换为新的符号地址
  * 通过`__la_symbol_ptr`的`reserved1`字段找到第一个需要动态绑定的符号在`Dynamic Symbol Table`中的位置
  * `Dynamic Symbol Table`中获取到`Symbol Table`的Index
  * 在`Symbol Table`获取`String Table`的偏移量，得到`符号名`

    {% img /images/post/fish/data-la-symbol-ptr-reserved1.png %}
    {% img /images/post/fish/dynamic-symbol-table-reserved1.png %}
    {% img /images/post/fish/symbol-table-info.png %}
    {% img /images/post/fish/string-table-offset.png %}

* 恢复`__DATA_CONST`内存权限为只读
