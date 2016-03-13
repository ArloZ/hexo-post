---
title: 使用 Hexo 轻松搭建博客
date: 2016-03-07 23:23:34
tags: [config,blog,tool]
category: 杂七杂八
---

一直希望能够用MarkDown方便的写博客，工具用得爽了，写起来心情也会更好。

之前使用`jekyll`搭建静态博客并在github上托管，但是由于使用`jekyll`的配置比较多且杂，而且对MarkDown的支持其实并不是很好，对代码块需要使用特有的标签，因此尝试使用`Hexo`。

`Hexo`是一个基于node的静态博客框架，[详细文档](https://hexo.io/zh-cn/docs/)

### Step 1 安装&启动
* 安装`node`/`git`请自行google
> 国内node的npm访问速度很慢，可以使用淘宝的node npm镜像：http://npm.taobao.org/

* 安装`hexo`
``` bash
$ npm install -g hexo-cli
```

* 使用`hexo`新建一个blog
``` bash
$ hexo init demo
```

* 本地启动`hexo`
``` bash
$ hexo server
```
执行命令之后，即可通过浏览器访问本地的静态博客了，默认地址是：http://localhost:4000

### Step 3 一些基本的配置
`hexo`的配置项均在`_config.yml`文件中管理，可以配置站点的基本信息、站点主题样式、网址、目录等等内容，详细的配置可参见：[https://hexo.io/zh-cn/docs/configuration.html](https://hexo.io/zh-cn/docs/configuration.html)

`Hexo`提供了多种博客主题，可以进行自由的挑选：[https://hexo.io/themes/](https://hexo.io/themes/)


### Step 2 部署到github pages
`Github pages`可以用来托管静态网站，详细介绍：[https://pages.github.com/](https://pages.github.com/)，如果要想将`Hexo`框架创建的静态网站托管在`Github pages`上，我们需要做以下的两个事情。

* 在Github上创建一个`Github pages`的代码仓库，仓库的名称为`username.github.io`，访问`http://username.github.io`这个地址的时候就可以直接访问托管在`Github pages`上的静态网站。

* 安装hexo插件，用于将静态网站部署到`Github pages`上
``` bash
$ npm install hexo-deployer-git --save
```

* 配置`hexo git`插件信息，在`_config.xml`文件中配置如下内容
``` yaml
deploy:
    type: git
    repo: git@github.com:username/username.github.io.git
```

* 配置好之后使用下面的命令就可以将博客部署到`Github pages`上面了
``` bash
$ hexo deploy -g
```

##### 使用hexo框架搭建个人博客就OK了，开始享受轻松写博客吧！



