# 实施 AWS Landing Zone 1: 背景及架构介绍

 发表于 2020-03-08 分类于 [AWS](https://notes.yuanlinios.me/categories/AWS/) ， [Landing Zone](https://notes.yuanlinios.me/categories/AWS/Landing-Zone/) 阅读次数： 1245 Disqus： 本文字数： 1.8k 阅读时长 ≈ 2 分钟

本文介绍了 AWS 多账号管理的需求, Landing Zone 方案的背景及基本结构, Landing Zone 方案和 Control Tower 的对比与选择



## 多账户的需求和面临的挑战

随着越来越多的工作负载迁移到 AWS 公有云, 过去的单个账户逐渐无法满足复杂环境的需要: 难以清晰界定的职责边界, 资源无法有效隔离, 对不同部门团队的计费和审计等. 此时会自然而然的引入多账户策略: 不同部门/团队, 不同环境使用不同的 AWS 账号. 优点很明显

- 降低触及单个账户资源上限的风险
- 不同团队的职责分离
- 开发/测试/生产环境的资源隔离
- 减少人为故障的影响范围

AWS Organization 提供了多账户环境的基础. 然而创建账户, 加入组织并完成初始化基线配置是一项繁琐的工作. 如何确保各个账户的合规性和配置的一致性? 如果后期配置基线发生变更, 又如何将基线变更追溯的应用到所有的现存账户上? Landing Zone 方案旨在解决这些问题.

## Landing Zone 简介

Landing Zone 是 AWS 在 2018 年推出的一套解决方案 (注意: 并不是一个 AWS 原生服务), 用来帮助用户根据 AWS 最佳实践, 快速建立安全的多账户环境

![img](https://notes.yuanlinios.me/2804254/aws-landing-zone-architecture.png)

上图为 Landing Zone 的基本结构:

- 核心账户: master, shared services, log archive, security
  - Master: 也称为 Organization account. 是创建 AWS Organization, Landing Zone 基础组件 (Service Catalog, CodePipeline, State Machines) 的地方
  - Shared Services: 公共服务所在的账号
  - Log Archive: 存放所有账户 Cloudtrail 和 Config log的 S3 bucket 所在的账户
  - Security: Guarduty master 所在账户, 全局 admin/readonly 角色所在账户
- Accounting Vending Machine (AVM): 是 Landing Zone 在 Service Catalog 里创建的一个产品, 通过运行 AVM 来创建新账户并应用配置基线
- Landing Zone Codepipline: 配置变更流水线. 触发该流水线来更改核心账户资源, Service Control Policies 和基线资源等. 该流水线最终会调用 AVM 将基线资源变更应用到所有账户上

各个组件/服务更深入的介绍将在后续的系列文章中给出.

## Control Tower 还是 Landing Zone?

Landing Zone方案涉及多种AWS服务, 其部署较为复杂, 因此在 2019 年 AWS 推出了 Control Tower 服务, 旨在帮助用户通过最简单的方式设置**全新**的多账户 AWS 环境. 其背后依然是 Landing Zone, 但是隐藏了实现细节和基础服务.

用户在实际生产环境下该如何选择?

- Control Tower 只适用于全新部署 (greenfield). 如果你已经建立了多账户的环境 (Organization, SSO), 又不想迁移账户, 此时应该选择 Landing Zone
- Landing Zone 具有更强的定制化 (当然这也正是它更加复杂的原因), 所有的基线资源都通过 Cloudformation template 创建, 用户可以方便的修改预设基线资源或者添加自定义资源
- Landing Zone提供了 CI/CD 流水线, 用户可以将其配置保存在代码仓库中以实现版本控制

## 在现存多账户环境下实施 Landing Zone

由于实际环境的复杂性, Landing Zone 方案在实施 AWS 最佳实践的同时, 不可避免的会对用户的环境做一些假设, 因此其部署的某些服务或者服务的设置对特定的用户可能并不适用. 此时需要对其预设配置进行调整.

举个例子, 默认 Landing Zone 会在所有账号和共享服务账号 (shared services account) 之间做 VPC peering 以便所有账户可以访问共享服务账号中的公共服务. 然而 VPC peering 本身有诸多限制, 企业环境下可能更多会选择 transit VPC 架构或者使用 transit gateway. 此时用户需要修改 Landing Zone 配置模板以匹配自身环境. 在系列的后续文章中我将逐一列示.

AWS 声明 Landing Zone 必须由客户的 AWS account 团队或者经过认证的合作伙伴来部署, 以确保方案的实施成功. 如果你想自己实施, 请自行评估风险

- **本文作者：** L. Yuan
- **本文链接：** https://notes.yuanlinios.me/2804254/
- **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh) 许可协议。转载请注明出处！

[# AWS](https://notes.yuanlinios.me/tags/AWS/) [# Landing Zone](https://notes.yuanlinios.me/tags/Landing-Zone/)

[实施 AWS Landing Zone 2: 实施前场景与目标规划 ](https://notes.yuanlinios.me/4237658498/)



https://notes.yuanlinios.me/2804254/