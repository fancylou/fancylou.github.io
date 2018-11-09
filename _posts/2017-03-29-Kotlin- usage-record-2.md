---
layout: post
title: "Kotlin使用记录2"
categories: kotlin
date: 2017-03-29 19:59:00
---



#### sealed class

Kotlin和java都有枚举类，而且两者蛮类似的，但有时候面对一些特殊的业务的时候枚举类显的比较简单，Kotlin有一种类叫作sealed 类，有点类似枚举类，但是枚举类可扩展性要强，使用在一些特殊业务上面相当好用，比如Android上面经常能看到的有层级的列表。



<!-- more -->



![层级列表](http://img.muliba.net/kotlin_sealed_class_1.jpg)

图上就是有多个大厦，每个大厦里面有多个会议室，这样的一个ListView，在java里面一般就是两个对象继承一个父对象。Kotlin中使用sealed类是这样的：

```kotlin
sealed class MeetingRoom {
    class Building(val name: String) : MeetingRoom()
    class Room(val id: String, val name: String, val roomNumber: String, val device: String, val floor: Int, val capacity: Int,
               val available: Boolean, val idle: Boolean): MeetingRoom()
}
```

然后在Adapter中就可以用`when`判断：

```kotlin
when(items[position]){
        is MeetingRoom.Building -> {...}
        is MeetingRoom.Room -> {...}
    }
```

官方文档还介绍说这个sealed类可以被data类继承：

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()

fun eval(expr: Expr): Double = when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
}
```



#### when

上面提到这个`when`关键字类似于java的`switch`不过要强大的多。它可以判断上面说的sealed类：

```kotlin
when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
}
```

还可以判断Boolean：

```kotlin
when (isGrid) {
  true -> {...}
  false -> {...}
}
```

甚至还有区间：

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

还有直接不写参数，然后用一个返回Boolean值的函数来做判断：

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```



#### 再说map

用过Rxjava的应该对map不会陌生，Kotlin里面也有这么一个`map`表达式，相当给力：

```kotlin
mView.setFolderList(response.list.map(CooperationFolderJson::copyToVO))
```

这么简单的一句代码搞定了一个相当复杂的逻辑操作，拆开来就是（java代码）:

```java
List<CooperationFolderJson> list = response.getList();
List<FolderVO> voList = new ArrayList<>();
for(int i=0; i<list.size(); i++) {
  CooperationFolderJson json = list.get(i);
  FolderVO vo = json.copyToVO();
  voList.add(vo);
}
mView.setFolderList(voList);
```

是不是有种再也不想用java写代码的感觉 :-D



