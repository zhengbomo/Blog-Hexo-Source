---
title: 理解flutter中的Key
tags: [flutter, key]
date: 2020-07-04 09:22:15
updated:
categories: flutter
---

我们知道，flutter有三颗树，widget树在每次setState的时候都会重建，而element树不会，而是会通过diff算法，计算出哪些element需要重建，哪些element可以重用，我们通过一个例子来开始

## 例子

```dart
import 'dart:math';
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

final _random = Random();

class _HomePageState extends State<HomePage> {
  var _items = [
    ListItem(title: "aaa"),
    ListItem(title: "bbb"),
    ListItem(title: "ccc"),
  ];
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("key demo"),
      ),
      body: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: _items,
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          // 删除第一个元素
          _items.removeAt(0);
          setState(() {});
        },
      ),
    );
  }
}

/// 定义一个item
class ListItem extends StatelessWidget {
  final String title;
  // color放在widget
  final Color color = Color.fromARGB(
      255, _random.nextInt(256), _random.nextInt(256), _random.nextInt(256));

  ListItem({this.title});
  @override
  Widget build(BuildContext context) {
    return Container(
      alignment: Alignment.center,
      child: Text(
        this.title,
        style: TextStyle(color: Colors.white, fontSize: 20),
      ),
      color: this.color,
      width: 100,
      height: 100,
    );
  }
}
```

{% img /images/post/flutter/flutter-key-stateless.gif 300 %}

运行正常，接下来我们把`ListItem`换成stateful

```dart
class ListItem extends StatefulWidget {
  final String title;

  ListItem({this.title});
  @override
  _ListItemState createState() => _ListItemState();
}

class _ListItemState extends State<ListItem> {
    // color放在state
  final Color color = Color.fromARGB(
      255, _random.nextInt(256), _random.nextInt(256), _random.nextInt(256));

  @override
  Widget build(BuildContext context) {
    return Container(
      alignment: Alignment.center,
      child: Text(
        widget.title,
        style: TextStyle(color: Colors.white, fontSize: 20),
      ),
      color: this.color,
      width: 100,
      height: 100,
    );
  }
}
```

{% img /images/post/flutter/flutter-key-stateful.gif 300 %}

从上图可以看出，颜色和文字混了，这是由于element树判断增量更新重用element导致的

{% img /images/post/flutter/flutter-widget-element-tree.png 800 删除前 %}

{% img /images/post/flutter/flutter-widget-element-tree2.png 800 删除后 %}

当widget重建的时候，element通过对比新旧两个widget是否需要更新，从而判断是否重用，默认的逻辑是对比`runtimeType`和`key`，我们上面的例子中显然会返回true（我们没有定义key），则表示可以element可以直接使用新的widget

```dart
static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
}
```

由于颜色是state持有的，没有变化，所以蓝色的element，直接使用了灰色的widget，而最后一个绿色的element，没有用到，会被释放

通过`canUpdate`方法可以看到，我们可以设置key来标识element是否可以直接更新widget，flutter中的key有两种

* LocalKey
* GlobalKey

## LocalKey

LocalKey有下面三种，其成员key用于比较，使用起来类似

* ValueKey: 使用一个泛型数据作为key
* ObjectKey: 使用一个对象作为key
* UniqueKey: 自动生成key，并且保证唯一，比较少用

```dart
// 在StatefulWidget构造方法添加参数key，并传给super
class ListItem extends StatefulWidget {
  final String title;

  ListItem({this.title, Key key}) : super(key: key);
  @override
  _ListItemState createState() => _ListItemState();
}

// 使用ListItem的时候，传入key
var _items = [
    ListItem(title: "aaa", key: ValueKey<String>("aaa")),
    ListItem(title: "bbb", key: ValueKey<String>("bbb")),
    ListItem(title: "ccc", key: ValueKey<String>("ccc")),
];
```

这样在判断的时候不同的ListItem会在canUpdate方法就会返回false，就不会重用了

> 通常我们在自定义`StatefulWidget`的时候，需要在构造函数添加可选参数key

## GlobalKey

GlobalKey可以获取到context（element），widget，和state，通常用于在父widget操作子widget

```dart
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget {
  final GlobalKey<_TestWidgetState> _globalKey = GlobalKey();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("global key"),
      ),
      body: Center(
          child: TestWidget(
        key: _globalKey,
      )),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          // 直接操作子widget的state
          _globalKey.currentState.increseCount();
        },
      ),
    );
  }
}

class TestWidget extends StatefulWidget {
  TestWidget({Key key}) : super(key: key);

  @override
  _TestWidgetState createState() => _TestWidgetState();
}

class _TestWidgetState extends State<TestWidget> {
  int _count = 0;

  increseCount() {
    setState(() {
      _count = _count + 1;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Text("$_count");
  }
}
```
