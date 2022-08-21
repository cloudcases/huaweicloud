# AWS Application Modernization vs Well Architected

- [Cloud](https://best.practices.cloud/archive.html?tag=Cloud)
- [AWS](https://best.practices.cloud/archive.html?tag=AWS)
- [Application-Modernization](https://best.practices.cloud/archive.html?tag=Application-Modernization)
- [应用现代化](https://best.practices.cloud/archive.html?tag=应用现代化)

-  2021年 08月12日
-  0 阅读

近两年，云应用领域出现了 `Application Modernization`(`应用现代化`)概念，业界云厂商、咨询公司，都在提这个概念。我们知道云应用架构的经典技术指导，是AWS的[应用架构完善系列](https://aws.amazon.com/cn/architecture/well-architected/)，此技术指南在2012年就有了第一版，其内容是AWS解决方案架构师根据客户需求以及客户应用在AWS上使用过程中沉淀出的经验总结。本文阐述AWS在应用现代化与架构完善两个概念之间差异，让读者有一个明确的认知。

## AWS 架构完善

### 缘起

最初是AWS解决方案架构师、专业服务顾问、企业客户支撑团队从与客户的交流中总结的经验，经过一段时间积累，大家觉得有必要把这些经验系统化整理并公布出来，让更多的AWS客户受益。

### 发展历程

- 2012：第一版，仅内部使用，用于解决方案架构师、客户支撑团队与客户交流，给客户建议使用。
- 2015：第四版，以[AWS架构完善白皮书](https://d1.awsstatic.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf)方式对外发布。此版本只有4个支柱，没有`卓越运营`这个支柱。
- 2016：第五版，增加`卓越运营`支柱。
- 2017：第六版，将5个支柱以[独立白皮书](https://aws.amazon.com/cn/whitepapers/?whitepapers-main.sort-by=item.additionalFields.sortDate&whitepapers-main.sort-order=desc&awsf.whitepapers-content-category=content-category%23well-arch-framework&whitepapers-main.q=pillar&whitepapers-main.q_operator=AND&awsf.whitepapers-content-type=*all&awsf.whitepapers-tech-category=*all&awsf.whitepapers-industries=*all&awsf.whitepapers-business-category=*all&awsf.whitepapers-global-methodology=*all)方式发布。
- 2018：AWS Console控制台增加架构完善评估工具。

### 更新机制

每年更新一个版本。

AWS 客户技术团队（解决方案架构师、专业服务顾问、企业客户支撑团队）使用[KAIZEN改善](https://www.kaizen.com/what-is-kaizen.html)流程来收集客户数据，通过这些客户反馈来决定架构完善内容的更新。

## AWS 应用现代化

### 是什么

AWS Application Modernization 是一套方法论，指导遗留应用(Legacy Application)(包含人员/流程)、如何迁移到AWS([Migrate to AWS](https://aws.amazon.com/cloud-migration/))，实现应用现代化。 应用现代化方法论中，包含经典微服务改造的方法论。

应用现代化给企业带来的价值如下（约等于AWS对企业提供的价值）：

1. 降低[TCO](https://aws.amazon.com/economics/)
2. 用户增长
3. 保护已有投资

### 理论支撑

AWS 应用现代化与AWS提出的应用迁移到云的[6R理论](https://docs.aws.amazon.com/whitepapers/latest/aws-migration-whitepaper/the-6-rs-6-application-migration-strategies.html)是相对应的。既然应用现代化就是应用迁移到AWS上，理论支撑也就是[AWS云迁移白皮书](https://docs.aws.amazon.com/whitepapers/latest/aws-migration-whitepaper/welcome.html)。

## 两者对比

### 1. 是什么

- 应用现代化：应用迁移到AWS并获得相应价值的一套方法论。
- 架构完善：应用托管在AWS上的架构最佳实践，是一套规则集合。还配套了AWS Audit Manager服务的[架构完善框架](https://docs.aws.amazon.com/audit-manager/latest/userguide/well-architected.html)，帮助客户持续审计架构是否符合AWS最佳实践。

### 2. 受众

- 应用现代化：管理者，通常是CXO。
- 架构完善：技术人员，架构师、开发者。

### 3. 价值

- 应用现代化：降低TCO、用户正常、保护已有投资。
- 架构完善：在AWS上构建`安全、高可靠、低成本、自动化运维`的应用架构。

### 4. 落地方式

- 应用现代化：方法论、或专家咨询服务。
- 架构完善：技术白皮书 + [评估工具](https://aws.amazon.com/cn/well-architected-tool/) + 行业应用实践（AWS 架构完善的剖析） + 自动化持续评估服务（Audit Manager 架构完善框架）。

## Reference

[AWS re:Invent 2020: Application modernization](https://www.youtube.com/watch?v=CdEDhWdmutQ)

[Modernize Your Applications, Drive Growth and Reduce TCO](https://aws.amazon.com/cn/enterprise/modernization/)

[History of AWS well architected framework](https://aws.amazon.com/cn/blogs/architecture/announcing-the-new-version-of-the-well-architected-framework/)

[KAIZEN](https://www.kaizen.com/what-is-kaizen.html)



[AWS Application Modernization vs Well Architected - Best practice on cloud (practices.cloud)](https://best.practices.cloud/2021/08/12/AWS-Application-Modernization-and-Well-Architected.html#:~:text=AWS Application Modernization 是一套方法论，指导遗留应用 (Legacy,Application) (包含人员%2F流程)、如何迁移到AWS (Migrate to AWS)，实现应用现代化。)