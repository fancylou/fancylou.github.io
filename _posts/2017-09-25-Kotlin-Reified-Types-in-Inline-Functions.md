---
layout: post
title: "Kotlin Reified Types in Inline Functions"
categories: kotlin
date: 2017-09-25 15:54:00

---

翻译自：[Kotlin Reified Types in Inline Functions](https://blog.simon-wirtz.de/kotlin-reified-types/)

我发现很多人没有听说过 `reified` 类型，或是见过但是还不太理解他到底能干嘛。这篇文章可以带领你在认识 `reified` 类型的黑暗过程中找到一丝曙光。

### Starting situation

```kotlin
fun <T> myGenericFun(c: Class<T>)
```

在一般的泛型函数中，就像`myGenericFun` ，你不能使用`T`这个类型，因为在java中运行时它会被抹去。当然如果你想使用这个泛型作为一个普通的实例在函数体内使用，你 必须显式的作为一个参数传入，就像我上面的例子里面的`c`。 这样完全能正常的跑起来，但是使用的时候就非常不美观。



<!-- more -->



### Inlined function with reified to the rescue

另一方面，如果你使用了 `reified` 泛型的`T` 在 `inline` 函数，这样你的`T`的值就能在运行时访问，并且你不需要额外的传入这个泛型`Class<T>`作为参数。你可以使用`T`就像一个普通的`Class` ，例如，你可能需要检查变量是否是`T`类型的实例，这里你就可以简单的写成：`myVar is T` 。

一个`reified`类型的`inline`函数看起来就像这样：

```kotlin
inline fun <reified T> myGenericFun()
```

必须清楚的是，`reified`类型只能和`inline`函数组合使用，一个`inline`函数会使编译器在编译阶段以字节码复制到每个使用这个函数的地方（这就是我们说的这个函数已经`inlined`）。当调用到这个使用`reified`类型的`inline`函数的时候，编译器知道这个真实类型作为参数类型，并且直接使用对应的类生成字节码替换。这样调用`myVar is T`的地方就变成了`myVar is String`（如果这里参数的类型是String的话）

### Reified in Action

让我们来看一个例子，来看看这个`reified`是怎么用的。我们想给`String`写一个叫`toKotlinObject`的扩展，它用来转化json字符串成为一个kotlin对象，使用的是泛型`T`。我们可以使用`com.fasterxml.jackson.module.kotlin` 实现，方法如下：

**Compile Error**

```kotlin
fun <T> String.toKotlinObject(): T {
      val mapper = jacksonObjectMapper()
      //does not compile!
      return mapper.readValue(JsonObject(this).encode(), T::class.java)
}
```

这里的`readValue` 方法需要我们提供是使用什么类型供它转化这个json对象，但是我们给的是`T`这个泛型。所以它就不能通过编译，编译器会说：**Cannot use ‘T’ as reified type parameter. Use a class instead.**

**Working example without reified**

```kotlin
fun <T> String.toKotlinObject(c: Class<T>): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(JsonObject(this).encode(), c)
}
```

这次我们传入了这个泛型的类，这样`readValue` 它能通过传入的`c`判断到它需要的类型。这就是我们平常在java里面使用的方式。这样调用的时候这样写：

```kotlin
"{}".toKotlinObject(MyJsonType::class.java)
```

**With reified**

现在我们使用`reified`类型的`inline`函数是怎么实现的：

```kotlin
inline fun <reified T> String.toKotlinObject(): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(JsonObject(this).encode(), T::class.java)
}
```

这样就不再需要传入T的实例，而直接可以作为普通类进行。调用方式如下：

```kotlin
"{}".toKotlinObject<MyJsonType>()
```

**Important**

`inline` `reified` 的函数不能在java代码中使用，但是普通的`inline`函数是可以的。这可能就是为什么不是每个`inline`函数的参数类型用`reified`作为默认类型的原因

### Conclusion

这是一个快速的`reified`类型介绍。在我看来使用`reified`类型在函数中使用是一个更好的方式，因为当有相关的泛型的时候我们可以用普通的<>语法来表示。最后，这样的方式写的代码可读性也比java那样传入`Class`参数要好的多。所有详细信息可以读[官方文档](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md)

如果你想知道更多kotlin的精彩的特点，我推荐你阅读这本[《Kotlin in Action》](https://www.amazon.de/gp/product/1617293296/ref=as_li_tl?ie=UTF8&camp=1638&creative=6742&creativeASIN=1617293296&linkCode=as2&tag=simonwirtzde-21&linkId=0aacf80e18c7cea57cc77f7556eef2d2)的书，同时也欢迎你来看我写的[博客文章](https://blog.simon-wirtz.de/)

Keep coding!





