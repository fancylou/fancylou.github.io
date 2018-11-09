---
layout: post
title:  "Android透明状态栏"
categories: android kotlin

---

原来一直用Android主题中`colorPrimary`和`colorPrimaryDark`用同一种颜色，模拟了透明状态栏的样式。后来发现原来低版本的Android系统状态栏底色不是用`colorPrimaryDark`。

因为我们的APP的使用版本是4.4及以上，所以查了下4.4版本的Android系统中如何使用透明状态栏。



<!-- more -->



在Android 4.4版本系统中，在Activity的setContentView(layoutResId)方法之前加入如下代码：

```kotlin
        if (Build.VERSION_CODES.LOLLIPOP > Build.VERSION.SDK_INT && Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS)
        }
```

![](http://img.muliba.net/statusBar1.jpg)

发现这个ActionBar和整体内容都往上了，到了状态栏下面。然后就加了`fitsSystemWindows`这个属性。

![](http://img.muliba.net/statusBar2.jpg)

这个状态栏就变成白色的了。应该是透明了，但是因为下面没有内容所以就看到了这个白色状态栏，但是他没有办法设置状态栏的背景色。于是我还是去掉了`fitsSystemWindows`属性然后在内容区域添加了状态栏的高度。

计算状态栏高度：

```kotlin
    /**
     * 获取状态栏高度
     */
    fun getStatusBarHeight(context: Context): Int {
        var result = 0
        val resourceId = context.resources.getIdentifier(
                "status_bar_height", "dimen", "android")
        if (resourceId > 0) {
            result = context.resources.getDimensionPixelSize(resourceId)
        }
        return result
    }
```

给内容区域设置top padding 

```kotlin
    /**
     * 给内容区域设置top padding 高度是状态栏的高度
     * 透明状态栏之后 内容会往上
     */
    fun setContentViewTopPaddingWithStatusBarHeight(context: Context, contentView:View?) {
        var result = getStatusBarHeight(context)
        if (contentView!=null) {
            contentView?.setPadding(0, result, 0, 0)
        }
    }
```



然后就成功了，如下图：

![](http://img.muliba.net/statusBar3.jpg)

后来又发现了大于5.0版本的情况下有问题，前面说到了高版本的状态栏我用修改`colorPrimaryDark`的方式模拟成了透明状态栏的样子，但是我的首页使用了`DrawerLayout`，发现拉出左边菜单的时候就不对了，因为状态栏的背景色是`colorPrimaryDark`的颜色。

![状态栏有背景色](http://img.muliba.net/statusBar4.jpg)

原来大于5.0版本的可以正确的设置他的状态栏为透明背景

```kotlin
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS)
            window.decorView.systemUiVisibility = (View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN or View.SYSTEM_UI_FLAG_LAYOUT_STABLE)
            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS)
            window.statusBarColor = Color.TRANSPARENT
        }
```



这样就是真正的透明背景的状态栏

![透明状态栏](http://img.muliba.net/statusBar5.jpg)



最后的代码是在`setContentView(layoutResId)`之前写入：

```kotlin
        if (Build.VERSION_CODES.LOLLIPOP > Build.VERSION.SDK_INT && Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS)
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS)
            window.decorView.systemUiVisibility = (View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN or View.SYSTEM_UI_FLAG_LAYOUT_STABLE)
            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS)
            window.statusBarColor = Color.TRANSPARENT
        }
```



