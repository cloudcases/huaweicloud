# Junit 5官方文档中文版

https://www.bookstack.cn/read/junit5/README.md



## 介绍

JUnit 5是JUnit的下一代。目标是为JVM上的开发人员端测试创建一个最新的基础。这包括专注于Java 8及更高版本，以及启用许多不同风格的测试。

JUnit 5团队于2017年9月10日发布了第一个[GA版本 5.0.0][ga]。

这里是Junit5官方文档的中文翻译版，由[doczh.cn](http://doczh.cn/)组织翻译并更新维护。

文档内容发布于gitbook，请点击下面的链接阅读或者下载电子版本:

- 在线阅读
  - [国外服务器](https://doczhcn.gitbooks.io/junit5/)：gitbook提供的托管，服务器在国外，速度比较慢，偶尔被墙，HTTPS
  - 国内服务器：~~腾讯云加速，国内网速极快，非HTTPS~~ 即将推出
- [下载pdf格式](https://www.gitbook.com/download/pdf/book/doczhcn/junit5)
- [下载mobi格式](https://www.gitbook.com/download/mobi/book/doczhcn/junit5)
- [下载epub格式](https://www.gitbook.com/download/epub/book/doczhcn/junit5)

本文内容可以任意转载，但是需要注明来源并提供链接。

**请勿用于商业出版**。

## doczh.cn

doczh.cn是一个纯技术的，面向技术社区，非商业性质的组织，专注于IT产品的中文文档翻译。

http://doczh.cn/

如果对文档翻译工作感兴趣，欢迎加入，也欢迎为现有文档的改善提供帮助。

## 来源(书栈小编注)

> https://github.com/doczhcn/junit5
> 注意：当前文档，译者并未翻译完全部内容，请实时关注上面的地址库，以跟进译者的最新翻译。



# Unit5用户指南

JUnit5官方user guide文档翻译，原英文文档地址为:

http://junit.org/junit5/docs/current/user-guide/

英文文档的github托管地址为：

https://github.com/junit-team/junit5/tree/master/documentation

当前版本是：Junit Version **5.0.2**



# unit 5是什么？

与以前的JUnit版本不同，JUnit 5由三个不同子项目的多个不同模块组成。

JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

**JUnit Platform**为在JVM上[启动测试框架](https://www.bookstack.cn/read/advanced-topics/launcher-api.md)提供基础。它还定义了[TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API, 用来开发在平台上运行的测试框架。此外，平台提供了一个[控制台启动器](https://www.bookstack.cn/read/running-tests/console-launcher.md)，用于从命令行启动平台，并为[Gradle](https://www.bookstack.cn/read/running-tests/build.md#gradle)和[Maven](https://www.bookstack.cn/read/running-tests/build.md#maven)提供构建插件以及[基于JUnit 4的Runner](https://www.bookstack.cn/read/running-tests/junit-platform-runner.md)，用于在平台上运行任意`TestEngine`。

**JUnit Jupiter**是在JUnit 5中编写测试和扩展的新型[编程模型](https://www.bookstack.cn/read/writing-tests/)和[扩展模型](https://www.bookstack.cn/read/extensions/)的组合.Jupiter子项目提供了`TestEngine`，用于在平台上运行基于Jupiter的测试。

**JUnit Vintage**提供`TestEngine`，用于在平台上运行基于JUnit 3和JUnit 4的测试。



# 支持的Java版本

JUnit 5在运行时需要Java 8（或更高版本）。当然，您仍然可以测试使用以前版本的JDK编译的代码。