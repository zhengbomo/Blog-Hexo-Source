---
title: Flutter实现一个ProgressHUD
tags: [iOS, flutter]
date: 2019-04-24 23:02:17
updated:
categories: flutter
---

用惯了iOS的[SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)，但是在flutter pub上的并没有找到类似的实现，于是自己实现一个

<!-- more -->

主要实现四个基本功能

* Loading显示
* 成功显示
* 错误显示
* 进度显示：环形进度条和文字

## 库地址

[https://pub.dartlang.org/packages/bmprogresshud](https://pub.dartlang.org/packages/bmprogresshud)

```yaml
dependencies:
  bmprogresshud: ^0.0.2
```

## 实现效果

![演示效果](https://user-gold-cdn.xitu.io/2019/4/24/16a4fd4d6d48cc8f?w=302&h=599&f=gif&s=387661)

1. 由于HUD是盖在视图上面的，通常是整个页面，故考虑直接在目标Widget上套一层`ProgressHUD`
2. 我们需要在特定的地方获取`ProgressHUD`进行操作，这个有点类似`Navigator`，参考Navigator的用法，通过`of`方法获得

实际效果如下

```dart
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text("hud demo"),
    ),
    body: ProgressHud(
      child: Container(
        child: Builder(builder: (context) {
          return RaisedButton(
            onPressed: () async {
              ProgressHud.of(context).show(ProgressHudType.loading, "加载中...");
              await Future.delayed(const Duration(seconds: 1));
              ProgressHud.of(context).dismiss();
            },
            child: Text("加载数据"),
          );
        }),
      ),
    )
  );
}
```

## 实现效果

### 1. 显示和隐藏渐变

通过属性`opacity`和`AnimationController`控制透明度，当透明度为0时候，通过`Offstage`控制控件的隐藏

```dart
class ProgressHudState extends State<ProgressHud> with SingleTickerProviderStateMixin {
  AnimationController _animation;
  var _opacity = 0.0;
  var _isVisible = false;

  @override
  void initState() {
    _animation = AnimationController(
      duration: const Duration(milliseconds: 200), 
      vsync: this
    )..addListener(() {
      setState(() {
        // 修改透明度
        _opacity = _animation.value;
      });
    })..addStatusListener((status) {
      if (status == AnimationStatus.dismissed) {
        setState(() {
          // 隐藏动画结束，隐藏控件
          _isVisible = false;          
        });
      }
    });
    super.initState();
  }

  ...
}
```

我们通过动画的执行方向控制动画

```dart
// 显示动画
_animation.forward();
setState(() {
  _isVisible = true;
});

// 隐藏动画
_animation.reverse();
```

### 2. 通过`BuildContext`获得Element树的`ProgressHUD`

```dart
class ProgressHud extends StatefulWidget {
  static ProgressHudState of(BuildContext context) {
    return context.ancestorStateOfType(const TypeMatcher<ProgressHudState>());
  }

  ...
}
```

### 3. 创建HUD

```dart
Widget _createHudView(Widget child) {
  return Stack(
    children: <Widget>[
      // 如果不想屏蔽用户操作，ignoring设置为true，这里设置为无法响应
      IgnorePointer(
        ignoring: false,
        child: Container(
          color: Colors.transparent,
          width: double.infinity,
          height: double.infinity,
        ),
      ),
      Center(
        child: Container(
          // 这里设置一定的偏移，因为iPhoneX有下方安全区域，看起来会偏下
          margin: EdgeInsets.fromLTRB(10, 10, 10, 10 - widget.offsetY * 2),
          decoration: BoxDecoration(
            color: Color.fromARGB(255, 33, 33, 33), 
            borderRadius: BorderRadius.circular(5)
          ),
          // 设置最小宽高，如果文字比较多，可以自适应
          constraints: BoxConstraints(
            minHeight: 130,
            minWidth: 130
          ),
          child: Padding(
            padding: EdgeInsets.all(12),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                Container(
                  padding: EdgeInsets.all(15),
                  child: child,
                ),
                Container(
                  child: Text(
                    _text,
                    textAlign: TextAlign.center,
                    style: TextStyle(color: Colors.white, fontSize: 16)
                  ),
                )
              ],
            ),
          ),
        ),
      ),
    ],
  );
}
```

### 4. 环形进度

通过Painter画两个圆

```dart
import 'dart:math';
import 'package:flutter/material.dart';


class CircleProgressBarPainter extends CustomPainter {
  final double progress;
  final double strokeWidth;
  final Color color;
  final Color fillColor;
  const CircleProgressBarPainter({
    this.progress = 0, 
    this.strokeWidth = 3,
    this.color = Colors.grey,
    this.fillColor = Colors.white
  });

  @override
  void paint(Canvas canvas, Size size) {
    final paint = new Paint()
      ..color = this.color
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;
    final double diam = min(size.width, size.height);
    final centerX = size.width * 0.5;
    final centerY = size.height * 0.5;
    final radius = diam / 2.0;

    canvas.drawCircle(Offset(centerX, centerY), radius, paint);
    paint.color = this.fillColor;
    // draw in center
    var rect = Rect.fromLTWH((size.width - diam) * 0.5, 0, diam, diam);
    canvas.drawArc(rect, -0.5 * pi, progress * 2 * pi, false, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

完整代码见[这里](https://github.com/zhengbomo/bmprogresshud)：