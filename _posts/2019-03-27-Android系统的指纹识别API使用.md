---
layout: post
title: "Android系统的指纹识别API使用"
categories: android
date: 2019-03-27 19:56:00
---


Android的指纹识别系统API是从Android M开始支持的。有专门的`FingerprintManager`类来提供指纹识别的能力，但是最新的Android P开始这个类被废弃了，取而代之的是`BiometricPrompt`类，从名字上看是生物识别，说明提供了更多的识别的支持，比如人脸识别、虹膜识别？我随便猜的😄

所以你的应用要使用指纹识别来进行验证的话，如果你的工程compileSdkVersion大于等于Android P就需要写两种调用方式，如果低于Android P，但是大于等于Android M就只需要写一种调用方式就行了。

### FingerprintManager
FingerprintManager是从API 23开始的所以使用的时候必须判断系统版本：

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    val fingerprintManager = context.getSystemService(FingerprintManager::class.java)
}
```


<!-- more -->



然后调用`FingerprintManager`的`authenticate`方法就开始识别指纹，这个方法的参数如下：
![Screen Shot 2019-03-27 at 20.41.07](http://img.muliba.net/Screen%20Shot%202019-03-27%20at%2020.41.07.png)

1. FingerprintManager.CryptoObject 一个加密对象，为了保证安全性，指纹识别过程中传递的加密验证结果的凭证，可以为null，null就代表指纹识别返回的结果app都认为是可信任的。
2. CancellationSignal 取消的回调函数，告诉系统要取消当前的指纹识别等待。
3. flags 不知道是干啥的，API上说传0就行
4. FingerprintManager.AuthenticationCallback 系统指纹识别回调函数，告诉你识别结果
5. Handler 这个好像是自己写的Handler，这样指纹识别就会使用你传递进去的Handler的looper来处理消息。我想不到有啥用，传null就行，它默认会使用mainLooper。

所以验证的代码就是类似：
```kotlin
fingerprintManager.authenticate(
    null, 
    cancel, 
    0, 
    object : FingerprintManager.AuthenticationCallback() {
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence?) {
                    Log.i(tag, "onAuthenticationError。。。code:$errorCode err:$errString。。")
                }

                override fun onAuthenticationFailed() {
                    Log.i(tag, "onAuthenticationFailed。。。。。")
                }

                override fun onAuthenticationHelp(helpCode: Int, helpString: CharSequence?) {
                    Log.i(tag, "onAuthenticationHelp。。。code:$helpCode  message:$helpString。。")
                }

                override fun onAuthenticationSucceeded(result: FingerprintManager.AuthenticationResult?) {
                    Log.i(tag, "onAuthenticationSucceeded。。。。 ")
                }
            }, 
    null)
```

这个识别过程都是后台的，前台UI啥都不会发生，不像IOS的指纹识别，会弹出一个Dialog，告诉你当前的识别情况，如下：
![](http://img.muliba.net/IMG_0851.jpg)
不过这个可以自己写一个Dialog，根据FingerprintManager.AuthenticationCallback的回调结果更新Dialog上面的提示文字来实现。
具体可以看我的demo里面的实现。[DEMO](https://github.com/fancylou/FingerprintDemo)


### BiometricPrompt

从API 28开始`FingerprintManager`就被废弃了，改用了`BiometricPrompt`，用法跟`FingerprintManager`几乎一样。但是它多了一个Builder类，用来生成指纹识别的提示Dialog，方便了很多。
初始化：

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    biometricPrompt = BiometricPrompt
            .Builder(this)
            //Dialog的标题
            .setTitle("指纹识别Demo")
            //Dialog的副标题
            .setSubtitle("这里是subTitle")
            //Dialog的提示内容
            .setDescription("这里是描述。。。。。。。")
            .build()
}
```
`authenticate`方法的参数也差不多：
![Screen Shot 2019-03-27 at 20.55.40](http://img.muliba.net/Screen%20Shot%202019-03-27%20at%2020.55.40.png)
就是多了一个`executor`线程执行器，方便你进行线程控制。
验证代码：

```kotlin
biometricPrompt?.authenticate(
    cancel, 
    this@MainActivity.mainExecutor, 
    object: BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence?) {
                    Log.i(tag, "onAuthenticationError。。。code:$errorCode err:$errString。。")
                }

                override fun onAuthenticationFailed() {
                    Log.i(tag, "onAuthenticationFailed。。。。。")
                }

                override fun onAuthenticationHelp(helpCode: Int, helpString: CharSequence?) {
                    Log.i(tag, "onAuthenticationHelp。。。code:$helpCode  message:$helpString。。")
                }

                override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult?) {
                    Log.i(tag, "onAuthenticationSucceeded。。。。")
                }
            }
    )
```


PS:全部代码，[https://github.com/fancylou/FingerprintDemo](https://github.com/fancylou/FingerprintDemo)