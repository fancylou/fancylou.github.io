---
layout: post
title: "Android SDK version 26 kotlin使用遇到的问题"
categories:  kotlin
date:  2017-08-28 13:58:00

---



## Part One

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



## Part Two

`Android Support Library` 从 `26.0.0` 开始`supportFragmentManager`加入了一个函数`isStateSaved()`判断当前FragmentManger的状态是否已经被宿主给保存了。也就是当前的Activity是否已经调用了`onSaveInstanceState()`函数保存状态。

因为我们如果在状态保存之后进行Fragment的提交执行了`Commit()`函数， 就会抛出这个异常：

```
java.lang.IllegalStateException:Can not perform this action after onSaveInstanceState
```

所以有了这个函数，我们在进行Fragment提交的时候就可以更加安全了。

怎么安全呢，写了个扩展函数，kotlin不写扩展函数怎么行呢

```kotlin
fun AppCompatActivity.replaceFragmentSafely(fragment: Fragment, tag: String, @IdRes containerViewId: Int,
                                            allowState: Boolean = false,
                                            isAddBackStack: Boolean = false,
                                            @AnimRes enterAnimation: Int = 0,
                                            @AnimRes exitAnimation: Int = 0,
                                            @AnimRes enterPopAnimation: Int = 0,
                                            @AnimRes exitPopAnimation: Int = 0) {
    val ft = supportFragmentManager.beginTransaction()
    ft.setCustomAnimations(enterAnimation, exitAnimation, enterPopAnimation, exitPopAnimation)
    if (isAddBackStack) {
        ft.addToBackStack(tag)
    }
    ft.replace(containerViewId, fragment, tag)
    if (!supportFragmentManager.isStateSaved) {// isStateSaved is from Android Support Library version 26.0.0
        ft.commit()
    } else if (allowState) {
        ft.commitAllowingStateLoss()
    }
}
```

这样就避免了，因为判断调用commit的时机错误而引起的异常。

```kotlin
replaceFragmentSafely(fragment, "tag", R.id.fragment_container, true)
```



