---
title: flutter切换tab后保留tab状态
tags: [flutter]
date: 2019-02-23 12:43:52
updated: 2019-02-23 12:43:52
categories: flutter
---

## 前言

最近在用flutter写一个小项目，在写主页面（底部导航栏+子页面）时遇到的一个问题：当点击底部item切换到另一页面, 再返回此页面时会重走它的initState方法（我们一般在initState中发起网络请求，或者初始化的操作），导致不必要的开销

<!-- more -->

## 根据Tab动态加载页面
我们先定义两个页面`PageA`和`PageB`

```dart

class PageA extends StatefulWidget {
  _PageAState createState() => _PageAState();
}

class _PageAState extends State<PageA> {
  @override
  void initState() {
    super.initState();
    print("pageA init state");
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.orangeAccent,
      child: Center(
        child: Text("page A"),
      ),
    );
  }
}

class PageB extends StatefulWidget {
  _PageBState createState() => _PageBState();
}

class _PageBState extends State<PageB> {
  @override
  void initState() {
    super.initState();
    print("pageB init state");
  }
  
  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.blueAccent,
      child: Center(
        child: Text("page B"),
      ),
    );
  }
}
```

定义Tab主页面

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatefulWidget {
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  int _tabIndex = 0;

  List<Widget> _tabWidget = [
    PageA(),
    PageB()
  ];

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'tab demo',
      home: Scaffold(
        bottomNavigationBar: BottomNavigationBar(
          items: _createBottomItems(),
          currentIndex: this._tabIndex,
          onTap: (index) {
            setState(() {
              this._tabIndex = index;
            });
          },
        ),
        body: this._tabWidget.elementAt(this._tabIndex),
      ),
    );
  }
}

// 创建底部导航item
List<BottomNavigationBarItem> _createBottomItems() {
  return [
    BottomNavigationBarItem(
      icon: Icon(Icons.home),
      title: Text("首页")
    ),
    BottomNavigationBarItem(
      icon: Icon(Icons.insert_emoticon),
      title: Text("我的")
    )
  ];
}
```

运行后发现，每次切换tab都会调用`initState`，这显然不符合我们的正常的需求，有下面两种解决方式

## IndexedStack

`IndexedStack`可以控制子元素的显示和隐藏，并且会缓存所有的元素，不会每次都重新创建子元素

```dart
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'tab demo',
      home: Scaffold(
        bottomNavigationBar: BottomNavigationBar(
          items: _createBottomItems(),
          currentIndex: this._tabIndex,
          onTap: (index) {
            setState(() {
              this._tabIndex = index;
            });
          },
        ),
        body: IndexedStack(
          children: this._tabWidget,
          index: this._tabIndex,
        )
      ),
    );
  }
```

运行后发现还是有个问题，IndexedStack在初始化的时候会初始化所有的子元素，pageA和pageB的`initState`会同时调用，这明显还是不符合我们的需求

正确来说应该是切换到具体页面的时候才进行初始化，而不是一开始就加载所有的页面的数据，避免资源浪费

## PageView + AutomaticKeepAliveClientMixin

使用PageView支持多个view切换，并且不会一次加载完所有的页面

```dart
  PageController _pageController;
  
  @override
  void initState() {
    super.initState();
    this._pageController =PageController(initialPage: this._tabIndex, keepPage: true);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'tab demo',
      home: Scaffold(
        bottomNavigationBar: BottomNavigationBar(
          items: _createBottomItems(),
          currentIndex: this._tabIndex,
          onTap: (index) {
            setState(() {
              this._tabIndex = index;
              _pageController.jumpToPage(index);
            });
          },
        ),
        body: PageView(
          children: this._tabWidget,
          controller: _pageController,
        ),
      ),
    );
  }
}
```

使用PageView可以正常切换，但是每次切换Tab的时候还是会重复调用`initState`，我们还需要在子页面实现`AutomaticKeepAliveClientMixin`

```dart
class _PageAState extends State<PageA> with AutomaticKeepAliveClientMixin {
  @override
  bool get wantKeepAlive => true;
  
  ...
}
class _PageBState extends State<PageB> with AutomaticKeepAliveClientMixin {
  @override
  bool get wantKeepAlive => true;

  ...
}
```

实现了`AutomaticKeepAliveClientMixin`就不会每次切换Tab都调用`initState`了，这也是google推荐的方式

最后发现`PageView`可以左右滑动切换，这个可以通过设置`physics`为`NeverScrollableScrollPhysics()`来禁止滑动

```dart
PageView(
  children: this._tabWidget,
  controller: _pageController,
  physics: NeverScrollableScrollPhysics(),
)
```