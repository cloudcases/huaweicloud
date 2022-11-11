# Terraform Module大赛来啦，AirPods Pro免费送！

2019-11-12 5397

**简介：** Terraform 模板征集大赛现火热招募中，价值两千元的AirPods Pro免费送，快来参加吧！

【注】本文章转自【[阿里云开放平台开发者挑战赛](https://developer.aliyun.com/article/726543)】

# 赛程介绍

[![750x312.png](https://ucc.alicdn.com/pic/developer-ecology/465b6e64b4f34f429fbb67531a8b5915.png)](https://survey.aliyun.com/apps/zhiliao/T0WeJX7u)

Terraform 模板征集大赛现火热招募中！！！本次大赛邀请所有对Terraform感兴趣的开发者加入“阿里云 Terraform Module”建设队伍，全程在线自主报名和提交作品。足不出门，动动指尖就可提升自身影响力，更有**价值两千元的AirPods Pro耳机**和**Logitech无线键盘大奖**拿回家，**优秀的模板提供者还有开放平台官网展示机会**。还在等什么？快快报名参加吧！

![屏幕快照 2019-11-11 上午11.23.49.png](https://ucc.alicdn.com/pic/developer-ecology/bdb544b274ca43b7a9dc973b5953a2c3.png)

# [点击报名](https://survey.aliyun.com/apps/zhiliao/T0WeJX7u)

**Terraform是什么？**
[Terraform](https://registry.terraform.io/)是由Hashicorp公司于2014年推出的一个开源项目，是一个典型的IaC工具。阿里云作为国内第一家与 Terraform 集成的云厂商，经过两年多的努力，目前已经提供了超过 148 个 Resource 和 98 个 Data Source，覆盖计算，存储，网络，负载均衡，CDN，容器服务，中间件，访问控制，数据库等超过30款产品，现公开有奖征集 Terraform Module。**参赛前请详细阅读以下活动规则，大奖轻松拿回家**~~~

## 1.活动规则



#### 1.1 代金券申请规则：

**为了解决您参加活动的云资源使用问题，大赛组委会将提供价值300元的全站云资源代金券**。请于12月06日前点击”[代金券申请](https://survey.aliyun.com/apps/zhiliao/aTgxW_AJ)“，上传Module的简单介绍或初步架构图，同时提供您开发Module的阿里云的账号ID（[具体可登录阿里云官网个人中心查看](https://account.console.aliyun.com/?spm=a2c81.fb365c1.amxosvpfn.7.459c1127ttjhOO#/secure)）。
活动审核专家将在3个工作日对审核通过的申请账号ID发放代金券。（**注：同质化的架构图，以第一个申请者为准发放代金券**）。使用有效期为一个月，上限100，先到先得！

#### 1.2 Terraform Module编写规则：

为了保证所提供的Module的质量和使用体验，**编写的Module请尽量实现如下几条规则，会帮助您更快速的通过奖品评审：**

- 所提交的Module可以是对已注册Module https://registry.terraform.io/ 的重构和完善，也可以是对ROS模板或者其他友商模板的转换，也可以基于自定义的架构。来源不限，但要保证所要实现的**Module必须从实际使用场景出发，以解决某种使用需求为目标**，为Module使用者提供“开箱即用”的体验；
- 所提交的Module只能包含对阿里云资源的管理，在此基础上，所使用的Resource和Data Source 可以是包括其他Provider提供的；
- 每个Module需要包含一个README（英文），并且**README中至少要包含Module具体使用场景描述，涉及到的阿里云资源的介绍，Module背后的资源拓扑图，Module中所有的入参和出参的介绍和展示**，可参考官方提供的Module Demo：[terraform-provider-demo](https://github.com/terraform-alicloud-modules/terraform-alicloud-demo)；
- 架构图中所使用的图标推荐使用阿里巴巴官方提供的图标，可在 https://www.iconfont.cn/ 中查询并下载；
- 每个Module都需要显示声明Provider，并在Provider中设置参数 `configuration_source=<Github ID>/<Module 名字后缀>` 来对所编写的Module进行打标，同时Provider需要支持对Region的设置，以支持多Region的。可参考Module Demo：[terraform-provider-demo](https://github.com/terraform-alicloud-modules/terraform-alicloud-demo/blob/master/main.tf#L1)；
- 每个Module都需要配置[Terratest测试](https://github.com/gruntwork-io/terratest)；
- 每个Module需要通过“terraform fmt”命令来格式化代码；
- 每个Module都应该在Terraform version 0.12.x 上开发。

#### 1.3 Module提交规则

参赛者完成模板编写后，请提交Terraform Module至Github，并尽快根据[官方发布规则](https://www.terraform.io/docs/registry/modules/publish.html)完成对Module的注册和发布，同时尽快（**截止日期12月12号前**）点击“[**提交模板**](https://survey.aliyun.com/apps/zhiliao/H30MBTn1)”上传模板URL以确认提交，奖品评审会在此处提交信息后启动，请务必按时确认提交。（注：提交至GitHub的Module将是开源的状态）

# [**Module提交模板**](https://survey.aliyun.com/apps/zhiliao/H30MBTn1)

#### 1.4 奖品评审规则：

收到Module提交确认问卷后，大赛组委会技术专家将其进行审核和评比，审核结果将以邮件/电话/钉钉群的方式通知。最终的奖品评审分为“优秀奖评审”和“特等奖评审”，优秀奖提供**50份Logitech无线键盘**，先到先得；特等奖提供**3份最新款Apple AirPods Pro耳机**，按照特等奖评审规则质量最高的前3名获得。（注：特等奖获得者不占用优秀奖名额）

**4.1优秀奖评审规则：**

- 态度端正，Module代码无抄袭行为；
- 每个Module应包括完整的包含README（英文编写），Module代码，Module的Terratest测试；
- 符合评审对“优秀Module”的判定预期。

**4.2特等奖评审规则：**

- 包含所以优秀奖评审规则；
- Module的可用性：所提交的Module需要通过内部的CI机制进行验收，不通过的会以邮件的方式或者Github Issues的方式进行反馈。可持续提交和完善，直到验收通过；

**加分项：**

- 含README（中文）：除了英文的README，还写了一个中文的README-ZH；
- Module Example：为Module配备了相应的Example，帮助开发者熟悉和使用Module；
- Terratest测试case的数量多，覆盖面广：Terratest的case越多，覆盖面越广，Module的稳定性越高，评分越高；
- 基于对Module的打标，Module的实际调用量越多，评分越高。



## 2.相关教程

**1.Terraform Module开发指南**：https://yq.aliyun.com/articles/642624
**2.Terraform 相关课程**:https://developer.aliyun.com/article/720999?spm=a2c6h.12873581.0.0.31631f1e18J5nN&groupCode=openapi
**3.阿里云现有Module**：https://registry.terraform.io/browse/modules?provider=alicloud



[Terraform Module大赛来啦，AirPods Pro免费送！-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/726705?spm=a2c6h.12873639.article-detail.75.5ce132e69ddZZL)