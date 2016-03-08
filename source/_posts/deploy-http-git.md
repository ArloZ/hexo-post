---
title:  "部署HTTP服务的git"
date:   2014-05-16 17:21:53
category: install&configure
tags: git apache nginx
---

最近使用git的频率不断增加，需要与人共享git工程，如果使用原生的git协议进行访问的话不是特别方便，因此着手通过部署HTTP服务来实现git的管理，一些配置如下介绍。

### 1.系统环境
- ubuntu12.04

### 2.安装

``` bash
$ sudo apt-get install git git-core apache2
$ cd /etc/apache2
$ sudo a2enmod dav dav_fs dav_lock
$ sudo a2enmod auth_digist
```

### 3.初始化git仓库
``` bash
$ cd /home & mkdir git & cd git & mkdir test.git & cd test.git
$ git --bare init
$ mv hooks/post-update.sapmle hooks/post-update
$ chmod a+x hooks/post-update
$ ./hooks/post-update
```

### 4.apache配置

在site-available下新建 git，内容如下：
```
<VirtualHost>
Servername localhost
SetEnv GIT_HTTP_EXPORT_ALL
SetEnv GIT_PROJECT_ROOT /home/git
ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
<Location /git>
    Dav on
    Options +Indexes +FollowSymLinks
    #AllowOverride None
    Order Allow,Deny
    Allow from all

    AuthType Digest
    AuthName "git"
    AuthDigestDomain git
    AuthUserFile /home/git/git.htpasswd
    Require valid-user
</Location>
</VirtualHost>
```

添加密码(第二次添加时不用加 -c)
```
$ htdigest -c ./git.htpasswd git name
$ sudo service apache2 restart
```

### 5.使用

```
$ git clone http://username:password@localhost/git/test.git
```

忽略ssl临时：
```
$ env  GIT_SSL_NO_VERIFY=true git clone https://username:passwd@localhost/git/test.git
```

配置：
```
$ git config  http.sslVerify false
```

__refs__
[在Linux上用Apache搭建Git服务器](http://www.cnblogs.com/dudu/archive/2012/12/09/linux-apache-git.html)
