---
layout: post
title: "Google Architecture Components 框架使用"
categories: android
date: 2018-04-13 15:00:00
---



Android应用开发原先一直用MPV模式，公司的应用所有数据都是在线异步获取，最近发现时不时会有一些闪退，发现是数据返回后到了UI层，结果UI被销毁了。我在返回数据的时候已经判断了当前的Activity是否存在了，但是还是偶尔会出现这种崩溃的情况，也就是刚好在那个点上被销毁了？很郁闷。。。

Google去年提出来一个Android开发框架叫 [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)  ，说是能帮开发者方便管理生命周期的，让UI和数据分离，防止内存泄露等等，当然还有很多其他内容，包括数据库管理的Room库、数据处理的Paging库等等。于是写了一个简单的[Demo](https://github.com/fancylou/EyeOpener/tree/master)，试试大概的使用过程。Demo里面的数据来源还有图片资源啥的都是从一个开源的学习项目上拿来的 [**Eyepetizer-in-Kotlin**](https://github.com/LRH1993/Eyepetizer-in-Kotlin)



<!-- more -->



### Lifecycle

首先是Lifecycle，框架提供了android.arch.lifecycle包里面的一系列类和接口，让你可以方便的感知应用的各个组件的生命周期。里面有LifecycleObserver接口以一个观察者来感知组件生命周期的各个阶段，这个接口如何观察呢，有一个生命周期组件拥有者LifecycleOwner，Activity等拥有生命周期的组件实现LifecycleOwner这个接口，并且让LifecycleObserver来观察，你就拥有生命周期感知能力的(*lifecycle-aware*),不再依赖Activity的生命周期。

从Support Library 26.1.0开始 ，里面Activity、Fragment等类都已经实现了LifecycleOwner

实现自己的生命周期观察者：

```kotlin
class MyLocationListener implements LifecycleObserver {
    private boolean enabled = false;
    public MyLocationListener(Context context, Lifecycle lifecycle, Callback callback) {
       ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
        if (enabled) {
           // connect
        }
    }

    public void enable() {
        enabled = true;
        if (lifecycle.getCurrentState().isAtLeast(STARTED)) {
            // connect if not connected
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
        // disconnect if connected
    }
}
```

在Activity里面开始监听：

```kotlin
 myLocationListener = new MyLocationListener(this, getLifecycle(), location -> {
            // update UI
        });
```



### LiveData

LiveData是一种可以被观察的数据对象，它有生命周期感知能力，它会在UI活跃的状态下通知UI更新数据。

#### 创建LiveData

```kotlin
class HomeViewModel : AndroidViewModel {

    private val homeRepository: HomeRepository

    constructor(application: Application): super(application) {
        val homeItemDao = (application as EyeOpenerApplication).database.homeItemDao()
        val api = RetrofitClient.getInstance(application).apiService()
        homeRepository = HomeRepository(api, homeItemDao)
    }

    private val TAG = "HomeViewModel"

    var homeItemList: LiveData<List<RoomHomeItem>>? = null

    fun loadHomeItemList(createDate: String) {
        Log.i(TAG, "createDate: $createDate")
        if (homeItemList == null) {
            homeItemList = homeRepository.getHomeItemList(createDate)
        }
    }
}
```

#### 观察LiveData中的数据并更新UI

```kotlin
viewModel.homeItemList?.observe(this, Observer<List<RoomHomeItem>>{list ->
    //更新UI
    if (list!=null) {
        adapter.setList(list.toMutableList())
    }
})
```

> //TODO 这个LiveData我觉得是这个框架比较核心的东西，但是我开发中发现个问题，不知道如何处理，后面知道了再补充，就是上拉加载更多数据，这个LiveData只能setValue ，那是不是每次查询都要把原来的value和新查询出来的数据合并成一个新的list后再setValue给这个LiveData呢？？？？



### ViewModel

ViewModel代码上面已经贴出来了，它设计目的是处理UI相关的数据的，它为UI提供数据，并且能够在旋转屏幕等Configuration Change发生时，仍能保持里面的数据。当UI组件恢复时，可以立刻向UI提供数据

当UI主动被销毁的时候ViewModel就会清空数据。

而且ViewModel还能在Fragment之间共享数据。同一个activity下的多个Fragment，以为通过activity持有这个ViewModel

```kotlin
model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
```

所有多个Fragment都相当于用的同一个数据，不会每次Fragment消失、显示等时候都查询数据浪费资源。





### Room

这个是一个Android数据库sqlite的封装库，极大的提高了Android开发者对数据库开发的效率。

比如数据库表对象：

```kotlin
@Entity
open class RoomHomeItem(
        @PrimaryKey
        open var itemId: String = "",
        /**
         * 创建日期 视频的获取日期
         */
        @ColumnInfo(name = "eye_create_date")
        open var createDate:String? ="",
        open var type: String? = "",
        @Embedded
        open var data: RoomHomeItemData? = null
)
```

使用@Entity注解标注对象，Room会自动帮你在数据库中建表

Dao，数据库操作类，自动帮你生成实现代码：

```kotlin
@Dao
interface HomeItemDao {


    @Insert
    fun insert(item: RoomHomeItem)


    @Query("select count(*) from roomhomeitem where eye_create_date = :createDate ")
    fun countHomeItemByCreateDate(createDate: String): Long


    @Query("select * from roomhomeitem where eye_create_date = :createDate  order by date DESC, releaseTime ASC ")
    fun findHomeItemListByCreateDate(createDate: String): LiveData<List<RoomHomeItem>>

}
```



Database，自动创建数据库、表等等：

```kotlin
@Database(entities = [RoomHomeItem::class, RoomHomeItemData::class, RoomHomeItemCoverBean::class,
    RoomHomeItemAuthor::class, RoomHomeItemConsumption::class], version = 1)
abstract class MyDatabase : RoomDatabase() {

    abstract fun homeItemDao(): HomeItemDao
}
```



写完这些必要的代码，你只要在应用中build一下，它就把创建数据库、建表等等事情都做完了

```kotlin
database = Room.databaseBuilder(applicationContext, MyDatabase::class.java, "eyeopener").build()
```





### Paging

这个库还没用，后续补充。。。。。。。。



