---
layout: post
title: "Flutter 学习7：Dart语言基础"
categories: flutter
date: 2019-01-11 13:00:00
---

1. [Flutter 学习1：开发环境、开发工具、初始化一个项目](http://www.muliba.net/ios/android/2018/11/16/Flutter-%E5%AD%A6%E4%B9%A0-%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83-%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7-%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html)
2. [Flutter 学习2：从main.dart文件说起](http://www.muliba.net/flutter/2018/11/23/Flutter-%E5%AD%A6%E4%B9%A0-%E4%BB%8Emain.dart%E6%96%87%E4%BB%B6%E8%AF%B4%E8%B5%B7.html)
3. [Flutter 学习3：转场、导航](http://www.muliba.net/flutter/2018/12/04/Flutter-%E5%AD%A6%E4%B9%A03-%E8%BD%AC%E5%9C%BA-%E5%AF%BC%E8%88%AA.html)
4. [Flutter 学习4：集成到原有的项目中](http://www.muliba.net/flutter/2018/12/09/Flutter-%E5%AD%A6%E4%B9%A04-%E9%9B%86%E6%88%90%E5%88%B0%E5%8E%9F%E6%9C%89%E7%9A%84%E9%A1%B9%E7%9B%AE%E4%B8%AD.html)
5. [Flutter 学习5：开发Dart包和插件包](http://www.muliba.net/flutter/2018/12/14/Flutter-%E5%AD%A6%E4%B9%A05-%E5%BC%80%E5%8F%91Dart%E5%8C%85%E5%92%8C%E6%8F%92%E4%BB%B6%E5%8C%85.html)
6. [Flutter 学习6：绘制动画](http://www.muliba.net/flutter/2018/12/28/Flutter-%E5%AD%A6%E4%B9%A06-%E7%BB%98%E5%88%B6%E5%8A%A8%E7%94%BB.html)
7. Flutter 学习7：Dart语言基础
8. [Flutter 学习8：BottomSheet](http://www.muliba.net/flutter/2019/01/26/Flutter-%E5%AD%A6%E4%B9%A08-BottomSheet.html)
9. [Flutter 学习9：Positioned、Transform等控件使用以及手势控制](http://www.muliba.net/flutter/2019/01/31/Flutter-学习9-Positioned-Transform等控件使用以及手势控制.html)
10. [Flutter 学习10：NestedScrollView、SliverAppBar、TabBar](http://www.muliba.net/flutter/2019/02/17/Flutter-学习10-NestedScrollView-SliverAppBar-TabBar.html)

这些概念其实一开始就应该去弄明白的，当时接触Flutter的时候发现跟Java很类似，也就没有花时间去看看文档，后来在用的过程中发现还是有必要看看文档的，不然在开发中经常会遇到很多语法上面的疑惑再去查文档，蛮浪费时间的。

## 关键概念
* 在Dart里面有一个重要的概念是所有的变量都是`Object`对象，比如Numbers、Strings等等，甚至functions、null等都是`Object`对象，他们最终都继承自`Object`。所以它相当于可以接受任何类型的值。

```dart
 Object s = 'this is string';
  s = 8;
  print(s.toString()); // 这里s是Object对象，只能执行Object的函数
```
* 还有一个可以接受所有类型值的是`dynamic` 。它和上面`Object`的区别是 dynamic是告诉了编译器这个变量不需要你来推断类型，后续代码会自己处理变量类型的。

```dart
  dynamic s = 'this is string';
  if (s is String) {
    print(s); // 打印this is string
  }
  s = 8;
  if (s is int) {
    s++; // 自动推断是int类型
    print(s.toString()); // 打印9
  }
```
这里的`is`关键字就是判断变量类型的，并且还有智能推断作用，只要通过了判断，if内部的语句中这个变量类型就是对应的判断成功的类型。

## 变量
和其他语言一样Dart定义了很多常见的基本变量类型，int、double、String、bool等，其中int、double都是64位的。但前面说了Dart是有类型推断的，所以一般变量可以用`var`关键字表示。还有常量可以用`final`和`const`表示，他们两区别是`final`是运行时常量，`const`是编译时常量。

<!-- more -->

### 基本类型

```dart
  String s = 'this is string';
  int n = 1;
  double d = 1.0;
  bool b = true;
  
  final String s = 'this is string';
  final int n = 1;
  final double d = 1.0;
  final bool b = true;
  
  const String s = 'this is string';
  const int n = 1;
  const double d = 1.0;
  const bool b = true; 
```
也可以这样写：

```dart
  var s = 'this is string';
  var n = 1;
  var d = 1.0;
  var b = true;
  
  final s = 'this is string';
  final n = 1;
  final d = 1.0;
  final b = true;
  
  const s = 'this is string';
  const n = 1;
  const d = 1.0;
  const b = true; 
```
#### String
字符串可以用 `+` 连接，字符串内部还可以用`$`表示变量。

```dart
  var s1 = 'this is string';
  var s2 = 's1:$s1';
  var s3 = 's1:'+ s1;
  if (s2 == s3) {
    print('true');
  }
```

### Lists、Maps、Sets
#### Lists
List 就跟js里的数组差不多，它可以直接用字面量新建：

```dart
var list = [1, 2, 3];
//指定泛型
vat list1 = <int>[];
```
也可以用List的构造器新建：

```dart
var list = List<int>();
```
定义不可变数组：

```dart
var list2 = const [1, 2]; //数组前加了const就不能改变这个数组了
list2.add(3);// 错误
list2 = [2, 3]; // list2本身是变量 所以这样是可以的
```
可以通过下标修改值：

```dart
var list = [1, 2, 3];
list[1] = 1;
assert(list[1] == 1);
```
#### Maps
Map和List一样可以通过字面量和构造器定义：

```dart
  var map = const{
    'key1': 'value1',
    'key2': 'value2'
  };
  var map1 = Map();
  map1['key1'] = 'value1';
  map1['key2'] = 'value2';
  // 指定泛型
  var map2 = Map<String, String>();
```
如果没有对应的key找出来的值就是null：

```dart
assert(map['key'] == null);
```

#### Sets
Set只能通过构造器定义：

```dart
  var set = Set<String>();
  set.add('123');
  if(set.contains('123')){
    print('true');
  }
```


## 函数
Dart中的函数和Java有点类似。

```dart
void abc() {
}
```
支持可选参数：

```dart
void abc(int i,[int l]){
}
// 也支持默认值
void fo(int i, [int l = 9]) {
}
void main() {
abc(1);
abc(1, 2);
}
```
还支持具名参数，具名参数也可以有默认值：

```dart
void abc({int x, int y = 0}){
}
void main() {
abc(x: 1);
abc(x: 1, y:2);
}
```
函数还可以定义为变量，可以用别名表示：

```dart
typedef abc = int Function(int, int);
void main() {
  abc a = (int i, int j) {
    return i+j;
  } ;
  var r = a(1, 2);
  print(r);
}
```
甚至还能在函数内部定义函数：

```dart
typedef abc = int Function(int, int);

abc generateAbc(int z) {
  int xyz(int x, int y) {
    return x + y + z;
  }
  return xyz;
}

void main() {
  var r = generateAbc(1);
  print(r(2, 3));
}
```
如果函数内部只有一句语句还可以用 `=>` 符号简化：

```dart
  int xyz(int x, int y) {
    return x + y ;
  }
  //跟上面等价
  int xyz(int x, int y)  => x+y;
```

## 操作控制语句

```dart
// 瀑布流写法 这个比较有意思
querySelector('#confirm') // Get an object.
  ..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```
上面这种中间两个点连接的写法是啥意思呢，跟下面的写法是等价的：

```dart
var button = querySelector('#confirm');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```
### 控制语句

```dart
//if
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
//for
for (var i = 0; i < 5; i++) {
  print('i：$i');
}
//for in
for (var x in collection) {
  print(x); 
}
//while
while (!isDone()) {
  doSomething();
}
//do while
do {
  printLine();
} while (!atEndOfPage());
//switch
var command = 'OPEN';
switch (command) {
  case 'CLOSED':
    executeClosed();
    break;
  case 'PENDING':
    executePending();
    break;
  case 'APPROVED':
    executeApproved();
    break;
  case 'DENIED':
    executeDenied();
    break;
  case 'OPEN':
    executeOpen();
    break;
  default:
    executeUnknown();
}
```
## 异常
抛出异常：

```dart
throw FormatException('Expected at least 1 section');
```
它还可以抛出普通类型的对象：

```dart
throw 1001;
```
捕获异常：

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
} on Exception catch (e) {
  // Anything else that is an exception
} catch (e) {
  // No specified type, handles all
}
```

## 类
类定义跟Java差不多，参数传入构造函数：
```dart
class P {
  var x;
  var y;
  P(int x, int y) {
    this.x = x;
    this.y = y;
  }
}
```
但是Dart可以简化：

```dart
class P {
  var x;
  var y;
  P(this.x, this.y) ;
}
```
当然跟函数一样也可以定义具名参数：

```dart
class P {
  var x;
  var y;
  P({this.x, this.y}) ;
}

void main() {
  P(x: 1, y: 2);
}
```
Dart类还有一个`Initializer lists`的概念，初始化类内部参数用的：

```dart
class P {
  var x;
  var y;
  P(int x, int y): x = x, y = y ;
}
```
这里冒号后面的语句就是`Initializer lists`，它会在构造函数执行前执行，它会识别等号前面那个x是类内部的x变量。
类继承很简单使用`extends`关键字，super是跟在 `Initializer lists` 后面：

```dart
class P {
  var x;
  var y;
  P(int x, int y): x = x, y = y ;
}

class Q extends P {
  int z;
  Q(int x, int y, int z): z = z, super(x, y) ;
}
```

## 访问控制
Dart没有public、private这样的访问权限关键字，它都是用package分割的。默认如果不想让外部访问命名的时候可以用下划线开头：

```dart
class _Abc {
}
class Abc {
    int _i;
}
```
访问外部包需要引入：

```dart
import 'dart:html';
```
也可以只引入某一个需要的文件：

```dart
import 'package:test/test.dart';
```
防止冲突还可以使用别名：

```dart
import 'package:lib2/lib2.dart' as abc;
//使用的时候
abc.Element element2 = abc.Element();
```
控制某些单独的对象引入：

```dart
// 这里只引入了foo对象
import 'package:lib1/lib1.dart' show foo;
// 这里是除了foo对象
import 'package:lib2/lib2.dart' hide foo;
```
甚至极端一点还可以运行时加载外部包，这样可以更快速的启动应用，或者有些业务场景下异步加载模块的时候可以用：

```dart
import 'package:greetings/hello.dart' deferred as hello;
```
使用的时候需要异步加载外部包：

```dart
Future greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```






