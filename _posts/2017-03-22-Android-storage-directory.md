---
layout: post
title: "Android本地存储目录记录"
categories: android kotlin

---

Android手机存储分为内部存储和外部存储。

内部存储有3个目录，分别是Data目录、DownloadCache目录、Root目录

```kotlin
        var dataPath = Environment.getDataDirectory().absolutePath
        Logger.d("内部存储 data 目录：" + dataPath)
        var cachePath = Environment.getDownloadCacheDirectory().absolutePath
        Logger.d("内部存储 cache 目录：" + cachePath)
        var rootPath = Environment.getRootDirectory().absolutePath
        Logger.d("内部存储 root 目录：" + rootPath)
```

日志：



`内部存储 data 目录：/data`

`内部存储 cache 目录：/cache`

`内部存储 root 目录：/system`





外部存储就是原来一直说的SD卡，现在一般手机都有内置的一块区域作为这个外部存储

```kotlin
        var externalPath = Environment.getExternalStorageDirectory().absolutePath
        Logger.d("外部存储目录：" + externalPath)
```

日志：`外部存储目录：/storage/emulated/0`

这个外部存储目录下分别有10个公共目录，分别是Alarms、DCIM、Download、Movies、Music、Notifications、Pictures、Podcasts、Ringtones、Documents

```kotlin
        var alarmPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_ALARMS)
        Logger.d("公共目录Alarms:"+alarmPath)
        var dcimPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM)
        Logger.d("公共目录DCIM:"+dcimPath)
        var documentPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOCUMENTS)
        Logger.d("公共目录document:"+documentPath)
        var downloadPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS)
        Logger.d("公共目录Downloads:"+downloadPath)
        var moviesPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MOVIES)
        Logger.d("公共目录Movies:"+moviesPath)
        var musicPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MUSIC)
        Logger.d("公共目录Music:"+musicPath)
        var notificationPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_NOTIFICATIONS)
        Logger.d("公共目录Notification:"+notificationPath)
        var picturePath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES)
        Logger.d("公共目录Pictures:"+picturePath)
        var podcastPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PODCASTS)
        Logger.d("公共目录Podcasts:"+podcastPath)
        var ringtonesPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_RINGTONES)
        Logger.d("公共目录Ringtones:"+ringtonesPath)
```



日志：

`公共目录Alarms:/storage/emulated/0/Alarms`

`公共目录DCIM:/storage/emulated/0/DCIM`

`公共目录document:/storage/emulated/0/Documents`

 `公共目录Downloads:/storage/emulated/0/Download`

`公共目录Movies:/storage/emulated/0/Movies`

`公共目录Music:/storage/emulated/0/Music`

`公共目录Notification:/storage/emulated/0/Notifications`

`公共目录Pictures:/storage/emulated/0/Pictures`

`公共目录Podcasts:/storage/emulated/0/Podcasts`

`公共目录Ringtones:/storage/emulated/0/Ringtones`



不过官方不建议我们把APP的文件存放到这些公共目录内，而是把文件放到APP的私有目录下。

```kotlin
var privateTestPath = applicationContext.getExternalFilesDir("test").absolutePath
        Logger.d("私有目录 test："+privateTestPath)
```

打印日志：

`私有目录 test：/storage/emulated/0/Android/data/包名/files/test`

如果传入参数是null就是获取私有目录的根目录`/storage/emulated/0/Android/data/包名/files`

