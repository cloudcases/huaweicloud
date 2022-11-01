# 《SDL安全体系实践》话题材料分享

2020-04-14阅读 8410

学习材料owasp主动控制项目SDL 成熟度框架:bsimm & OWASP samm威胁建模：McGraw SARA；威胁建模McGrawSARA什么阶段做安全评估适用范围方法论OWASP ASVS缓解机制列表（含公共组件）参考OWASP_Cheat_Sheet_SeriesIEEE安全设计中心（CSD）提出的Top-10-Flaws参考材料

# **学习材料**

​    近日学习了《大型互联网应用安全SDL体系建设实践》，希望各位大神多介绍下工作沉淀的宝贵经验，一起建设应用安全生态，通过学习更加认识到并没有唯一的最佳安全实践，适合各个公司可以落地的就是对的方向，对SDL建设期间选择正确的技术决定工作的重心，安全并非是技术的争论，是非安全的，工程化的，体系化的探索。分享材料最后特意提到了几个材料引用：

1. owasp主动控制项目
2. SDL 成熟度框架:bsimm & OWASP samm
3. 威胁建模：McGraw SARA
4. OWASP ASVS
5. 缓解机制列表（含公共组件）参考OWASP_Cheat_Sheet_Series
6. IEEE安全设计中心（CSD）提出的Top-10-Flaws

学海无涯，回头是岸。趁着假期学习一下，现记录如下与众读者共勉。

# **owasp主动控制项目**

第一项owasp主动控制项目可以参考本公众号之前介绍过的《[项目发布 | OWASP Top 10 Proactive Controls V3中文版](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804532&idx=1&sn=dd88fb286b77bc928b2d691cd3442089&chksm=8445142ab3329d3c1ff3daf558cb888d0bcfa05f830ba11eaaaf552b98843d116e21412ea10a&scene=21#wechat_redirect)》，不同于我们熟悉的owasp top10关注主流漏洞排行，主动控制项目关注软件的安全防御构建需求，包括十项：

C1：定义安全需求（Define Security Requirements）

C2：使用安全框架和库（Leverage Security Frameworks and Libraries）

C3：安全的[数据库](https://cloud.tencent.com/solution/database?from=10680)访问（Secure Database Access）

C4：数据编码与转义（Encode and Escape Data）

C5：验证所有输入（Validate All Inputs）

C6：实现数字身份（Implement Digital Identity）

C7：实施访问控制（Enforce Access Controls）

C8：保护所有的数据（Protect Data Everywhere）

C9：实施安全日志记录和监控（Implement Security Logging and Monitoring）

C10：处理所有错误和异常（Handle All Errors and Exceptions）

​    在安全工作中主动控制项目可以作为安全需求确定的checkpoint，安全审计的checklist。主动控制项目会是安全人员和开发人员的共同语言，从安全角度来定义产品做了哪些事可以让“漏洞的利用时间更长”；让开发人员了解：为了实现所负责的产品安全性，需要增加哪些安全任务量。详细的章节各位读者可以访问owasp官网下载阅读。

# **SDL 成熟度框架:bsimm & OWASP samm**

​    再一个是SDL 成熟度框架，可以参考本公众的文章《[软件安全构建成熟度模型 (BSIMM) 介绍](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804260&idx=1&sn=aa84528d8fb66e5fdf2535633c153126&chksm=84451b3ab332922c4c7ca261dd4bcbaa8b7fbf790ec491383b5e14eb91214c19d247d0de78ec&scene=21#wechat_redirect)》，滴滴使用了bsimm8的版本评估出来level2的成熟度，笔者升级了Bsimm9，有兴趣的读者可以搭建环境评估所在单位的成熟度,做为安全能力建设的指标。owasp SAMM适合不以软件开发为主业的公司，疏于维护，不算好用，材料倒是可以看看。

# **威胁建模：McGraw SARA；**

## **威胁建模**

​    系列材料有《[威胁建模系统教程-简介和工具（一）](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804345&idx=1&sn=64c85d75942a2353f67e80fd1728ff7f&chksm=84451be7b33292f13b307fd2137e3d35e886c629859f5604ad881143d3e3f708f749d643a846&scene=21#wechat_redirect)》和《[SDL安全设计工具，一款支持多人协作实施威胁建模的微信小程序](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804412&idx=1&sn=c904fafcde9d7209290c79f8bc17b1b3&chksm=84451ba2b33292b42237ec86707e8754d4c549ec2cdaed08f8d05fa838268bf9282acac2270c&scene=21#wechat_redirect)》。

## **McGraw**

​    是指Gray McGraw,他的个人简介材料可以看下本公众号的文章[破框和跨界：一个外国大牛离职后的活法](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804459&idx=1&sn=ff38f4d7059e7637ea4cc1b1556dba2b&chksm=84451475b3329d636810286b0eb46ed4a573927394242318dd0862a5109c68be3859b54745e7&scene=21#wechat_redirect)，McGraw也是发明Bsimm的大神。我曾经有幸和他有过交流 ：），

![img](https://ask.qcloudimg.com/http-save/yehe-4752717/gryugs2932.jpeg?imageView2/2/w/1620)

image-20200404192423361

读者们知道他的邮件签名是什么意思吗？

## **SARA**

​    这个是一个相对较新的软件架构风险分析方法材料，在加拿大军方，法国电信，欧洲汽车安全行业有实际应用，下面将用大量的篇幅介绍SARA。

### **什么阶段做安全评估**

​    DevSecOps的理念是糅合了开发、安全及运营理念以创建解决方案（以后有时间会详细介绍SDL和DevSecOps的区别和联系）。在DEV（Development）和OPS（Operation）中有许多安全的介入点：    在开发环节可以做的事情有：定义安全需求，引入安全设计原则，对开发人员进行技能培训，使用白盒扫描工具，使用更安全的编程语言（go语言是世界上最好的语言=_=）,在迭代中进行安全测试。    在Operation环节确保安全的配置（配置不当漏洞），进行安全专项审计(众测)，应急新发现的安全漏洞（WAF、SRC），检测和审计应用是否有异常行为（hids\nids\rasp\蜜罐\欺骗防御）。    架构安全风险评估可以在设计阶段实施避免后续开发环节引入的安全体系缺陷，也可以在存量系统已经上线时，评估变更和现有安全风险的影响。

​    我们不要仅仅以为安全架构评审仅适用了SDL的安全需求和设计阶段，实际中安全专家入职后的第一件事就是评估现在入网众多核心系统的安全性，目前正在开发的系统并没有那么难搞，可以划分一到三周时间内分别评估多个组件模块，跟随迭代功能一次评估一类，最后形成评估的整体报告。由小到到的好处是容易出成果，当业务发现软件内置的安全越来越完善，合作的安全团队越来越靠谱的时候会容易接受安全的介入，大家一起找到相对安全的平衡点。

### **适用范围**

​    SARA作为一种架构分析框架和SDLC(软件安全开发生命周期)之间是什么关系呢？SARA强调使用组织现有的风险评估流程，为安全架构师使用，需要业务系统架构师，开发、安全、甚至用户视角的参与，专注于快速对目标系统进行评估。SARA流程涉及SDCL除了“安全培训”环节以外的每个流程。从安全实际工作来看，应用安全性提升的阶段分为风险评估，风险环节，再评估闭环三个阶段。SARA只做风险评估的事情。SARA直接包含威胁建模活动和代码审查、安全测试阶段。

### **方法论**

![img](https://ask.qcloudimg.com/http-save/yehe-4752717/8qhk0pabgq.jpeg?imageView2/2/w/1620)

1-9个步骤

​    下面详细介绍从步骤1到步骤9的实施过程。

1. 架构访谈及问卷    收集并基本了解目标的架构相关文档，同业务方的架构师根据架构设计进行访谈和问题review，确定系统关键模块是什么？核心功能有哪些？安全措施是什么？了解系统的关键数据流的流向（可以从GDPR文档中获取一些材料）。必要时登录操作下实际业务，熟悉用户使用的流程。在这一环节形成对关键组件模块的数据流转、安全控制机制、系统内的敏感数据定级三项材料。梳理环节可以有遗漏，但是不能不准确。
2. 威胁识别   在该阶段实施威胁建模，可以用微软的Stride或者攻击树模型，注意不要关心业务是否已经有安全加固措施，只需要列出风险即可，风险来源可以是组件历史漏洞、专家经验、同类系统的风险，输出为威胁列表，实施完成的输出材料可以参考本公众号的文章《[代码审计-dubbo admin <=2.6.1远程命令执行漏洞](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804212&idx=1&sn=c3067e0b1b959671d606e082f07d9d9b&chksm=84451b6ab332927c133e14d9b33d1f96ca8e668674fe506e7e658d7c4f384bf56a69ec43ef79&scene=21#wechat_redirect)》。
3. 漏洞识别    这一步是最简单的，使用漏扫工具、白盒扫描、渗透测试的安全验证手段，尽可能提早发现问题，这个节点发现的问题并非是全的，但是起码是当前最紧要的。“未知攻，焉知防”，从攻击小伙伴们获取帮助，对安全架构分析形成更全面的认识。这一步要注意生成安全测试报告时，要和业务团队一一确认责任人和排期，避免后续开发中没有把安全风险纳入开发计划。
4. 已有控制措施分析    安全专家必须知道系统目前已经采取的哪些控制措施是有效的，控制措施的列表见《[项目发布 | OWASP Top 10 Proactive Controls V3中文版](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804532&idx=1&sn=dd88fb286b77bc928b2d691cd3442089&chksm=8445142ab3329d3c1ff3daf558cb888d0bcfa05f830ba11eaaaf552b98843d116e21412ea10a&scene=21#wechat_redirect)》。这一步识别出来的工作量和业务息息相关，是产品接下来要实现的功能安全。双方分歧最大的是确定安全技术细节，这时候对安全评审人员的要求不是挖洞，而是安全防御方案的设计能力，最好懂开发，了解同类安全措施的漏洞原理和主流防护方案、业界同行的最优实践。
5. 威胁可能性评估 业务可以接受的安全建议一定不能是来自“安全迫害妄想症”的臆想，根据上一步输出的已有的安全措施和详细渗透测试报告，可以清晰了解系统已经实现的安全措施的有效性。举个例子是：业务以为启用了md5就是具备足够安全等级的加密，这时候就需要安全专家指出来存在的攻击可能。    需要于识别到的威胁必须从实际出发去评估发生的可能性。举例来说对于只暴露内网系统，修复的优先级实际上是可以沟通放松的，非得说来自NSA国家级专业团队“精心构造的POC”,有点用自己的最高标准，衡量别人最低要求的意思。    确定攻击者使用的技能、资源和安全防护措施匹配，指着业务使用的一个流行的加密算法，非得说量子加密可以破解就是抬杠了。安全投入不能无限大，攻击的投入也不可能无限大。

![img](https://ask.qcloudimg.com/http-save/yehe-4752717/oy6bywhgr.jpeg?imageView2/2/w/1620)

威胁要结合可能性进行合理评估

1. 影响分析    对分析出来的每一项功能探讨如果发生安全问题会带来什么影响，同业务leader沟通安全后果是什么。举例来说在银保证监行业里的业务角度，一个越权身份信息泄露的影响，远比一个XXE的技术漏洞影响高，尊重业务意见，说清安全评估的风险。
2. 风险确定    经过以上几步同业务各个方面的关键人员的沟通，确定达成对潜在攻击发生的风险的认知一致。

![img](https://ask.qcloudimg.com/http-save/yehe-4752717/hmws41jirj.jpeg?imageView2/2/w/1620)

攻击可能性评估表

1. 缓解措施    风险按照级别和发生的可能性排定风险优先级，制定缓解计划，对业务的缓解或者修复方案进行评估和记录，专业的人干专业的事情。安全负责结果的验收，业务负责方案的制定。如果业务选择接受风险，安全在明确告知风险的前提下也可以接受。
2. 结果输出    编写完整报告，说明威胁、风险、级别，安全建议，这一步注意不要忽视低优先的安全风险，让这个文档动起来，避免当时视为低危的风险随着迭代推移风险不断上升。    最后一定要为建立安全文化而鼓励业务部门，融洽关系，让安全带来正能量，让业务感到安全是为业务提升价值，不是因为安全多了工作量。为下一次安全评审的合作打好基础。

## **OWASP ASVS**

ASVS就是Web应用安全评估标准，

![img](https://ask.qcloudimg.com/http-save/yehe-4752717/8m0huo89sm.jpeg?imageView2/2/w/1620)

这本书没啥用

就是这本书，挺薄的，里面是列出来的安全验证checklist。

V1：架构、设计和威胁建模

V2：认证

V3：会话管理

V4：访问控制

V5：验证、清理和编码

V6：存储加密

V7：错误处理和日志记录

V8：数据保护

V9：通信安全

V10：恶意代码

V11：业务逻辑

V12：文件和资源

V13：API和WEB服务

V14：安全配置

# **缓解机制列表（含公共组件）参考OWASP_Cheat_Sheet_Series**

​    缓解机制列表（含公共组件）参考OWASP_Cheat_Sheet_Series，简称OWASP CSS，目的是为实施应用程序加固方案的开发者提供一套简单可靠的指南。CSS作为备忘录是owasp 主动控制和owasp ASVS项目的补充。

接地气可以阅读http://www.toolfk.com/handbook-wizardforcel-owasp-cheat-sheet-zh 项目，介绍了每项cheat背后实际的攻击技术。

# **IEEE安全设计中心（CSD）提出的Top-10-Flaws**

   是指在一份题为“避免软件安全设计的十大弊端”的报告中，CSD组织希望将重点从发现错误转移到识别设计缺陷，帮助软件开发人员从他们的学习中吸取教训。根据来自Twitter，Google和Hewlett-Packard等各种组织的专家的意见，这份长达31页的报告将建议分为10大类：

-赢得或给予，但从不假设信任

-使用无法绕过或篡改的身份验证机制

-验证后授权

-严格分开数据和控制指令，切勿处理从不可信来源收到的控制指令

-定义一种确保所有数据都经过明确验证的方法

-正确使用加密

-识别敏感数据以及如何处理

-始终考虑用户

-了解集成外部组件如何改变您的攻击面

-在考虑对象和参与者的未来变化时要保持灵活性

这个报告也是上面提到的McGraw同志提出来….关于软件安全设计原则，国内鲜少有资料，我将在下次更新为大家介绍，也为自己提供学习的机会。

# **参考资料**

https://www.owasp.org/images/f/fd/SAMM-1.0-cn.pdf

http://www.owasp.org.cn/OWASP_Training/SAMM

[SDL已死，应用安全路在何方？](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804543&idx=1&sn=2a8f48524d3ecfec73f7b66ebad4b8f7&chksm=84451421b3329d373d69dba106380764af59f0f09c219cc6145ee217cd1bcbb102d4486f7814&scene=21#wechat_redirect)

[项目发布 | OWASP Top 10 Proactive Controls V3中文版](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804532&idx=1&sn=dd88fb286b77bc928b2d691cd3442089&chksm=8445142ab3329d3c1ff3daf558cb888d0bcfa05f830ba11eaaaf552b98843d116e21412ea10a&scene=21#wechat_redirect)

[供应链安全：安全建设中的第三方组件依赖问题](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804524&idx=1&sn=9d2dfdd17fc5260b308ecdfc6d39cd1b&chksm=84451432b3329d2406ed4b113a6c1879f62ad2c5373435c15d6c311f72dac7936f4ff3036b5e&scene=21#wechat_redirect)

[从微软、FB、华为的网络安全备忘录说开去](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804397&idx=1&sn=4f9810e4767d515cf8d1fd2d9c24a5f4&chksm=84451bb3b33292a56c3f627599b83f3ad602cea7661b732054a94c4d4c01818c563b8262449c&scene=21#wechat_redirect)

[软件安全构建成熟度模型 (BSIMM) 介绍](http://mp.weixin.qq.com/s?__biz=MzA5Mzg3NTUwNQ==&mid=2447804260&idx=1&sn=aa84528d8fb66e5fdf2535633c153126&chksm=84451b3ab332922c4c7ca261dd4bcbaa8b7fbf790ec491383b5e14eb91214c19d247d0de78ec&scene=21#wechat_redirect)

本文分享自微信公众号 - 安全乐观主义（gh_d6239d0bb816），作者：Ramos

原文出处及转载信息见文内详细说明，如有侵权，请联系 yunjia_community@tencent.com 删除。

原始发表时间：2020-04-05

本文参与[腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。