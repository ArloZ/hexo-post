---
title: docker-start
tags: [docker,hadoop]
date: 2016-10-04 20:14:53
categories: docker
---

[Docker](https://www.docker.com/)是目前十分流行的一种虚拟容器技术，能够很方便的实现应用的统一部署和运维管理。最近学习`hadoop`和`spark`，正好可以利用`docker`在本地部署集群用于运行`hadoop`和`spark`应用，本文主要介绍`docker`的基本使用以及镜像制作。

<!--more-->
`Docker`是`GO`语言实现的一个轻量级的开源容器，其在Linux的基础上提供一个抽象层，用于管理Linux资源，为应用程序提供独立的运行环境。

# Get Start
> Mac上安装Docker：https://docs.docker.com/engine/installation/mac/
> 国内访问网速原因，可以使用镜像服务器：
> 网易的docker镜像：https://c.163.com/hub#/m/home/
> Aliyun Docker镜像：https://dev.aliyun.com/list.html，可以注册账户并登陆后可以使用Aliyun的镜像加速Docker Image

### 获取镜像到本地
```
$ docker pull ubuntu                            # 获取镜像到本地
$ docker images                                 # 列出本地已有的镜像
```

### 与宿主机共享目录
```
$ docker run -ti -h ubuntu -v /Users/share:/sharedata ubuntu /bin/bash     # 将宿主机的/Users/share挂载到docker容器的/sharedata目录
```

### 启动Docker容器
```
$ docker run -t -i ubuntu /bin/bash             # 启动ubuntu镜像，并执行bash命令
$ docker ps -a                                  # 查看所有的docker容器
$ docker logs [container ID or NAMES]           # 查看docker容器的标准输出
$ docker start/stop [container ID or NAMES]     # 启动/停止一个已存在的容器
$ docker rm [container ID or NAMES]             # 删除一个docker容器
$ docker attach [container ID or NAMES]         # 通过终端连接已启动运行的容器
$ docker run -t -i --name hadoop-base arlozql/hadoop-base /bin/bash -c 'su admin'
```
### 基于ubuntu创建镜像
> 在Aliyun私有镜像管理页面创建镜像仓库：https://cr.console.aliyun.com/#/imageList

```
$ docker commit -m "add base software" -a "ubuntu-dev-env" [container ID] arlozql/ubuntu-dev:0.1    # 创建docker镜像
$ docker login --username=arlozql@gmail.com registry.cn-hangzhou.aliyuncs.com                       # 使用aliyun账户登录
$ docker tag 107b2838e90d registry.cn-hangzhou.aliyuncs.com/arlozql/ubuntu-dev:1.0                  # 给image打tag
$ docker push registry.cn-hangzhou.aliyuncs.com/arlozql/ubuntu-dev:1.0                              # 将镜像push到私有仓库
```

### 基于docker搭建spark集群

* 相关资源下载
    * hadoop-2.6.4.tar.gz
    * jdk-8u101-linux-x64.tar.gz
    * scala-2.11.8.tgz
    * spark-2.0.0-bin-hadoop2.6

* 将下载好的所有包解压放在`/home/spark/`目录下
```
root@ubuntu:/home/spark# ll
total 24
drwxr-xr-x  6 root root 4096 Oct 11 15:07 ./
drwxr-xr-x  3 root root 4096 Oct 10 15:52 ../
drwxr-xr-x  9 root root 4096 Oct 11 15:06 hadoop-2.6.4/
drwxr-xr-x  8 root root 4096 Oct 11 15:06 jdk1.8.0_101/
drwxr-xr-x  6 root root 4096 Oct 11 15:07 scala-2.11.8/
drwxr-xr-x 12 root root 4096 Oct 11 15:07 spark-2.0.0-bin-hadoop2.6/
```

* 基础环境变量配置
    * 安装ssh：
    ```
    $ apt-get install openssh-server
    $ ssh-keygen                                                # 一路回车
    $ cd ~ && cat .ssh/id_rsa.pub > .ssh/authorized_keys        # 设置免密ssh登录
    $ 在 .bashrc 文件末添加 `service ssh start`                   # 登录时启动ssh服务
    $ ssh localhost                                             # 验证ssh服务是否ok
    ```

* Java安装配置
    * 在`~/.bashrc`文件中添加如下配置
    ```
    export JAVA_HOME=/home/spark/jdk1.8.0_101
    export JRE_HOME=$JAVA_HOME/jre
    export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
    export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
    ```
    * 执行`source ~/bashrc`使配置生效
    * 执行`java -version`检查Java是否配置成功

* Scala安装配置
    * 在`~/.bashrc`文件中添加如下配置
    ```
    export SCALA_HOME=/home/spark/scala-2.11.8
    export PATH=$PATH:$SCALA_HOME/bin
    ```
    * 执行`source ~/bashrc`使配置生效
    * 执行`scala -version`检查Java是否配置成功


* hadoop配置
    * `hadoop-2.6.4/etc/hadoop/hadoop-env.sh`中配置
    ```
    export JAVA_HOME=/home/spark/jdk1.8.0_101/
    ```
    * `hadoop-2.6.4/etc/hadoop/yarn-env.sh`中配置
    ```
    export JAVA_HOME=/home/spark/jdk1.8.0_101/
    ```
    * `hadoop-2.6.4/etc/hadoop/slaves`中配置Slave服务器的主机名
    ```
    slave1
    slave2
    slave3
    ```
    * `hadoop-2.6.4/etc/hadoop/core-site.xml`中配置
    ```
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://master:9000/</value>
        </property>
        <property>
             <name>hadoop.tmp.dir</name>
             <value>file:/home/spark/hadoop-2.6.4/tmp</value>
        </property>
    </configuration>
    ```
    * `hadoop-2.6.4/etc/hadoop/hdfs-site.xml`中配置
    ```
    <configuration>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>master:9001</value>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/home/spark/hadoop-2.6.4/dfs/name</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/home/spark/hadoop-2.6.4/data</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>3</value>
        </property>
    </configuration>
    ```
     * `hadoop-2.6.4/etc/hadoop/mapred-site.xml`中配置
    ```
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    ```
     * `hadoop-2.6.4/etc/hadoop/yarn-site.xml`中配置
    ```
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
            <name>yarn.resourcemanager.address</name>
            <value>master:8032</value>
        </property>
        <property>
            <name>yarn.resourcemanager.scheduler.address</name>
            <value>master:8030</value>
        </property>
        <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>master:8035</value>
        </property>
        <property>
            <name>yarn.resourcemanager.admin.address</name>
            <value>master:8033</value>
        </property>
        <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>master:8088</value>
        </property>
    </configuration>
    ```

* Spark配置
    * `cp spark-env.sh.template spark-env.sh`
    * 在`spark-env.sh`文件末尾添加以下配置
    ```
    export SCALA_HOME=/home/spark/scala-2.11.8
    export JAVA_HOME=/home/spark/jdk1.8.0_101
    export HADOOP_HOME=/home/spark/hadoop-2.6.4
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    SPARK_MASTER_IP=master
    SPARK_LOCAL_DIRS=/home/spark/spark-2.0.0-bin-hadoop2.6
    SPARK_DRIVER_MEMORY=1G
    ```
    * 在slaves文件中添加slave主机名称
    ```
    slave1
    slave2
    slave3
    ```



