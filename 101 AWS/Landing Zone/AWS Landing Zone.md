# AWS Landing Zone

[奥利奥](https://www.zhihu.com/people/xia-xia-31-26)

AWS解决方案架构师

AWS Landing Zone是在 AWS 上部署您的应用程序之前，您需要设计和配置一个基础环境，其中设计AWS多账户结构、设置安全规则、配置默认网络和其他基础服务。 

这样的基础环境称为Landing Zone，也是为拥有多个IT团队或项目客户的最佳实践。

### 1、利用企业多账号策略建立企业的AWS账号体系

Landing Zone的一个**核心要素就是多账号体系**，您可以全部在一个账户中工作，但是随着时间的增加、团队成员越来越多，IAM权限管理将会非常困难。例如测试环境和生产环境如果在一个账户内并且还在一个region中，那么极有可能发生误操作。

企业通常通常会采用多AWS账号策略来管理云上资源，这样做的好处一方面是能够提供足够强的**安全隔离**，另一方面是能够使用**最大的资源数**（因为每个账号是有资源限制的，如VPC限制为5）

常见的AWS账号体系如下：

### 身份账户：

假设你有5个AWS账号，那么每入职一名新员工，作为管理者都要创建5个AWS账号，不仅管理起来麻烦，而且员工也要维护5套账号密码，这样下去身份管理工作将会越来越难做，中央身份账户则能够通过switch role的方式，使员工只有一个aws登录入口，然后再通过switch role登录到其他的aws账户。具体实现方法可以从我历史文章中找到。

### 日志账户

多账户环境通常会把所有的日志、cloudtrail、vpc 日志、config日志等集中存储到一个账户中，方便定期的审计和分析，我们可以在中央日志账户中建立一个S3存储桶，将其他账户的日志发送到中央账户中，实现通过单一节点对AWs账户的日志存储、分析、监控等需求。

### 发布账户

安全团队可以将批准过的安全镜像共享给研发账户或测试、生产账户使用，确保各个团队使用安全、统一的镜像，您可以使⽤AWS的ServiceCatalog服务来实现。

### 账单账户

当企业是多AWS账户环境时，您可以使⽤AWS Organizations的整合账户功能，建⽴组织的主账户并整合和⽀付所有成员AWS⼦账户，使得您可以在⼀个主账户上追踪整个企业的AWS⼦账户的账单，并可为多个AWS⼦账户在主账户上进⾏统⼀⽀付。

### 2、开启AWS CloudTrail

在每个账户中创建一个 CloudTrail 跟踪并对其进行配置，以将日志发送到日志存档账户中集中管理的 Amazon Simple Storage Service (Amazon S3) 存储桶。    

### 3、开启AWS Config

启用 AWS Config，并将账户配置日志文件存储在日志存档账户中的集中管理 Amazon S3 存储桶中。

### 4、设置AWS Config 规则

启用 AWS Config 规则以监控存储加密（Amazon Elastic Block Store、Amazon S3 和 Amazon Relational Database Service）、AWS Identity and Access Management (IAM) 密码策略、根账户多重验证 (MFA)、Amazon S3 公共读取和写入及不安全的安全组规则。

5、创建IAM角色和策略

为不同职责的员工创建管理员和只读的角色和策略

### 6、配置IAM密码策略

AWS Identity and Access Management 可用于配置 IAM 密码策略。

### 7、设置跨账户访问

跨账户访问可使IAM用户通过中央身份账户跨越到其他账户中访问资源。

### 8、Amazon Virtual Private Cloud (VPC)

Amazon VPC 可为账户配置初始网络。

### 9、开启AWS Landing Zone 通知

Amazon CloudWatch 警报和事件被配置为，在根账户登录、控制台登录失败和账户内的 API 身份验证失败时发送通知。也可设置资源增删通知，和账单告警通知。

发布于 2021-07-18 19:05

[亚马逊云科技](https://www.zhihu.com/topic/19558548)