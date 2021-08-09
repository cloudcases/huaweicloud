# AWS 开源混沌工程工具 AWSSSMChaosRunner



- Hrishikesh Barua

- 王者



**2020 年 9 月 30 日

**[云计算](https://www.infoq.cn/topic/cloud-computing)[开源](https://www.infoq.cn/topic/opensource)[AWS](https://www.infoq.cn/topic/AWS)



![AWS开源混沌工程工具AWSSSMChaosRunner](https://static001.infoq.cn/resource/image/a5/e1/a53c0588ff2a106ef449d26daed5dde1.jpeg)

AWS 的工程师们最近写了一篇文章，介绍了一个叫作 AWSSSMChaosRunner 的开源混沌工程工具，他们用它来测试 Prime Video 的故障注入。这个工具使用 AWS Systems Manager 构建，可以在 EC2 实例上执行任意命令，团队可以用它缓解与延迟相关的问题。



AWSSSMChaosRunner 是使用 AWS Systems Manager 构建的，用于针对一组特定的 EC2 实例远程执行命令。通过声明方式指定的命令集合创建了一组注入错误。



Prime Video 软件工程师 Varun Jewalikar 和 AWS 首席开发者(架构)布道师 Adrian Hornsby 写道，典型的混沌工程实验包括模拟资源耗尽和缓慢的网络。对于这样的场景有一些对策，但“它们很少得到充分测试，因为单元测试或集成测试通常不能充分验证它们”。



[AWS Systems Manager](https://aws.amazon.com/systems-manager/)是一个工具，可以通过一个叫作[SSM Agent](https://github.com/aws/amazon-ssm-agent/)的代理组件跨 AWS 资源执行各种运维任务。默认情况下，代理被预先安装在某些 Windows 和 Linux AMI 上——它们也有“文档”的概念，类似于可以执行的 Runbook。它还可以执行简单的 shell 脚本，AWSSSMChaosRunner 就是利用了这个特性。SSM 的 SendCommand API 允许跨多个实例执行命令，这些实例可以通过 AWS 标记来过滤。CloudWatch 可以用于在一个地方查看来自所有实例的日志。



安全方面的问题由代理负责，比如创建在 EC2 实例上执行的用户。AWSSSMChaosRunner 可以做的事情包括在一个特定端口上悄悄地中断所有传出的 TCP 流量、在一个接口上引入网络延迟、占用 CPU，等等。需要注意的是，当前支持的故障注入要么是在基础设施上，要么是在 AWS 服务层上。



AWSSSMChaosRunner 源自一组[SSM文档](https://github.com/adhorn/chaos-ssm-documents)，这些文档与将故障注入 AWS 资源有关。根据文中所写，在使用标准 SSM Agent API 执行文档之后，负载生成组件根据应用程序模拟真实的流量。AWSSSMChaosRunner 也可以用于 ECS，但不能用于 Lambda，因为后者是一个完全托管的服务。还有[其他方法](https://medium.com/@adhorn/failure-injection-gain-confidence-in-your-serverless-application-ce6c0060f586)可以在 AWS Lambda 中进行故障注入。



Prime Video 背后使用了 AWS 服务，它利用 AWSSSMChaosRunner 来测试依赖服务出现高延迟时的性能。Jewalikar 和 Hornsby 提到，AWSSSMChaosRunner 助他们修复了 Elasticache 超时配置的一个 bug。



还有[其他](https://github.com/dastergon/awesome-chaos-engineering#notable-tools)可用于执行混沌工程实验的库，早期的一个库是 Netflix 的[Chaos Monkey](https://github.com/Netflix/chaosmonkey)。其他公司也开发了自己的框架，比如 LinkedIn 的[Waterbear](https://www.infoq.com/news/2018/06/linkedout-failure-injection/)项目和 Twitter 的[Python库](https://blog.twitter.com/engineering/en_us/a/2015/how-we-break-things-at-twitter-failure-testing.html)。[Gremlin](https://www.infoq.com/gremlin/)公司还提供了故障注入服务。



AWSSSMChaosRunner 的[源代码](https://github.com/amzn/awsssmchaosrunner)可以在 GitHub 上找到。



**原文链接**：



[An Open Source Chaos Engineering Library from AWS](https://www.infoq.com/news/2020/08/aws-chaos-engineering/)



2020 年 9 月 30 日 10:321112

文章版权归极客邦科技InfoQ所有，未经许可不得转载。



https://www.infoq.cn/article/IZLGTWoGlp3GufGVMey7?utm_source=related_read_bottom&utm_medium=article