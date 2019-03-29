---
layout: post
title: "Flutter使用自定义的Icon"
categories: flutter
date: 2019-03-29 13:00:00
---

Flutter默认开启了`uses-material-design`。
![Screen Shot 2019-03-29 at 13.02.50](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2013.02.50.png)
这使得我们在开发中可以轻松使用很多定义好的图标，非常方便，也节约了很多设计或者找图的时间。

但需求无止境嘛，虽然这个material design的Icons库里面已经有很多图标了，有些需求或者场景下想找到一个符合语境的图标还是很难。不过最近发现了一个网站 [Flutter Icon](http://fluttericon.com) ，可以通过上传svg文件生成自定义字体文件，然后就可以使用自定义的Icon了。


<!-- more -->


### SVG生成字体文件
打开网站，首先你就能看到`Custom Icons`的一个区域就是上传SVG文件的，右上角`DOWNLOAD`按钮左边还有一个输入框是定义你的Dart类名的，这个名字干啥用的后面会讲到。
![Screen Shot 2019-03-29 at 13.11.59-w1179](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2013.11.59.png)

我这边测试放入了5个SVG图标文件，自定义了一个Dart类名为：`CustomIcons` 。
![Screen Shot 2019-03-29 at 12.47.18-w1199](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2012.47.18.png)
这几个SVG文件我是从[iconfont](www.iconfont.cn)上面下载下来的，那上面下载下来的文件都有控制图标大小的代码，可以去掉，这样我们在Flutter中使用Icon的时候可以自定义大小了。如下图，删除width和height标签。
![Screen Shot 2019-03-29 at 13.17.55-w811](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2013.17.55.png)

然后选中上传的那几个图标，点击`DOWNLOAD`按钮，就会下载到一个zip包，解压后看到的一个json文件，一个dart文件和一个fonts目录里面是一个ttf字体文件：
![Screen Shot 2019-03-29 at 10.34.55](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2010.34.55.png)
这样文件就生成好了，下面是在Flutter中使用了。

### Flutter中使用字体文件
把上面生成的fonts目录拖到项目工程目录下，然后配置`pubspec.yaml` ，找到`flutter:` 标签，配置如下代码：

```yaml
fonts:
   - family: CustomIcons
     fonts:
       - asset: fonts/CustomIcons.ttf
```

![Screen Shot 2019-03-29 at 13.24.07-w721](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2013.24.07.png)

然后把上面生成的dart文件也拖进工程lib目录下
![Screen Shot 2019-03-29 at 13.27.28-w1111](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2013.27.28.png)

现在只要在我们要使用icon的地方引入这个custom_icons_icons.dart文件就可以使用了，比如我测试的代码：

```dart
import 'custom_icons_icons.dart';
...
    Column(
          children: <Widget>[
            IconButton(
                iconSize: 48,
                icon: Icon(
                  CustomIcons.guanlianshebei,
                  color: Colors.red,
                ),
                onPressed: null),
            IconButton(
                iconSize: 48,
                icon: Icon(
                  CustomIcons.jishufuwu,
                  color: Colors.red,
                ),
                onPressed: null),
            IconButton(
                iconSize: 48,
                icon: Icon(
                  CustomIcons.jizhanguanli,
                  color: Colors.red,
                ),
                onPressed: null),
            IconButton(
                iconSize: 48,
                icon: Icon(
                  CustomIcons.quanxianshenpi,
                  color: Colors.red,
                ),
                onPressed: null),
            IconButton(
                iconSize: 48,
                icon: Icon(
                  CustomIcons.shebeikaifa,
                  color: Colors.red,
                ),
                onPressed: null)
          ],
        )
```
显示结果：
![Screen Shot 2019-03-29 at 13.31.10-w389](http://img.muliba.net/Screen%20Shot%202019-03-29%20at%2013.31.10.png)
上面的代码我们可以非常方便的自定义icon的大小、颜色等属性。

PS：关于上面生成自定义图标的网站，它在GitHub上开源的，这个wiki可以帮助你如何使用这个网站 [https://github.com/ilikerobots/polyicon/wiki/Help](https://github.com/ilikerobots/polyicon/wiki/Help)

