---
layout: post
title: "使用CursorLoader查询图片和相册"
categories: android
date: 2017-10-19 16:16:00
---



原来项目中使用到需要选择Android本地文件，就写了这个文件选择器[FancyFilePicker](https://github.com/fancylou/FancyFilePicker) ，其中还有一个仿微信的图片选择器。原来这个图片选择器查询图片使用的是`ContentResolver`， 然后自己根据图片类型什么的条件组合查询。最近看到一个官方提供的异步查询的工具`CursorLoader`，于是本着学习的心态，重写了这个图片选择器，就使用的这个异步工具`CursorLoader`。

首先，Activity提供了一个`LoaderManager`的管理工具来管理这个`CursorLoader`，如果用的是support-v4包，里面也同样有这个`getSupportLoaderManager`，

```kotlin
supportLoaderManager.initLoader(PHOTO_LOADER_ID, args, callback)
```

通过这个`initLoader`初始化这个`CursorLoader`并在后台查询数据返回给前台，具体是通过这个`callback`进行操作的。这个callback是这个接口的实现`LoaderManager.LoaderCallbacks<Cursor>`

这个接口有下面3个函数：

```kotlin
override fun onCreateLoader(id: Int, args: Bundle?): Loader<Cursor>? {
   when (id) {
            PHOTO_LOADER_ID -> {
                var albumId = args?.getString(ALBUM_ID_KEY) ?: ""
                return createCursorLoader(albumId)
            }
            else -> {
                Log.e(TAG, "do not recognize onCreateLoader id , id:$id")
                return null
            }
        }
}
override fun onLoaderReset(loader: Loader<Cursor>?) {
  mAdapter.changeCursor(null)
}
override fun onLoadFinished(loader: Loader<Cursor>?, cursor: Cursor?) {
  mAdapter.changeCursor(cursor)
}

```

`onCreateLoader`就是提供一个查询的`CursorLoader`给后台查询使用，`onLoadFinished`返回查询结果的`Cursor`，`onLoaderReset`如果数据或者内容有变化就会执行。

比如查询全部图片，我们在`onCreateLoader`返回一个如下的Loader 

```kotlin
 val PROJECTION = arrayOf(MediaStore.Images.Media._ID,
            MediaStore.Images.Media.DISPLAY_NAME,
            MediaStore.Images.Media.DATA,
            MediaStore.Images.Media.DATE_TAKEN)
 val ORDER_BY = " ${MediaStore.Images.Media.DATE_TAKEN } DESC"
 val SELECTION_SIZE =  "${MediaStore.Images.Media.SIZE} > ? or ${MediaStore.Images.Media.SIZE} is null"

CursorLoader(context,
                    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                    PROJECTION,
                    SELECTION_SIZE,
                    arrayOf("0"),
                    ORDER_BY)
```



然后`CursorLoader`后台查询数据完成后调用这个`onLoadFinished`返回查询结果，这个结果要提供给前台展现，因为前台使用的是`RecyclerView`，返回的cursor提供给`RecyclerView`的`Adapter`，根据官方的`android.support.v4.widget.CursorAdapter` ，写了这个[PictureCursorRecyclerViewAdapter](https://github.com/fancylou/FancyFilePicker/blob/master/fancyfilepickerlibrary/src/main/java/net/muliba/fancyfilepickerlibrary/adapter/PictureCursorRecyclerViewAdapter.kt) ，主要是给Adapter提供数据以及数据绑定进行一些封装。`changeCursor`每次数据查询后将`Adapter`中的`cursor`替换并刷新`RecyclerView`。



```kotlin
fun changeCursor(cursor: Cursor?) {
        var old:Cursor? = swapCursor(cursor)
        if (old!=null) {
            old.close()
        }
    }
    
    private fun swapCursor(newCursor: Cursor?): Cursor? {
        if (newCursor == mCursor) {
            return null
        }
        var old = mCursor
        if (old!=null) {
            old.unregisterContentObserver(mChangeObserver)
            old.unregisterDataSetObserver(mDataSetObserver)
        }
        mCursor = newCursor
        if (newCursor!=null) {
            newCursor.registerContentObserver(mChangeObserver)
            newCursor.registerDataSetObserver(mDataSetObserver)
            mRowIDColumn = newCursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID)
            mDataValid = true
            notifyDataSetChanged()
        }else {
            mRowIDColumn = -1
            mDataValid = false
            notifyDataSetChanged()
        }
        return old
    }
```



还有个相册的查询，方式是一样的，就是查询条件不一样，然后因为要有一个全部图片的功能，所以会多一个叫“**最近**”的相册，这个我就继承了`CursorLoader`，定义了一个叫`AlbumCursorLoader`的Loader。

```kotlin
/**
     * 重写CursorLoader 里面的loadInBackground函数
     * 主要是添加一个“最近”的相册出来，按照使用时间排序
     */
    open class AlbumCursorLoader : CursorLoader {

        constructor(context: Context?): super(context,
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                arrayOf(MediaStore.Images.Media.BUCKET_ID,
                        MediaStore.Images.Media.BUCKET_DISPLAY_NAME,
                        MediaStore.Images.Media.DATA,
                        "count(bucket_id) as cou",
                        MediaStore.Images.Media._ID),
                " _size > ? or _size is null ) GROUP BY  1,(2",
                arrayOf("0"),
                "MAX(datetaken) DESC")

        override fun loadInBackground(): Cursor {
            val albums =  super.loadInBackground()
            val recentAlbum = MatrixCursor(arrayOf(MediaStore.Images.Media.BUCKET_ID,
                    MediaStore.Images.Media.BUCKET_DISPLAY_NAME,
                    MediaStore.Images.Media.DATA,
                    "count(bucket_id) as cou",
                    MediaStore.Images.Media._ID))
            var count = 0L
            if (albums.count>0) {
                while (albums.moveToNext()) {
                    count += albums.getLong(3)
                }
            }

            recentAlbum.addRow(arrayOf(Utils.RECENT_ALBUM_ID,
                    context.getString(R.string.recent_photo),
                    "",//TODO 没有图片
                    "$count",
                    Utils.EMPTY_MEDIA_ID))
            return MergeCursor(arrayOf(recentAlbum, albums))
        }
    }
```

就是重写了这个`loadInBackground`， 把查询的结果，就是所有的相册，查询到的照片数量加起来，作为这个“**最近**”相册的照片数量。



PS：`supportLoaderManager.initLoader(PHOTO_LOADER_ID, args, callback)`是第一次查询使用，如果是多次查询，那查询使用的是同一个Loader，所以后面使用的是`supportLoaderManager.restartLoader(PHOTO_LOADER_ID, args, callback)`



上面已经把使用CursorLoader的主要代码片段都提了一遍，具体可以参考Github上的全部代码。

[https://github.com/fancylou/FancyFilePicker/blob/master/fancyfilepickerlibrary/src/main/java/net/muliba/fancyfilepickerlibrary/ui/PictureLoaderActivity.kt](https://github.com/fancylou/FancyFilePicker/blob/master/fancyfilepickerlibrary/src/main/java/net/muliba/fancyfilepickerlibrary/ui/PictureLoaderActivity.kt)





