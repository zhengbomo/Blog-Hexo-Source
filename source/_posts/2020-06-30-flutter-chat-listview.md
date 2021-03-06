---
title: flutter仿微信聊天交互
tags: [flutter]
date: 2020-06-30 08:16:15
updated: 2020-06-30 08:16:15
categories: flutter
---

最近在做一个聊天的页面，参考微信的聊天页面，对`ListView`有下面几个需求

1. 使用列表Widget，例如：ListView, CustomScrollView等
2. 支持`scrollToEnd`，当键盘，表情面板，工具面板弹出时，消息滑动到底部
3. 支持获取位置用于跳转`getCurrentIndexInfo`，用于保持加载数据时候的位置不变
4. 支持`jumpToIndex`和`scrollToIndex`，避免手动计算位置
5. 滑动位置要准确，没有误差
6. 滑动到底部不会出现bounce
7. 由于键盘上移的时候scrollToEnd

<!-- more -->

{% img /images/post/flutter/flutter-chat-message-list.gif 300 %}

## scrollToEnd

以`ListView`为例，网上推荐的做法是

```dart
/// 滑动到底部
void _scrollToEnd() {
  final offset = _scrollController.position.maxScrollExtent;
    _scrollController.animateTo(offset,
    duration: Duration(milliseconds: 250),
    curve: Curves.easeInOut
  );
}
```

但上面做法存在一个问题，就是误差，当内容是高度可变的时候就会有误差，如下面例子

```dart
final _random = Random();

class _TestPageState extends State<TestPage> {
  List<double> _items =
      List.generate(180, (index) => _random.nextInt(100) + 100.0);
  ScrollController _scrollController = ScrollController();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("scrollToEnd"),
        actions: <Widget>[
          IconButton(
            icon: Icon(Icons.add),
            onPressed: _scrollToEnd,
          ),
        ],
      ),
      body: ListView.builder(
          controller: _scrollController,
          itemBuilder: (c, i) {
            return Container(
              margin: EdgeInsets.all(8),
              height: _items[i],
              color: Colors.orange,
              child: Text("$i"),
            );
          },
          itemCount: _items.length),
    );
  }
}
```

{% img /images/post/flutter/scroll_controller_scroll_to_end.gif 300 %}

### 存在问题

从截图中可以看到，`maxScrollExtent`有时候偏小，有时候偏大，如果是偏大的情况，scrollView在scroll到屏幕外后再反弹（bounce的效果），这是由于ListView在渲染的时候，没出现在屏幕的Widget是不会被渲染的，这个时候还不能确定所有Widget的实际高度，ScrollView会根据当前渲染的Widget估算其他Widget的高度，所以带来误差，而如果widget是等高的，则不会有误差问题

如果能`scrollToIndex`应该可以解决问题，直接滑动到某一项

## scrollToIndex

自带的ScrollController不支持scrollToIndex，找到下面2个第三方库，支持scrollToIndex，两个库都可以精确滑动到对应的位置

1. [`scroll_to_index`](https://pub.dev/packages/scroll_to_index): 通过分段滑动，边滑动边计算，在滑动的过程中可以得到widget的高度，达到scrollToIndex的目的
2. [`scrollable_positioned_list`](https://pub.dev/packages/scrollable_positioned_list)
   * 对于滚动列表进行`extendCache`，缓存多两个屏幕的widget
   * 为了计算位置，我们知道滚动到第0项，位置肯定是准的，也就是`offset=0`，scrollable_positioned_list用一个辅助的列表做滚动位置，让滚动的目标为0，这样就可以避免计算的误差
   * 保持缓存区间所有Widget的位置信息，当目标位置在当前列表的缓存区间的时候，直接scrollToOffset，否则，使用辅助列表配合滚动，两个列表都只缓存开始和结束位置的widget，而不需要计算中间的widget，当列表增大时，不会带来太大的性能消耗

### 存在问题

* `scroll_to_index`: 由于是采用多次滚动的方式，对于数据量大的话滑动会持续时间比较长，而且看起来非常不顺滑，抖动厉害，性能消耗比较大
* `scrollable_positioned_list`: 当滑动到底部（最后一个项）的时候，他会把index项滑动到0的位置再回弹，会出现bounce

## bounce问题

基于上面问题，考虑对`scrollable_positioned_list`滑动之前添加溢出检查，避免溢出造成bounce，具体代码见[这里](https://github.com/zhengbomo/flutter.widgets/tree/master/packages/scrollable_positioned_list)

scrollable_positioned_list默认使用相对位置alignment，这里改为offset

```dart
// 这里去掉了alignment，改用offset
void _jumpTo({@required int index, double offset}) {
    cancelScrollCallback?.call();

    final controller =
        _showFrontList ? frontScrollController : backScrollController;
    final lastTarget = _showFrontList ? frontTarget : backTarget;
    // 方向
    final direction = index > lastTarget ? 1 : -1;
    // 更新index
    setState(() {
      if (lastTarget != index) {
        if (_showFrontList) {
          frontTarget = index;
        } else {
          backTarget = index;
        }
      }
    });
    // 加上偏移量offset
    var jumpOffset = 0 + offset;
    if (direction == -1) {
      controller.jumpTo(jumpOffset);
    } else {
      controller.jumpTo(jumpOffset);
      // 渲染后如果发现溢出，马上修正
      WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
        var offset = min(jumpOffset, controller.position.maxScrollExtent);
        if (controller.offset != offset) {
          controller.jumpTo(offset);
        }
      });
    }
}
```

对于scrollToIndex

```dart
...

if (itemPosition != null) {
    // 不用切换列表
    final localScrollAmount = itemPosition.itemLeadingEdge *
        startingScrollController.position.viewportDimension;

    var animateOffset =
        startingScrollController.offset + localScrollAmount - offset;
    // 添加溢出check
    animateOffset =
        min(animateOffset, startingScrollController.position.maxScrollExtent);
    await startingScrollController.animateTo(animateOffset,
        duration: duration, curve: curve);
} else {
    // 需要切换两个列表
    ...
    startAnimationCallback = () {
        SchedulerBinding.instance.addPostFrameCallback((_) async {
          frontOpacity.parent = _opacityAnimation(startingListDisplay).animate(
              AnimationController(vsync: this, duration: duration)..forward());
          startAnimationCallback = () {};
          var endJump = -direction *
              (_screenScrollCount *
                      startingScrollController.position.viewportDimension -
                  offset);
          var startScroll =
              startingScrollController.offset + direction * scrollAmount;

          endingScrollController.jumpTo(endJump);
          // 修正位置，避免溢出
          var endScroll = min(
              0.0 + offset, endingScrollController.position.maxScrollExtent);
          endScroll =
              max(endScroll, endingScrollController.position.minScrollExtent);
          endCompleter.complete(endingScrollController.animateTo(endScroll,
              duration: duration, curve: curve));

          startCompleter.complete(startingScrollController
              .animateTo(startScroll, duration: duration, curve: curve));

          cancelScrollCallback = () => _cancelScroll(startingListDisplay);
        });
    };
    ...
}
```

## 获取index位置

当列表滑动到顶部的时候，需要加载上一页的聊天数据，我们希望加载数据后刷新页面，用户所在的位置不变（不要跳动），可以保留当前位置，在刷新后更新位置

```dart
// 第一个为index，第二个为offset
List<dynamic> _getCurrentIndexInfo(bool wholeVisible) {
    final controller =
        _showFrontList ? frontScrollController : backScrollController;
    final notifier =
        _showFrontList ? frontItemPositionNotifier : backItemPositionNotifier;
    /// 获取viewport上元素的Position信息
    var visibleItems = notifier.itemPositions.value.where((i) {
      if (wholeVisible) {
        return i.itemLeadingEdge >= 0 && i.itemTrailingEdge <= 1;
      } else {
        return i.itemTrailingEdge > 0 && i.itemLeadingEdge < 1;
      }
    });
    /// 取最小index
    ItemPosition firstVisibleItem = visibleItems.fold(null, (v, i) {
      if (v == null) {
        return i;
      } else if (i.index < v.index) {
        return i;
      } else {
        return v;
      }
    });
    // 计算偏移量
    var offset = controller.position.viewportDimension *
        firstVisibleItem.itemLeadingEdge;
    return [firstVisibleItem.index, offset];
}
```

{% img /images/post/flutter/flutter-listview-insert-keep-position.gif 300 %}

## 键盘处理

键盘弹出的时候，我们希望ChatBar是动画上移的，并且listview需要scrollToEnd，`Scaffold`有个属性`resizeToAvoidBottomInset`用于控制键盘弹出时的内容区域，但是没有动画，直接变化看起来非常突兀，这里关掉了这个属性，我们自己来控制键盘弹出时的UI变化

```dart
Scaffold(
    resizeToAvoidBottomInset: false,
    ...
)
```

如果使用动画修改ListView的高度，则无法和scrollToEnd配合起来，应为scrollToEnd无法根据动画一致滚动，这样性能上会比较差，这里采用占位的方式，键盘弹出的时候，不修改ListView的高度，而是在ListView底部添加一个`占位item`，修改这个占位item高度（不需要动画），然后scrollToEnd，这个滚动可以做到平滑，另外ChatBar键盘弹出时添加上动画即可

仿写微信项目在[这里](https://github.com/zhengbomo/flutter_wechat)
