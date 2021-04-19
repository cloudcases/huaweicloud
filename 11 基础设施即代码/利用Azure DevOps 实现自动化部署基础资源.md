# 利用Azure DevOps 实现自动化部署基础资源

[![img](https://upload.jianshu.io/users/upload_avatars/20063139/35205e2b-ec34-4136-a6dd-0caa2ca036b9.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/bf0665ecc306)

[Java互联网架构师小马](https://www.jianshu.com/u/bf0665ecc306)关注

0.6072021.01.18 22:17:35字数 1,451阅读 227

# 前言

上一篇我们结合学习 Azure Traffic Manger 的内容，做了一个负载均衡的基础设施架构。通过 Terraform 部署执行计划，将整个 Azure Traffic Manager 结合 Azure Web App 的架构快速部署到云上。然后再将我们的示例项目代码部署到对应的不同区域的Azure Web 应用程序上。最后Azure Traffic Manager 将不同地理位置的用户的访问请求转发到后端的 Azure Web 应用上。

这时，又有人提问了，现在都流行 DevOps ，整个应用层面的项目代码都可以实现 CI/CD 整个过程，那这些基础设施代码可以实现 CI/CD 吗？

答案是肯定的，今天要演示的正如文章标题那样，利用 Azure DevOps 快速实现自动化部署基础设施资源。

开始内容之前，我们先看看整个 pipeline 过程

![img](https://upload-images.jianshu.io/upload_images/20063139-5734dda9fea494d1?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

# 正文

**1、Azure DevOps 创建项目**

Azure DevOps 上创建 "CnBateBlogWeb_Infrastructure" 项目，Azure DevOps 地址：dev.azure.com。

![img](https://upload-images.jianshu.io/upload_images/20063139-6f3573723c2207ea?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

**2、配置Azure DevOps Pipeline 环境**

选择 “Pipelines =》Releases”，并且点击 “New pipeline” 创建新的 pipeline

![img](https://upload-images.jianshu.io/upload_images/20063139-22c40bcbdb428bc7?imageMogr2/auto-orient/strip|imageView2/2/w/1124/format/webp)

利用Azure DevOps 实现自动化部署基础资源

选择模板页面，我们先选择 “Empty job”

![img](https://upload-images.jianshu.io/upload_images/20063139-a7101a66a8e941be?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

修改 "Stage name"，并且点击 “x” 进行关闭此页面

![img](https://upload-images.jianshu.io/upload_images/20063139-8a0d5ab2d3d9fca6?imageMogr2/auto-orient/strip|imageView2/2/w/993/format/webp)

利用Azure DevOps 实现自动化部署基础资源

接下来，我们需要给当前 "pipeline" 添加 “artifact”

![img](https://upload-images.jianshu.io/upload_images/20063139-89966c18181cd306?imageMogr2/auto-orient/strip|imageView2/2/w/1028/format/webp)

利用Azure DevOps 实现自动化部署基础资源

选择 “GitHub”，点击 “Service” 旁边的 “+” ，添加新的 “github connection”

![img](https://upload-images.jianshu.io/upload_images/20063139-324102367c939d22?imageMogr2/auto-orient/strip|imageView2/2/w/643/format/webp)

利用Azure DevOps 实现自动化部署基础资源

输入 “Connection name” ：“github_connection_yunqian44”，并且点击 “Authorize using OAuth” 进行登陆 github 认证

![img](https://upload-images.jianshu.io/upload_images/20063139-0675d7721c7464dc?imageMogr2/auto-orient/strip|imageView2/2/w/633/format/webp)

利用Azure DevOps 实现自动化部署基础资源

等待验证完成后，我们需要选择 terraform 对应的代码仓库，点击图中的箭头指向的位置

![img](https://upload-images.jianshu.io/upload_images/20063139-32ae7528dc042d2b?imageMogr2/auto-orient/strip|imageView2/2/w/645/format/webp)

利用Azure DevOps 实现自动化部署基础资源

输入相应参数

Source（repository）: "Terraform_Cnbate_Traffic_Manager"

Default branch（默认仓库）：“remote_stats”

Default version：“Lastest from the default branch”

Source alias 选择默认后，点击 “Add” 进行添加

![img](https://upload-images.jianshu.io/upload_images/20063139-858e9efe87f01ffe?imageMogr2/auto-orient/strip|imageView2/2/w/631/format/webp)

利用Azure DevOps 实现自动化部署基础资源

接下来就需要我们添加 “task” 了，点击图中箭头指向

![img](https://upload-images.jianshu.io/upload_images/20063139-c1da2ae1894cd5ed?imageMogr2/auto-orient/strip|imageView2/2/w/846/format/webp)

利用Azure DevOps 实现自动化部署基础资源

点击 “Agent Job” 旁白 “+”，并在右边的输入框中输入 “Azure Key Vault”，选中图中的“Azure Key Vault”。点击 “Add”，添加 task。

![img](https://upload-images.jianshu.io/upload_images/20063139-aa667393fed5d47f?imageMogr2/auto-orient/strip|imageView2/2/w/981/format/webp)

利用Azure DevOps 实现自动化部署基础资源

修改相关参数

Display name：“Azure Key Vault：Get Storage Access Secret”

Azure subscription 选择当前自己的订阅

Key vault 选择：“cnbate-terraform-kv”

Secrets filter（机密过滤器）：“terraform-stste-storage-key”，如果选择默认 “*”，则下载选定密钥库的所有机密

![img](https://upload-images.jianshu.io/upload_images/20063139-646b9e4b28d09595?imageMogr2/auto-orient/strip|imageView2/2/w/968/format/webp)

利用Azure DevOps 实现自动化部署基础资源

然后添加新的 task，搜索 “Terraform”，选择 “Terraform tool install”

![img](https://upload-images.jianshu.io/upload_images/20063139-c1b6ef1853a72397?imageMogr2/auto-orient/strip|imageView2/2/w/966/format/webp)

利用Azure DevOps 实现自动化部署基础资源

修改 Terraform 版本 “0.14.3”

![img](https://upload-images.jianshu.io/upload_images/20063139-de1686632b5364b4?imageMogr2/auto-orient/strip|imageView2/2/w/988/format/webp)

利用Azure DevOps 实现自动化部署基础资源

接下来再添加 Terraform 新的 task，选择 “Terraform”，点击 “Add”

![img](https://upload-images.jianshu.io/upload_images/20063139-d422926b74e16f36?imageMogr2/auto-orient/strip|imageView2/2/w/992/format/webp)

利用Azure DevOps 实现自动化部署基础资源

修改相应参数：

Display name：“Terraform：Init”

Command 选择：“init”

Addition command arguments：”-backend-config="access_key=$(terraform-stste-storage-key)"“ （tf 代码中没有access_key 的配置信息，所以我们需要在 terraform init 过程中传递此参数）

**AzureRM backend configuration**:

Azure subscription：选择当前自己的订阅

Resource group：”Web_Test_TF_RG“

Storage account：”cnbateterraformstorage“

Container：”terraform-state“

Key：”cnbate.terraform.stats“

![img](https://upload-images.jianshu.io/upload_images/20063139-8b27320367a1c1e1?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

设置完 terraform init 相应参数后，我们需要修改 terraform init 的工作目录，选择完成后，点击 ”Add“ 进行添加操作

![img](https://upload-images.jianshu.io/upload_images/20063139-d14b06e7f695a5b6?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

再次添加 ”Terraform“ task，用来配置 生成 terraform 部署计划

![img](https://upload-images.jianshu.io/upload_images/20063139-8d0fbe75ce7320c1?imageMogr2/auto-orient/strip|imageView2/2/w/964/format/webp)

利用Azure DevOps 实现自动化部署基础资源

Display name：”Terraform：plan“

Command 选择：”plan“

Configuration directory: 选择 terraform 代码所在目录

Azure subscription 选择当前自己的Azure订阅

![img](https://upload-images.jianshu.io/upload_images/20063139-b0892e295492474d?imageMogr2/auto-orient/strip|imageView2/2/w/966/format/webp)

利用Azure DevOps 实现自动化部署基础资源

最后一步，再次添加 Terraform 新的 task，选择 “Terraform”，点击 “Add”

![img](https://upload-images.jianshu.io/upload_images/20063139-a06ec1ede134e6c7?imageMogr2/auto-orient/strip|imageView2/2/w/979/format/webp)

利用Azure DevOps 实现自动化部署基础资源

修改相应参数：

Display name：”Terraform：auto-apply“

Command：”validate and apply“

Configuration directory：选择 terraform 代码的工作目录

Additional command arguments：“-auto-approve”

Azure subscription：选择当前自己订阅

![img](https://upload-images.jianshu.io/upload_images/20063139-37a86cb8426832ca?imageMogr2/auto-orient/strip|imageView2/2/w/973/format/webp)

利用Azure DevOps 实现自动化部署基础资源

修改当前 pipeline 名称，点击 “Save” 进行保存

![img](https://upload-images.jianshu.io/upload_images/20063139-6e9495ea472043f4?imageMogr2/auto-orient/strip|imageView2/2/w/998/format/webp)

利用Azure DevOps 实现自动化部署基础资源

最后，我们需要设置 pipeline 的触发条件

开启持续部署触发，每次在所选存储库中发生Git推送时触发pipeline，接下来添加分支筛选条件

Type：Include，Branch：“remote_stats”，也就是说每当 “remote_stats” 发生git 推送的时候，触发此 pipeline

设置完毕后，点击 “Save” 进行保存

![img](https://upload-images.jianshu.io/upload_images/20063139-3bae44ad192ada39?imageMogr2/auto-orient/strip|imageView2/2/w/992/format/webp)

利用Azure DevOps 实现自动化部署基础资源

**3、对Service principal 设置Key Vault 访问策略**

具体操作步骤，我就不再演示了，大家可以参考：Azure Kay Vault（一）.NET Core Console App 获取密钥保管库中的机密信息

![img](https://upload-images.jianshu.io/upload_images/20063139-03cc99e95b5a4b11?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

**4、测试 Terraform 自动化部署**

回到 terraform 代码上，我们提交并且推送新的代码到 “remote_stats” 远端分支上

![img](https://upload-images.jianshu.io/upload_images/20063139-e1c7636a0461aabd?imageMogr2/auto-orient/strip|imageView2/2/w/985/format/webp)

利用Azure DevOps 实现自动化部署基础资源

这个时候，我们回到Azure DevOps 上我们就可以看到，当前 pipeline 已被触发。

![img](https://upload-images.jianshu.io/upload_images/20063139-18a04be472be344a?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

等待稍许时间，我们可以看到当前 pipeline 执行日志，部署成功

![img](https://upload-images.jianshu.io/upload_images/20063139-c280fef315d5b3ec?imageMogr2/auto-orient/strip|imageView2/2/w/985/format/webp)

利用Azure DevOps 实现自动化部署基础资源

同样，我们需要登陆Azure Portal 上确认 使用 terraform 管理的资源是否创建成功

![img](https://upload-images.jianshu.io/upload_images/20063139-50c9d85040536ef4?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

利用Azure DevOps 实现自动化部署基础资源

查看Storage Account 的 Bolb 中的 terraform 状态文件

![img](https://upload-images.jianshu.io/upload_images/20063139-ff70805bd4accc99?imageMogr2/auto-orient/strip|imageView2/2/w/598/format/webp)

利用Azure DevOps 实现自动化部署基础资源

Bingo，今天的内容就到此结束，下期我们继续/

**最后，大家试验完，记得删除刚刚使用 terraform 部署的资源，防止 Auzre 扣费**

# 结尾

今天的内容比较多，基本上全都是需要在Azure DevOps 上进行操作的，大家要多加练习，熟能生巧。本文所分享的内容也存在着很多我自己的一些理解，有理解不到位的，还希望多多包涵，并且指出不足之处。

> 作者：Allen
>
> 链接：[https://www.cnblogs.com/AllenMaster/p/14274035.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2FAllenMaster%2Fp%2F14274035.html)





原文：https://www.jianshu.com/p/5659ccdef4cd