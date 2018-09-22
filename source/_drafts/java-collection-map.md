---
title: Java 容器(集合)
tags: [java,collection,map]
date: 2016-09-02 13:32:08
categories: java
---

Java容器(集合)是十分重要的一类基础工具类库，本文详细阐述集合的基本使用以及源码实现。

<!--more-->

# 基本概念
所谓Java容器(集合)即用来存储一组元素或元素的引用的类库，`数组`是程序设计中最常见最普通的一种容器，从我们跨入程序猿的大门时，就一定跟他发生过些什么。使用`数组`可以让我们很方便的存储一组同类型的元素，并且我们可以及其方便的（使用数组下标）访问指定的某个元素。

根据他们保存的元素的方式，可以分为以下两类：
* Collection：一组独立元素的序列。
* Map：一组成对的'键-值对'的元素对象。

Java容器(集合)类库给我们提供了大量的方便存储和访问一组元素的工具，其最基础的接口为`Collection`，定义了Java容器的一些基础的方法，其他具体类型的容器接口均继承自该接口，如：`List`、`Set`、`Queue`、`Map`(没有直接继承`Collection`接口)等。


# Java容器(集合)类图

![Java容器(集合)类图](http://7xrny8.com1.z0.glb.clouddn.com/blog/1473080494694.png)

>上面的容器(集合)类图中，绿色背景的类代表常用的基础集合类，红色背景的类代表具有多线程并发特性的集合类，灰色背景的类为老版本的Java遗留的容器类，不推荐使用。`Collections`和`Arrays`是容器工具类，提供操作容器的一些基本方法以及排序算法等，大部分的容器(集合)实现了`Serializable`和`Clonable`接口。

从上面的容器(集合)类图可以看出，容器主要分为4类：`List`、`Set`、`Queue`、`Map`,下面分别介绍每一类容器。

#### List容器

`ArrayList`是程序中最常使用的一个基础容器类，类似于基本的数组但不像数组是定长的，`ArrayList`支持自动扩容，但是其内部也是基于数组实现的，同时提供了一组十分便捷的访问容器元素的方法。基础的`ArrayList`使用示例如下：

``` java
// 创建list
ArrayList<Integer> arrayList = new ArrayList<Integer>();

// 在数组最后追加元素
arrayList.add(1);
arrayList.add(1);
arrayList.add(2);
arrayList.add(3);

// 输出:[1, 1, 2, 3]
System.out.println(arrayList);

// 在指定的位置插入元素
arrayList.add(2, 4);

// 输出:[1, 1, 4, 2, 3]
System.out.println(arrayList);

// 删除下标为1的元素,即第2个元素
arrayList.remove(1);

// 输出:[1, 4, 2, 3]
System.out.println(arrayList);

// 删除指定的元素
arrayList.remove(new Integer(3));

// 输出:[1, 4, 2]
System.out.println(arrayList);
```

`LinkedList`是实现链表数据结构的一个基础容器类，能够实现快速的元素插入和删除动作。
`CopyOnWriteArrayList`是Java并发包里面的一个用于多线程并发的List容器，是一个线程安全的List容器类。

#### Queue容器
`PriorityQueue`是一个基础的优先级队列容器。
`ConcurrentLinkedQueue`是常用于多线程的链表队列

#### Set容器
`HashSet`
`TreeSet`
`LinkedHashSet`
`CopyOnWriteArraySet`
`ConcurrentSkipListSet`

#### Map容器
`HashMap`
`LinkedHashMap`
`WeakHashMap`
`EnumMap`
`TreeMap`
`IdentityHashMap`
`ConcurrentHashMap`
`ConcurrentSkipListMap`





