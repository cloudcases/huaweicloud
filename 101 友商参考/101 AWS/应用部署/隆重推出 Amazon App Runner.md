# 隆重推出 Amazon App Runner

by [AWS Team](https://aws.amazon.com/cn/blogs/china/author/victor-yuan/) | on 10 JUN 2021 | in [Containers](https://aws.amazon.com/cn/blogs/china/category/containers/) | [Permalink](https://aws.amazon.com/cn/blogs/china/grand-launch-of-aws-app-runner/) | [ Share](https://aws.amazon.com/cn/blogs/china/grand-launch-of-aws-app-runner/#)

今天，我们很高兴地宣布推出 [Amazon App Runner](https://console.aws.amazon.com/apprunner/home)，这是在 Amazon Web Services中构建和运行容器化 Web 应用程序的最简单方法。App Runner 为您提供**完全托管的容器原生服务**。**无需配置编排器、无需构建流水线、无需优化负载均衡器，也无需轮换 TLS 证书**。当然，也没有需要管理的服务器。

App Runner **按秒计费**，可提供运行安全生产工作负载所需的一切。只需单击几下，即可运行一个具有公共终端节点、经过验证的 TLS 证书并可自动扩展的容器。

通过 App Runner，您可以使用现有容器，也可以使用集成容器构建服务直接从代码仓库转移到已部署的应用程序。**构建服务可以连接到 GitHub仓库，提供一个可自动部署更改的 `git push`工作流**。为了更好地控制构建过程，App Runner 使用最新版本的 [Amazon Copilot](https://github.com/aws/copilot-cli) 来容器化应用程序并自动化其他 Amazon Web Services服务（例如 [Amazon DynamoDB](http://aws.amazon.com/dynamodb)）。

为了展示 App Runner 的一些独特优势并了解如何使用它，我们来创建一项服务。

 

## 部署您的第一个容器

我们来演示如何在 App Runner 上部署示例容器。在本教程中，我们已经在[这里](http://gallery.ecr.aws/aws-containers/hello-app-runner)为您发布了一个预构建容器示例。此应用程序为您提供了一个基本的首页，并为每项服务生成一个独有的横幅图。[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/image-1(1).jpg)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/image-1(1).jpg)

在 [App Runner 控制台](https://console.aws.amazon.com/apprunner/home?region=us-east-1)中，您仅需提供三条信息就可以部署成容器。首先指定容器注册表类型。对于本例，请指定 **Amazon ECR Public** 作为提供商。在容器镜像中输入`public.ecr.aws/aws-containers/hello-app-runner:latest`[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner2.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner2.jpg)

接下来，为服务命名（可按喜好随意命名），然后指定端口 `8000 `。您还可以为应用程序提供配置，例如环境变量、运行状况检查终端节点、扩展要求和服务标签。对于本例，将使用默认值。
[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner3.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner3.jpg)

单击 **Create & Deploy (****创建和部署)**，您的应用程序将部署在具有 TLS 证书的托管负载均衡器之后。自动扩展是基于并发请求的。每项服务还为容器提供 CloudWatch 指标和日志。在 Amazon Web Services处理基础设施时，您可以专注于构建应用程序。
[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner4.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner4.jpg)

 

## 自动构建容器

如果您想将代码转换为 App Runner 服务，可以直接使用 GitHub 仓库，而不必构建或推送容器。**构建服务为运行时（例如 Python 和 Node.js）提供了基本容器**，因此您可以减少安全修补。**App Runner 具有集成的容器构建服务**，按分钟计费，可满足您的所有 App Runner 容器需求

例如，开源应用程序托管在[此处的](http://github.com/aws-containers/hello-app-runner) GitHub 上。它使用暂存盘空间生成横幅图，并在 `/metrics`导出该服务的 Prometheus指标，以显示在服务请求之间如何在内存缓存中使用。

要使用完全托管的 App Runner 构建管道部署它，请在 GitHub 账户中创建一个仓库分支。在 App Runner 中创建新服务，然后选择 **Source code repository (****源代码仓库****)** 类型。连接到 GitHub 账户并选择仓库和分支。
[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner5.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner5.jpg)

选择 **Automatic (****自动)** 作为部署触发器类型，以便在向主分支提交时自动部署容器。
[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner6.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner6.jpg)

指定应用程序运行时 – **Python 3** 或 **Node.js 12**。然后，指定构建命令、启动应用程序的命令以及要公开的端口。
[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner7.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/grand-launch-of-aws-app-runner7.jpg)

您还可以通过在仓库中创建` apprunner.yaml`文件来控制构建过程。以下是示例 Python 应用程序的一个最小` apprunner.yaml`文件。

```python
version: 1.0
runtime: python3
build:
 commands:
  build:
  - yum install -y pycairo
  - pip install -r requirements.txt
run:
 command: python app.py
 network:
   port: 8000
```

Python

您可以[在文档中](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html)查看更多 `apprunner.yaml`示例。

在仓库中拥有 `apprunner.yaml`文件后，您可以直接在仓库中控制容器构建，而无需在控制台中更新服务。**推送代码后，将构建新版本的容器、推送到注册表并进行部署**。App Runner 会为您处理一切。

 

## 合作伙伴在行动

我们还与 Pulumi 和 Logz.io 等 Amazon Web Services合作伙伴合作，将他们的产品与 App Runner 集成。同样，Trek10 等 Amazon Web Services咨询合作伙伴可以帮助客户利用 App Runner 进行云原生架构设计。

[**Pulumi**](https://www.pulumi.com/blog/deploy-applications-with-aws-app-runner?utm_campaign=app_runner&utm_source=amazon.com&utm_medium=partner-blog) –“Amazon App Runner 简直太棒了。它采用领先的容器技术构建，但开发人员无需具备任何容器专业知识即可运行他们的 Web 应用程序和服务。App Runner 已成为 Pulumi 中的资源，我们很高兴能够为它提供支持。”

[**Logz.io**](http://logz.io/blog/aws-app-runner) –“我们很高兴地宣布，我们将利用我们领先的开源工具，即与 ELK 兼容的日志管理、基于 Prometheus 的基础设施监控和基于 Jaeger 的分布式跟踪，为 Amazon App Runner 提供监控支持。通过将 Logz.io 与 Amazon App Runner 集成，客户可以全面了解其应用程序和基础设施的性能，从而更好地排查技术堆栈问题并扩展技术堆栈，同时节省工程资源和时间。”

[**Trek10** ](https://www.trek10.com/blog/aws-app-runner)–“在 Trek10，我们构建解决方案，帮助客户在 Amazon Web Services上更快地构建和创新。我们对 Amazon App Runner 的推出感到非常兴奋，因为它可以通过分离基础设施和简化 Web 应用程序部署来加速创新。我们期待能够利用 App Runner 来帮助我们的客户构建和运行云原生应用程序。”

我们还有来自 [MongoDB](https://www.mongodb.com/blog/post/build-applications-and-apis-faster-with-mongodb-atlas-and-aws-app-runner)、[Datadog](https://www.datadoghq.com/blog/aws-app-runner-monitoring/) 和 [HashiCorp](https://www.hashicorp.com/blog/announcing-launch-day-support-for-aws-app-runner-in-the-terraform-aws-provider) 的激动人心的集成，这可让 App Runner 客户使用其已知和信任的工具与服务。

 

## 了解更多

有关 App Runner 的更多信息，请查看[文档](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html)和 [apprunnerworkshop.com](http://apprunnerworkshop.com/) 上的研讨会。研讨会将引导您完成自动滚动更新，并演示如何将应用程序连接到数据库。您还可以在 [5 月 19 日星期三下午 12:00 (PDT)/下午 3:00 (EDT)](https://www.youtube.com/watch?v=97Ua6Gv_HSo) 访问 [Containers from the Couch](https://youtube.com/containersfromthecouch) 观看我们的直播，届时，我们将介绍该研讨会并解答您的问题。

App Runner 是最新推出的 Amazon Web Services服务，可帮助您大规模运行容器。它将 Amazon Web Services上运行数十亿容器的多年运营经验融入其中，任何人都能轻松上手。告诉我们您希望看到哪些功能，并分享您在 [GitHub 上的公共 App Runner 路线图上](https://github.com/aws/apprunner-roadmap/projects/1)构建的内容。

 

## 本篇作者

![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/Author/jgarrison.jpg)

### Justin Garrison

Amazon Web Services容器团队的高级开发人员宣传官。长期起来，他一直深切关注开源社区，并为开源做出自己的贡献。在加入 Amazon Web Services之前，Justin 曾为 Disney+ 和动画电影（《冰雪奇缘 2》和《海洋奇缘》）构建基础设施。您可以通过 @rothga 在 Twitter 上关注他



[隆重推出 Amazon App Runner | 亚马逊AWS官方博客](https://aws.amazon.com/cn/blogs/china/grand-launch-of-aws-app-runner/)