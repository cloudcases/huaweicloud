# Twilio 的混沌工程实践



- Hrishikesh Barua

- 薛命灯



**2017 年 12 月 27 日

**[DevOps](https://www.infoq.cn/topic/Devops)



[Twilio ](https://www.infoq.com/twilio/)团队分享了他们的初次[混沌工程](https://www.infoq.com/articles/chaos-engineering)实践，他们使用[ Gremlin ](https://www.infoq.com/news/2017/12/gremlin-chaos-engineering)往自家的队列系统中注入故障，测试系统的自动恢复能力。

Twilio 提供 SMS 和电话网关服务，开发者可以在他们的代码中调用 Twilio 的 API。Twilio[架构](https://www.infoq.com/news/2011/04/twilio-cloud-architecture)的核心部分是他们的分布式队列系统和速率限定系统。它提供了持久化的队列，解决了系统故障和消息处理的延迟问题，避免消息丢失。这个队列系统叫作[ Ratequeue ](https://vimeo.com/52569901?width=1080)，由 Twilio 团队开发。Ratequeue 对消息出队速率进行了限定——每个电话号码都有一个自己的临时队列。因为开发者有可能以很快的频率调用 API，所以速率限定是很有必要的，而 Twilio 将消息推送到电话网络的速度也需要加以控制。Ratequeue 是基于 Redis 开发的，可以进行横向扩展。单个分片故障并不会影响到其他分片。另外，为了高可用，每个分片都有自己的主节点和副本。

之前，如果有分片发生故障，需要由人工手动将副本提升为主节点。这就要求先定位到拥有相同分片数量的主机，然后把它加到负载均衡器中。Twilio 团队开发了两个系统，旨在对这一过程进行自动化——一个自动化的失效备援系统和一个用于测试前者的故障注入系统。故障注入系统属于混沌工程，通过制造随机的故障来测试系统的自我恢复能力。

测试的首要目的是保证零数据丢失，其次要保证能够自动检测出故障，并推举出新的主节点。Twilio 团队基于 Amazon Kinesis、Nagios 和 Lazarus 开发了自己的解决方案——也就是他们的集群自动化服务。每个 Ratequeue 副本将主节点的心跳情况发送给 Nagios，如果达到某个阈值，Nagios 就向 Kinesis 推送通知。Lazarus 监听 Kinesis，检查集群的健康状况，如果出现故障，就进行恢复。

为了测试自动故障恢复能力，Twilio 团队开发了一个叫作 Ratequeue Chaos 的工具，它会选择一个分片，停掉它的主节点，然后监控其自我恢复过程。他们有一个叫作 Gremlin 的服务，用于将故障注入到系统中，触发失效备援。Gremlin 支持可控的故障注入，Ratequeue Chaos 通过 Gremlin 提供的 API 来调用它。Twilio 的 staging 环境每 4 个小时会重复一次这个过程。

Twilio 团队也分享了他们从这一实践中学到的东西——基于测试模型的假设、用于运行测试的框架、在生产环境中需要有一个回退计划。

**查看英文原文**：[ Chaos Engineering at Twilio](https://www.infoq.com/news/2017/12/twilio-chaos-engineering)

2017 年 12 月 27 日 18:001189

文章版权归极客邦科技InfoQ所有，未经许可不得转载。



https://www.infoq.cn/news/2017/12/twilio-chaos-engineering/