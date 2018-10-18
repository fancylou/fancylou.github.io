---
layout: post
title: "Kotlin使用记录4-Realm使用问题"
categories: kotlin
date: 2017-04-11 22:55:00

---

今天在项目中添加了一个小功能，论坛版块收藏，还是老样子使用kotlin来写，因为原来项目中本地存储使用的是[Realm](http://realm.io), 所以就刚好第一次使用kotlin来使用Realm库。

RealmObject样例：

```kotlin
open class Person(
        @PrimaryKey open var id: Int = 0,
        open var name: String = "",
        open var sex: Int = 0,
        open var age: Int = 0
) : RealmObject()
```

这里必须使用`open`这个关键字，不然Realm会提示错误，`Error:(31, 66) error: cannot inherit from final Person`

然后在Activity中测试新增一条数据：

```kotlin
realm.beginTransaction()
realm.copyToRealmOrUpdate(Person(id, name, sex, age))
realm.commitTransaction()
```

结果报错了：

![Realm和kotlin的错误](http://img.muliba.net/blog/post/error1.png)

这个错误一看之下以为是Realm没有把新的表加入到Schema中，因为原来项目中的Realm表增删改查都是好的。

然后我想起来没有添加`migration`，schema没有这个新增加的表，于是去config里面添加`migration`。

```kotlin
val config: RealmConfiguration = RealmConfiguration.Builder()
                .schemaVersion(1)
                .migration { realm, oldVersion, newVersion ->
                    var version = oldVersion
                    val schema = realm.schema
                    if (version == 0L) {
                        (schema as RealmSchema).create(Person::class.java.simpleName)
                                .addField("id", Int::class.java, FieldAttribute.PRIMARY_KEY)
                                .addField("name", String::class.java)
                                .addField("sex", Int::class.java)
                                .addField("age", Int::class.java)

                        version++
                    }
                }.build()
```

运行后还是原来的错误，看来只能找Google帮忙，有人提到kotlin和Realm一起使用的时候，gradle的plugin顺序有关系的

```kotlin
apply plugin: 'kotlin-android'
apply plugin: 'realm-android'
```

这个kotlin的plugin必须在realm的之前，我一看我的build文件还真是realm写前面了，于是调整位置试了试，果然问题就解决了。

![kotlin with Realm](http://img.muliba.net/blog/post/kotlinWithRealm.png.jpg)





