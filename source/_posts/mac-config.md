---
title: 'Mac开发环境配置'
date: 2015-09-08 22:40:19
category: 杂七杂八
tags: [config,mac]
---

主要用于记录一些MAC上的个人的开发环境及工具的配置，留着以后备用,避免每次都去google.

### 编辑器-Sublime Text 3

#### 个人的一些默认配置
* 通过cmd+,打开配置文件，配置内容如下：

```
// 使光标闪动更加柔和
"caret_style": "phase",

// 高亮当前行
"highlight_line": true,

// 高亮有修改的标签
"highlight_modified_tabs": true,

// The number of spaces a tab is considered equal to
"tab_size": 4,

// Set to true to insert spaces when tab is pressed
"translate_tabs_to_spaces": true,
```

#### 安装Sublime Text的包管理器
* [https://packagecontrol.io/installation](https://packagecontrol.io/installation)

#### 配置MarkDown编辑环境
* 配置MarkDown高亮
    * 安装`Monokai Extended`主题:`cmd+shift+p`呼出提示命令，安装之
    * 选择应用`Monokai Extended`主题：`Preference->Color Scheme->Monokai Extended`
* 配置Markdown Preview
    * 安装`Markdown Preview`插件：`cmd+shift+p`呼出提示命令，安装之mark
* 设置 MarkDown预览快捷键
    * 选择Preference->Key Binding-User
    * 输入如下内容：

    ```
    [
        { "keys": ["shift+super+m"], "command": "markdown_preview", "args": { "target": "browser"} }
    ]
    ```

#### SFTP插件

通过该插件可以很方便的将文件上传到服务上，免除了不停scp的痛苦

安装插件
* `cmd+shift+p`呼出提示命令，输入`install package`,然后搜索”sftp“即可
* 安装完成后会打开配置文件，在其中配置需要同步上传文件的服务器地址、目标路径、用户名密码等

GBK编码显示
* 插件安装时搜索`ConvertToUTF8`
* Sublime3有可能会遇到解码错误，此时需要安装`Codecs33`

#### Git插件

通过该插件可以在Sublime中直接使用git命令

安装插件

* 搜索安装GIT插件，此插件将git集成到了Sublime中
* `cmd+shift+p`呼出Sublime的命令行，然后输入GIT则可以看到能使用git命令了
* `GIT:add`:添加要提交的文件，`GIT：commit`:输入提交注释后关闭窗口即可自动提交......

#### 侧边栏管理文件
通过该插件可以直接对文件进行拖动、在finder打开等操作

* 通过control package安装即可，[SideBarEnhancements](https://github.com/titoBouzout/SideBarEnhancements)

#### HTML插件Emmet

* 可以用来快速的补全html，参考文档 [http://emmet.io/](http://emmet.io/)


## Intellij IDEA配置

#### 配置新建文件的文件头及类描述
* `CMD+,`->`File and Code Templates`设置includes和Templates

#### 添加getter、setter注释

* 在需要添加getter和setter的源文件中：`ctrl+enter`弹出菜单选择`getter`，然后在弹出对话框中添加一个getter模板（复制default模板的内容），新增如下内容：

    ```
    /**
    * Getter method for property <tt>${field.name}</tt>.
    *
    * @return property value of ${field.name}
    */
    public ##
    #if($field.modifierStatic)
      static ##
    #end
    $field.type ##
    #set($name = $StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project))))
    #if ($field.boolean && $field.primitive)
      #if ($StringUtil.startsWithIgnoreCase($name, 'is'))
        #set($name = $StringUtil.decapitalize($name))
      #else
        is##
    #end
    #else
      get##
    #end
    ${name}() {
      return $field.name;
    }
    ```

* 新增一个setter模板，方法同getter，添加内容如下：

    ```
    /**
    * Setter method for property <tt>${field.name}</tt>.
    *
    * @param ${field.name} value to be assigned to property ${field.name}
    */
    #set($paramName = $helper.getParamName($field, $project))
    public ##
    #if($field.modifierStatic)
      static ##
    #end
    void set$StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project)))($field.type $paramName) {
      #if ($field.name == $paramName)
        #if (!$field.modifierStatic)
          this.##
        #else
          $classname.##
        #end
      #end
      $field.name = $paramName;
    }
    ```

#### 自动生成serialVersionUID

* 启用该项检查功能
    Setting->Inspections->Serialization issues->Serializable class without ’serialVersionUID’

* 在class定义处按 `alt+enter` 即可自动生成

在Tab中标识出修改过的文件

* setting -> Editor -> editor Tabs -> Mark Modified Tabs...

## Mac 配置Java1.6开发环境

#### JDK1.6

由于Oracl官方提供mac版本jdk是从1.7开始的，而在项目中需要使用jdk1.6，因此需要通过apple官方提供的jdk进行安装。

* apple官方jdk1.6下载并安装[地址](https://support.apple.com/kb/DL1572?locale=zh_CN)。
* 设置JAVA_HOME环境变量，Mac中通过`/usr/libexec/java_home`命令能够进行java_home的管理，
如果添加参数`-v 1.6` 可以得到jdk1.6的路径，默认输出1.8版本的jdk路径（如果有安装）。
* 验证JDK安装：

    ```
    $java -version
    java version "1.6.0_65"
    Java(TM) SE Runtime Environment (build 1.6.0_65-b14-466.1-11M4716)
    Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-466.1, mixed mode)
    ```

#### Maven

想要在mac安装maven版本2.2.1，可以通过brew安装此版本，`brew install homebrew/versions/maven2`

* 验证Maven安装：
    
    ```
    $mvn -version
    Apache Maven 2.2.1 (r801777; 2009-08-07 03:16:01+0800)
    Java version: 1.6.0_65
    Java home: /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
    Default locale: zh_CN, platform encoding: EUC_CN
    OS name: "mac os x" version: "10.10.3" arch: "x86_64" Family: "mac"
    ```

## 配置淘宝ruby镜像
[http://ruby.taobao.org/](http://ruby.taobao.org/)

## 配置git
* git config --global push.default simple
* git config --global alias.st status
* git config --global alias.co checkout
* git config --global user.name "xxx"
* git config --global user.email "xxx"
* git config --global core.editor /usr/bin/vim `解决通过vi提交时报错：error: There was a problem with the editor 'vi'.`
    参见：[http://blog.csdn.net/forever_crying/article/details/8641545](http://blog.csdn.net/forever_crying/article/details/8641545)

## iTerm2配置自动ssh登录服务器免输密码
通常是使用ssh登录服务器的时候是需要密码的，但是作为懒人的我们的是不喜欢总是输入密码，之前使用securitCRT的时候是可以保存密码的，但是转成iTerm2之后没有保存密码的功能了，这样需要使用脚本来实现了，一段expect脚本如下：

```
#!/usr/bin/expect

# args check
if {$argc != 3} {
    puts "-------------------------------------"
    puts "Usage:"
    puts "    auto input password when ssh host"
    puts "    sshexpect.sh user host pwd"
    puts "-------------------------------------"
    exit 0
}

set timeout 30
spawn ssh [lindex $argv 0]@[lindex $argv 1]
expect {
    "(yes/no)?"
    {send "yes\n";exp_continue}
    "password:"
    {send "[lindex $argv 2]\n"}
}
interact
```

将该段脚本放置在`/usr/local/bin/`目录下之后，就可以使用命令`login.sh username hostname pwd`的方式来登录了。

**当然如果是经常登录同一台服务器的话，可以直接使用ssh 公钥的方式实现啦。**

还有一种情况就是登录服务器的时候密码是通过token校验的，每次登录时token都是不相同的，那这样就只能输入密码了，而不能使用上面的自动登录脚本，但是如果需要开多个终端tab来登录同一个服务器的话，作为懒人的我们的当然是不愿意输入多次密码的啦。这就需要类似securitCRT中复制会话的功能啦，要实现这个功能只需要创建`~/.ssh/config`文件[参考链接](http://linux.die.net/man/5/ssh_config)，并在文件中作如下配置即可：
```
host *
ControlMaster auto
ControlPath ~/.ssh/master-%r@%h:%p
```
这样配置以后，只要是登录同样hostname的服务器，就可以自动copy当前建立的soket而不需要再次输入密码了。如果这样配置，再加上使用前面的自动登录脚本，会出现卡住的情况，原因很简单，前面的自动登录脚本是等待终端输出类似password字样，然后输入密码，配置了`~/.ssh/config`之后，是直接复制会话登录，就不会出现再次需要输入密码的情况了，所以导致卡住，这种情况可以不配置`host *`而配置成`host hostname`的形式，指定特定的机器复制会话，而其他的机器使用原来的登录脚本。


