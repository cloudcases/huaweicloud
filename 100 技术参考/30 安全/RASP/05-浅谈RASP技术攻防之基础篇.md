# 浅谈RASP技术攻防之基础篇

[安百科技 ](https://www.freebuf.com/author/安百科技) 2019-03-18 08:30:53 1126577 1

## 引言

**本文就笔者研究RASP的过程进行了一些概述，技术干货略少，偏向于普及RASP技术。中间对java如何实现rasp技术进行了简单的举例，想对大家起到抛砖引玉的作用，可以让大家更好的了解一些关于web应用程序安全防护的技术。文笔不好，大家轻拍。**

## 一 、什么是RASP？

在2014年的时候，Gartner引入了“Runtime application self-protection”一词，简称为RASP。它是一种新型应用安全保护技术，它**将保护程序像疫苗一样注入到应用程序中，应用程序融为一体，能实时检测和阻断安全攻击，使应用程序具备自我保护能力，当应用程序遭受到实际攻击伤害，就可以自动对其进行防御，而不需要进行人工干预**。

RASP技术可以快速的**将安全防御功能整合到正在运行的应用程序中，它拦截从应用程序到系统的所有调用，确保它们是安全的，并直接在应用程序内验证数据请求**。Web和非Web应用程序都可以通过RASP进行保护。该技术不会影响应用程序的设计，因为RASP的检测和保护功能是在应用程序运行的系统上运行的。

## 二、RASP vs WAF

很多时候大家在攻击中遇到的都是基于流量规则的waf防御，waf往往误报率高，绕过率高，市面上也有很多针对不同waf的绕过方式，而RASP技术防御是根据请求上下文进行拦截的，和WAF对比非常明显，比如说：

攻击者对url为http://http.com/index.do?id=1进行测试，一般情况下，扫描器或者人工测试sql注入都会进行一些sql语句的拼接，来验证是否有注入，会对该url进行大量的发包，发的包可能如下：

```
http://xxx.com/index.do?id=1' and 1=2--
```

但是应用程序本身已经在程序内做了完整的注入参数过滤以及编码或者其他去危险操作，实际上访问该链接以后在数据库中执行的sql语句为：

```
select id,name,age from home where id='1 \' and 1=2--'
```

可以看到这个sql语句中已经将单引号进行了转义，导致无法进行，但是WAF大部分是基于规则去拦截的（也有小部分WAF是带参数净化功能的），也就是说，如果你的请求参数在他的规则中存在，那么waf都会对其进行拦截（上面只是一个例子，当然waf规则肯定不会这么简单，大家不要钻牛角尖。），这样会导致误报率大幅提升，但是R**ASP技术可以做到程序底层拼接的sql语句到数据库之前进行拦截**。也就是说，**在应用程序将sql语句预编译的时候，RASP可以在其发送之前将其拦截下来进行检测，如果sql语句没有危险操作，则正常放行，不会影响程序本身的功能。如果存在恶意攻击，则直接将恶意攻击的请求进行拦截或净化参数**。

## 三、国外的RASP

我搜集到国外RASP的产品有的下面几个，可能列表不足，欢迎补充:P

|     公司名称      |                           产品官网                           |
| :---------------: | :----------------------------------------------------------: |
|    Micro Focus    | https://www.microfocus.com/en-us/products/application-defender/features |
|      Prevoty      |                   https://www.prevoty.com/                   |
|      waratek      |    https://www.waratek.com/application-security-platform/    |
|  OWASP AppSensor  |                    http://appsensor.org/                     |
|      Shadowd      |      https://shadowd.zecure.org/overview/introduction/       |
|       immun       |                https://www.immun.io/features                 |
| Contrast Security | https://www.contrastsecurity.com/runtime-application-self-protection-rasp |
|  Signal Sciences  | https://www.signalsciences.com/rasp-runtime-application-self-protection/ |
|     BrixBits      |        http://www.brixbits.com/security-analyzer.html        |

笔者只列举了部分国外产品，关于更多这些产品的介绍请大家自行咨询。

根据上述收集到的国外RASP技术产品，发现大家都各有千秋，甚至还有根据RASP技术衍生出来的一些其他技术名词以及解决方案。

大家对数据可视化越来越注重，让使用者可以一目了然的看清楚入侵点以及攻击链，以此来对特定的点进行代码整改或者RASP技术层面的攻击拦截。

## 四、国内RASP技术实现进度以及状态

国内目前做RASP技术的厂家不多，我收集的只有以下几家。如有不足，欢迎补充。

| 产品名称 |                        产品官网                         |                             概述                             |
| :------: | :-----------------------------------------------------: | :----------------------------------------------------------: |
|   灵蜥   |            http://www.anbai.com/lxPlatform/             | 灵蜥是北京安百科技研发的一款RASP防御产品，而RASP安全研究团队的核心是之前乌云核心成员，据我了解，乌云在2012年后半年左右的时候就对RASP技术开始进行研究了。目前灵蜥RASP安全防御技术以及升级到了2.0版本，对灵蜥1.0版本的架构以及防御点进行了大版本的升级，应该是目前防御比较完善的一家了。 |
| OneRasp  |                 https://www.oneapm.com/                 | 然后蓝海讯通研究的OneRasp也出现在市场，后面不知什么原因，蓝海讯通好像放弃了对oneRasp的研究以及售卖，现在oneRasp官网以及下线。 |
| OpenRasp |                 https://rasp.baidu.com/                 | 在随后可能就是大家都听过或者用过百度开源的OpenRasp，百度开源的rasp产品让更多的企业以及安全研究者接触和认识到了rasp技术。该项目目前社区活跃度挺高，插件开发也很简单，大大降低了使用门槛。 |
|   云锁   | [https://www.yunsuo.com.cn](https://www.yunsuo.com.cn/) | 云锁我是在18年的360安全大会上了解到的，官网将RASP技术列入了一个小功能，主打是的微隔离技术，关于微隔离技术，在这边我们不做探讨。有兴趣的可以搜索一下相关的技术文章。 |
|  安数云  |         http://www.datacloudsec.com/#/product-4         | 安数云WEB应用实时防护系统RASP是在我写这篇文章的时候，发现他们也开发了一款RASP产品才知道的。不过没有找到其技术白皮书等可参考的文章。 |

## 五、各种语言RASP技术实现方式

### 1.JAVA

Java是通过Java Agent方式进行实现（Agent本质是java中的一个动态库，利用JVMTI暴露的一些接口实现的），具体是使用ASM(`或者其他字节码修改框架`)技术实现RASP技术。

Java Agent有三种机制，分别是Agent_OnLoad、Agent_OnAttach、Agent_OnUnload，大部分时候使用的都是Agent_OnLoad技术和Agent_OnAttach技术，Agent_OnUnload技术很少使用。

具体关于Java Agent的机制大家可以看一下下面这些文章：

[JVMTM Tool Interface](https://docs.oracle.com/javase/8/docs/platform/jvmti/jvmti.html#startup) [javaagent加载机制分析](https://nijiaben.iteye.com/blog/1847212) [JVM 源码分析之 javaagent 原理完全解读](https://www.infoq.cn/article/javaagent-illustratedhttps://www.infoq.cn/article/javaagent-illustrated)

### 2.PHP

PHP是通过开发第php扩展库来进行实现。

### 3..NET

.NET是通过IHostingStartup（承载启动）实现，具体的大家可以下面这篇文章：

[在 ASP.NET Core 中使用承载启动程序集](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/platform-specific-configuration?view=aspnetcore-2.2)

### 4.other

RASP技术其实主要就是**对编程语言的危险底层函数进行hook**，毕竟在怎么编码转换以及调用，最后肯定会去执行最底层的某个方法然后对系统进行调用。由此可以反推出其hook点，然后使用不同的编程语言中不同的技术对其进行实现。

## 六、举“栗子”

### 1.如何在Java中实现RASP技术

目前笔者我参与研究&研发了JAVA版本的RASP技术，下面给大家简单的进行讲解。

在jdk1.5之后，java提供一个名为Instrumentation的API接口，Instrumentation 的最大作用，就是类定义动态改变和操作。开发者可以在一个普通Java程序（带有 main 函数的 Java 类）运行时，通过 – javaagent参数指定一个特定的 jar 文件（包含 Instrumentation 代理）来启动 Instrumentation 的代理程序。

关于premain的介绍，大家可以去我博客看一下：[java Agent简单学习](https://www.03sec.com/3232.shtml)。

关于ASM的介绍以及使用，大家可以看一下下面的介绍：[ASM官方操作手册](https://asm.ow2.io/asm4-guide.pdf)，[AOP的利器：ASM3.0介绍](https://www.ibm.com/developerworks/cn/java/j-lo-asm30/index.html)关于ASM的时序图，我这边引用一下IBM上的一张图： 

![1.jpg](https://image.3001.net/images/20190311/1552284115_5c85f9d3cb36c.jpg!small)

**正文开始**

首先编写一个premain函数，然后在premain中添加一个我们自定义的Transformer。 

![2.jpg](https://image.3001.net/images/20190311/1552284142_5c85f9eeb7828.jpg!small)

我们的Transformer需要实现ClassFileTransformer接口中的transform方法，我这边就简单的先进行一个打印包名的操作。 

![3.jpg](https://image.3001.net/images/20190311/1552284157_5c85f9fd7ba30.jpg!small)

在编译以后，运行结果如下： 

<img src="https://image.3001.net/images/20190311/1552284169_5c85fa0998bc7.jpg!small" alt="4.jpg" style="zoom:67%;" />

由此可见我们的transform已经生效了。 我们来简单的类比一下加入agent以及未加入agent的过程。 

![5.jpg](https://image.3001.net/images/20190311/1552284184_5c85fa186e8f7.jpg!small)

由此可见，在使用了transform以后，可以对jvm中的未加载的类进行重写。已经加载过的类可以使用retransform去进行重写。 在java中有个比较方便的字节码操作库，ASM（ASM是一个比较方便的字节码操作框架，可以借助ASM对字节码进行修改，这样就可以实现动态修改字节码的操作了。[ASM介绍](https://www.ibm.com/developerworks/cn/java/j-lo-asm30/index.html)）

ClassVisitor接口，定义了一系列的visit方法，而这些visit方法。

下面这段代码是对方法进行重写的一个过程。

首先定义一个ClassVisitor，重写visitMethod方法。 

![6.jpg](https://image.3001.net/images/20190311/1552284198_5c85fa2648875.jpg!small)

可以看到该方法返回的对象为MethodVisitor(后面简称mv)，我们如果要修改其中的逻辑过程，需要对mv进行操作，下面展示一个简单的修改代码的过程。 

![7.jpg](https://image.3001.net/images/20190311/1552284218_5c85fa3a3f5bc.jpg!small)

上面visitCode中代码的大概意思为，插入一个调用TestInsert类中的hello方法，并且将一个数字8压入进去。 TestInsert类的代码如下：

<img src="https://image.3001.net/images/20190311/1552284235_5c85fa4b9dfcd.jpg!small" alt="8.jpg" style="zoom:67%;" />

如果不出意外，运行jar包以后将输出类似下面这样的信息：

![9.jpg](https://image.3001.net/images/20190311/1552284252_5c85fa5c5028a.jpg!small)

可以看一下加入agent运行以后的类有什么变化。

源代码：

![10.jpg](https://image.3001.net/images/20190311/1552284265_5c85fa699b9ee.jpg!small)

加入Agent运行以后的源代码：

![11.jpg](https://image.3001.net/images/20190311/1552284280_5c85fa7852b39.jpg!small)

可以看到代码：

```
System.out.println(TestInsert.hello(8));
```

成功在：

```
System.out.println("Hello,This is TestAgent Program!");
```

之前运行，而调用TestInsert.hello这个方法在源代码中原来是没有的，我们是通过java的agent配合asm对运行的字节码进行了修改，这样就达到了埋点hook的目的。

在了解了上面的技术以后，就可以其他对关键的点进行hook了，hook的方法大同小异，这里仅仅是抛砖引玉，大家自行发挥。可以给大家推荐一个园长师傅写的一个简单例子：[ javaweb-codereview](https://github.com/anbai-inc/javaweb-codereview)

笔者展示的该例子项目地址为：[javaagent](https://gitlab.com/iiusky/javaagent)

### 2.如何在PHP中实现RASP技术

关于php中的rasp技术实现，笔者我还没有进行深入研究，大家可以参考以下两个开源的扩展进行研究，笔者接下来也准备研究php方面的rasp技术实现，欢迎大家一起讨论。

> （1）[xmark](https://github.com/fate0/xmark)
>
> （2）[taint](https://github.com/laruence/taint)

## 七、RASP技术的其他方面应用场景

在RASP技术实现的过程中，我横向了遐想一些其他的用法以及参考了一下其他rasp产品的功能点，比如：

### **代码审计**

对于rasp中运用到的技术，换一种思维方式，可以不进行拦截，而进行记录，对所有记录的日志结合上下文进行代码审计。

### **0day防御**

对已经hook的关键点进行告警通知并且要拦截攻击行为，然后在公网部署多种不同cms的web蜜罐。如若已经触发到了告警通知，那么已经证明攻击已经成功。且拦截到的漏洞可能为0day。

### **攻击溯源**

对所有攻击ip以及攻击的文件进行聚合，用时间轴进行展示。这样就可以定位到黑客是从上面时候开始进行攻击的，攻击中访问了哪些文件，触发了哪些攻击拦截。然后对所有大致相同的ip进行归类，可以引出来一个专业用于攻击溯源的产品。

### **DevOps**

对所有事件详细信息提供完整的执行路径，包括代码行，应用程序中使用的完整上下文查询以及丰富的属性详细信息。

### **其他方面**

当然Rasp技术的应用场景肯定不止这么多，还有其他很多方面的一些应用场景，大家可以发散思维去想。本文章仅用于抛砖引玉。

## 八、RASP技术有什么缺陷

● 不同的编程语言可能编译语言和应用程序的版本不一致都导致RASP产品无法通用，甚至导致网站挂掉；

● 如果RASP技术中对底层拦截点不熟悉，可能导致漏掉重要hook点，导致绕过；

● 对于csrf、ssrf、sql语句解析等问题目前还是基于部分正则进行防护(对于sql语句的解析问题可以使用AST语法树进行解析)。

……

## 九、总结

目前RASP还处于一个发展的阶段，尚未像防火墙等常见的安全产品一样有非常明确的功能边界(scope)，此文仅抛砖引玉，更多由RASP防御技术的用法可以发挥想象，比如日志监控、管理会话、安全过滤、请求管理等。 在研究RASP技术的时候，我发现RASP与APM有着部分相同的技术，APM本意是用于监控应用程序的，反转思路，在APM中监控应用程序的时候加入漏洞防护功能，即可形成了一个简单的RASP demo。在APM这方面，大家参考的对象可就很多了，比如：[skywalking](http://skywalking.apache.org/)、[newrelic](https://www.newrelic.com/)等，skywalking是开源产品，在github上就可以直接查看其源代码，而且是Apache的项目，而newrelic的所有agent都没有进行任何代码混淆，官方安装文档也很丰富，这些都是可以进行参考的。 由此可见，有时候技术换个平台、换个想法，就是一个完全不一样的东西了。所以对于任何技术都可以发散性的去想象一下其他的应用场景，万一恰到好处呢？ 不论研究的技术是什么样的，可能大家实现的方式不一样，使用的技术不一样，使用的名称不一样，不过最终的目的就是为了保护应用程序安全，防止攻击者入侵成功。

## 十、参考

https://searchsecurity.techtarget.com/opinion/McGraw-on-why-DAST-and-RASP-arent-enterprise-scale

http://www.cnblogs.com/kingreatwill/p/9756222.html

https://techbeacon.com/security/it-time-take-rasp-seriously

https://www.gartner.com/doc/2856020/maverick-research-stop-protecting-apps

https://github.com/anbai-inc/javaweb-codereview/tree/master/javaweb-codereview-agent

https://www.03sec.com/3232.shtml

***本文作者：安百科技，转载请注明来自FreeBuf.COM**

本文作者：安百科技， 转载请注明来自[FreeBuf.COM](https://www.freebuf.com/)



[浅谈RASP技术攻防之基础篇 - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/web/197823.html)