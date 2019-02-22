---
layout: post
title: "Flutter 学习8：BottomSheet"
categories: flutter
date: 2019-01-26 21:00:00
---

1. [Flutter 学习1：开发环境、开发工具、初始化一个项目](http://www.muliba.net/ios/android/2018/11/16/Flutter-%E5%AD%A6%E4%B9%A0-%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83-%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7-%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html)
2. [Flutter 学习2：从main.dart文件说起](http://www.muliba.net/flutter/2018/11/23/Flutter-%E5%AD%A6%E4%B9%A0-%E4%BB%8Emain.dart%E6%96%87%E4%BB%B6%E8%AF%B4%E8%B5%B7.html)
3. [Flutter 学习3：转场、导航](http://www.muliba.net/flutter/2018/12/04/Flutter-%E5%AD%A6%E4%B9%A03-%E8%BD%AC%E5%9C%BA-%E5%AF%BC%E8%88%AA.html)
4. [Flutter 学习4：集成到原有的项目中](http://www.muliba.net/flutter/2018/12/09/Flutter-%E5%AD%A6%E4%B9%A04-%E9%9B%86%E6%88%90%E5%88%B0%E5%8E%9F%E6%9C%89%E7%9A%84%E9%A1%B9%E7%9B%AE%E4%B8%AD.html)
5. [Flutter 学习5：开发Dart包和插件包](http://www.muliba.net/flutter/2018/12/14/Flutter-%E5%AD%A6%E4%B9%A05-%E5%BC%80%E5%8F%91Dart%E5%8C%85%E5%92%8C%E6%8F%92%E4%BB%B6%E5%8C%85.html)
6. [Flutter 学习6：绘制动画](http://www.muliba.net/flutter/2018/12/28/Flutter-%E5%AD%A6%E4%B9%A06-%E7%BB%98%E5%88%B6%E5%8A%A8%E7%94%BB.html)
7. [Flutter 学习7：Dart语言基础](http://www.muliba.net/flutter/2019/01/11/Flutter-%E5%AD%A6%E4%B9%A07-Dart%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80.html)
8. Flutter 学习8：BottomSheet
9. [Flutter 学习9：Positioned、Transform等控件使用以及手势控制](http://www.muliba.net/flutter/2019/01/31/Flutter-学习9-Positioned-Transform等控件使用以及手势控制.html)
10. [Flutter 学习10：NestedScrollView、SliverAppBar、TabBar](http://www.muliba.net/flutter/2019/02/17/Flutter-学习10-NestedScrollView-SliverAppBar-TabBar.html)

**`BottomSheet`** 是一个Flutter提供的一个组件，有点类似Android的PopWindow，从底部弹出一个Widget。今天就说说这个`BottomSheet`的用法。

首先这个BottomSheet，Flutter提供了两种方式显示，形式有点不同，一个是ModalBottomSheet， 一个是PersistentBottomSheet。

### ModalBottomSheet
这个ModalBottomSheet就是类似一个Dialog，有一个半透明的背景层，然后上面显示你自定义的内容。
用法非常简单，Flutter提供了一个`showModalBottomSheet`的方法弹出一个BottomSheet。

```dart
showModalBottomSheet(
        context: context,
        builder: (context) {
          return Container(
            height: 300,
            color: Colors.greenAccent,
            child: Center(
              child: Text('ModalBottomSheet'),
            )
          );
        })
```

![](http://img.muliba.net/post/modalBottomSheet.2019-01-26 21_29_17.gif)

只要在空白处点击就能关闭这个窗口，而且他这个方法是让你生成一个WidgetBuilder对象，你完全可以自定义任何内容在里面，非常方便实用。



<!-- more -->



### PersistentBottomSheet
这个BottomSheet和上面那个ModalBottomSheet的区别是少了那个半透明的背景层，没有点击关闭的事件了，所以可以操作这个BottomSheet下层的那些UI界面，Button可以点击，ListView可以滚动等等。
![](http://img.muliba.net/post/persistentBottomSheet.2019-01-26 21_40_24.gif)

还有一些其他的区别是，他的使用方式有一定的限制，看上面的动图你会发现他弹出来后，ActionBar多一个返回按钮，点击返回按钮能够关闭这个窗口。

这个`PersistentBottomSheet`是Material Design形式的一个BottomSheet，所以他必须依附于Material的包，他必须在Scaffold的context下才能使用。

```dart
// 这里的context必须是来自Scaffold
showBottomSheet(
context: context,
builder: (context) {
          return Container(
            height: 300,
            color: Colors.greenAccent,
            child: Center(
              child: Text('PersistentBottomSheet'),
            )
          );
        })
        
//或者直接用Scaffold 的state创建
final scaffoldKey = GlobalKey<ScaffoldState>();
...
scaffoldKey.currentState
        .showBottomSheet((context) {
          return Container(
            height: 300,
            color: Colors.greenAccent,
            child: Center(
              child: Text('PersistentBottomSheet'),
            )
          );
        })
```

这两种形式的BottomSheet都有一个下拉关闭的事件，可以往下拉动关闭这个BottomSheet。

### 遇到的问题
1. 这种弹出窗我们为了美观上边会做圆角处理。我们上面的例子用的Container有一个decoration的属性可以处理圆角：

```dart
Container(
            decoration: BoxDecoration(
              color: Colors.greenAccent,
              borderRadius: BorderRadius.vertical(
                top: Radius.circular(16)
              )
            ),
            height: 300,
            child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Text('ModalBottomSheet'),
                ],
            ),
          )
```
但是运行后发现有问题，出现了一个白色的底：
![](http://img.muliba.net/post/Screen Shot 2019-01-26 at 22.04.46.png)
解决办法：
在主题里面把canvasColor改成透明，这样整个背景会变黑色，所以在外层还需要加一个背景色。

```dart
theme: ThemeData(
        primarySwatch: Colors.blue,
        canvasColor: Colors.transparent // 透明
      ),
```
最后结果：
![](http://img.muliba.net/post/Screen Shot 2019-01-26 at 22.06.54.png)

2. 还有是PersistentBottomSheet关闭的问题
有些场景可能你在操作别的地方的时候关闭这个BottomSheet，如何关闭呢。这个`showBottomSheet`方法返回的是一个`PersistentBottomSheetController`对象，这个对象提供了一个close方法，可以这样关闭：

```dart
bottomSheetController.close();
```
但是如果这个窗口已经关闭了执行这个方法会报错，这个Controller还提供了一个回调`closed`，就是关闭后回调一下。
所以我们一般这样处理：

```dart
bottomSheetController = scaffoldKey.currentState
        .showBottomSheet((context) {
          return Container(
            height: 300,
            color: Colors.greenAccent,
            child: Center(
              child: Text('PersistentBottomSheet'),
            )
          );
        })
        .closed
        .whenComplete(() {
        // closed的时候把bottomSheetController设为null
          bottomSheetController = null
        });
```
然后在关闭的地方：

```dart
if(bottomSheetController!=null) {
      bottomSheetController.close();
    }
```


如上，这就是一个BottomSheet组件的使用。Flutter提供了很多类似这样的组件方便开发者们能够短时间内构建一个漂亮的应用。赞一个😬

