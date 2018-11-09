---
layout: post
title: "Kotlin使用记录5-文件选择控件"
categories: kotlin
date: 2017-04-28 09:24:00
---



最近应用开发中使用到了文件选择器，需要选择文件进行上传。原本使用的是自带的选择器，这个Android自带的选择器各版本体验不一样，而且样子难看，最近又在学习kotlin，于是就想到用kotlin写一个文件选择器，并把它打包成aar，传到了Jcenter，这样方便以后使用。

项目已经在Github上开源了，https://github.com/fancylou/FancyFilePicker 

开始我按照传统的思路，根据系统实际的目录结构进行展现，类似windows上的文件选择。

![](http://img.muliba.net/blog/post/filePicker1.2.0-1.jpeg?imageMogr2/auto-orient/thumbnail/720x/blur/1x0/quality/75)



<!-- more -->



然后还设定了一些可定义项，标题的文字、标题的背景色等：

![](http://img.muliba.net/blog/post/filePicker1.2.0-2.jpeg?imageMogr2/auto-orient/thumbnail/720x/blur/1x0/quality/75)

当然还有单选和多选的选择类型的分别，单选：

![](http://img.muliba.net/blog/post/filePicker1.2.0-3.jpeg?imageMogr2/auto-orient/thumbnail/720x/blur/1x0/quality/75)

使用方法也简单，用现在流行的链式编程：

```kotlin
FilePicker()
        .withActivity(this)
        .requestCode(0)
        .start()
```



接收结果：

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (resultCode == Activity.RESULT_OK) {
        if (requestCode == 0) {
            val array = data?.getStringArrayListExtra(FilePicker.FANCY_FILE_PICKER_ARRAY_LIST_RESULT_KEY)
            ...
            return
        }
    }
    super.onActivityResult(requestCode, resultCode, data)
}
```



单选方式：

```kotlin
FilePicker()
	.withActivity(this)
    .requestCode(0)
    .chooseType(FilePicker.CHOOSE_TYPE_SINGLE)
    .start()
```



单选接收结果：

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (resultCode == Activity.RESULT_OK) {
        if (requestCode == 0) {
           val result = data?.getStringExtra(FilePicker.FANCY_FILE_PICKER_SINGLE_RESULT_KEY)
            ...
            return
        }
    }
    super.onActivityResult(requestCode, resultCode, data)
}
```



然后在实际使用过程中发现，这个模式还是有些不方便，有些目录深的你得找好久，一层层翻下去很累。于是我新增了一种模式，根据大分类进行选择，通过Android的`ContentResolver`将系统的文件根据 **图片**、**音频**、**视频**、**文档**、**压缩包**、**应用** 这六类分别按时间倒序查询展现，这样方便有目的的查询到想要的文件。

![](http://img.muliba.net/blog/post/FilePicker_2.0.0.jpg)

分类选择模式：

```kotlin
FilePicker()
                .withActivity(this)
                .requestCode(0)
                .mode(FilePicker.CHOOSE_MODE_CLASSIFICATION)
                .start()
```



还有许多细节需要慢慢磨合改进的，我会持续更新。要是您看到了，有啥意见或者建议，请告诉我