# AWS App Runner 的第一印象

[m0_60199850](https://blog.csdn.net/m0_60199850)![img](https://csdnimg.cn/release/blogv2/dist/pc/img/newCurrentTime2.png)于 2021-08-19 16:09:41 发布

版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：https://blog.csdn.net/m0_60199850/article/details/119804444

## 是什么让 App Runner 与其他服务不同？

让我们看看 AWS 如何描述 App Runner。

> [AWS](https://so.csdn.net/so/search?q=AWS&spm=1001.2101.3001.7020) App Runner 是一项完全托管的服务，可让开发人员轻松快速地大规模部署容器化 Web 应用程序和 API，并且无需具备基础设施经验。从您的源代码或容器映像开始。App Runner 自动构建和部署 Web 应用程序，并通过加密对流量进行负载平衡。App Runner 还会自动放大或缩小以满足您的流量需求。使用 App Runner，您无需考虑服务器或扩展，而是有更多时间专注于您的应用程序。

对我而言，App Runner 的主要卖点在于它是一项非常易于使用、安全且**完全托管的**服务。这是一种固执己见的架构，使在 AWS 中运行容器化应用程序变得非常容易。App Runner 为容器运行时和编排选项的复杂生态系统提供了另一个抽象级别。与现有的 AWS 容器服务相比，几乎没有学习曲线，因为您只需要指向 GitHub 存储库或 ECR 中的容器映像，并提供一些安全配置设置、默认实例数量和单个实例所需的资源，它将创建一个安全、完全负载平衡和自动缩放的服务。对技术感兴趣朋友可以加这个扣扣2779571288交流。

![AWS 控制台显示不同的源和部署选项](https://img-blog.csdnimg.cn/img_convert/5f6009fc22f4f5c54f334ee6601e9686.png)


AWS 控制台的屏幕截图，显示了 App Runner 服务的不同来源和部署选项。

 

![AWS 控制台显示不同的服务设置](https://img-blog.csdnimg.cn/img_convert/4b007746a44b7a4f7cce9ec5016acda9.png)

AWS 控制台的屏幕截图，显示 App Runner 服务的不同服务设置。

## AWS App Runner 背后隐藏着什么？

App Runner 构建在各种不同的 AWS 服务之上：

- [AWS CodeBuild](https://aws.amazon.com/codebuild/) – 用于构建、测试和打包应用程序。仅在您选择**从源代码构建时使用**。支持的语言是 Node.js 和 Python。
- [AWS Fargate](https://aws.amazon.com/fargate/)和[AWS ECS](https://aws.amazon.com/ecs/) – 用于**底层托管容器编排平台**。
- [AWS Auto Scaling](https://aws.amazon.com/autoscaling/) – 确保应用程序根据并发请求数进行扩展。
- [AWS Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/) – 确保负载在服务的不同实例之间均匀分布。
- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) – 用于存储 App Runner 日志（事件、部署和应用程序）和指标。
- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) – 用于为服务终端节点提供开箱即用的 SSL/TLS 证书。
- [AWS KMS](https://aws.amazon.com/kms/) – 用于加密源存储库和服务日志的副本。

除此之外，您还可以为该服务配置/注册您自己的（子）域，该域可能正在使用 Route53（在任何地方都找不到）。

如您所见，您无需考虑一大堆服务、配置、供应和管理。AWS App Runner 对您隐藏/抽象了这一点，让您专注于您的应用程序和业务问题。很整洁吧？

## 那么缺少什么？

这是一项新服务，因此某些功能（尚未）可用，您可能会在类似服务中找到这些功能。在试用 App Runner 时，我注意到了其中的几个，让我们来看看。

- 配置 App Runner 服务时，没有对 **Parameter Store 或 Secrets Manager** 的本机支持。您仍然可以通过应用程序中的 SDK 使用这些服务，但不能使用它们来为容器配置一些环境变量。
- 据我所知，App Runner（还）不允许您使用私有 VPC 内的资源（例如 RDS 实例）。
- 如果您是 AWS CDK 的粉丝，则必须记住，CDK 目前仅提供用于创建 App Runner 服务的 L1 构造。尚无 L2 支持，但似乎[在路线图上](https://github.com/aws/apprunner-roadmap/issues/7)。
- App Runner 仅支持**蓝/绿部署**模型，因此其他选项（例如滚动、金丝雀或流量拆分）目前不可用。
- 除 ECR 外，不支持其他容器存储库。
- 目前不支持除 GitHub 之外的其他 git 存储库。我希望他们很快添加 CodeCommit。
- 与大多数新服务一样，App Runner 目前仅在少数几个地区可用：欧洲（爱尔兰）、亚太地区（东京）、美国东部（弗吉尼亚北部）、美国东部（俄亥俄）、美国西部（俄勒冈）。
- 目前尚不支持 Java、Go、Rust、Ruby 等语言。因此，如果其中一种是您最喜欢的编程语言，则必须先创建容器映像，然后才能在 App Runner 中启动该服务。
- App Runner 不允许您扩展到零个实例。您将始终运行单个实例，并将为分配的资源付费。

## 价钱

当您查看AWS App Runner 页面上提到的[定价](https://aws.amazon.com/apprunner/pricing/)时，您可能会认为每月 5 美元非常便宜，但还有更多。他们提到它每月的费用约为 5 美元，但在阅读小写字母时，它表示运行单个实例的应用程序每天暂停约 22 小时就是这种情况。

使用 App Runner，您需要为**应用程序使用的计算和内存资源付费**。我在运行服务时注意到的是，从 CPU 的角度来看，您似乎只需要为实际花费的 CPU 资源付费。如果您的应用程序没有被使用，则您不会消耗 CPU，因此无需为 CPU 使用率付费。 

另一方面，内存将为您的应用程序保留，您将被收取相应的费用。您还需要为额外的 App Runner 功能付费，例如从源代码构建或自动部署。因此，定价一开始可能并不简单，但也不太复杂。在开始将应用程序迁移到 App Runner 之前，请务必尝试估算您的账单，因为从微型 EC2 实例迁移到 App Runner 可能不值得。

![AWS App Runner 定价](https://img-blog.csdnimg.cn/img_convert/37c1707be9f93d8f7fa52cf7ac3f7ac8.png)

## 最后的想法

我第一次阅读有关 App Runner 的公告时，就让我想到了 Google Cloud Run。我将 App Runner 视为 AWS 对 Google Cloud Run 的响应。它拥有强大的、固执己见的架构，并建立在其他出色的 AWS 服务之上。易用性非常好，无需大量容器经验即可快速上手。

我已经用一堆 Spring Boot 应用程序测试了 App Runner，在几分钟内启动并运行应用程序真的很容易。我认为 App Runner 对于使用 **REST API 或 Web 应用程序等用例创建小型应用程序**非常有用。我认为它非常适合**快速原型设计和部署 PoC 和 MVP**。

有一个[公共路线图](https://github.com/aws/apprunner-roadmap)开始形成一些很棒的功能。AWS 社区使用的流行工具正在积极添加对 App Runner 的支持。我很期待看到 App Runner 在一年后的发展。对技术感兴趣朋友可以加这个扣扣2779571288交流。