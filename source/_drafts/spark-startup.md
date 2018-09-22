---
title: spark-startup
tags: [spark]
date: 2016-09-24 15:39:46
categories: spark
---

Spark Hello World
<!--more-->

# 源码
* 下载源码`git clone git@github.com:apache/spark.git`

### 编译打包
* 使用`Java 8`，`Maven 3.3.9`
    * maven编译命令：`./build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package`

### IDEA
* 安装scala、SBT插件
* 以mvn工程的方式导入spark工程,Profile界面选中YARN和Hadoop2.4
* 经过上面使用`mvn`命令编译通过后，IDEA可梳理编译通过
* 编译遇到问题：
```
Error:(45, 66) not found: type SparkFlumeProtocol val transactionTimeout: Int, val backOffInterval: Int) extends SparkFlumeProtocol with Logging {
```
    * 打开`View->Tool Window->Maven Projects`,选择`Extranal Flume Sink`右键`Generate Sources And Update Folders`
    * 然后再重新编译

* 跑example报错
    * http://stackoverflow.com/questions/31683306/running-java-spark-with-intellij
    * -Dspark.master=local


### RDD实现数据共享：
* 针对共享数据集，加载之后记录应用与共享数据上的计算操作，而不是记录真实数据，这样减少数据复制的消耗。需要恢复时，仅需对该分区执行一遍所有的计算操作即可。

* RDD：通过从稳定的存储或其他RDD数据集上执行确定性的操作来创建，确定性的操作常被称为`变换`；容错：针对丢失部分的数据进行重新执行变换即可。

* 窄依赖：父RDD仅被一个子RDD依赖
* 宽依赖：父RDD被多个子RDD依赖


