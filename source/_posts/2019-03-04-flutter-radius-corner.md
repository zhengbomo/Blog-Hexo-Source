---
title: Flutter给控件设置圆角
tags: [flutter]
date: 2019-02-20 19:51:44
updated: 2019-02-20 19:51:44
categories:
---

Flutter给控件设置圆角有几种方式，可以通过裁切Widget包装，也可以通过装饰器设置

<!-- more -->

## 裁切控件Clip

```dart
// 圆角
new ClipRRect(
  borderRadius: BorderRadius.circular(6.0),
  child: Container(
    width: 100,
    height: 100,
    color: Colors.redAccent,
  )
)

// 圆形
new ClipOval(
  child: Container(
    width: 100,
    height: 100,
    color: Colors.redAccent,
  ),
)
```

## 装饰器BoxDecoration

```dart
// 圆角
DecoratedBox(
  decoration: BoxDecoration(
    border: new Border.all(color: Colors.black54, width: 0.5),
    color: Colors.greenAccent,
    shape: BoxShape.rectangle,
    borderRadius: BorderRadius.circular(12.0),
  ),
  child: Container(
    padding: EdgeInsets.all(12),
    child: Text("内容"),
  ),
)

// 圆形
DecoratedBox(
  decoration: BoxDecoration(
    border: new Border.all(color: Colors.black54, width: 0.5),
    color: Colors.greenAccent,
    shape: BoxShape.circle,
  ),
  child: Container(
    padding: EdgeInsets.all(12),
    child: Text("内容"),
  ),
)

```

> 注意：DecoratedBox与Clip控件不同，这里只进行装饰，不进行裁切，如果child的内容超出了范围，装饰器不会进行裁切，`BoxDecoration`通常可以用来设置边框，例如标签列表等

对于圆形头像我们也用的比较多，这里有一个圆形头像的控件，效果与`BoxDecoration`一致

```dart
CircleAvatar(
  radius: 100,
  backgroundImage: NetworkImage("https://flutter.io/images/intellij/hot-reload.gif"),
)
```

