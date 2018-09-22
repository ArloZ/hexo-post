---
title: Scala与Java语法差异
tags: [scala,java]
date: 2016-09-22 08:01:56
categories: scala
---

最近学习一下`scala`，以下列举一些在语法上与`Java`之间的差异。

<!--more-->

## Scala语法
* 在`Scala`中你不能使用`Thread.yield()`是因为`yield`为`Scala`中的关键字，你必须使用`Thread.\`yield\`()`来使用这个方法。
* 行末不用加`;`
* 包定义，除了在文件首行定义之外，还可以按如下方式定义包，以实现一个文件定义多个包
```
package com.runoob {
  class HelloWorld 
}
```
* 引用包
```
import java.awt.Color  // 引入Color
 
import java.awt._  // 引入包内所有成员

import java.awt.{Color, Font} // 引入Color和Font

import java.util.{HashMap => JavaHashMap} // 重命名成员
 
import java.util.{HashMap => _, _} // 隐藏成员，引入了util包的所有成员，但是HashMap被隐藏了
```
* `Scala`具有和`java`相同的数据类型，所有类型均为对象，没有像java一样的原生类型，增加了
    * Unit 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。
    * Null null 或空引用
    * Nothing 类型在Scala的类层级的最低端；它是任何其他类型的子类型。
    * Any 是所有其他类的超类
    * AnyRef 类是Scala里所有引用类(reference class)的基类
* 多行字符串用三个双引号来表示分隔符，格式为：""" ... """。
* `Scala`用`var`声明变量，用`val`声明常量，定义语法：`var VariableName : DataType [=  Initial Value]`
* 作用域的声明可以用使用`private[x]`方式来表示允许x及其对象可见，而其他均不可见
* `>>>`无符号右移运算符，
* 先算术运算，后移位运算，最后位运算。请特别注意：`1 << 3 + 2 & 7` 等价于 `(1 << (3 + 2))&7` 
* `Scala`不支持`break`语句，但是可以通过使用`Breaks`对象来实现；
```
// 导入以下包
import scala.util.control._

// 创建 Breaks 对象
val loop = new Breaks;

// 在 breakable 中循环
loop.breakable{
    // 循环
    for(...){
       ....
       // 循环中断
       loop.break;
   }
}
```
## Scala函数

* 传名调用：`def delay(t : => Long)`设置为传名调用，表示仅传入函数名，使用变量`t`时执行函数。

* 不定参数函数：`def printStrings( args:String* )`最后一个参数名使用* 来表示不定参数个数

* 匿名函数：`var add = (x: Int, y: Int) => x + y`将函数赋值给变量`add`，可以通过`add(1,2)`调用函数

* 柯里化(Currying)函数：将原来接受两个参数的函数变成新的接收一个参数的函数并返回一个新的接收一个参数的函数，以下两个函数调用等效：
```
def add(x: Int, y: Int) = x + y       // 普通具有两个参数的函数
add(1,2)                              // 函数调用执行结果为 3

def add(x: Int)(y: Int) = x + y       // 柯里化的等效的函数
add(1)(2)                             // 函数调用执行结果为 3

def add(x: Int) = (y: Int) => x + y   // 定义第一次函数调用返回一个匿名函数，匿名函数以第二个参数作为自己的入参
add(1)(2)                             // 函数调用执行结果为 3
```
从上可以看出柯里化函数实际上是执行了两个普通的函数调用，以第一个参数为入参的函数返回了一个匿名函数，该匿名函数以第二个参数为入参。

* 函数闭包：一个函数引用了自身外部的变量，将函数本身和外部变量一同返回给一个函数变量，被引用的变量与该函数一同存在。
```
var fac = 10
val mul = (i: Int) => i * fac
mul(2)                              // 函数变量mul构成了一个“闭包”，输出结果：20

fac = 12                            // 修改变量的值
mul(2)                              // 函数中引用的外部变量值也一起修改，输出结果：24

var fac = 5                         // 重新同名的变量，不影响“闭包”中的变量值
mul(2)                              // “闭包”中引用的变量值仍然为12，输出结果24
```
