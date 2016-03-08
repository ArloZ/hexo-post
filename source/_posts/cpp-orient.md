---
title: 'C++面向对象简记'
date: 2014-09-09 10:18:32
category: 数据结构与算法
tags: C++ OOP
---

面向对象程序设计（OOP）以对象的概念为中心，对象封装了数据和对数据的操作，对象之间通过消息来相互通信，
基于对象的结构化程序能够达到几个目的：

* 1、这种数据和操作之间的强耦合关系是一种对现实世界的仿真，是对现实的一种建模；
* 2、对象更容易debug，因为对象的封装性使得操作都局限在对象内部，容易跟踪；
* 3、对象可以隐藏对数据的操作。

C++是一种面向对象语言，具有很多面向对象的特性，以下做一些简单的记录。___才疏学浅，
理解有误的地方还请指正___

### 1.类和对象

C++中的类通过关键字`class`来定义，类中定义的变量称为成员变量或属性，定义的函数称为成员函数或方法，
通过类来定义一个变量称为一个对象实例，类的方法可以通过实例来调用。
以下是一个简单的类和对象的定义、方法调用。

``` cpp
// 定义一个类
class A{
private:
    char name[32];
    int age;
public:
    A(char *s="",int i=0){
        strcpy(name,s);
        age = i;
    }
    void hello(){
        cout<<name<<" is "<<age<<" years old."<<endl;
    }
};

// 定义类的对象实例
A obj1("Bob",12),obj2("Lucy",13);

// 调用类方法
obj1.hello();
obj2.hello();
```

### 2.继承

OOL允许创建类的层次关系，所以一个对象不一定是单一类的实例，如下代码。
``` cpp
/*
 * 类的继承关系
  */
class Base{
    private:
        int id;
    protected:
        int age;
    public:
        char name[32];
    public:
        Base(){}
        Base(char* s="", int i=0,int ii=1){
            strcpy(name,s);
            age = i;
            id = ii;
            cout<<"Base construct"<<endl;
        }
        void hello(){
            cout<<"Base hello: "<<name<<" is "<<age<<" years old."<<endl;
        }
};
class Student:protected Base{
    private:
        char school[32];
    public:
        Student(char* s="",int i=0, int ii=1,char* sch=""):Base(s,i,ii){
            strcpy(school,sch);
            cout<<"Student construct"<<endl;
            Base::hello();
        }
        void hello(){
            cout<<"Student hello: "<<name<<" is "<<age<<" years old.";
            cout<<" in school: "<<school<<endl;
            //cout<<"the id is: "<<id<<endl; // ERROR: id is a private member of Base
        }
};
class Worker:public Base{
    private:
        char companay[64];
    public:
        Worker(char* s="",int i=0, int ii=1,char* com=""):Base(s,i,ii){
            strcpy(companay,com);
            cout<<"Worker construct"<<endl;
        }

        void hello(){
            cout<<"Worker hello: "<<name<<" is "<<age<<" years old.";
            cout<<" in companay: "<<companay<<endl;
        }
};
// 类的继承
Student s("studentA",23,10000,"tju");
Worker w("workerB",32,10001,"baidu");
w.hello();
s.hello();
cout<<w.name<<endl;
//cout<<s.name<<endl;// ERROR: Student is protected extend from Base class

```

类Base称为基类或超类，其他的类称为子类或派生类，因为他们都是从基类继承而来，在从基类继承的时候可以指定为
protected或者public继承，子类继承了基类的所有数据成员和成员函数，不需要重复定义，同时也可以定义自己的
数据成员和成员函数。类中的成员(数据成员和成员函数)的访问限制如下：

* public成员：类自身、子类和类的对象实例可以访问
* protected成员：类自身和子类可以访问
* private成员：仅类自身可以访问

子类继承父类之后，父类的成员的private、protected、public修饰的变化：

* public继承：父类的所有成员保持原修饰不变
* protected继承：父类的public修饰的成员变为protected修饰，其他的保持不变
* private继承：父类的所有成员变为private修饰

### 3.指针

形象的我们可以把程序中的变量看做一个不为空的盒子，在定义时由程序员填上一些内容或者由系统填上默认的内容。
这样一个变量就至少具有了两种属性：内容（变量的值）和位置（变量的内存地址），内容可以是数字、字符或复杂的数据结构，
当然内容也可以是另一个变量的位置，具有这种内容的变量就称为指针。
举个例子，声明变量如下：

``` cpp
int i = 15, j, *p, *q;
```

其中i,j都是数值型的变量，p,q都是指向数值的指针。将变量i的地址赋值给指针p：`p = &i`，此时p指向变量i在内存中的存储地址，
该地址处存储的值为15，同时可以给指针q赋值：`q = p`，此时将p指向的地址赋值给了q，这样q和p指向同样的位置。通过指针修改
其指向变量的值可以：`*p = 20`，此时变量i的值为20。

上面指针是指向了一个整数变量内存块，同样指针也可以指向一个动态创建的数据结构。在C++中数组的大小是必须提前定义的，如：
`int a[5],*p;`。有很多情况下是不能够提前知道数组大小的，这样就需要能够在程序运行时动态的开辟数组的大小，在C++中可以
使用new关键字，如：`int* a = new int[n]`，这样分配了n个int空间的数组，当不再使用该数组时需要释放分配的存储空间，如：
`delete [] p;`，这里的括号表示p是指向一个数组。

当将数据从一个对象复制到另一个对象的时候，包含指针数据成员的需要做一些特殊的处理，否则就会有一些问题。如下定义：

``` cpp
/*
 * 指针和复制构造函数
 */
struct Node{
    char *name;
    int age;
    Node(char* s="",int a=0){
        name = strdup(s);
        age = a;
    }
};

// 指针和复制构造函数
Node node1("kobe",20),node2(node1);
cout<<"node1 "<<node1.name<<" "<<node1.age<<endl;
cout<<"node2 "<<node2.name<<" "<<node2.age<<endl;
// output
// node1 kobe 20
// node2 kobe 20
strcpy(node1.name,"john");
node1.age = 23;
cout<<"node1 "<<node1.name<<" "<<node1.age<<endl;
cout<<"node2 "<<node2.name<<" "<<node2.age<<endl;
// output
// node1 john 23
// node2 john 20

```

由上可知，改变了node1的name值的同时也改变了node2的name的值，这是因为在定义node2的时候`Node node2(node1);`，将node1的数据成员name的值（也就是指向name字符串的地址）赋值给了node2的name，这样node1和node2的数据成员name则指向了同样的内存地址，当修改node1的name指向的字符串时`strcpy(node1.name,"john")`，node2的name指向的值也随着改变了。要解决这个问题就需要在Node的构造函数中做一些修改，如下，这样node1复制给node2的数据成员name就是其指向的值而不是其保存的地址。注意，常见的赋值操作`=`,也存在这样的问题，因此需要重载赋值操作符。同样，对于指针数据成员，类的析构函数也需要做相应的处理，在析构函数中需要调用`delete`删除指针数据成员所指向的内存。

``` cpp
struct Node{
    char *name;
    int age;
    Node(char* s="",int a=0){
        name = strdup(s);
        age = a;
    }
    // 构造函数的指针复制
    Node(const Node &n){
        name = strdup(n.name);
        age = n.age;
    }
    // 指针数据成员的赋值操作
    Node& operator = (const Node& n){
        if(this != &n){ // no assignment to it self
            if(name != null)
                delete [] name;
            name = strdup(n.name);
            age = n.age;
        }
        return *this;
    }
};
```

### 4.多态

多态性指同样的函数名称表示很多的函数，这些函数是不同对象的成员函数。如下例子：
``` cpp

/*
 * 多态性
 */
class ClassA{
    public:
        virtual void hello(){
            cout<<"hello in ClassA"<<endl;
        }
        void bye(){
            cout<<"bye in ClassA"<<endl;
        }
};
class ClassB{
    public:
        virtual void hello(){
            cout<<"hello in ClassB"<<endl;
        }
        void bye(){
            cout<<"bye in ClassB"<<endl;
        }
};
class ClassC{
    public:
        void hi(){
            cout<<"hi in ClassC"<<endl;
        }
};

// 多态性
ClassA obja,*p;
ClassB objb;
ClassC objc;

p = &obja;
p->hello();
p->bye();

p = (ClassA*) &objb;
p->hello();
p->bye();

p = (ClassA*) &objc;
//p->hello(); // ERROR: segmentation fault
p->bye();
//p->hi();// ERROR: no member named 'hi' in ClassA

```

分别声明了类ClassA、ClassB、ClassC的三个对象obja、objb、objc和一个ClassA类型的指针p，首先将obja对象地址赋值给指针p，
这样通过p调用函数hello和bye均为分别调用ClassA的函数成员，这也是常见的一种使用方式；其次将objb对象赋值给指针p，这样通过
p调用hello和bye函数时，由于hello函数是定义为virtual函数的，此时将根据当前p执行的对象类型来决定调用哪个类的方法，
此时p是指向的ClassB类型，所以调用ClassB中的hello，而由于bye函数不是虚函数，则其在编译时已经确定了，p声明时是ClassA类型，
所以调用ClassA中的函数bye；最后将objc对方赋值给指针p，此时由于ClassC中没有定义虚函数hello，当程序执行到此处时，找不到ClassC的hello函数
则会出现segmentation fault错误，调用bye函数则是在编译时已经确定的调用ClassA中的bye函数，调用ClassC中定义的hi函数时，
由于p指针是ClassA类型，所以在编译时已经确定了调用ClassA中的hi函数，由于ClassA没有定义改函数，则在编译会提示错误。

多态性是OOP中的一种强大且复杂的功能，通过该特性可以将同一个消息发送给很多不同的对象，而不需要指定如何处理消息，也不需要指定对象的类型。

### 5.标准模板库STL

C++是一种面向对象的程序设计语言，得到了广泛的应用，其中也包含了程序员最常用的一些数据结构和算法，
这个库就是[标准模板库STL](http://zh.wikipedia.org/wiki/标准模板库)。
这个库包含了3中通用的实体：容器、迭代器和算法。


___参考：《数据结构与算法----C++版(第三版)》，清华大学出版社___

