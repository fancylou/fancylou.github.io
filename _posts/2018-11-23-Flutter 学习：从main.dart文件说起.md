---
layout: post
title: "Flutter 学习：从main.dart文件说起"
categories: flutter
date: 2018-11-23 14:10:00
---

这个是我学习Flutter的一个系列文章：
1. [Flutter 学习：开发环境、开发工具、初始化一个项目](http://www.muliba.net/ios/android/2018/11/16/Flutter-%E5%AD%A6%E4%B9%A0-%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83-%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7-%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html)
2. Flutter 学习：从main.dart文件说起

本文第二篇《Flutter 学习：从main.dart文件说起》。

<!-- more -->
### flutter的命令
上次那篇文章写了环境安装，开发工具插件安装然后用插件生成一个`flutter`项目。生成一个新的项目非常简单，点一下插件的`New Project`就行了。其实点击这个`New Project`就是执行了一个命令：

```shell
$ flutter create appName
# 这个appName就是你要生成的项目名
```
这个时候生成的项目里面，android项目用的是默认的java语言，IOS项目用的是Object-C。如果你比较新潮用了kotlin、swift语言。那只需要在那个命令后面加个参数就行了。

```shell
$ flutter create -i swift -a kotlin appName
```
### 热更新
Flutter提供了热更新功能来提升开发者的效率，还是用那个生成的项目来测试下。我把中间的那串英文改成了中文，然后保存一下这个`main.dart`文件：
![](http://img.muliba.net/post/Screen Shot 2018-11-23 at 14.45.57.png-500.500)
再看模拟器，文字已经改了。非常方便！

### 理解main.dart的内容
刚才看模拟器上显示的文字，在`main.dart`里面找到了对应的地方，然后修改文字就行了。对于整个文件和模拟器上显示的UI界面是怎么对应的还不太清楚。
我把代码折叠看了一下：
![](http://img.muliba.net/post/20181123145209.png-500.500)
首先第一行是：

```dart
void main() => runApp(new MyApp());
```
这个main函数是一个入口。启动了这个`MyApp`，这个`MyApp`继承自`StatelessWidget`。下面还有一个是`MyHomePage`继承自`StatefulWidget`，`_MyHomePageState`继承自`State` 。这些都是啥意思呢？
其实，`Flutter`的思路就是来自于Facebook的React，它的整个UI界面就是由这些`Widget`组合起来的。它就像一棵树，最顶部是这个`MyApp`，里面有子元素`MyHomePage`，`MyHomePage`还有子元素`AppBar`、`FloatingActionButton`、`Text`，一层层往下展开。

 `Widget`主要有两种：`StatelessWidget`、`StatefulWidget` 。
 
* `StatelessWidget`是不可变的，它可以在生成之前接收一些不可变的参数，一旦生成了就不可变了。比如基础的一些组件`Text`（显示文字用的）、`Container`（一个容器）等。

* `StatefulWidget`是可变的，它是如何变化的呢？靠的是`State`，这种`StatefulWidget`都包含了一个`State createState();`，就比如上面`main.dart`里面的`MyHomePage`

```dart
class MyHomePage extends StatefulWidget {
    @override
    _MyHomePageState createState() => new _MyHomePageState();
}
class _MyHomePageState extends State<MyHomePage> {
    ...
}
```

`State`的生命周期比`Widget`长，`Widget`是临时的，`State`的`build()`生成需要显示的`StatefulWidget`，当状态参数变化的时候执行`setState() ` 它会再次调用`build()`生成新的`StatefulWidget`，所以我们在界面上就看到了变化！
#### MaterialApp
`main.dart`里面还有一个比较有意思的就是`MyApp`，它其实就是一个`MaterialApp`，这个Widget是Flutter提供的按照Material Design设计的一个主题框架。这个开发Android的应该都很熟悉。Android有很多这种Material Design组件。这个组件提供了主题功能，所以切换主题就变的分分钟的事情了.
这是默认生成的代码，它的主题色是蓝色：

```dart
new MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: new MyHomePage(title: 'Flutter Demo Home Page'),
    );
```
修改成紫色：
```dart
new MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData(
        primarySwatch: Colors.purple,
      ),
      home: new MyHomePage(title: 'Flutter Demo Home Page'),
    );
```
![](http://img.muliba.net/post/20181123164742.png-500.500)

还有暗黑模式：
```dart
new MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData.dark(),      
      home: new MyHomePage(title: 'Flutter Demo Home Page'),
    );
```
![](http://img.muliba.net/post/20181123164727.png-500.500)

### 制作一个列表页面
#### 引入外部包
我把main.dart修改一下，把它做成一个列表，列表在移动端是很常见的东西，看看Flutter怎么实现。
首先看下我们的项目根目录下有个文件`pubspec.yaml`，这个是项目的配置文件，配置了项目名称，flutter依赖的包，环境版本等等信息。这个配置文件信息量很大，我现在只了解了依赖包的使用。
添加依赖包就在`dependencies`节点下面，关于依赖包可以去[官方的开源包网站](https://pub.dartlang.org/flutter/)查找更多：

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^0.1.2
  # 新增一个单词包,列表展现用的
  english_words: ^3.1.0
```
添加了包之后，需要到Terminal上去执行一下命令：

```shell
$ flutter packages get
```
然后就可以在我们的main.dart头部引入这个包了：

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';
```
修改main.dart代码：

```dart
class MyApp extends StatelessWidget {
  final wordPair = WordPair.random();
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData.dark(),
      home: Scaffold(
        appBar: new AppBar(
          title: new Text('Welcome to Flutter'),
        ),
        body: new Center(
          child: Text(wordPair.asPascalCase),
        ),

      ),
    );
  }
}
```
这样就能在我们的屏幕中间显示一个随机的英文单词。
![](http://img.muliba.net/post/20181124102140.png-500.500)
#### 定义一个StatefulWidget
我们把中间这个Text改成一个前面说的`StatefulWidget`。

```dart
class RandomWordsState extends State<RandomWords> {
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();
    return Text(wordPair.asPascalCase);
  }
}

class RandomWords extends StatefulWidget {
  @override
  RandomWordsState createState() => new RandomWordsState();
}
```
#### ListView
现在列表的元素已经有了，那我们可以引入列表了`ListView`，生成一个无限滚动的列表。
修改下`RandomWordsState` ：

```dart
Widget  _buildSuggestions() {
    return ListView.builder(
      padding: const EdgeInsets.all(16.0),
      itemBuilder: (context, i){
        //分隔符
        if (i.isOdd) return Divider();
        final index = i ~/ 2;
        if (index >= _suggestions.length) {
          // 每次获取10个单词加入数组，让它有无限长。。。
          _suggestions.addAll(generateWordPairs().take(10));
        }
        return _buildItem(_suggestions[index]);
      },
    );
  }
```
#### ListTile
这个`_buildSuggestions`生成一个`ListView`，这个`ListView`是Flutter的提供的组件，里面有很多参数可以调整。`itemBuilder`是生成列表子项的build方法，每次显示一个子项就会来调用这个`itemBuilder`。
列表子项的生成：

```dart
Widget _buildItem(WordPair pair) {
    return ListTile(
      title: Text(
        pair.asPascalCase,
        style: _biggerFont,
        ),
    );
  }
```
它是一个`ListTile`，`ListView`的一个子项，就是一般列表的一行。我们这里就显示一个单词，所以就定义了它的title用`Text`显示，`ListTile`还可以定义leading，subtitle等。
这样列表和列表的行都有了，我们把`RandomWordsState`的build方法修改下用这个列表替换:

```dart
@override
Widget build(BuildContext context) {
    return _buildSuggestions();
}
```
保存后列表就出现了，可以一直往下滚动：
![](http://img.muliba.net/post/Screen Shot 2018-11-24 at 11.04.07.png-500.500)










