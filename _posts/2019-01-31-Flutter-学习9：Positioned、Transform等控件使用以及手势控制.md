---
layout: post
title: "Flutter 学习9：Positioned、Transform等控件使用以及手势控制"
categories: flutter
date: 2019-01-31 15:00:00
---

1. [Flutter 学习1：开发环境、开发工具、初始化一个项目](http://www.muliba.net/ios/android/2018/11/16/Flutter-%E5%AD%A6%E4%B9%A0-%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83-%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7-%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html)
2. [Flutter 学习2：从main.dart文件说起](http://www.muliba.net/flutter/2018/11/23/Flutter-%E5%AD%A6%E4%B9%A0-%E4%BB%8Emain.dart%E6%96%87%E4%BB%B6%E8%AF%B4%E8%B5%B7.html)
3. [Flutter 学习3：转场、导航](http://www.muliba.net/flutter/2018/12/04/Flutter-%E5%AD%A6%E4%B9%A03-%E8%BD%AC%E5%9C%BA-%E5%AF%BC%E8%88%AA.html)
4. [Flutter 学习4：集成到原有的项目中](http://www.muliba.net/flutter/2018/12/09/Flutter-%E5%AD%A6%E4%B9%A04-%E9%9B%86%E6%88%90%E5%88%B0%E5%8E%9F%E6%9C%89%E7%9A%84%E9%A1%B9%E7%9B%AE%E4%B8%AD.html)
5. [Flutter 学习5：开发Dart包和插件包](http://www.muliba.net/flutter/2018/12/14/Flutter-%E5%AD%A6%E4%B9%A05-%E5%BC%80%E5%8F%91Dart%E5%8C%85%E5%92%8C%E6%8F%92%E4%BB%B6%E5%8C%85.html)
6. [Flutter 学习6：绘制动画](http://www.muliba.net/flutter/2018/12/28/Flutter-%E5%AD%A6%E4%B9%A06-%E7%BB%98%E5%88%B6%E5%8A%A8%E7%94%BB.html)
7. [Flutter 学习7：Dart语言基础](http://www.muliba.net/flutter/2019/01/11/Flutter-%E5%AD%A6%E4%B9%A07-Dart%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80.html)
8. [Flutter 学习8：BottomSheet](http://www.muliba.net/flutter/2019/01/26/Flutter-%E5%AD%A6%E4%B9%A08-BottomSheet.html)
9. Flutter 学习9：Positioned、Transform等控件使用以及手势控制

上次说了一个`BottomSheet`的控件，非常实用的一个弹出控件。Flutter提供了很多有意思的控件，好好利用这些控件就能搭建出任何你想要的效果。

### Positioned
看这个名字就大概能了解到这个是定位用的，但是它不是随便哪里都能定位的，它必须配合Stack控件用。它通过 **[left], [right], [width]，[top], [bottom], [height]** 这六个属性将Stack的某个子控件定位到特定的位置上。

```dart
    Stack(
        children: <Widget>[
          Positioned(
            top: 150,
            left: 30,
            child: Container(
              color: Colors.red,
              width: 200,
              height: 200,
            ),
          ),
          Positioned(
            left: 40,
            top: 200,
            width: 50,
            height: 50,
            child: Container(
              color: Colors.teal,
            ),
          )
        ],
      )
```
![Screen Shot 2019-01-31 at 14.14.28-w417](http://img.muliba.net/Screen%20Shot%202019-01-31%20at%2014.14.28.png-500.500)


### Transform
Transform已经有写好的三种用法分别是scale、rotate、translate 。用起来非常方便，一个个来看看怎么用
#### Transform.scale
它就一个参数`scale`。在上面代码的基础上，我们把那个红色方块用Transform.scale包起来就可以放大缩小了。

```dart
var myScale = 1.0;
void _scale(bool plus) {
    if(plus) {
      myScale += 0.1;
    }else {
      myScale -= 0.1;
    }
    setState(() {});
  }
  ...
    Stack(
        children: <Widget>[
          Positioned(
            top: 150,
            left: 30,
            child: Transform.scale(
              scale: myScale,
              child: Container(
              color: Colors.red,
              width: 200,
              height: 200,
            ),
            ),
          ),
          Positioned(
            left: 40,
            top: 200,
            width: 50,
            height: 50,
            child: Container(
              color: Colors.teal,
            ),
          ),
          Positioned(
            bottom: 20,
            right: 40,
            child: Container(
              width: 200,
              height: 64,
              child: Row(
                children: <Widget>[
                  FlatButton(
                    child: Icon(Icons.add_circle),
                    onPressed: () => _scale(true),
                  ),
                  FlatButton(
                    onPressed:  () => _scale(false),
                    child: Icon(Icons.remove_circle),
                  )
                ],
              ),
            ),
          )
        ],
      )
```
![Transform.2019-01-31 15_16_14](http://img.muliba.net/Transform.2019-01-31%2015_16_14.gif)


#### Transform.rotate
它是用来旋转一定的角度用的，也只有一个参数angle。

```dart
    Positioned(
            left: 40,
            top: 200,
            width: 50,
            height: 50,
            child: Transform.rotate(
              angle: math.pi / 2, //90度
              child: Container(
                color: Colors.teal,
                child: Center(child: Text('中国', style: TextStyle(color: Colors.white),)),
              ),
            ),
          )
```
![Screen Shot 2019-01-31 at 15.38.26-w417](http://img.muliba.net/Screen%20Shot%202019-01-31%20at%2015.38.26.png-500.500)

#### Transform.translate
最后一个是变化位置，有点类似Positioned的，就是x轴y轴的移动。

```dart
    Positioned(
            left: 40,
            top: 200,
            width: 50,
            height: 50,
            child: Transform.translate(
              offset: Offset(130, 0),//x轴方向移动130
              child: Container(
                color: Colors.teal,
                child: Center(child: Text('中国', style: TextStyle(color: Colors.white),)),
              ),
            ),
          )
```
![Screen Shot 2019-01-31 at 15.51.17-w417](http://img.muliba.net/Screen%20Shot%202019-01-31%20at%2015.51.17.png-500.500)


### 手势控制
做移动开发的看到这里其实很容易就能想到我们经常用到的图片查看器，图片可以用双指放大缩小，单指移动位置。
那用上面两个控件结合手势控制的`GestureDetector`是否就可以实现图片查看器的功能了呢，试试！

网上随便找了一个小姐姐的图片，我随便找你们随便看！放大缩小移动的效果如下：

![ScreenRecording_01-31-2019 17-30-23.2019-01-31 17_33_36](http://img.muliba.net/ScreenRecording_01-31-2019%2017-30-23.2019-01-31%2017_33_36.gif)


下面是实现的代码：

```dart
class _MyHomePageState extends State<MyHomePage> {
  var myScale = 1.0;
  var myPosition = Offset(0.0, 0.0);
  var mySize = Size(0.0, 0.0);
  var lastPosition = Offset(0.0, 0.0);
  var startMovePosition = Offset(0.0, 0.0);
  var edgePosition = Offset(0.0, 0.0);
  int numPointers = 0;
  var lastScale = 1.0; // 多次放大缩小的时候保存上一次的结果。
  var scaling = false;

  @override
  Widget build(BuildContext context) {
    final size = MediaQuery.of(context).size;
    mySize = size * myScale;
    final dx = -(mySize.width - size.width);
    final dy = -(mySize.height - size.height);
    edgePosition = Offset(dx, dy);
    return Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
        ),
        body: Listener(
            onPointerDown: (_) => numPointers++,
            onPointerUp: (_) => numPointers--,
            child: GestureDetector(
              child: Stack(
                children: <Widget>[
                  Positioned(
                    top: myPosition.dy,
                    left: myPosition.dx,
                    width: mySize.width,
                    height: mySize.height,
                    child: Transform.scale(
                      scale: myScale,
                      child: Image.network(
                        'http://img.muliba.net/post/timg.jpeg',
                        fit: BoxFit.cover,
                      ),
                    ),
                  ),
                ],
              ),
              onScaleStart: _scaleStart,
              onScaleEnd: _scaleEnd,
              onScaleUpdate: _scaleUpdate,
            )));
  }

  void _scaleStart(ScaleStartDetails detail) {
    if (numPointers == 1 && !scaling) {
    lastPosition = myPosition; //标记下位置
    startMovePosition = detail.focalPoint;
    }
    if (numPointers == 2) {
      if (!scaling) {
        scaling = true;
      }
    }
  }

  void _scaleUpdate(ScaleUpdateDetails detail) {
    if (numPointers == 1 && !scaling) {
    final distance = detail.focalPoint - startMovePosition; // 移动距离
    final newPosition = lastPosition + distance; //移动
    var x = newPosition.dx;
    if (x > 0) {
      x = 0.0;
    }
    if (x < edgePosition.dx) {
      x = edgePosition.dx;
    }
    var y = newPosition.dy;
    if (y > 0) {
      y = 0.0;
    } else if (y < edgePosition.dy) {
      y = edgePosition.dy;
    }
    myPosition = Offset(x, y); //移动
    setState(() {});
    }
    if (numPointers == 2) {
      if (scaling) {
        var newScale = lastScale * detail.scale;
        if (newScale < 1.0) {
          newScale = 1.0;
        } else if (newScale > 5.0) {
          newScale = 5.0;
        }
        myScale = newScale;
        // 计算大小
        setState(() {});
      }
    }
  }

  void _scaleEnd(ScaleEndDetails detail) {
    if (scaling) {
      lastScale = myScale;
      scaling = false;
    }
  }
}
```



是不是很方便实用，Flutter确实让开发越来越简单了！👍

