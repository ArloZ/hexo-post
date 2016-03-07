---
title: hexo-setup
date: 2016-03-07 23:23:34
tags: 杂七杂八,工具,配置
---

## Hexo 搭建静态博客并在github上托管
>一直希望能够用MarkDown方便的写博客，工具用得爽了，写起来心情也会更好。

>之前使用`jekyll`搭建静态博客并在github上托管，但是由于使用`jekyll`的配置比较多且杂，而且对MarkDown的支持其实并不是很好，对代码块需要使用特有的标签，因此尝试使用`Hexo`。

>`Hexo`是一个基于node的静态博客框架，[详细文档](https://hexo.io/zh-cn/docs/)

### Step 1 安装&启动
* 安装`node`/`git`请自行google
> 国内node的npm访问速度很慢，可以使用淘宝的node npm镜像：http://npm.taobao.org/

* 安装`hexo`
```
$ npm install -g hexo-cli
```
* 使用`hexo`新建一个blog
```
$ hexo init demo
```

* 本地启动`hexo`
```
$ hexo server
```
执行命令之后，即可通过浏览器访问本地的静态博客了，默认地址是：http://localhost:4000

### Step 3 一些基本的配置

### Step 2 部署到github pages





