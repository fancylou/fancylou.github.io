### Android开发调试工具Stetho（FaceBook出品）

![logo](http://facebook.github.io/stetho/static/logo.png)

[Stetho](http://facebook.github.io/stetho/)是一个FaceBook开发的一个帮助开发者在debug模式下进行Android应用调试的工具，方便开发者对调试应用的SharedPreference存储数据、Sqlite数据的查看和修改，还可以对network访问情况进行监控，甚至可以看到当前页面的布局tree等，**以上这一切都可以在你的Chrome浏览器上进行**，对于Android开发者而言这是一个相当给力的调试利器。

**集成：**

首先引入核心包

```json
dependencies { 
    compile 'com.facebook.stetho:stetho:1.4.2' 
  } 
```

然后在项目的Application类的onCreate方法内加入代码：

```java
public class MyApplication extends Application {
  public void onCreate() {
    super.onCreate();
    Stetho.initializeWithDefaults(this);
  }
}
```

启动调试应用后，打开你的chrome浏览器，在地址栏输入： **chrome://inspect**

![](http://facebook.github.io/stetho/static/images/inspector-discovery.png)

然后点击对应项目下面的那个 **Inspect** 超链接，就会打开一个开发者工具窗口

存储数据操作：

![Databse inspection](http://facebook.github.io/stetho/static/images/inspector-sqlite.png)

界面View的树状结构：

![View Hierarchy](http://facebook.github.io/stetho/static/images/inspector-elements.png)



然后要监控网络访问情况，还需要集成对应的包，如果项目用的okhttp，集成就比较简单

OkHttp3 用如下的包

```json
dependencies { 
    compile 'com.facebook.stetho:stetho-okhttp3:1.4.2' 
  } 
```

集成代码：（这里注意是用的 **addNetworkInterceptor** 而不是 **addInterceptor**）

```java
new OkHttpClient.Builder()
    .addNetworkInterceptor(new StethoInterceptor())
    .build();
```

Okhttp2 用如下的包

```json
dependencies { 
    compile 'com.facebook.stetho:stetho-okhttp:1.4.2' 
  } 
```

集成代码：

```java
OkHttpClient client = new OkHttpClient();
client.networkInterceptors().add(new StethoInterceptor());
```



查看连接情况还是刚才那个开发者窗口，在 **Network**  标签页:

![Network inspection](http://facebook.github.io/stetho/static/images/inspector-network.png)



Stetho还为使用HttpURLConnection进行交互的项目提供了网络访问情况监控的集成方式，需要添加对应的包

```json
dependencies { 
    compile 'com.facebook.stetho:stetho-urlconnection:1.4.2' 
  } 
```

可以使用包内提供的 `StethoURLConnectionManager` 进行集成，具体集成方式可以查看FaceBook提供的[Sample](https://github.com/facebook/stetho/tree/master/stetho-sample)应用



官方还提供了关于Dump输出自定义的说明，这个我还没用到过，有需要的可以查看官方的说明。

