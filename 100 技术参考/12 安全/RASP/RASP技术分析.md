### RASP技术分析

[2018-01-03](http://blog.nsfocus.net/rasp-tech/)[陈涛](http://blog.nsfocus.net/author/chentao/)[RASP](http://blog.nsfocus.net/tag/rasp/), [WAF](http://blog.nsfocus.net/tag/waf/), [绿盟科技](http://blog.nsfocus.net/tag/绿盟科技/)

 阅读： 32,893

什么是RASP?RASP和WAF性能比较怎么样？RASP的实现思路是什么，都将在本文详细介绍

## RASP概述

### 什么是RASP

RASP（Runtime application self-protection）运行时应用自我保护，

RSAP将自身注入到应用程序中，与应用程序融为一体，实时监测、阻断攻击，使程序自身拥有自保护的能力。并且应用程序无需在编码时进行任何的修改，只需进行简单的配置即可

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/7195ace984937ccbddc171ece82e237e.png)

Ø运行在应用程序内部；

Ø检测点位于应用程序的输入输出位置；

Ø输入点包括用户请求、文件输入等；

Ø输出点包括包括数据库、网络、文件系统等。

### RASP能做什么

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/276c128e902ed117cf7992464eab999f.png)

### RASP *vs* WAF  <部署>

**WAF**

Ø外部边界入口统一部署

Ø支持透明(串联)、旁路、反向代理三种方式

Ø容易形成单点故障，影响面大

**RASP**

Ø服务器上单独部署，嵌入在应用程序内部，应用代码无感知

*java**程序，启动时加上**–**javaagent* *rasp.jar**参数即可*

Ø开发语言强相关，但防护插件可共用

Ø更了解应用程序上下文

### RASP *vs* WAF  <性能>

**WAF**

Ø正则匹配的规则越多，性能越低

Ø和硬件规格相关

Ø业务报文多了一次socket转发，延迟大

Ø对服务器CPU没有影响

**RASP**

Ø由于只在关键点检测，不是所有请求都匹配所有规则

Ø某些厂商声称的对服务器CPU性能影响能够降低到2%

Ø非防护状态延迟增大3-5%，防护状态延迟增大4.6 – 8.9%

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/8f7db1e979a4143a5eaabc7e21377f22.png)

### RASP *vs* WAF <产品特性>

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/a57a6de47b9031c1d923bf55d700505e.png)

### RASP *vs* WAF <检测能力>

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/45cfe44daa41bcf8f7deffa5c7fb6794.png)

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/f9616de940711a237397694018f81d0f.png)

（来自https://www.oneasp.com/topic/raspwaf.html）

### ![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/66860ae43346b77b33468a99e59f7552.png)RASP前景

“The RASP market size is expected to grow from USD 294.7 million in 2017 to USD 1,240.1 million by 2022, at a Compound Annual Growth Rate (CAGR) of 33.3% during the forecast period. “

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/d45e5fe61f81886a3b238c69f6e6e85d.png)

## RASP技术实现(JAVA)

### RASP实现思路

**如何****注入检测代码**

在哪些访问控制点注入

注入之后如何检测攻击

检测到攻击后如何处理

### RASP 注入方法(JAVA)

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/fca02f0c30846c349a6f3f35998724dd.png)

**Servlet Filter:** 在请求响应路径上，只能对http报文过滤处理

**JVM****重构****:** 植入JVM内部, 基于JVM的安全控制层实现RASP容器。需要对JVM非常熟悉，难度很大。国外waratek采用这种方法

**Java Instrument****:** 最普遍的做法

### Java基础: 源码、编译、运行

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/74396276217bda013f502e84a6f71083.png)

### 什么是Java Instrument

Java SE 5 的新特性，依赖于 JVMTI。

用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些类的定义

–javaagent 参数指定Instrumentation功能的 jar 文件来运行程序:

​    *java*  *–**javaagent:K:\java\MyAgent.jar*  *app.**HelloWorld*

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/5e930d7bace8364416a733f21252d2e7.png)

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/9c864c662e1892b174d609d93bba6b0b.png)

### 什么是JVMTI

Ø全名JVM Tool Interface，是JVM暴露出来的一些供用户扩展的本地编程接口集合。

Ø基于事件驱动的，JVM每执行到一定的逻辑就会主动调用一些事件的回调接口，这些接口可以供开发者扩展自己的逻辑。

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/ef21a6f1579eb5b1b26bd11884204152.png)

### Java Instrument原理

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/00218c2023a2ba140887543f88a4fd99.png)

### Intrument的实现方式

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/202162e0f3b0b4a51b33c834b1783d62.png)

### JDK反射

**概念**

Java在运行时识别对象和类的信息，有2种方式:

Ø传统的RTTI: 编译时确定某个class是否被JVM加载；

Ø反射机制: 在运行状态中，对于任意一个类(包括未加载的)，都能够知道这个类的所有属性和 方法；对于任意一个对象，都能够调用它的任意一个方法和属性；

借助这种对类结构探知的”自审”能力和多态特性，充分发挥Java的灵活性。

**使用**

Java提供了一个叫做reflect的库，封装了Method,Constructor,field,Proxy,InvocationHandler 等类

**功能**

Ø在运行时判断任意一个对象所属的类。

Ø在运行时构造任意一个类的对象。

Ø在运行时判断任意一个类所具有的成员变量和方法。

Ø在运行时调用任意一个对象的方法

Ø生成动态代理

### JDK反射 – 实例1

**动态加载**

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/e8bdac98bbf0b5f7269f727c7f723b19.png)

### JDK反射 – 实例2

**动态代理**

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/46b6b4a0a54d8fa214d1a8fea65d657f.png)

### JDK反射 – 结论

动态代理虽然灵活性高，但仍然需要使用相关的类库，进行动态代理的配置，并融合到应用的源代码中，不是理想的解决方案。

没有找到合适的方法在不修改应用代码的前提下，对已有类进行代理

Javassist

**概念**

一个开源的分析、编辑和创建Java字节码的类库。

由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建。它已加入了开放源代码JBoss 应用服务器项目，通过使用Javassist对字节码操作为JBoss实现动态”AOP”框架。

允许开发者自由的在一个已经编译好的类中添加新的方法，或者是修改已有的方法。

Java Instrumention的一种方式。

**优点**

简单，直接使用java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类。

**使用**

Javassist库，封装了ClassPool、CtClass、CtMethod等类;

### Javassist – 实例1

**动态构造类**

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/91a59ec81de45eaa46e0e9ed016c228f.png)

### Javassist – 实例2

**Instrument**

premain函数:

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/1364f38beb6a40acc1def73dfdea92c5.png)

transform函数:

如果是期望修改的类，找到类中期望被注入的函数，在函数前后添加检测函数(下图只是示意)。

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/ccd6c408fa79c4a774130bb531c7b189.png)

### ASM

**概念**

一个JAVA字节码分析、创建和修改的开源应用框架。

允许开发者自由的在一个已经编译好的类中添加新的方法，或者是修改已有的方法。

Java Instrumention的一种方式。

**优点**

可以用JVM指令直接操作字节码，更灵活；

性能高；

功能更丰富；

**使用**

ASM库/工具  http://asm.ow2.org/

封装ClassReader、ClassVisitor与ClassWriter等很多功能丰富的类

需要了解jvm指令和class文件结构

### ASM – 常用类

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/7913fb32bde28fb350af59c99b84b544.png)

### ASM – 类关系图

**典型的静态代理模式：**

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/b408133550b9c994d046c9087fa76965.png)

### ASM – 字节码解析流程

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/fe386c8cc83b2d9013899e1a64a8d4f2.png)

### ASM – 实例1

**动态构造类**

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/70380a9505fd56653b0406186437fcc2.png)

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/39f7c3d3bfdcc254d6029c051273a8ae.png)

### ASM – 实例2

**修改类**

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/5b3fbee880aa697e8bbbafdb75e3b17b.png)

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/1d26664653813c75926ea9b38e98832a.png)

### ASM – 利用Method的扩展类

![img](http://blog.nsfocus.net/wp-content/uploads/2018/01/6eab875721d3f4449e1f21a4c2b5aa14.png)



[RASP技术分析 – 绿盟科技技术博客 (nsfocus.net)](http://blog.nsfocus.net/rasp-tech/)