# Netflix 新放出来的开源工具 Chaos Monkey



- Richard Seroter

- 郭晓刚



**2012 年 8 月 04 日

**[AWS](https://www.infoq.cn/topic/AWS)[云计算](https://www.infoq.cn/topic/cloud-computing)[DevOps](https://www.infoq.cn/topic/Devops)[架构](https://www.infoq.cn/topic/architecture)



Netflix 刚刚[开源了](https://github.com/Netflix/SimianArmy)他们那被人惦记好一阵子的“Chaos Monkey”，这是一套用来故意把服务器搞下线的软件，可以测试云环境的恢复能力。Netflix 专门开发的一系列捣乱工具，已经有不少被拿出来和技术社区自由分享，现在Chaos Monkey 也加入了这个行列。

Netflix 团队让 Chaos Monkey 亮相的时间，最早是在 2010 年 12 月的一篇[官博文章](http://techblog.netflix.com/2010/12/5-lessons-weve-learned-using-aws.html)，文章内容是他们在 AWS 云上托管其热门视频流服务所得到的经验教训。文中总结了一点，叫做“避免失败的最好办法是经常失败”, 反映 Netflix 通过主动破坏自身环境来发现弱点的做法。

> 我们的工程师在 AWS 上最早建立的系统之一叫 Chaos Monkey。这猴子的工作是随机杀掉架构中的运行实例和服务。如果我们不经常检验自己战胜挫折的能力，那么在最需要的时候，遇到意料之外的故障事件的时候，能力很可能使不出来。

Netflix 技术团队在 2012 年 7 月 20 日的[官博文章](http://techblog.netflix.com/2012/07/chaos-monkey-released-into-wild.html)上宣布，Chaos Monkey 作为[开源项目](https://github.com/Netflix/SimianArmy)公开。文中解释了 Chaos Monkey 的设计意图和运营中的注意事项。Netflix 声称软件可以成功运行在在 AWS 以外的云上，主要给用户检测自身环境中的失败条件。考虑到有人会担心在数据中心里随便放跑猴子，不知道会闯出多大的祸事，Netflix 预设了一些配置选项作为防备。首先，Chaos Monkey 可以被设定为只在支持人员现场待命，准备救灾的时候才运行。

> 服务具有可配置的执行计划，默认只在非假日的周一到周五上午 9 点至下午 3 点执行。一般来说，我们的应用设计可以保证一两个实例下线不至于导致服务中断，但万一有什么特殊情况，我们还是希望边上有人能解决问题和吸取经验。根据这样的思路，我们设计只有在预料警报会被工程师发现并作出响应的有限时间段，才把 Chaos Monkey 放出来。

第二，用户可以决定 Chaos Monkey 对新应用的攻击强度。Netflix 喜欢让 Chaos Monkey 对所有的应用无差别攻击（除非应用负责人明确选择退出），但一般用户不需要完全照学，可以养一只温顺点的猴子，叫它只跟特定的应用过不去，还可以把彻底搞死实例的几率设得低一些。

> 不是所有的应用都能把实例下线不当回事。有时候恢复实例必须拿人力去填，说不定还要折腾备份。简化回复流程，加快恢复速度，最终实现自动化，这些都需要时间。对于禁不起下线的应用，Chaos Monkey 允许自主退出。Chaos Monkey 中止实例的几率也是可调的。

比 Chaos Monkey 还早一点点喜迎开源的是另一个重要项目：[ Asgard ](https://github.com/Netflix/asgard)。[ Netflix 团队在另一篇文章](http://techblog.netflix.com/2012/06/asgard-web-based-cloud-management-and.html)中说，Asgard 是用于部署和管理 AWS 环境的一套 web 界面。[ AWS 本身的管理控制台](http://aws.amazon.com/console/faqs/)提供了完整的服务器规划能力，不过 Netflix 团队觉得它缺少了“应用”的概念，所以打算让 Asgard 填补空缺。

> 当一个面向服务的架构中充满了巨量云对象的时候（如同 Netflix 所面临的情况），让用户从中找到与特定应用关联的全部对象，是很重要的一件事情。Asgard 依靠存放在 SimpleDB 里的应用登记表和命名约定，把应用和相关的云对象联系到一起。

另外，Asgard 将[ AWS Auto Scaling Groups ](http://aws.amazon.com/autoscaling/)聚合为“clusters”对象，更便于在部署失败的时候回滚。Asgard 之所以设计“应用”和“集群”的概念，以及支持自动化部署任务，都是为了简化 AWS 的管理，减少人为出错。

[Netflix 正在积极的开源其软件](http://techblog.netflix.com/2012/07/open-source-at-netflix-by-ruslan.html)，惠及更大的社群，Chaos Monkey 和 Asgard 只是其中的一小部分而已。在他们的 Github 页面上可以看到[ Netflix 全部开源项目](http://netflix.github.com/)。

**查看英文原文：**[ Netflix Unleashes Chaos Monkey as its Latest Open Source Tool](http://www.infoq.com/news/2012/07/chaos-monkey)

2012 年 8 月 04 日 18:0617430

文章版权归极客邦科技InfoQ所有，未经许可不得转载。



https://www.infoq.cn/news/2012/08/chaos-monkey?utm_source=related_read_bottom&utm_medium=article