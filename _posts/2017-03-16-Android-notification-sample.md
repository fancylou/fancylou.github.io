---
layout: post
title:  "Android 通知处理"
date:   2017-03-16 21:42:00 +0800
categories: android
---



为了解决Android系统版本兼容问题，于是使用Android官方提供的`NotificationCompat`兼容类来帮助实现统一的Notification。

一般简单的通知需要三个内容：图标、标题、文本内容，代码如下：

```java
 NotificationCompat.Builder builder = new NotificationCompat.Builder(context);
        builder.setContentTitle("Fancy Lou");
        builder.setContentText("Hello, this is a notification !");
        builder.setSmallIcon(R.mipmap.ic_launcher);
        NotificationManagerCompat.from(context).notify(0, builder.build());
```

这段代码执行效果如下：

![](http://img.muliba.net/notification.jpg)



<!-- more -->



这样只是简单的展现了通知，但是不可以点击，需要添加响应效果，这时需要构造一个`PendingIntent` ，如：

```java
PendingIntent pendingIntent = ***;
builder.setContentIntent(pendingIntent);
```

还可以给通知增加辨识度，还可以这样设置：

```java
builder.setColor(Color.RED);

Bitmap largePic = ***;
builder.setLargeIcon(largePic);
```



高版本的Android系统还支持通知展开模式，所以Notification出现几种风格

#### BigPictureStyle

可以展现一张大图的通知样式

示例代码：

```java
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setContentTitle("BigPictureStyle");
        builder.setContentText("BigPicture演示示例");
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setDefaults(NotificationCompat.DEFAULT_ALL);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.pic_zxgg));
        android.support.v4.app.NotificationCompat.BigPictureStyle style = new android.support.v4.app.NotificationCompat.BigPictureStyle();
        style.setBigContentTitle("BigContentTitle");
        style.setSummaryText("SummaryText");
        style.bigPicture(BitmapFactory.decodeResource(getResources(),R.drawable.pic_sample));
        builder.setStyle(style);
        builder.setAutoCancel(true);
        Intent intent = new Intent(this,MainActivity.class);
        PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
        builder.setContentIntent(pIntent);
        Notification notification = builder.build();
        manger.notify(TYPE_BigPicture,notification);
```

通知效果初始和普通的样式差不多，点击箭头展开后可以看到大图

![默认样式](http://img.muliba.net/bigPicture1.jpg)

![展开样式](http://img.muliba.net/bigPicture2.jpg)



#### BigTextStyle

可以展现大段文字的通知样式

示例代码：

```java
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setContentTitle("BigTextStyle");
        builder.setContentText("BigTextStyle演示示例");
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setDefaults(NotificationCompat.DEFAULT_ALL);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.pic_zxgg));
        android.support.v4.app.NotificationCompat.BigTextStyle style = new android.support.v4.app.NotificationCompat.BigTextStyle();
        style.setBigContentTitle("BigContentTitle");
        style.setSummaryText("SummaryText");
        style.bigText("这里就是需要展现的大段文字内容，这个内容可以换行显示，长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点长一点");
        builder.setStyle(style);
        builder.setAutoCancel(true);
        Intent intent = new Intent(this,MainActivity.class);
        PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
        builder.setContentIntent(pIntent);
        Notification notification = builder.build();
        manger.notify(TYPE_BigText,notification);
```

通知效果和上一个类似

![普通样式](http://img.muliba.net/IMG_20170316_202900.png)

![展开样式](http://img.muliba.net/IMG_20170316_202930.png)



#### InboxStyle

InBoxStyle和上面的BigTextStyle差不多，它控制多行文本显示

示例代码：

```java
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setContentTitle("InboxStyle");
        builder.setContentText("InboxStyle演示示例");
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.pic_zxgg));
        android.support.v4.app.NotificationCompat.InboxStyle style = new android.support.v4.app.NotificationCompat.InboxStyle();
        style.setBigContentTitle("BigContentTitle")
                .addLine("第一行，第一行，第一行，第一行，第一行，第一行，第一行")
                .addLine("第二行")
                .addLine("第三行")
                .addLine("第四行")
                .addLine("第五行")
                .setSummaryText("SummaryText");
        builder.setStyle(style);
        builder.setAutoCancel(true);
        Intent intent = new Intent(this,MainActivity.class);
        PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
        builder.setContentIntent(pIntent);
        builder.setDefaults(NotificationCompat.DEFAULT_ALL);
        Notification notification = builder.build();
        manger.notify(TYPE_Inbox,notification);
```

效果：

![普通效果](http://img.muliba.net/inBox1.jpg)

![展开样式](http://img.muliba.net/inBox2.jpg)



#### MediaStyle

就是音乐播放器使用的通知样式，它可以保持在通知栏，不被清除

示例代码：

```java
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setContentTitle("MediaStyle");
        builder.setContentText("Song Title");
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.pic_zxgg));
        builder.setDefaults(NotificationCompat.DEFAULT_ALL);
        Intent intent = new Intent(this,MainActivity.class);
        PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
        builder.setContentIntent(pIntent);

        //这里是设置操作 第一个参数是图标，第二个参数是名称，第三个是操作PendingIntent
        builder.addAction(android.R.drawable.ic_media_rew,"",null);
        builder.addAction(android.R.drawable.ic_media_previous,"",null);
        builder.addAction(android.R.drawable.ic_media_play,"",pIntent);
        builder.addAction(android.R.drawable.ic_media_next,"",null);
        builder.addAction(android.R.drawable.ic_media_ff,"",null);
        NotificationCompat.MediaStyle style = new NotificationCompat.MediaStyle();
        style.setMediaSession(new MediaSessionCompat(this,"MediaSession",
                new ComponentName(NotificationActivity.this,Intent.ACTION_MEDIA_BUTTON),null).getSessionToken());
        //这里是设置默认显示的图标，会显示在通知右边区域，点击后展开显示全部的action，参数是上面的action的下标 最多三个
        style.setShowActionsInCompactView(1, 2, 3);
        builder.setStyle(style);
        builder.setShowWhen(false);
        Notification notification = builder.build();
		notification.flags = FLAG_ONGOING_EVENT;
        manger.notify(TYPE_Media,notification);
```



展现效果如下：

![普通样式](http://img.muliba.net/media1.jpg)

![展开样式](http://img.muliba.net/media2.jpg)

**这里用到的Action在普通样式里面也可以使用，这样普通样式的通知就可以展开了，展开之后就能看到你添加的Action按钮**



还有一个最灵活的是自定义样式的通知，使用的是一个`RemoteView`，具体代码和样列待续。。。。。。^_^
