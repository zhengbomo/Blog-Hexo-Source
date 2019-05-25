---
title: StafulWidget自动释放资源Mixin
tags: [flutter]
date: 2019-04-25 21:48:37
updated: 2019-04-25 21:48:37
categories: [flutter]
---

平常开发中，经常会用到释放资源的问题，最常见的就是网络请求了，也是最经常被忽略的问题，例如，我们进入一个新的页面会请求数据，在请求回来之前，这个时候如果用户退出了该页面，就需要及时的释放资源（cancel掉之前的请求），避免资源被释放带来的其他问题，例如空指针，而页面中，可能不止网络请求，可能有定时器，动画，等资源都需要及时的释放，这使得我们管理起来非常麻烦

<!-- more -->

我们用一个网络请求的例子来，我们进入页面后，点击按钮，请求`github`首页，请求成功或失败后更新文字，网络请求使用`dio`库

```dart
import 'package:flutter/material.dart';
import 'package:dio/dio.dart';


class TestPage extends StatefulWidget {
  @override
  _TestPageState createState() => _TestPageState();
}

class _TestPageState extends State<TestPage> {
  CancelToken _token = CancelToken();
  String _content = "";

  @override
  void dispose() {
    // 如果没有被释放，则释放掉
    if (_token != null && !_token.isCancelled) {
      _token.cancel();
    }
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("test cancel")
      ),
      body: Center(
        child: Column(
          children: <Widget>[
            RaisedButton(
              onPressed: () async {
                try {
                  var url = "https://www.github.com";
                  _token = CancelToken();

                  Dio dio = new Dio();
                  Response response = await dio.get(url, cancelToken: _token);
                  int code = response.statusCode;

                  if (code == 200) {
                    setState(() {
                      _content = "请求成功";
                    });  
                  } else {
                    setState(() {
                      _content = "请求失败";
                    });
                  }
                } catch (e) {
                  if (!CancelToken.isCancel(e)) {
                    setState(() {
                      _content = "请求失败";
                    });
                  }
                }
              },
              child: Text("请求数据")
            ),
            Text(_content)
          ],
        )
      )
    );
  }
}
```

如果我们不在`dispose`取消请求的话，用户离开页面后，由于请求没有被`cancel`，当网络请求回来后，会出现资源被释放导致`崩溃`

上面例子可以看到，我们需要管理`CancelToken`，在dispose的时候进行释放，如果需要dispose的对象比较多的时候，管理起来是崩溃的，我们可以把释放的操作封装到`mixin`中进行统一管理，释放对象（例如：CancelToken）通过闭包管理，而不需要单独维护

```dart
import 'package:flutter/material.dart';

@optionalTypeArgs
mixin AutomaticDisposeStatefulWidgetMixin<T extends StatefulWidget> on State<T> {
  List<Function> _disposeFunc = List<Function>();

  void addToDispose(Function func) {
    _disposeFunc.add(func);
  }

  @override
  void dispose() {
    // dispose all func
    _disposeFunc.forEach((f) => f());
    super.dispose();
  }
}
```

上面代码可以改为

```dart
class TestPage extends StatefulWidget {
  @override
  _TestPageState createState() => _TestPageState();
}

class _TestPageState extends State<TestPage> with AutomaticDisposeStatefulWidgetMixin {
  String _content = "";

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("test")
      ),
      body: Center(
        child: Column(
          children: <Widget>[
            RaisedButton(
              onPressed: () async {
                try {
                  var url = "https://www.github.com";
                  var token = CancelToken();

                  // 在widget释放时候执行
                  addToDispose(() {
                    token.cancel();
                  });

                  Dio dio = new Dio();
                  Response response = await dio.get(url, cancelToken: token);
                  int code = response.statusCode;

                  if (code == 200) {
                    setState(() {
                      _content = "请求成功";
                    });  
                  } else {
                    setState(() {
                      _content = "请求失败";
                    });
                  }
                } catch (e) {
                  if (!CancelToken.isCancel(e)) {
                    setState(() {
                      _content = "请求失败";
                    });
                  }
                }
              },
              child: Text("请求数据")
            ),
            Text(_content)
          ],
        )
      )
    );
  }
}
```

当然，除了dispose网络请求，可以在block做任何操作，例如关闭streamController，关闭通知监听等

> 对于网络请求的释放，一定要在不需要的时候释放
> 对于网络请求的释放，一定要在不需要的时候释放
> 对于网络请求的释放，一定要在不需要的时候释放