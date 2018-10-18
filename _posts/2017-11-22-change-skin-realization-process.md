---
layout: post
title: "换肤控件的开发过程"
categories: kotlin 换肤
date: 2017-11-22 14:46:00
---



目前市面上有很多应用都是有换肤功能，也有叫切换主题，就是把主色、背景色、图标、字体颜色等资源替换掉。虽然我们的项目是比较严肃的办公系统，但我觉得应该加点活泼的东西，在严肃的工作之余也有一丝乐趣。于是去网上搜索了下大家是如何实现的，就发现我平常关注的洪洋大神很早就写过这么一个换肤框架[ChangeSkin](https://github.com/hongyangAndroid/ChangeSkin)。于是在他的思路下用kotlin改写了这个框架。

整体思路是这样的，通过为LayoutInfalter去设置自定义Factory，对加载的View进行分析和提取。

![](http://img.muliba.net/blog/20171122/Activity.onCreate%28savedInstanceState-+Bundle-%29.png)

在自定义Factory里面创建各个View，并且分析这些View的属性，是否有需要进行替换资源的属性（就是换肤），把它们记录下来，然后进行换肤操作。下面是自定义Factory的部分实现：

```kotlin
override fun onCreateView(parent: View?, name: String, context: Context?, attrs: AttributeSet?): View? {
        var view: View? = null
        val skinAttrList = filterSkinAttr(attrs, context)
        if (skinAttrList.isEmpty()) {
            return null
        }
        if (context == null || attrs == null) {
            return null
        }
        view = mAppCompatActivity.delegate.createView(parent, name, context, attrs)
        if (view == null) {
            view = createViewFromTag(context, name, attrs)
        }
        if (view != null) {
            val skinView = SkinView(view!!, skinAttrList)
            skinViewsMap.put(view!!, skinView)
            skinView.apply()
        }
        if (view == null ) {
            Log.i("oncreateView", "create view is null")
        }
        return view
    }
```

把需要替换资源的view放入SkinView，然后进行换肤操作，SkinView就是做一件事换肤，把当前这个view里面需要换肤的属性一个个替换下，比如background、textColor等。

```kotlin
data class SkinView(val view:View, val skinAttrList: List<BaseSkinAttr>) {
    fun apply() {
        skinAttrList.map { skinAttr ->
            skinAttr.apply(view)
        }
    }
}
```

因为需要替换的资源千奇百怪，也有自定义的，不好弄，就写了个BaseSkinAttr

```kotlin
open abstract class BaseSkinAttr(var attrName:String, var originResId: Int, var resName: String) {

    abstract fun apply(view: View)
    abstract fun copy(attrName:String, originResId: Int, resName: String): BaseSkinAttr

    override fun toString(): String {
        return "SkinAttr{attrName:$attrName, originResId:$originResId, resName:$resName }"
    }
}
```

在apply方法中进行各个属性自己实现替换资源的方式，因为平常最多的是background、src、textColor这几个属性。

```kotlin
class BackgroundSkinAttr(attrName:String = "", originResId: Int = 0, resName: String = "" ): BaseSkinAttr(attrName, originResId, resName) {

    override fun apply(view: View) {
        val bottom = view.paddingBottom
        val top = view.paddingTop
        val right = view.paddingRight
        val left = view.paddingLeft
        /**
         * 资源是color的时候 通过getDrawable获取到drawable 去setBackground 会有颜色获取不正确的情况  ColorDrawable 里面的mBaseColor 和 mUseColor两个值不一致
         * 所以如果是color资源，通过getColor获取color值 setBackgroundColor
         */
        val isColorId = FancySkinManager.instance().getResourceManager()?.getColorResIdByName(resName) ?: 0
        if (isColorId!=0) {
            val color = FancySkinManager.instance().getResourceManager()?.getColorByResId(isColorId) ?: -1
            if (color != -1){
                view.setBackgroundColor(color)
            }
        }else {
            val drawable = FancySkinManager.instance().getResourceManager()?.getDrawable(originResId, resName)
            if (drawable!=null) {
                view.background = drawable
            }
        }
        view.setPadding(left, top, right, bottom)

    }

    override fun copy(attrName: String, originResId: Int, resName: String): BaseSkinAttr {
        return BackgroundSkinAttr(attrName, originResId, resName)
    }
}
```

这是background属性换肤的实现，里面特别提到了，如果是背景是用颜色的，要专门提取颜色值然后进行`view.setBackgroundColor(color)`,  用Drawable对象进行`view.background = drawable` 设置经常会无效。还没搞明白是为啥。

这样从创建View到设置View需要替换皮肤资源的属性进行各自设置的整个过程就完成了。然后换肤的还有一个关键点，这些换肤的资源怎么获取。就是上面属性apply方法中的`FancySkinManager.instance().getResourceManager()?.getDrawable（）`这些代码。它获取的怎么就是我要的皮肤资源。

![](http://img.muliba.net/blog/20171122/%E7%9A%AE%E8%82%A4%E8%B5%84%E6%BA%90.png)

内部资源获取的思路就是加后缀。ResourceManager里面获取资源的实现：

```kotlin
private fun appendSuffix(name: String): String {
        var mName = name
        if (!TextUtils.isEmpty(mSkinSuffix)) {
            mName = "${mName}_$mSkinSuffix"
        }
        return mName
    }
fun getColor(originResId: Int, name: String): Int {
        try {
            val originColor = ContextCompat.getColor(mContext, originResId)
            val trueName = appendSuffix(name)
            val id = mResources.getIdentifier(trueName, DEFTYPE_COLOR, mSkinPackageName)
            if (id != 0) {
                return mResources.getColor(id)
            }
            return originColor
        } catch (e: Resources.NotFoundException) {
            return -1
        }
    }
```

这是获取颜色的实现，获取Drawable的也是类似的。先给资源名称添加后缀，然后根据新的名称找对于的资源的id，然后根据id获取资源。

FancySkinManager会把当前皮肤的后缀保存起来，这样每次打开这个页面的时候看到的就是换肤后的资源样式了。

```kotlin
/**
     * 应用内部换肤
     * @param suffix 资源后缀
     */
    fun changeSkinInner(suffix: String) {
        //清理当前的插件皮肤
        cleanPluginSkin()
        mCurrentSkinSuffix = suffix
        PreferenceUtil.putPluginSuffix(mContext, suffix)
        mResourceManager = ResourceManager(mContext, mContext.resources!!, mContext.packageName!!, mCurrentSkinSuffix)
        notifySkinChanged()
    }

```

上面是应用内换肤的整个过程。还有一块是使用应用外部的皮肤资源进行换肤。其它过程都是一样的主要是怎么样把资源加载进来，并且根据这些加载的资源创建ResourcesManager。

其实外部资源就是一个apk包，没有代码只有资源，这时候需要加载这个apk包：

```kotlin
 /**
     * 加载皮肤包
     * @param prefSkinPath 皮肤包存放路径
     * @param prefSkinPackageName 皮肤包的packageName
     * @param suffix 皮肤包内的资源是否需要后后缀
     */
    private fun loadSkinPlugin(prefSkinPath: String, prefSkinPackageName: String, suffix: String) {
        val assetManager = AssetManager::class.java.newInstance()
        val addAssetPath = assetManager.javaClass.getMethod("addAssetPath", String::class.java)
        addAssetPath.invoke(assetManager, prefSkinPath)

        val superRes = mContext.resources
        mResources = Resources(assetManager, superRes?.displayMetrics, superRes?.configuration)
        mResourceManager = ResourceManager(mContext, mResources!!, prefSkinPackageName, suffix)
        usePlugin = true
    }
```

就是利用AssetManager 加载apk的资源，然后创建一个新的Resources，并用这个新的Resources创建一个新的ResourcesManager，用于我们后面换肤的地方获取皮肤资源。这样获取到的资源就是加载进来的新的皮肤资源了。

切换外部皮肤：

```kotlin
/**
     * 使用外部皮肤资源插件包换肤
     * @param skinPath 插件地址，apk地址
     * @param skinPackageName 插件包名
     * @param suffix 资源后缀，默认为空
     */
    fun changeSkin(skinPath: String, skinPackageName: String, suffix: String = "", callback: PluginSkinChangingListener = DefaultPluginSkinChangingListener()) {
        callback.onStart()
        checkPluginParamsThrow(skinPath, skinPackageName)
        if (skinPath == mCurrentSkinPath && skinPackageName == mCurrentSkinPackageName) {
            return
        }
        /**
         * 后台线程加载 apk皮肤资源插件包
         */
        doAsync {
            try {
                loadSkinPlugin(skinPath, skinPackageName, suffix)
            }catch (e: Exception) {
                callback.onError(e)
            }
            uiThread {
                try {
                    updatePluginInfo(skinPath, skinPackageName, suffix)
                    notifySkinChanged()
                    callback.onCompleted()
                }catch (e: Exception) {
                    callback.onError(e)
                }
            }
        }
    }
```



具体代码都放在了Github上了FancyChangeSkin](https://github.com/fancylou/FancyChangeSkin)

全部的换肤过程就完了，使用的话，就是把需要换肤的Activity都继承我们的FancySkinActivity， 在我们自己的Application中onCreate方法中初始化我们的FancySkinManager实例`FancySkinManager.instance().init(this)`



最后更新下，因为需要每个Activity都要继承我们的FancySkinActivity比较麻烦，后来看到有人利用了Application.ActivityLifecycleCallbacks接口来实现，这样就不要每个Activity都继承我们的FancySkinActivity。

```kotlin
class FancySkinActivityLifeCycle : Application.ActivityLifecycleCallbacks {


    private val inflaterFactoryMap: WeakHashMap<Context, FancySkinLayoutInflaterFactory> = WeakHashMap()

    private constructor(application: Application) {
        application.registerActivityLifecycleCallbacks(this)
        installLayoutFactory(application)
    }
    companion object {
        private var INSTANCE: FancySkinActivityLifeCycle? = null
        fun instance(application: Application): FancySkinActivityLifeCycle {
            if (INSTANCE==null) {
                INSTANCE = FancySkinActivityLifeCycle(application)
            }
            return INSTANCE!!
        }
    }

    override fun onActivityCreated(activity: Activity?, savedInstanceState: Bundle?) {
        if (activity !=null ) {
            installLayoutFactory(activity)
            changeStatusColor(activity)
            FancySkinManager.instance().addSkinChangedListener(activity, object : SkinChangedListener{
                override fun onSkinChanged() {
                 getSkinLayoutInflaterFactory(activity).applySkin()
                    changeStatusColor(activity)
                }
            })
        }
    }
    ...
    
    override fun onActivitySaveInstanceState(activity: Activity?, bundle: Bundle?) {

    }
    override fun onActivityDestroyed(activity: Activity?) {
        if (activity!=null) {
            getSkinLayoutInflaterFactory(activity).clean()
            FancySkinManager.instance().removeSkinChangedListener(activity)
        }
    }

    private fun installLayoutFactory(context: Context) {
        val layoutInflater = LayoutInflater.from(context)
        try {
            val field = LayoutInflater::class.java.getDeclaredField("mFactorySet")
            field.isAccessible = true
            field.setBoolean(layoutInflater, false)
            LayoutInflaterCompat.setFactory(layoutInflater, getSkinLayoutInflaterFactory(context))
        } catch (e: NoSuchFieldException) {
            e.printStackTrace()
        } catch (e: IllegalArgumentException) {
            e.printStackTrace()
        } catch (e: IllegalAccessException) {
            e.printStackTrace()
        }

    }
  ....
}
```

FancySkinManager:

```kotlin
fun withoutActivity(application: Application): FancySkinManager {
    init(application)
    FancySkinActivityLifeCycle.instance(application)
    return instance()
}
```



然后我们自己应用的Application的onCreate方法中就要这样初始化了

```kotlin
FancySkinManager.instance().withoutActivity(this)
```

