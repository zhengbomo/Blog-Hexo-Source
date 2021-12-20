---
title: Flutter禁用水波纹
tags: [flutter]
date: 2020-06-12 22:49:19
updated: 2020-06-12 22:49:19
categories: Flutter
---

在做Flutter开发的时候，通常我们都使用MaterialApp来开发，最典型的样式就是点击会有水波纹效果，但有时候我们不希望有水波纹效果，例如在iOS上，使用TextFiled的时候禁用水波纹效果，让体验看起来更像原生

<!-- more -->

搜索了一番，没有特别好的方法，基本上是修改`splashColor`，感觉不够彻底，于是从源码入手看下，我们知道水波纹效果的Widget为`InkWell`

```dart
InkWell(
    onTap: () {},
    child: Text("button")
)
```

`InkWell`继承自`InkResponse`，在`_InkResponseState`中的方法`_handleTapDown`可以看到，在点击的时候会开启splash

```dart
void _handleTapDown(TapDownDetails details) {
    _startSplash(details: details);
    if (widget.onTapDown != null) {
        widget.onTapDown(details);
    }
}
```

在`_startSplash`创建了`InteractiveInkFeature`对象splash

```dart
void _startSplash({TapDownDetails details, BuildContext context}) {
    assert(details != null || context != null);

    Offset globalPosition;
    if (context != null) {
      final RenderBox referenceBox = context.findRenderObject() as RenderBox;
      assert(referenceBox.hasSize, 'InkResponse must be done with layout before starting a splash.');
      globalPosition = referenceBox.localToGlobal(referenceBox.paintBounds.center);
    } else {
      globalPosition = details.globalPosition;
    }
    final InteractiveInkFeature splash = _createInkFeature(globalPosition);
    _splashes ??= HashSet<InteractiveInkFeature>();
    _splashes.add(splash);
    _currentSplash = splash;
    updateKeepAlive();
    updateHighlight(_HighlightType.pressed, value: true);
}

InteractiveInkFeature _createInkFeature(Offset globalPosition) {
    final MaterialInkController inkController = Material.of(context);
    final RenderBox referenceBox = context.findRenderObject() as RenderBox;
    final Offset position = referenceBox.globalToLocal(globalPosition);
    final Color color = widget.splashColor ?? Theme.of(context).splashColor;
    final RectCallback rectCallback = widget.containedInkWell ? widget.getRectCallback(referenceBox) : null;
    final BorderRadius borderRadius = widget.borderRadius;
    final ShapeBorder customBorder = widget.customBorder;

    InteractiveInkFeature splash;
    void onRemoved() {
      if (_splashes != null) {
        assert(_splashes.contains(splash));
        _splashes.remove(splash);
        if (_currentSplash == splash)
          _currentSplash = null;
        updateKeepAlive();
      } // else we're probably in deactivate()
    }

    splash = (widget.splashFactory ?? Theme.of(context).splashFactory).create(
      controller: inkController,
      referenceBox: referenceBox,
      position: position,
      color: color,
      containedInkWell: widget.containedInkWell,
      rectCallback: rectCallback,
      radius: widget.radius,
      borderRadius: borderRadius,
      customBorder: customBorder,
      onRemoved: onRemoved,
      textDirection: Directionality.of(context),
    );

    return splash;
  }
```

该类为抽象类，看名字像是水波纹的实现，主要逻辑就在这里面，`InkResponse`通过自身属性或从主题中取到`Theme.of(context).splashFactory`，然后创建`InteractiveInkFeature`，我们把主题中的factory换成自己实现一个没有水波纹的`InteractiveInkFeature`对象，就可以间接的关闭掉水波纹的效果了

```dart
import 'package:flutter/material.dart';


// 空水纹实现工厂
class NoSplashFactory extends InteractiveInkFeatureFactory {
  @override
  InteractiveInkFeature create({required MaterialInkController controller, required RenderBox referenceBox, required Offset position, required Color color, required TextDirection textDirection, bool containedInkWell = false, RectCallback? rectCallback, BorderRadius? borderRadius, ShapeBorder? customBorder, double? radius, VoidCallback? onRemoved}) {
    return _NoInteractiveInkFeature(controller: controller, referenceBox: referenceBox, color: color, onRemoved: onRemoved);
  }
}

// // InkFeature空实现
class _NoInteractiveInkFeature extends InteractiveInkFeature {
  @override
  void paintFeature(Canvas canvas, Matrix4 transform) {

  }
  _NoInteractiveInkFeature({
    required MaterialInkController controller,
    required RenderBox referenceBox,
    required Color color,
    VoidCallback? onRemoved,
  }) : super(controller: controller, referenceBox: referenceBox, color: color, onRemoved: onRemoved);
}
```

把widget包到`Theme`中

```dart
Theme(
    data: ThemeData(
        splashFactory: Shares.noInkFeatureFactory
    ),
    child: FlatButton(
        child: Text("点击了"),
        onPressed: () {},
    ),
)
```

默认还有点击背景，如果需要把点击的背景也去掉，highlightedColor设置为透明即可

```dart
ThemeData(
    splashFactory: Shares.noInkFeatureFactory,
    highlightColor: Colors.transparent,
)
```

放到NavigationBottomTabBar

```dart
Scaffold(
  bottomNavigationBar: Theme(
    data: ThemeData(
      // 去掉水波纹效果
      splashFactory: Shares.noInkFeatureFactory,
      // 去掉点击效果
      highlightColor: Colors.transparent,
    ),
    child: BottomNavigationBar(
      ...
    )
  )
  ...
)
```

{% img /images/post/flutter/flutter_disable_ripple.gif 500 %}
