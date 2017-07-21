---
layout: post
title: "怎么样让你的代码更优雅更帅，Kotlin扩展和高阶函数"
categories: kotlin
date: 2017-07-21 13:57:00
---



好长一段时间没来写了，今天又突然想来写一个，在使用Kotlin的过程中，你总能体会到很多的惊喜，包括下面要说的扩展、还有高阶函数（high-order functions）

扩展的官方定义是这样的：Kotlin, similar to C# and Gosu, provides the ability to extend a class with new functionality without having to inherit from the class or use any type of design pattern such as Decorator. This is done via special declarations called *extensions*. Kotlin supports *extension functions* and *extension properties*.

不需要继承类就能直接新添加一个函数到类里面。这种逆天功能有啥好处？ 我觉得可以让我们的代码更帅。

比如，在Android开发中我们要显示或者隐藏一个View，原来的写法：

```kotlin
tv.visibility = View.VISIBLE
or
tv.visibility = View.GONE
```

我们给View写个扩展：

```kotlin
fun View.visible() {
    visibility = View.VISIBLE
}
fun View.gone() {
    visibility = View.GONE
}
fun View.inVisible() {
    visibility = View.INVISIBLE
}
```

那新的写法就是这样的：

```kotlin
tv.visible()
or 
tv.gone()
```

这样的代码看起来是不是更帅、更直观。

再比如，我们经常要写一个Activity跳转到另外一个Activity，代码：

```kotlin
val intent = Intent(this, AnotherActivity::class.java)
startActivity(intent)
```

或者你还想传数据过去：

```kotlin
val intent = Intent(this, AnotherActivity::class.java)
val bundle = Bundle()
...
intent.putExtras(bundle)
startActivity(intent)
```

现在可以给Activity写个扩展：

```kotlin
inline fun <reified T : Activity> Activity.go(bundle: Bundle? = null) {
    val intent = Intent(this, T::class.java)
    if (bundle != null) {
        intent.putExtras(bundle)
    }
    startActivity(intent)
}
```

OK，现在跳转是这么写的：

```kotlin
go<AnotherActivity>()
```

或者带参数：

```kotlin
go<AnotherActivity>(bundle)
```

awesome! 

还有更精彩的，high-order functions

继续举个栗子，Android开发经常用到的本地存储，存储各种基本类型的数据：

```kotlin
val prefs: SharedPreferences = ...
val editor = prefs.edit()
editor.putInt("key1", 1)
editor.putString("key2", "string")
...
editor.apply()

```

每次都要写这些固定的，重复的apply()等提交的函数，非常烦。

那kotlin怎么玩？

```kotlin
inline fun SharedPreferences.edit(func: SharedPreferences.Editor.()->Unit) {
    val editor = edit()
    editor.func()
    editor.apply()
}
```

这是啥意思，看下怎么用就知道 这上面的是啥意思了，

使用：

```kotlin
val prefs: SharedPreferences = ...
prefs.edit {
    putInt("key1", 1)
    putString("key2", "string")
    ...
}
```

kotlin的高阶函数，可以将函数作为一个参数传入到一个方法中也作为一个返回值返回。

其实上面的edit定义简单解释就是给SharedPreferences添加了一个叫edit的扩展函数，这个扩展函数的传入值是一个函数，而这个函数是SharedPreferences.Editor的一个扩展函数。这个叫edit的扩展做的事情就是执行这个传入的func，然后提交apply().

有点绕口，稍微理解下就明白，看看新的写法是不是更优雅、更帅。

再来一个栗子，也是经常用到的场景：

kotlin可以使用Lambdas写出优雅的单个函数的接口实现，比如：

```kotlin
view.setOnClickListener { do some thing ... }
```

但是有些需要实现多个函数的接口就比较麻烦， 比如：

```kotlin
viewPager.addOnPageChangeListener(object : ViewPager.OnPageChangeListener {
  override fun onPageScrollStateChanged(state: Int) { 
  		do some thing ...
  }
  override fun onPageScrolled(position: Int, positionOffset: Float, positionOffsetPixels: Int) {
  		do some thing ...	
  }
  override fun onPageSelected(position: Int) {
  		do some thing ...
  }
})
```

这种代码每次一大坨，又难看，而且经常是我们可能只需要实现里面的一个函数就够了。

看看使用kotlin的高阶函数，如何优雅的处理这个问题。

```kotlin
class _OnPageChangeListener : ViewPager.OnPageChangeListener {

    private var _onPageScrollStateChanged: ((state: Int) -> Unit)? = null
    private var _onPageScrolled: ((position: Int, positionOffset: Float, positionOffsetPixels: Int) -> Unit)? = null
    private var _onPageSelected: ((position: Int) -> Unit)? = null


    override fun onPageScrollStateChanged(state: Int) {
        _onPageScrollStateChanged?.invoke(state)
    }
    fun onPageScrollStateChanged(func: (state: Int) -> Unit) {
        _onPageScrollStateChanged = func
    }

    override fun onPageScrolled(position: Int, positionOffset: Float, positionOffsetPixels: Int) {
        _onPageScrolled?.invoke(position, positionOffset, positionOffsetPixels)
    }
    fun onPageScrolled(func: (position: Int, positionOffset: Float, positionOffsetPixels: Int) -> Unit) {
        _onPageScrolled = func
    }

    override fun onPageSelected(position: Int) {
        _onPageSelected?.invoke(position)
    }
    fun onPageSelected(func: (position: Int) -> Unit) {
        _onPageSelected = func
    }
}
```

```kotlin
inline fun ViewPager.addOnPageChangeListener(func:_OnPageChangeListener.()-> Unit) {
    val listener = _OnPageChangeListener()
    listener.func()
    addOnPageChangeListener(listener)
}
```

通过这些封装，我们就可以简单的使用了：

```kotlin
viewPager.addOnPageChangeListener {
	onPageScrolled {
      	do some thing ...
	}
  	onPageSelected {
    	do some thing ...
  	}
}
```

这样看起来代码有没有更清晰，更明白，而且里面的实现函数可以写一个，也可以写多个。

如果你是java的使用者，我真心觉得该转用kotlin了，难道这一个个精彩的特性没有打动你吗？ 

IT‘S COOL ！