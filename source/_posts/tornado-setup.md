---
title: 'Tornado搭建stepByStep'
date: 2015-07-25 11:01:30
category: install&configure
tags: tornado
---

由于需要在本地搭建一个轻量级的web服务，用来提供一些Web页面小工具，最近工作一直使用java，但是又不想落下Python，所以选择Tornado来进行这个小工具页面的服务的开发，至少保持不对Python生疏。下面进入正题，StepByStep的搭建一个Tornado服务，详情参见[官方文档](http://www.tornadoweb.org/en/stable/)。

### 下载安装最新版本的Tornado

* 通过命令安装`easy_install tornado`，简单方便
* 下载源码安装：[源码](https://pypi.python.org/packages/source/t/tornado/tornado-4.2.1.tar.gz)，解压后执行`sudo python setup.py install`即可

### 简单粗暴HelloWorld

* 新建一个工程目录及文件
``` python
mkdir mytool
cd mytool
touch server.py
```

* 在`server.py`文件中输入以下内容
``` python
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")

application = tornado.web.Application([
    (r"/", MainHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.current().start()

```

* 运行服务
通过命令`python server.py`执行，然后在浏览器访问`http://localhost:8888/`即可看到helloworld了。

至此一个简单的Tornado服务已经搭建好了，是不是特别简单~~That's it！

### Tornado简单介绍

从上面的Helloworld程序来看，首先我们创建了一个继承自`tornado.web.RequestHandler`的类`MainHandler`，这个类就是用来处理HTTP请求的，在`MainHandler`类中定义了一个get方法，直觉告诉我们，这就是用来处理Http的GET请求的，你还可以定义个post方法，用来处理HTTP的POST请求。

然后我们定义了一个`Application`对象，并且传入了一个参数（列表中仅包含一个元组），这个参数的含义就是将HTTP的请求的`/`目录映射给`MainHandler`，也就是说，所有的根目录请求都由`MainHandler来处理`。

最后我们让`application`监听8888端口，然后启动Tornado，想了解IOLoop请[移步](http://www.oschina.net/question/565065_79182)。至此Tornado将会监听8888端口的请求，当收到请求时就将其分发给`RequestHandler`来处理。

### 增加一些功能

helloworld确实十分简单粗暴，很容易就写出了自己的web服务，是不是很嗨森，但是在一步步往web服务里面增加功能的时候，你一会需要下面的这些简单方便的功能。

#### Tornado模板
在前面的HelloWorld中我们直接在Python代码里面输出了字符串到页面上，但是通常来说我们的页面不会简单到只有那么短的字符串，这个时候我们就可以把每一个页面写成HTML文件，在处理请求的时候把处理的数据填充到我们的HTML文件中，再返回这个文件就行，这个时候就需要Tornado的模板系统了。
所谓模板就是我们写的HTML文件，处理请求时通过模板系统将数据和模板渲染成最终的HTML页面返回给浏览器。在模板中可以使用跟Python相同的`if,for,while,try`等语句。如下示例：
``` html
<html>
   <head>
      <title>{{ title }}</title>
   </head>
   <body>
     <ul>
       {% for item in items %}
         <li>{{"{{ item "}}}}</li>
       {%end%}
     </ul>
   </body>
 </html>
```
上面的`{%raw%}{{}}{%endraw%}`标签是指输出python中的变量title，`{%raw%}{% for item in items %}{%endraw%}`,`{%raw%}{% end %}{%endraw%}`语句块是循环变量python列表items的每一个元素，并输出一行，把上面的内容保存为template.html，然后在相同目录下就可以通过下面的方式来渲染模板文件了：

``` python
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        items = ["Item 1", "Item 2", "Item 3"]
        self.render("template.html", title="My title", items=items)
```

另一方面咱们常见的网页通常都是有页面的头部、内容、尾部组成，头部和尾部在同一网站多个不同内容页面基本是保持一致不怎么变化的，那么从设计的角度，这些公共的代码部分最好是能够通用，Tornado模板也提供了这样的功能。如下示例：

首先我们创建页面的头部模板：header.html

``` html
<p>
    this is header....
</p>
```

然后创建尾部模板：footer.html

``` html
<p>
    this is footer....
</p>
```

然后创建一个最基本的base.html模板文件，其中通过`include`标签引入了header.html和footer.html两个文本文件，那么在渲染的时候则会将头部和尾部模板添加到最终的结果中。通过`block`标签定义了可以在后续的模板中来填充的部分

``` html
<!DOCTYPE html>
<html>
<head>
    <title>My Tools</title>
</head>
<body>
    {%include header.html%}

    {%block content%}
    {%end%}

    {%include footer.html%}
</body>
</html>
```

最后我们定义一个我们需要最终渲染的页面：index.html

``` html
{%extends base.html%}

{%block content%}
<p>
    This is content
</p>
{%end%}
```

在index.html文件中，通过`extends`标签“继承”了base.html模板，然后通过`block`标签来填充base.html定义时预留的内容块，最后通过下面的语句渲染index.html模板得到最终在页面显示的html文档。

``` python
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render('index.html')
```

以上所有的模板文件和python文件都必须位于同一目录，这样当然很不便于文件的管理，因此自定义application的时候可以指定模板文件的路径。

#### 静态资源文件
一个web服务处理每一次的动态处理用户请求之后还有很多静态资源，比如javascript文件、css文件、icon文件等等，他们只需要加载到浏览器中，由浏览器渲染即可，而不需要在服务器上做任何的处理。通常来说这些静态文件都有专门的静态资源服务器（如nginx、Apache）提供，但在我们的这个简单的web服务中还不需要去配置他们，可以直接使用Tornado内置的专门进行静态资源管理的功能，使用他们很简单，只需要配置一个`StaticFileHandler`，然后通过正则表达式指定哪些请求使用静态资源服务即可，如下：

``` python
class Application(tornado.web.Application):
    def __init__(self):
        settings = dict(
            template_path = SETTINGS.TEMPLATE_DIR,
            static_path = SETTINGS.STATIC_DIR,
            debug = SETTINGS.DEBUG,
        )
        handlers = [
            (r"/",MainHandler),
            (r"/static/(.*)",tornado.web.StaticFileHandler,dict(path=settings['static_path'])),
        ]
        tornado.web.Application.__init__(self, handlers, **settings)


def main():
    app = Application()
    app.listen(8888)
    tornado.ioloop.IOLoop.instance().start()

if __name__ == "__main__":
    main()
```

在模板文件中引用静态资源：
``` html
<script src="{{static_url('js/jquery-1.10.2.min.js')}}"></script>
```

这里与Helloworld不同，我们定义了`Application`类继承自`tornado.web.Application`，然后在初始化的时候进行了一些配置，如`template_path`模板文件路径，`static_path`静态资源文件的路径，然后在URL映射的地方配置了所有`static`开头的请求都使用静态文件资源来处理。最后在html模板文件中通过`static_url`标签来引用静态资源，这个标签会在静态资源请求链接中加上一个版本号，如果没有修改静态资源文件，版本号就不会改变，那么浏览器在请求该资源的时候会优先从本地缓存中获取，当缓存中获取不到时，才从服务请求。

至此一个简单的基于Tornado的web服务就搭建好了，下面就开始我的小工具功能开发啦 ~~

如有错误，欢迎指正。



