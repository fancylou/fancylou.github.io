---
layout: post
title: "Flutter Widget - PageView的使用"
categories: flutter
date: 2019-04-10 13:00:00
---

PageView 在Android中也是很常用的一个控件，可以配合TabBar、BottomBar做页面切换。也有很多广告Banner页用PageView来实现。Flutter也有这个Widget，用法更灵活方便。


```dart
 PageView({
    Key key,
    this.scrollDirection = Axis.horizontal,
    this.reverse = false,
    PageController controller,
    this.physics,
    this.pageSnapping = true,
    this.onPageChanged,
    ...
  }) 
```
#### 参数说明
* scrollDirection 可以控制滚动方向，这个PageView不仅可以横向滚动，还能垂直滚动
* reverse 是控制滚动方向，比如横向滚动的时候，一般我们是从右往左滚动，这个参数设置成true就是默认反一反，变成从左往右滚动。
* controller 有很多操作可以做，可以控制跳转到某一页，可以控制PageView初始显示页面等
* physics 是处理滚动到最前或滚动到最后的时候的动画效果，Flutter默认有提供几个实现好的效果，`BouncingScrollPhysics`、`ClampingScrollPhysics`等
* pageSnapping 控制滚动方式默认true的情况下，PageView都是一页一页翻的，如果设置成false，真个PageView就可以一点点滚动，就是不用整页滚动过去，滚动到一半的时候也能停下来。
* onPageChanged 是一个监听事件，页面切换就调用一次，并传回当前显示页面的序号。

### 默认构造

这个默认构造，一般用在多个页面切换的场景下，只要放入固定的几个页面就行了。
例子代码：

```dart
PageView(
      children: <Widget>[
        Container(
          color: Colors.red,
          child: Center(child: Text('第一页', style: TextStyle(color: Colors.white),),),
        ),
        Container(
          color: Colors.blue,
          child: Center(child: Text('第二页', style: TextStyle(color: Colors.white),),),
        ),
        Container(
          color: Colors.green,
          child: Center(child: Text('第三页', style: TextStyle(color: Colors.white),),),
        ),
        Container(
          color: Colors.blueGrey,
          child: Center(child: Text('第四页', style: TextStyle(color: Colors.white),),),
        )
      ],
    )
```
![1554884412508646.2019-04-10 16_25_39](http://img.muliba.net/2019-04-10-1554884412508646.2019-04-10%2016_25_39.gif)



<!-- more -->


### PageView.builder
builder方式构造，提供了itemBuilder方式，而不是默认构造的children方式，能够更加方便处理动态UI，或者有固定规律的UI，比如广告Banner等。
这种方式需要注意一点，如果不设置itemCount,PageView默认是无限个数的，也就是能一直往后翻页下去，index一直往上增加。
例子代码：

```dart
PageView.builder(
      itemBuilder: (context, index){
        return Center(
          child: Image.network(urls[index], fit: BoxFit.fitWidth),
        );
      },
      itemCount: urls.length,
    )
```
![1554884372282330.2019-04-10 16_22_37](http://img.muliba.net/2019-04-10-1554884372282330.2019-04-10%2016_22_37.gif)


### PageView.custom
custom方式构造，这个其实就是上面两个方式的父，更加底层点，需要提供`SliverChildDelegate`，生成所有的子页面。一般使用不需要用到这个。
例子代码：

```dart
PageView.custom(
        childrenDelegate:
            SliverChildBuilderDelegate((BuildContext context, int index) {
      return Container(
        color: Colors.blue[100 * (index % 9)],
        child: Center(child: Text('这里是子 $index'),) ,
      );
    }, childCount: 10));
```
![1554884403592167.2019-04-10 16_24_45](http://img.muliba.net/2019-04-10-1554884403592167.2019-04-10%2016_24_45.gif)



### 翻页添加变幻动画
先加两个参数：

```dart
PageController controller = PageController();
var currentPageValue = 0.0;
```
然后在initState中添加监听：

```dart
@override
  void initState() {
    controller.addListener(() {
      setState(() {
        currentPageValue = controller.page;
      });
    });
    super.initState();
  }
```
使用Transform进行旋转：

```dart
    PageView.builder(
      controller: controller,
      itemBuilder: (context, index) {
        if (index == currentPageValue.floor()) {
          return Transform(
            transform: Matrix4.identity()..rotateX(currentPageValue - index),
            child: fourthPageViewItem(index),
          );
        } else if (index == currentPageValue.floor() + 1){
          return Transform(
            transform: Matrix4.identity()..rotateX(currentPageValue - index),
            child: fourthPageViewItem(index),
          );
        } else {
          return fourthPageViewItem(index);
        }
      },
    );
```
效果：

![1554887157369687.2019-04-10 17_07_32](http://img.muliba.net/2019-04-10-1554887157369687.2019-04-10%2017_07_32.gif)