---
title: '使用DigitalOcean搭建PPTPD的VPN服务'
date: 2015-01-10 19:23:45
category: 杂七杂八
tags: [DigitalOcean,VPN,VPS,PPTPD]
---

要自己搭建独立的VPN服务实现翻墙，首先需要有能够访问国外VPS，然后在VPS中安装VPN软件即可。
目前国外VPS比较好用的主要有Linode和DigitalOcean，其中DigitalOcean的性价比相对更高，
本文以DigitalOcean为例进行介绍。[DigitalOcean注册](https://www.digitalocean.com/?refcode=d127d9afd5f3)

### 1、注册DigitalOcean并创建VPS

进入官网[DigitalOcean](https://www.digitalocean.com/?refcode=d127d9afd5f3)后注册账户，邮件激活后需要填写信用卡信息或通过Paypal充值5刀，完成此步骤后很有可能发生账户被锁的情况，此时只需根据提示填写相关信息后等待处理。

解锁账户后（或者压根儿没被锁过）则可以直接创建droplet，创建时选择Ubuntu，机房选择新加坡或者旧金山，关于各机房速度的差别可以自行创建多个droplet进行测试。

>如果是学生用户，可以使用github提供的100刀的优惠券，[点击这里](https://education.github.com)

### 2、VPS初始化配置

初始创建的VPS可以直接通过root账户登录（登录密码会发送到注册时的邮箱中），然而在使用Linux的过程中通常不直接使用root账户，以免造成一些安全问题，因此当我们用root账户登录之后首先新创建一个用户，通过下面的命令则可以添加用户。

``` bash
$ adduser username
```

在Ubuntu中可以通过使用`sudo`命令来使普通用户进行需要root权限的操作，因此我们把创建的用户添加到`sudo`用户组。

``` bash
$ gpasswd -a username sudo
```

至此一个具有`sudo`权限的账户则创建成功了，我们退出root账户，改用刚刚创建的用户进行登录。另外，由于VPS处于公网，最好是通过公钥秘钥的方式来登录机器从而避免用户密码泄露等安全问题，在mac机器上生成ssh密钥对直接通过以下命令即可。

``` bash
$ ssh-keygen  #可以直接回车都默认信息即可
```

将 `~/.ssh/id_rsa.pub`中的内容复制到VPS机器上的`~/.ssh/authorized_keys`文件中，并将`authorized_keys`文件的权限改为400：`chmod 400 ~/.ssh/authorized_keys`，至此则可以通过ssh无密码登陆VPS：`ssh username@hostname`了。

将文件 `/etc/ssh/sshd_config`中的`PasswordAuthentication`后面的yes修改为no，并去掉前面的注释#。重启ssh服务 `sudo service ssh restart`之后则不能通过用户名密码的方式登陆了。这样设置之后如果ssh密钥对丢失，仍然可以通过DigitalOcean提供的VNC方式登陆机器，设置新的密钥对。

>推荐一个十分强大的bas：[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)，可以通过以下命令安装。
>
>sudo apt-get install zsh git
>
>git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
>
>cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
>
>chsh -s /bin/zsh
>
>至此重新开启一个会话则可以使用zsh


### 3、安装pptpd服务

在DigitalOcean上安装配置pptpd的VPN服务，[参考这里](http://www.zhihu.com/question/20113381)

#### 安装pptpd（VPN服务器）和ufw（防火墙）软件
执行以下命令即可。

``` bash
$ sudo apt-get install pptpd ufw
```


#### 修改ufw规则
添加需要的端口 22、1723，其中22端口为ssh登录端口，1723为pptpd服务的默认端口号。

``` bash
$ sudo ufw allow 22
$ sudo ufw allow 1723
$ sudo ufw enable
```
如果VPS还有其他的端口需要使用，则同样需要添加对应的端口。同时需要将文件`/etc/sysctl.conf`中对应行的注释去掉，以支持ipv4的转发，如下所示。

```
net.ipv4.ip_forward=1
```

编辑`/etc/default/ufw`文件，修改如下。将文件中的`DEFAULT_FORWARD_POLICY = "DROP"`修改为`DEFAULT_FORWARD_POLICY = "ACCEPT"`

在文件`/etc/ufw/before.rules`的开头处添加如下内容：

```
# NAT table rules
*nat

:POSTROUTING ACCEPT [0:0]
# Allow forward traffic to eth0 开启ipv4转发
# 此处的192.168。1.0/24 需要与前面设置的remoteip对应
-A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# Process the NAT table rules
COMMIT
```


#### 编辑pptpd相关配置

打开文件`/etc/ppp/pptpd-options`，修改如下：

```
# 注释掉以下三行
refuse-pap
refuse-chap
refuse-mschap

# 修改DNS
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

打开文件`/etc/pptpd.conf`，配置服务器和客户端的IP地址，修改如下：

```
localip vps-ip # 此处为VPS服务器的公网IP
remoteip 192.168.1.234-238,192.168.1.245 #此IP为分配给VPN客户端的可用ip地址
```

打开文件`/etc/ppp/chap-secrets`，配置VPN服务的用户名和密码，一行代表一个用户，格式如下：

```
# [username][service][password][allow-ip]
test pptpd passwd *
```


以上配置修改完成后，重启pptpd和ufw服务，使用如下命令：

``` bash
$ sudo sysctl -p      # 使系统ip转发功能生效
$ sudo ufw disable    # 
$ sudo ufw enable     # 重启ufw服务
$ sudo service pptpd restart # 重启pptpd服务
```

### 4、mac设置VPN连接

按照以上方式配置完成pptpd服务后，如无意外，则可以进行科学上网了。在MAC下可以通过如下方式添加VPN配置


打开`网络偏好设置`->`点击左下角的加号+创建一个新服务`->`添加VPN配置`填写相关信息，最后在VPN配置页面选择`高级`中`转发所有数据包`。Enjoy！！

