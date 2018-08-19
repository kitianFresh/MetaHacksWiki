---
title: "program-paradigm"
date: 2018-08-18 11:38
---

# 编程理念
## IOC
IOC(Iversion-of-Control) 控制反转，在IoC出现以前，组件之间的协调关系是由程序内部代码来控制的，或者说，以前我们使用New关键字来实现两组间之间的依赖关系的。

这种方式就造成了组件之间的互相耦合。这个主要是在对象组合的时候存在耦合，比如类A需要依赖类B的功能，一般会通过一下方式去依赖B, 比如定义一个B属性，或者传递B参数等。
<img src="/static/images/AComputingThinking/program-paradigm/di-types.png" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **DI** </u></font> </center></caption>

但是这种方式耦合度过高IoC(控制反转)就是来解决这个问题的，它将实现组件间的关系从程序内部提到外部容器来管理。也就是说，由容器在运行期将组件间的某种依赖关系动态的注入组件中。
IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。实现原理就是利用Java的反射机制，配置好需要的类之后，可以直接调用容器方法根据配置文件动态的把需要的类以及方法加载和组装起来提供给其他对象。这样就从代码层面实现了类的解耦。



## AOP
AOP(Aspect-oriented programming), 面向切面编程，这个概念是应该是从OOP(Obeject-oriented programming)延伸出来的概念，是对OOP不足的改善和辅助。面向对象的基本思维就是对现实世界的物理实体建模，把实体对象的属性和行为进行封装，从而得到类。类还可以通过继承来实现功能复用，也可以说是代码复用，进一步通过多态实现一个接口函数在运行时判断不同行为的特性。
这样来看，类之间除了通过继承产生联系，还可以通过组合产生联系，但是组合过多就会产生耦合，为了弥补这种不足，将多个类都可能用到的类（功能、代码）抽象出来，以动态插入的形式插入某个切面。

上面可能无法理解，但是如果学习过Python的装饰器的话，就容易理解了，其实Python装饰器就是切面编程的一种具体实现。在需要用到装饰器的切面，插入即可。例如，Web应用中，会写多个WebHandler类来处理不同的业务逻辑，你可以把这些类看成纵向的，然后他们在处理某个请求的开始，有一个共同的部分，请求验证。但是有些请求又不一定需要验证，那么，就可以在该切面插入验证逻辑代码。这也是切面的名称由来吧。在Python里面这个功能相当简单，只需要加入验证装饰器接口。

<img src="/static/images/AComputingThinking/program-paradigm/aop-example.png" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **AOP-EXMAPLE** </u></font> </center></caption>

本质上来说，和[Python里面的装饰器](http://blog.hacksmeta.com/2017/04/24/python-decorator-1/#more)类似，都是为了复用代码，而且是动态的插入代码来执行，可以说Python装饰器是AOP思想的一种具体实现方式之一。

### 参考
 - [Spring IOC原理](https://blog.csdn.net/mdcmy/article/details/8542277)
 - [IoC与DI详解](https://blog.csdn.net/Baple/article/details/53667767)
 - [Inversion of Control: Overview with Examples](https://www.codeproject.com/Articles/380748/Inversion-of-Control-Overview-with-Examples)