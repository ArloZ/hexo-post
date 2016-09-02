# Head First设计模式

### 设计模式

* 策略模式：定义算法族，并封装起来，让他们之间可相互替换，让算法的变化独立于使用算法的客户
* 观察者模式：定义了对象之间的一对多的依赖，这样当一个对象改变状态时，它的所有依赖都会收到更新通知，可以以报纸的出版者、订阅者比拟。
* 装饰者模式：动态的将责任附加到对象身上，想要扩展功能，装饰者提供有别于继承的另一种选择。
* 工厂方法模式：定义一个创建对象的接口，由子类决定要实例化的类是哪一个，工厂方法让类的实例化推迟到子类。
* 抽象工厂模式：定义一个接口，用于创建相关或依赖对象的家族，而不需要指定具体的类。
* 单件模式：确保一个类只有一个实例，并提供一个全局的访问点
* 命令模式：将请求封装成对象，以便使用不同的请求、队列或日志来参数化其他对象，命令模式也支持可撤销的操作。
* 适配器模式：将一个类的接口转换成客户期望的另一个接口，适配器让原本接口不兼容的类可以合作无间。
* 外观(facade)模式：提供一个统一的接口用来访问子系统中的一堆接口，外观定义了一个高层次的接口，使得子系统更容易使用。
* 模板方法模式：在一个方法中定义一个算法的骨架，而一些步骤延迟到子类中，模板方法使得子类可以在不改变算法结构的情况下重新定义算法的某些步骤。
* 迭代器模式：提供一种方法顺序的访问聚合对象中的每一个元素，而不对外暴露元素在内部的表示
* 组合模式：允许将对象组合成树形结构来表现“整体/部分”层次结构，组合能让客户使用统一的方式来处理对象以及对象组合。
* 状态模式：允许对象在内部状态改变时改变他的行为，对象看起来好像修改了他的类
* 代理模式：为另一个对象提供一个替身或占位符，以控制对这个对象的访问

### 设计原则

* 找出应用中可能变化之处，把他们独立出来，不要和不需要变化的代码混在一起
* 针对接口编程，而不是针对实现
* 多用组合，少用继承
* 为了交互对象之间的松耦合设计而努力
* 类应该对扩展开放，对修改关闭
* 要依赖抽象，不要依赖具体类
* 最少知识原则：之和你的密友谈话
* 好莱坞原则：别调用我们，我们会调用你的
* 单一责任原则：一个类应该只有一个引起这个类变化的原因