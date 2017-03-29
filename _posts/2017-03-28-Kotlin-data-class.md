---
layout: post
title: "Kotlin使用记录1"
categories: kotlin
date: 2017-03-28 20:13:00

---

#### data class

Kotlin的data class 相当于是java里面经常说的实体对象，也就是我们用来存放数据的对象。这种类一般都是除了一堆属性和getter、setter方法外，没有别的东西，但是又不得不写。在Kotlin里面有专门的关键字标识这种类 `data` ，它是这么写的：

```kotlin
data class User(val name: String, val age: Int)
```

在kotlin里面它就是这么简单的一行代码就行了，只要把属性放在构造器里面，不需要额外写getter、setter方法，也不需要重写`toString()`、`equals()/hashCode()`，这些它都有默认生成好的，除非你有特殊需求。比如 `toString()`  上面的User类：

```kotlin
val user = User("fancylou", 30)
println(user.toString())
```

打印结果：`User(name=fancylou, age=30)`

它还默认定义了一个`copy()`函数，可以方便同一对象的复制修改

```kotlin
val user = User("fancylou", 30)
val newUser = user.copy(age = 23)
println(newUser.toString())
```

打印结果：`User(name=fancylou, age=23)`

这个`copy`函数在这个User类中的实现是这样的：

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)     


```

意思很简单就是传入参数name和age，然后用默认的构造函数new一个User对象，其中传入参数的等号后面的this.name和this.age是默认值，在Kotlin中函数的传入参数可以有默认值。函数后面的等号是一种简写方式，如果函数体只有一句表达式，就可以省略大括号直接用等于号后面跟上这个句表达式就行了，也就是相当于:

```kotlin
fun copy(name:String = this.name, age: Int = this.age) : User {
        return User(name, age)
    } 
```

上面简单说明了Kotlin语言中的`data` 这个关键字，但过程中已经出现了很多Kotlin语言的特性，它简洁明了、可读性好、函数式编程，相比java可以减少你很多很多的代码量，当然还有重复劳动。