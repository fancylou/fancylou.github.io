---
layout: post
title: "kotlin特殊关键字，let、apply、also、with、run"
categories: kotlin
date: 2018-02-05 11:26:00
---



#### let

定义：

```kotlin
/**
 * Calls the specified function [block] with `this` value as its argument and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)

```

执行一个参数传入的函数，该函数的参数是当前对象。并且返回一个完全自由的结果。

##### 典型用法

- 转换类型

```kotlin
val answerToUniverse = strBuilder.let {
    it.append("Douglas Adams was right after all")
    it.append("Life, the Universe and Everything")
    42
}
```

- 处理Nullability

  ```kotlin
  str?.let { print(it) }
  ```



<!-- more -->



#### apply

定义：

```kotlin
/**
 * Calls the specified function [block] with `this` value as its receiver and returns `this` value.
 */
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```

执行一个以参数传入的函数，该函数在当前对象上面执行，并返回当前对象

##### 典型用法

- 初始化或者配置一个对象

  ```kotlin
  // old way of building an object
  val andre = Person()
  andre.name = "andre"
  andre.company = "Viacom"
  andre.hobby = "losing in ping pong"
  // after applying 'apply' (pun very much intended)
  val andre = Person().apply {
      name = "Andre"
      company = "Viacom"
      hobby = "losing in ping pong"
  }
  ```

#### also

定义：

```kotlin
/**
 * Calls the specified function [block] with `this` value as its argument and returns `this` value.
 */
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }
```

执行一个以参数传入的函数，函数的参数就是当前对象，并返回当前对象

##### 典型用法

- 保持链式编程

  ```kotlin
  // transforming data from api with intermediary variable
  val rawData = api.getData()
  Log.debug(rawData)
  rawData.map {  /** other stuff */  }
  // use 'also' to stay in the method chains
  api.getData()
      .also { Log.debug(it) }
      .map { /** other stuff */ }
  ```



#### with

定义：

```kotlin
/**
 * Calls the specified function [block] with the given [receiver] as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```

不是一个扩展函数，相关对象作为参数传入，然后在传入对象上面执行特殊的函数，并且返回执行函数的返回值

##### 典型用法

- 对象执行规范化、可读化

  ```kotlin
  // Every Android Developer 
      messageBoard.init(“https://url.com”)
      messageBoard.login(token)
      messageBoard.post(“Kotlin’s a way of life bro")

  // using 'with' to avoid repetitive references to identifier
  with(messageBoard) {
      init(“https://url.com”)
      login(token)
      post(“Kotlin’s a way of life bro")
  }
  ```



####run

定义：

```kotlin
/**
 * Calls the specified function [block] with `this` value as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R = block()
```

给当前对象执行一个函数，返回结果就是函数的返回结果，这个run还有一个不是扩展的版本

##### 典型用法

- 跟`let`类似但是它是以扩展函数执行的

  ```kotlin
  // GoT developers after season 7
  aegonTargaryen = jonSnow.run {
      makeKingOfTheNorth()
      swearsFealtyTo(daenerysTargaryen)
      realIdentityRevealed(“Aegon Targaryen”)
  }
  ```









