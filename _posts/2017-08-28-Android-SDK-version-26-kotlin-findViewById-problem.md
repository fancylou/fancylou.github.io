---
layout: post
title: "Android SDK version 26 后kotlin使用findViewById的错误问题"
categroies: kotlin
date: 2017-08-28 13:58:00

---



今天升级了项目Android Support Library到26.0.1版本，同时肯定把SDK版本也升级到26。结果编译项目的出了一堆错，发现是同一个错误：

```
Error:(132, 29) Type inference failed: Not enough information to infer parameter T in fun <T : View!> findViewById(id: Int): T!
Please specify it explicitly.
```

然后我转到代码一看：

![findViewById](http://muliba.u.qiniudn.com/blog/post/20170828/QQ%E6%88%AA%E5%9B%BE20170828140524.png)

我就很奇怪这个`findViewById`这个函数不是SDK本身提供的函数怎么会报错，点进源码发现这个函数好像改了。原来返回值是View，现在改成了:

```java
<T extends View> T findViewById(@IdRes int id)
```

不是直接返回View，而是泛型View的子类。

其实这个本来不是问题，但是使用kotlin语言的人一般习惯这么写：

```kotlin
var toolbar = findViewById(R.id.toolbar) as Toolbar
```

kotlin他会根据这个返回的对象类型来判断这个toolbar变量是什么类型，原来根据这个`findViewById`函数可以推导出这个对象是View，现在它改成泛型，它自己都不知道是什么类型 kotlin也就无法推导出来是什么类型了，所以就报错了。

其实改下很简单，只要直接注明这个变量的类型就行了：

```kotlin
var toolbar : Toolbar = findViewById(R.id.toolbar)
```

或者给这个泛型函数标明类型：

```kotlin
var toolbar = findViewById<Toolbar>(R.id.toolbar)
```



哦 对了 `findViewWithTag`函数也是一样的问题