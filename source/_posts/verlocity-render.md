---
title: velocity模板渲染之变量替换
date: 2016-03-09 11:53:49
tags: velocity spring
category: spring
---

velocity是在`spring mvc`中，使用十分普遍的一种模板渲染引擎，

### 现象
* VelocityView配置`allowSessionOverride`和`allowRequestOverride`均为默认的false，即不把session和request中的属性同model中数据做合并。
* 在Controller中往session和modelMap中分别设置属性`addAttribute`，使用相同的key，vm模板中的key对应的变量最终渲染得到的值是session中的值

### demo
``` java
@Controller
public class VelocityRenderController {

    public static String TOKEN         = "_token";

    public static String TOKEN_NEW     = "token-new";

    public static String TOKEN_OLD     = "token-old";

    @RequestMapping(value = "/velocity", method = RequestMethod.GET)
    public void doGet(HttpServletRequest request, ModelMap modelMap) {

        HttpSession session = request.getSession();

        session.setAttribute(TOKEN, TOKEN_OLD);

        modelMap.addAttribute(TOKEN, TOKEN_NEW);

        modelMap.addAttribute("hello", "world2");
    }
}
```

### 原因分析
* Spring请求处理

    当一个http请求到达服务端后，通过`org.springframework.web.servlet.DispatcherServlet`对请求进行分发处理，在调用handler处理一次请求之后得到结果ModelAndView
* Velocity模板渲染
    * 当`controller`处理完一个请求之后，使用`org.springframework.web.servlet.DispatcherServlet#processDispatchResult`对结果进行处理
    * 如果没有错误则调用`org.springframework.web.servlet.DispatcherServlet#render `方法进行模板渲染，最终的渲染逻辑是从ModelAndView中获取View对象并使用其render方法`org.springframework.web.servlet.View#render`进行渲染
    * 在渲染时首先通过`com.alipay.web.mvc.view.velocity.DefaultVelocityLayoutView#createVelocityContext`方法创建`velocity`的上下文，如下图：
    <img src="http://7xrny8.com1.z0.glb.clouddn.com/blog/1457495819616.png" width="624"/>

    在`new VelocityContext`的时候将modelMap中的数据存储到了context中，但是创建ChainedContext时,给context属性传入了空的数据，而将VelocityContext对象赋值给了innerContext属性，如下图：
    <img src="http://7xrny8.com1.z0.glb.clouddn.com/blog/1457495842082.png" width="626"/>

    至此，构造了velocity渲染的上下文对象，即当前对象context属性中没有model数据，而model数据存放在innerContext的属性中
    * 到模板进行渲染的时候，针对vm模板中的变量，通过`org.apache.velocity.runtime.parser.node.ASTReference#getVariableValue`方法获取变量的值,如下图：
    <img src="http://7xrny8.com1.z0.glb.clouddn.com/blog/1457495862871.png" width="629"/>

    * 最关键的地方来了
    <img src="http://7xrny8.com1.z0.glb.clouddn.com/blog/1457495883658.png" width="627"/>
    <img src="http://7xrny8.com1.z0.glb.clouddn.com/blog/1457495920117.png" width="596"/>

    在`getVariableValue`方法中，context属性是我们上面构造的`ChainedContext`类型的对象，首先使用了`internalGet`方法从当前对象的context属性来获取key对应的value，此时由于当前对象是ChainedContext，且在创建的时候context属性为空，所以通过key获取的value也是为空，这样就需要从request和session中去获取key对应的值，此时如果从session中获得了value，那么就不会从innerContext对象中去获取值了，还记得我们前面在构造VelocityContext时把model中的属性值设置到了innerContext中了。因此，VM模板中的变量渲染的值就是session中设置的值了。