# Netflix Chaos Monkey 2.0 发布



- 金灵杰



**2016 年 11 月 16 日

**[语言 & 开发](https://www.infoq.cn/topic/development)[架构](https://www.infoq.cn/topic/architecture)[运维](https://www.infoq.cn/topic/operation)



Chaos Monkey 是在 Netflix 整体微服务化的形势下开发的。为了增加微服务架构的弹性，需要确保当服务集群中有节点失败或者退出时不会影响整体服务。由于 Netflix 的内部文化，没有办法通过框架或者编码规范来形成一套能够满足弹性要求的框架。最终，Netflix 选择开发了 Choas Monkey：一个在生产环境随机选择并关闭服务的工具。对于这个选择，有人会觉得很疯狂，但是通过频繁的服务失败演练，使得开发团队对服务集群稳定性有了更高的重视，以确保不会因为这些演练对最终用户产生影响。

Netflix 将 Chaos Monkey 定位为提升服务质量的高效工具。最近发布的 2.0，除了带来更好的可维护性，也带来了一些新的特性。

**和 Spinnaker 集成**

[Spinnaker ](http://www.infoq.com/cn/news/2015/11/Netflix-Spinnaker)是 Netflix 的持续交付平台，Chaos Monkey 2.0 和它结合之后，可以在 Spinnaker 上对其进行配置。同时 Chaos Monkey 可以从 Spinnaker 获取服务部署的相关信息并通过 Spinnaker 关闭服务实例。

由于集成了 Spinnaker，Chaos Monkey 增加了对多种后端的支持，包括：AWS、GCP、Azure、Kubernetes、Cloud Foundry。

Chaos Monkey 2.0 还在配置上进行了优化，用户可以设置两次终止之间的平均时间，而不是在任意时段内的概率。另外，针对服务所在的环境进行分组，分组方式延续了 AWS 的概念，包括应用、[应用栈（stack）](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#d0e3929)和集群。配置页面如下：

![img](https://static001.infoq.cn/resource/image/86/9d/869508bf95de33b47906e3d64ea68c9d.jpg)

**追踪关闭行为**

Chaos Monkey 2.0 可以单独配置外部追踪器，当 Chaos Monkey 对某个实例进行了关闭操作后，它会向配置的追踪器发送通知。对于 Netflix 内部使用来说，Chaos Monkey 会将通知发送到[ Atlas ](http://techblog.netflix.com/2014/12/introducing-atlas-netflixs-primary.html)（Netflix 的检测系统）和[ Chronos ](http://techblog.netflix.com/2013/03/python-at-netflix.html)（Netflix 的事件追踪系统）。下图是 Atlas 系统的截图，展示了 Chaos Monkey 对于部分服务的关闭操作行为，值得注意的是，Chaos Monkey 还会关闭自己的服务实例。

![img](https://static001.infoq.cn/resource/image/85/76/85a90925b748bba7f5b36cf9fba3f176.jpg)

**其他改变**

之前版本的 Chaos Monkey，除了支持关闭服务实例之外，还支持其他一些操作系统级别的破坏，例如提高 CPU 占用率、阻塞网络 IO、写满硬盘空间等。Chaos Monkey 2.0 移除了这些功能，只支持关闭服务实例。对于这些功能移除，Netflix 的工程师认为，这些功能应该被放到故障注入服务中进行定向注入，而不是作为 Chaos Monkey 的随机操作之一。关于故障注入，Netflix 也有一些[介绍](http://techblog.netflix.com/2014/10/fit-failure-injection-testing.html)。

Chaos Monkey 2.0 源码在其[ Github 仓库](https://github.com/netflix/chaosmonkey)上已经可以下载和部署。详细部署方式参见 Chaos Monkey 的[ wiki 页面](https://github.com/Netflix/chaosmonkey/wiki/How-to-deploy)。

------

感谢[陈兴璐](http://www.infoq.com/cn/author/陈兴璐)对本文的审校。

给InfoQ 中文站投稿或者参与内容翻译工作，请邮件至[ editors@cn.infoq.com ](mailto:editors@cn.infoq.com)。也欢迎大家通过新浪微博（[ @InfoQ ](http://www.weibo.com/infoqchina)，[ @丁晓昀](http://weibo.com/u/1451714913)），微信（微信号：[ InfoQChina ](http://www.geekbang.org/ivtw)）关注我们。

2016 年 11 月 16 日 18:002016

文章版权归极客邦科技InfoQ所有，未经许可不得转载。



https://www.infoq.cn/news/2016/11/Netflix-Chaos-Monkey-2-0?utm_source=related_read_bottom&utm_medium=article