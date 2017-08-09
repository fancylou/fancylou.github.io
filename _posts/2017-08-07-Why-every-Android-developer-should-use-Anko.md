---
layout: post
title: "为啥每个Android开发者都应该使用Anko"
categories: kotlin
date: 2017-08-07 16:12:00
---



翻译自：[Why every Android developer should use Anko](https://www.kotlindevelopment.com/why-should-use-anko/)

Anko 是一个使用Kotlin语言编写，由JetBrains公司维护的 Android 开发库。其目的是提升那些使用Kotlin语言的Android开发者的开发速率，使得Android开发更加简单。这就是为什么这个库叫Anko（**An**droid **Ko**tlin）。让我们来看看Anko有那些精彩的工具给我们使用！

Anko有4个主要的模块组成：

- Commons
- Layouts
- SQLite
- Coroutines

Commons模块提供了各种各样有用的功能和函数。Layouts模块提供Kotlin代码创建UI的功能，该功能叫做Anko DSL。SQLite模块让android的SQLite数据库开发更简单。最后，Anko从kotlin 1.1 开始提供了一个重磅功能：Kotlin coroutine 。

## 开发Android应用从未如此简单！

这篇文章主要写的是Anko的Commons模块上面的功能函数。

### Anko Commons

#### 最基本的

让我们开始看看这些基础的简化功能！View.setOnClickListener在Android开发中无处不在，所以如果我们能简化它，将减少很多代码。

用Kotlin我们能写的最长的click事件处理是这样的：

```kotlin
button.setOnClickListener(object: View.OnClickListener{
  override fun onClick(view:View) {
  }
})
```

但是用Anko精简之后是这样的：

```kotlin
button.onClick{ }
```

#### 有意的

使用Intent是作为Android开发者初学的时候最早接触的，但是这个API应该更简单些。

```kotlin
val intent = Intent(this, MainActivity::class.java)
intent.putExtra("id", 5)
intent.putExtra("name", "John")
startActivity(intent)
```

现在来看看更简单的版本：

```kotlin
startActivity<MainActivity>("id" to 5, "name" to "John")
```

一行代码搞定。

Anko还有封装了很多常用的Intent：

```kotlin
browse("https://makery.co")
share("share", "subject")
email("hello@makery.co", "Great app idea", "potato")
```

#### 与陌生人聊天更简单

Anko也精简了Dialog的API，使代码更友好，不再需要builder模式。

```kotlin
val builder = AlertDialog.Builder(this)
builder.setTitle("title")
builder.setMessage("Java is ...Old")
builder.setPositiveButton("ok"){dialog, which -> toast("yes")}
builder.setNegativeButton("cancel"){dialog, which -> toast("no") }
builder.show()
```

```kotlin
alert(Appcompat, "Kotlin", "Kotlin is so fresh!") {
  customView{editText()}
  positiveButton("ok"){toast("yes")}
  negativeButton("cancel"){toast("no")}
}.show()
```

#### 尺寸问题

当我们需要对dpi进行计算的时候，以前的代码很难理解。

```kotlin
val dpAsPx = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 10f, getResources().getDisplayMetrics())
```

现在的方式无比简单：

```kotlin
val dpAsPx = dip(10)
```

当然字体大小也一样：

```kotlin
sp(16)
```

#### API Level 23

版本碎片话问题是每个Android开发者都会面对的问题。我们不想丢掉老版本Android用户，但是同时我们也希望能使用新版的那些酷炫的新特性。

我们怎么办，为了平衡这问题，我们经常要写这样的代码：

```kotlin
if (Build.VERSION.SDK_INT == Build.VERSION_CODES.LOLLIPOP){ }
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP){ }
```

让我们看看Anko怎么做：

```kotlin
doIfSdk(Build.VERSION_CODES.LOLLIPOP){ }
doFromSdk(Build.VERSION_CODES.LOLLIPOP){ }
```

#### I just had a snack at the snack bar

Android的Snack bar相关的API也是需要改进，你是否经常在make了snack bar后忘了执行show()方法，然后各种debug查找问题，为啥没有出现snack bar？ 不是吧？ 难道就我会忘了？

```kotlin
Snackbar.make(findViewById(android.R.id.content), "This is a snack!", Snackbar.LENGTH_LONG).show()
```

#### 你不再需要执行show()！

```kotlin
longSnackbar(findViewById(android.R.id.content), "This is a snack!")
```

如果你是toast粉，Anko同样有类似代码给你用：

```kotlin
toast("Message")
```

#### Threading the needle

处理多线程往往是一个难事，但是这对一个手机应用开发者而言是一个很常用的方式，因为很多处理不能在UI线程上进行。Anko给了我们一个漂亮、简单、明了的方式：

```kotlin
doAsync {
 //IO task or other computation with high cpu load
 uiThread {
   toast("async computation finished")
 }
}
```

#### Add Anko to your project

如果你喜欢上面的那些功能，那还等什么呢？马上添加Anko到你的项目中吧！

```gas
ankoVersion = "0.10.1"
dependencies {
 compile "org.jetbrains.anko:anko-appcompat-v7-listeners:$ankoVersion"
 compile "org.jetbrains.anko:anko-design-listeners:$ankoVersion"
 compile "org.jetbrains.anko:anko-design:$ankoVersion"
 compile "org.jetbrains.anko:anko-sdk15-listeners:$ankoVersion"
 compile "org.jetbrains.anko:anko-sdk15:$ankoVersion"
}
```

#### +1 Bye-bye findViewById()

你是否听说过Kotlin Android Extensions Gradle plugin？你从此可以摆脱烦人的findViewById()函数了，在你的Gradle Script中添加一行

```
apply plugin: 'kotlin-android-extensions'
```

现在你只要参考xml文件里面定义的view的id：

```xml
// activity_main.xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:orientation="vertical">
 
 <Button android:id="@+id/button"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text="OK"/>

</LinearLayout>
```

```kotlin
// MainActivity.kt

import kotlinx.android.synthetic.main.activity_main.*
...
override fun onCreate(savedInstanceState: Bundle?) {
 super.onCreate(savedInstanceState)
 setContentView(R.layout.activity_main)
 button.onClick {  }
}
```





