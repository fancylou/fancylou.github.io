---
layout: post
title: "Flutter 学习5：开发Dart包和插件包"
categories: flutter
date: 2018-12-14 14:10:00
---

这个是我学习Flutter的一个系列文章：
1. [Flutter 学习1：开发环境、开发工具、初始化一个项目](http://www.muliba.net/ios/android/2018/11/16/Flutter-%E5%AD%A6%E4%B9%A0-%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83-%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7-%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html)
2. [Flutter 学习2：从main.dart文件说起](http://www.muliba.net/flutter/2018/11/23/Flutter-%E5%AD%A6%E4%B9%A0-%E4%BB%8Emain.dart%E6%96%87%E4%BB%B6%E8%AF%B4%E8%B5%B7.html)
3. [Flutter 学习3：转场、导航](http://www.muliba.net/flutter/2018/12/04/Flutter-%E5%AD%A6%E4%B9%A03-%E8%BD%AC%E5%9C%BA-%E5%AF%BC%E8%88%AA.html)
4. [Flutter 学习4：集成到原有的项目中](http://www.muliba.net/flutter/2018/12/09/Flutter-%E5%AD%A6%E4%B9%A04-%E9%9B%86%E6%88%90%E5%88%B0%E5%8E%9F%E6%9C%89%E7%9A%84%E9%A1%B9%E7%9B%AE%E4%B8%AD.html)
5. Flutter 学习5：开发Dart包和插件包
6. [Flutter 学习6：绘制动画](http://www.muliba.net/flutter/2018/12/28/Flutter-%E5%AD%A6%E4%B9%A06-%E7%BB%98%E5%88%B6%E5%8A%A8%E7%94%BB.html)
7. [Flutter 学习7：Dart语言基础](http://www.muliba.net/flutter/2018/01/11/Flutter-%E5%AD%A6%E4%B9%A07-Dart%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80.html)
8. [Flutter 学习8：BottomSheet](http://www.muliba.net/flutter/2019/01/26/Flutter-%E5%AD%A6%E4%B9%A08-BottomSheet.html)
9. [Flutter 学习9：Positioned、Transform等控件使用以及手势控制](http://www.muliba.net/flutter/2019/01/31/Flutter-学习9-Positioned-Transform等控件使用以及手势控制.html)


在上面的列表中第二篇文章就说到过怎么使用一个外部的包，需要到`pubspec.yaml`的dependencies中引入，然后到需要使用的Dart文件中import。外部包这种东西在开发中是很常见的了，它有可能是某种需求的UI库，也有可能是方便开发的工具库等等。

在Flutter中这中外部包分两种类型，一种叫`Dart Packages` ，它是纯Dart开发的，不涉及Native层的一些开发工具包或者UI包，比如前面用到`english_words`。还有一种叫`Plugin Packages`，它是涉及到Native层的，需要调用Native的原生代码的，可能是要调用硬件，比如获取电池电量等类似的功能库的。一个外部包至少包含一个`pubspec.yaml`（声明一些包名称、版本、作者等数据的文件），一个`lib`文件夹（包含公开的源代码，最少应该要有一个包名称命名的dart文件）。

前面的文章已经提到了，这些外部包有个[官方的包管理的网站](https://pub.dartlang.org/flutter/) ，现在已经有很多可用的包了，如果开发中想要用什么包，可以先去上面找找，有的话直接拿来用就不用重复造轮子了。当然肯定会有不满足需求的时候，那我们得学会自己开发！😊

<!-- more -->

### Dart包开发
创建Dart包，跟前面建Flutter工程一样用 `flutter create` 命令，需要加一个`template`参数：

```shell
$ flutter create --template=package hello_package
......
All done!
Your package code is in hello_package/lib/hello_package.dart
```
完成后生成的项目结构很简单：

```
hello_package/
    lib/hello_package.dart
    test/hello_package_test.dart
    pubspec.yaml
```

在hello_package.dart实现你的包功能就行了。还有test文件夹下的hello_package_test.dart中可以进行单元测试。关于单元测试如何进行可以参考下[官方的指导](https://flutter.io/docs/testing#unit-testing)。


### 插件包开发

插件包相当于是Dart包的特殊版本，它包含了针对于特定平台（Android、IOS）有特定的实现。Flutter的API通过[平台的通道](https://flutter.io/docs/development/platform-integration/platform-channels)和指定平台进行通信。

创建插件包的命令和前面Dart包类似，就是把`template`参数改成`plugin` ：

```shell
$ flutter create --template=plugin hello_plugin
```
因为涉及到Android和IOS平台，所以会生成Android和IOS平台代码，所以多了几个参数，和创建Flutter工程一样，可以指定Android IOS的包名、开发语言等：

```shell
$ flutter create --org=muliba.net --template=plugin -i swift -a kotlin hello_plugin
```

生成的目录结构和Dart包不一样的是test目录变成了example目录，是一个完整的flutter项目目录。一个android目录，插件包的android端实现。一个ios目录，插件包的ios端实现。

```
hello_plugin/
    lib/hello_plugin.dart
    example/
        android/
        ios/
        lib/
        test/
        pubspec.yaml
    android/
    ios/
    pubspec.yaml
```

所以通过这个目录结构就可以很清楚了解到了，在`lib/hello_plugin.dart`中实现插件包的对外功能，比如包括提供给别人用的Widget、工具函数等。自动生成的代码就是获取当前手机平台的版本号：

```dart
static Future<String> get platformVersion async {
    final String version = await _channel.invokeMethod('getPlatformVersion');
    return version;
  }
```
代码就提供给使用者一个异步获取平台版本的函数。分别看Android端和IOS端的代码实现：

#### Android端

在`android`目录中实现Android 端获取版本号的代码，在编写代码之前，要先用Android Studio对这个android目录进行构建一次。

```kotlin
override fun onMethodCall(call: MethodCall, result: Result) {
    if (call.method == "getPlatformVersion") {
      result.success("Android ${android.os.Build.VERSION.RELEASE}")
    } else {
      result.notImplemented()
    }
  }
```

#### IOS端

在`ios`目录中实现IOS 端获取版本号的代码，在IOS编写代码之前也要用xcode构建一遍。

```swift
public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
    result("iOS " + UIDevice.current.systemVersion)
  }
```


### 发布packages

我们编写插件就是为了给别的项目或者其他人使用的，所以肯定要发布到[Pub](https://pub.dartlang.org/)上，这样别人才可以在自己的flutter项目中依赖使用。
发布之前先检查`pubspec.yaml`、README.md、CHANGELOG.md等文件是否已经准备好了。
运行 dry-run 命令以查看是否都准备OK了:

```shell
$ flutter packages pub publish --dry-run
```

最后发布：

```shell
$ flutter packages pub publish
```



