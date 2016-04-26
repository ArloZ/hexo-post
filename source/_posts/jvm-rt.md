---
title: Java虚拟机--执行系统
tags: [jvm,java,class]
date: 2016-04-10 23:29:59
categories: java
---

最近在阅读<深入理解Java虚拟机-Java高级特性与最佳实践>一书，对一些概要性的东西做一下记录，也算是一个简单的读书笔记吧，本篇问主要是记录一下类文件的结构及字节码的执行。
<!--more-->

# 类文件结构

Java文件通过编译器编译之后生成字节码文件（即class后缀的文件），JVM虚拟就是解析这些字节码文件中的字节码，然后执行对应的指令。JVM通过定义规范的字节码文件的，定义了虚拟机自身的指令集，这样就可以达到`Write once Run everywhere`的目的。一个类文件包含以下各项，他们严格的按照顺序定义，这样虚拟机才能正确的解析。

| 类型             | 名称（描述）                                   | 数量                    |
| -------------- | ---------------------------------------- | --------------------- |
| u4             | magic（魔数，用于唯一确定该文件是否为一个class文件:`0xCAFEBABE`） | 1                     |
| u2             | minor_version（次版本号）                      | 1                     |
| u2             | major_version（主版本号，从45开始，对应版本1.1，向下兼容）   | 1                     |
| u2             | constant_pool_count（常量池的大小，从1开始计数，第0项用来表示没有不引用任何常量） | 1                     |
| cp_info        | constant_pool（常量信息）                      | constant_pool_count-1 |
| u2             | access_flag                              | 1                     |
| u2             | this_class                               | 1                     |
| u2             | super_class                              | 1                     |
| u2             | interface_count                          | 1                     |
| u2             | interfaces                               | interface_count       |
| u2             | field_count                              | 1                     |
| field_info     | fields                                   | field_count           |
| u2             | methods_count                            | 1                     |
| method_info    | methods                                  | methods_count         |
| u2             | attributes_count                         | 1                     |
| attribute_info | attributes                               | attributes_count      |

### 常量池

常量池中主要存储两大类常量：字面量（通常指代码中定义的文本字符串或final常量）和符号引用。符号引用主要包含：

* 类和接口的全限定名
* 字段的名称和描述符
* 方法的名称和描述符

常量池中的每一项都是一个表，每一项都有一个u1类型的标志位(tag)。如下表

| 类型                               | 标志位（tag） | 描述                 |
| -------------------------------- | -------- | ------------------ |
| CONSTANT_Utf8_info               | 1        | Utf8编码的字符串         |
| CONSTANT_Integer_info            | 3        | 整数型字面量             |
| CONSTANT_Float_info              | 4        | 浮点型字面量             |
| CONSTANT_Long_info               | 5        | 长整型字面量             |
| CONSTANT_Double_info             | 6        | 双精度浮点型字面量          |
| CONSTANT_Class_info              | 7        | 类或接口的符号引用（全限定名）    |
| CONSTANT_String_info             | 8        | 字符串型字面量            |
| CONSTANT_Fieldref_info           | 9        | 字段的符号引用（名称和描述符）    |
| CONSTANT_Methodref_info          | 10       | 类中方法的符号引用（名称和描述符）  |
| CONSTANT_InterfaceMethodref_info | 11       | 接口中方法的符号引用（名称和描述符） |
| CONSTANT_NameAndType_info        | 12       | 字段或方法的部分符号引用       |

上面的这些常量类型都有自己特定的数据结构：

* CONSTANT_Utf8_info

  | 类型   | 名称                  | 数量     |
  | ---- | ------------------- | ------ |
  | u1   | 标志位：tag = 1         | 1      |
  | u2   | 字符串长度：length        | 1      |
  | u1   | utf8编码字符串字节内容：bytes | length |

* CONSTANT_Integer_info

  | 类型   | 名称                 | 数量   |
  | ---- | ------------------ | ---- |
  | u1   | 标志位：tag = 3        | 1    |
  | u4   | 按高位在前存储的int值：bytes | 1    |

* CONSTANT_Float_info

  | 类型   | 名称                   | 数量   |
  | ---- | -------------------- | ---- |
  | u1   | 标志位：tag = 4          | 1    |
  | u4   | 按高位在前存储的float值：bytes | 1    |

* CONSTANT_Long_info

  | 类型   | 名称                  | 数量   |
  | ---- | ------------------- | ---- |
  | u1   | 标志位：tag = 5         | 1    |
  | u8   | 按高位在前存储的long值：bytes | 1    |

* CONSTANT_Double_info

  | 类型   | 名称                    | 数量   |
  | ---- | --------------------- | ---- |
  | u1   | 标志位：tag = 6           | 1    |
  | u8   | 按高位在前存储的double值：bytes | 1    |

* CONSTANT_Class_info

  | 类型   | 名称                          | 数量   |
  | ---- | --------------------------- | ---- |
  | u1   | 标志位：tag = 7                 | 1    |
  | u2   | 类或接口全限定名指向的字符串引用：name_index | 1    |


* CONSTANT_String_info

  | 类型   | 名称               | 数量   |
  | ---- | ---------------- | ---- |
  | u1   | 标志位：tag = 8      | 1    |
  | u2   | 指向字符串常量的索引：index | 1    |

* CONSTANT_Fieldref_info

  | 类型   | 名称                                       | 数量   |
  | ---- | ---------------------------------------- | ---- |
  | u1   | 标志位：tag = 9                              | 1    |
  | u2   | 指向声明该字段的类或接口的描述符CONSTANT_Class_info的索引：index | 1    |
  | u2   | 指向该字段的描述符CONSTANT_NameAndType_info的索引：index | 1    |



- CONSTANT_Methodref_info

  | 类型   | 名称                                       | 数量   |
  | ---- | ---------------------------------------- | ---- |
  | u1   | 标志位：tag = 10                             | 1    |
  | u2   | 指向声明该方法的类的描述符CONSTANT_Class_info的索引：index | 1    |
  | u2   | 指向该方法的描述符CONSTANT_NameAndType_info的索引：index | 1    |


- CONSTANT_InterfaceMethodref_info

  | 类型   | 名称                                       | 数量   |
  | ---- | ---------------------------------------- | ---- |
  | u1   | 标志位：tag = 11                             | 1    |
  | u2   | 指向声明该方法的接口的描述符CONSTANT_Class_info的索引：index | 1    |
  | u2   | 指向该接口的描述符CONSTANT_NameAndType_info的索引：index | 1    |


- CONSTANT_NameAndType_info

  | 类型   | 名称                    | 数量   |
  | ---- | --------------------- | ---- |
  | u1   | 标志位：tag = 12          | 1    |
  | u2   | 指向声明该字段或方法的名称常量：index | 1    |
  | u2   | 指向该字段的描述符常量项索引：index  | 1    |

### 访问标志

访问标志位于常量池之后，主要用于标识是类还是接口，是否定义为public，是否声明为final等等信息

![access_flag](http://7xrny8.com1.z0.glb.clouddn.com/blog/1460803332513.png)

### 类索引、父类索引和接口集合

class文件通过这三项来决定这个类的继承关系，除了`java.lang.Object`类的父类索引为0之外，其他的任何java类的父类都不为0，他们均指向了`CONSTANT_Class_info`类型的常量。

### 字段表集合（field_info）

字段表集合用于描述类或接口中声明的变量，包括了类级别和实例级别的变量，但不包含方法内部申明的变量。

| 类型             | 名称                          | 数量               |
| -------------- | --------------------------- | ---------------- |
| u2             | 访问标志（类似于类的访问标志）：access_flag | 1                |
| u2             | 字段简单名称索引：name_index         | 1                |
| u2             | 字段描述符索引：descriptor_index    | 1                |
| u2             | 字段的其他属性个数：attributes_count  | 1                |
| attribute_info | 属性信息集合：attributes           | attributes_count |

描述符主要用于描述字段的数据类型、方法的参数列表和返回值，描述符标识符的含义：

![descriptor](http://7xrny8.com1.z0.glb.clouddn.com/blog/1460804670400.png)

字段集合中不会列出超类或父接口中继承来的字段。

### 方法表集合

方法表集合与字段表集合类似，用于描述类中定义的方法，如果没有进行方法重载，方法表集合中不包含父类的方法。

| 类型             | 名称                          | 数量               |
| -------------- | --------------------------- | ---------------- |
| u2             | 访问标志（类似于类的访问标志）：access_flag | 1                |
| u2             | 方法的简单名称索引：name_index        | 1                |
| u2             | 方法描述符索引：descriptor_index    | 1                |
| u2             | 方法的其他属性个数：attributes_count  | 1                |
| attribute_info | 属性信息集合：attributes           | attributes_count |

### 属性表集合

属性表用于描述Class文件、字段表、方法表中的专有信息，不要求有严格的顺序。

![](http://7xrny8.com1.z0.glb.clouddn.com/blog/1460805902136.png)

对于每个属性，他的名称指向常量池的一个`CONSTANT_Utf8_info`类型的常量，属性值的结构可以自定义，只要说明属性值信息的长度即可。

# 虚拟机类加载机制

java虚拟机类加载机制：虚拟机将Class文件加载到内存，对数据进行校验、转换解析和初始化，最终得到在Java虚拟机中可以直接使用的Java类型。Java类型的加载和链接都是运行前进行的，与语言的编译、链接过程不同，会带来运行期的一些额外性能开销。

Java类的生命周期包含：加载、验证、准备、解析、初始化、使用、卸载7个过程。

### 类的加载时机

Java类在如下时刻需要进行初始化

* 遇到new、getstatic、putstatic、invokestatic 4条指令时，如果类还没有进行初始化，则需要进行初始化。
* 使用reflect反射包对类进行反射调用时，如果没有进行初始化则需要进行初始化。
* 初始化一个类时，如果发现其父类还咩有初始化，则首先触发父类的初始化。
* 虚拟机启动时，用户指定要执行的类（包含main方法的），这个类需要进行初始化。

### 类的加载过程

* 加载

  * 通过类的全限定名找到类对应的二进制流
  * 将二进制流转换为方法区的运行时数据结构
  * 在堆中生成这个类对应的一个`java.lang.Class`对象

* 验证

  验证是链接的第一个步骤，主要目的是为了确保类对应的二进制流符合虚拟机的要求，并且不会影响虚拟机自身的安全

  * 文件格式验证--验证之后字节流将进入方法区存储，后面的阶段基于方法区数据结构来进行
  * 元数据验证
  * 字节码验证
  * 符号引用验证

* 准备

  准备阶段正式为类变量(static 变量，不包括实例变量)分配内存，并设置初始值（零值，而不是变量定义时设置的初始值），这些内存都在方法区进行分配

* 解析

  解析阶段是将常量池内的符号引用解析为直接引用的过程。

* 初始化

# 类加载器

### 类与类加载器

通过两个类加载器加载的类，在使用`instanceof`关键字的时候认为`不相等`

### 双亲委派模型

* 启动类加载器（Bootstrap ClassLoader）
* 扩展类加载器（Extension ClassLoader）
* 应用程序类加载器（Application ClassLoader）
* 其他用户自定义的类加载器

双亲委派模型要求每个类都有一个父类的引用，当需要加载一个类时首先使用父类尝试进行加载，如果父类不能加载，则自己进行加载，每一层的类加载器都如此。

### 破坏双亲委派模型











