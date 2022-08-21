# AWS推出开源microVM 支撑容器和无服务器计算服务

 [k8s中文技术社区](https://blog.51cto.com/u_13964361)2022-04-13 13:42:24©著作权

*文章标签*[基础设施](https://blog.51cto.com/search/result?q=基础设施)[服务器](https://blog.51cto.com/topic/the-server-2.html)[公有云](https://blog.51cto.com/search/result?q=公有云)*文章分类*[其他](https://blog.51cto.com/nav/other1)[其它](https://blog.51cto.com/nav/other)*阅读数*65

*AWS CEO Andy Jassy（左）与VMware CEO Pat Gelsinger（右）*

近日于2018 AWS re:Invent大会上，全球公有云服务提供商AWS首谈混合架构，正式推出其本地化部署方案AWS Outposts，方案中也坐实其早前与VMware的混合云战略合作。

AWS Outposts是将AWS的基础架构和运营模型本地化，在企业内部部署和AWS中使用相同的API、工具、硬件，以及功能，实现真正一致的混合体验。 AWS CEO Andy Jassy表示，一些客户的应用无法上云，他们希望拥有与云一致的服务与体验。

**AWS Outposts目前支持两种选项：**

**第一、VMware Cloud on AWS：**相当于是VMware Cloud on AWS的本地部署，在本地搭建的AWS环境中运行VMware。这就比较适合有数据可控需求，又希望有更好伸缩性和新技术的企业，以及想要上公有云的企业。

**第二、AWS native：**在本地交付AWS软硬件一体的解决方案，可配置的计算、存储机架甚至维保，客户可以使用用于在AWS云中运行的相同的控制台和API。

实际在几个月前宣布的Amazon RDS on VMware，混合架构的策略就已经初露端倪。企业可以在基于VMware的软件定义数据中心和混合环境中轻松地安装、运行和扩展数据库，并将它们迁移到AWS或AWS上的VMware云。

另一方面，AWS在本次大会中还发布了Firecracker，这是一款microVM，是基于Kernel的虚拟机（KVM）的轻量级实例，AWS已承诺将其作为开源项目提供。Firecracker将用于为使用托管AWS Fargate容器服务或AWS Lambda无服务器计算框架的客户更有效地隔离IT基础设施资源。

在开发Firecracker之前，AWS一直依靠EC2云服务的专用实例来运行AWS Fargate和AWS Lambda。AWS全球基础设施和客户支持副总裁Peter Desantis表示，AWS现在可以使用Firecracker虚拟机隔离AWS Fargate和AWS Lambda上运行的工作负载。这种方法可以使AWS更有效地支持这些服务，而无需为每个客户提供专用的基础设施。

每个Firecracker microVM消耗的内存略多于5 MiB，并且可以在125毫秒内启动，这大大减少了与传统虚拟机相关的延迟和开销。Firecracker基于为ChromeOS开发的crosvm，采用Rust编程语言开发。在Firecracker上运行的每个客户都会看到一个网络设备、一个块I / O设备、一个可编程间隔定时器、一个KVM时钟、一个串行控制台和一个部分键盘。

每个Firecracker进程都用Linux内核中的cgroups和seccomp Berkley Packet Filter（BPF）函数进行“监禁”，并且可以访问一个严格控制的小型系统调用列表。每个Firecracker进程都是静态链接的，可以从“jailer”启动，以确保主机环境的安全。

Desantis表示，AWS已经证明每秒可以在其服务上推出超过1.5亿个microVM。实际上，已经有数千万个容器在AWS服务上运行。

通过发布Firecracker，AWS正在考虑如何为容器和无服务器计算框架提供隔离，而不必依赖产生大量开销的传统虚拟机。现在大多数容器都在虚拟机上运行以确保隔离。但平均而言，需要为每个虚拟机部署10到25个容器。microVM可以在microVM之上的客户操作系统上运行数百个容器。

microVM和传统虚拟机之间的关系将如何发展还有待观察。在云和本地环境中运行的数百万个遗留应用程序针对传统虚拟机进行了优化。但是，随着时间的推移，许多应用程序将被重新设计为基于容器的一组微服务运行。

由于Firecracker是一个开源项目，因此AWS使企业IT组织不仅可以在AWS云中部署microVM，而且理论上也可以在本地IT环境甚至是竞争对手的云服务中部署。将这种功能与Kubernetes集群相结合，那么将AWS融入混合云计算战略可能会变得更加简单。

参考链接：

https://containerjournal.com/2018/11/28/aws-adds-microvms-for-containers-and-serverless-computing-frameworks/

http://cio.zhiding.cn/cio/2018/1129/3113596.shtml



[AWS推出开源microVM 支撑容器和无服务器计算服务_K8S中文技术社区的技术博客_51CTO博客](https://blog.51cto.com/u_13964361/5201806)