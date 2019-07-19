---
layout: post
title: "Flutter中StreamBuilder的用法"
categories: flutter
date: 2019-07-19 10:10:00
---

先大概说说Stream的概念，这个Stream就是类似平常大家用的比较多的RxJava、RxSwift等框架的思路，它是一个异步数据的源，它可以接受任何数据，通过StreamController可以给这个Stream添加数据，如果这个Stream有监听者就会接收到数据。
代码如下：

```dart
import 'dart:async';

void main() {
  // 初始化一个Stream controller
  final StreamController ctrl = StreamController();
  // 添加一个监听
  ctrl.stream.listen((data) => print('$data'));
  // 往Stream中添加数据
  ctrl.sink.add('这个是字符串'); //字符串类型
  ctrl.sink.add(1234);//int 类型
  ctrl.sink.add({'name': '姓名', 'content': 'json对象也行'});//对象类型
  ctrl.sink.add(123.45); //double类型
  // StreamController用完后需要释放
  ctrl.close();
}
```

<!-- more -->

### StreamBuilder
以前我就经常说Flutter有各种个样的Widget，只有你想不到的。所以既然有Stream，就有对应的Widget，叫StreamBuilder。它本身是一个Widget，然后会监听一个Stream，只要这个Stream有数据变化，它内部的UI就会根据这个新的数据进行变化，完全不需要`setState((){}) `。

这里我就拿平常很常见的一个例子来看看这个StreamBuilder怎么用的：发送验证码的按钮
![倒计时.2019-07-19 11_14_57-w410](http://img.muliba.net/2019-07-19-%E5%80%92%E8%AE%A1%E6%97%B6.2019-07-19%2011_14_57.gif)

首先建一个类处理倒计时的业务，这个有点类似BLoC模式，将UI和业务逻辑分离。

```dart

class CountTime {
  StreamController<int> _controller;
  final int maxSecond;
  CountTime({this.maxSecond = 60}){
    _controller = StreamController();
  }
  ///
  ///提供一个stream
  ///
  Stream<int> get countTimeStream => _controller.stream;
  ///
  /// 开始倒计时
  ///
  startCount() async {
    if (maxSecond <= 0) {
      return;
    }
    for(var i = maxSecond; i >= 0; i--){
      Duration duration;
      if (i == maxSecond) {
        duration = Duration(seconds: 0);
      }else {
        duration = Duration(seconds: 1);
      }
      await Future.delayed(duration, (){
        return i
      }).then((s){
        _controller.sink.add(s);
      });
    }
  }
  dispose() {
    _controller.close();
  }
}
```
这个业务逻辑类里面就是定义了一个StreamController，控制倒计时的输出，它提供一个countTimeStream供外部调用。
这时候在界面UI上，我们使用StreamBuilder监听这个Stream：

```dart
 @override
  void initState() {
  //初始化业务类
    _countTime = CountTime(maxSecond: 60);
    super.initState();
  }
  @override
  void dispose() {
  //销毁
    _countTime.dispose();
    super.dispose();
  }
  
...

StreamBuilder(
    //监听的Stream
    stream: _countTime.countTimeStream,
    //初始化值
    initialData: 0,
    //根据Stream变化进行修改的UI
    builder: (context, snap) {
      var buttonText;
      if (snap.data == 0) {
        onclick = (){
          _countTime.startCount();
        };
        buttonText = '发送验证码';
      }else {
        onclick = null;
        buttonText = '${snap.data} 秒后可以再次发送';
      }
      return FlatButton(
        onPressed: onclick,
        child: Text(buttonText),
        color: Colors.lightBlue,
        textColor: Colors.white,
        disabledColor: Colors.grey,
        disabledTextColor: Colors.white30,
      );
    }
)
```


这个倒计时按钮就完成了。

对了，全部源码：[https://github.com/fancylou/StreamBuilder_Demo](https://github.com/fancylou/StreamBuilder_Demo)

