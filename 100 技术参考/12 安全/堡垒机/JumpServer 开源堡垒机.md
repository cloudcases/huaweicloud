# [**JumpServer** 开源堡垒机](https://www.oschina.net/p/jumpserver)

授权协议[GPLv2](http://www.oschina.net/question/12_2826)

开发语言[Python](https://www.oschina.net/project/lang/25/python) [HTML/CSS](https://www.oschina.net/project/lang/269/html-css) [查看源码 »](https://github.com/jumpserver/jumpserver)

操作系统Windows

软件类型开源软件

所属分类[管理和监控](https://www.oschina.net/project/tag/14/sysmgr)、 [DevOps/运维工具](https://www.oschina.net/project/tag/201/nms)

开源组织无

地区[国产](https://www.oschina.net/project/zh)

投 递 者[ibuler](https://my.oschina.net/ibuler)

适用人群未知

收录时间2014-11-17

[软件首页](https://www.oschina.net/home/login?goto_page=https%3A%2F%2Fwww.oschina.net%2Fp%2Fjumpserver%3Fhmsr%3Daladdin1e1)[软件文档](https://www.oschina.net/home/login?goto_page=https%3A%2F%2Fwww.oschina.net%2Fp%2Fjumpserver%3Fhmsr%3Daladdin1e1)[官方下载](https://www.oschina.net/home/login?goto_page=https%3A%2F%2Fwww.oschina.net%2Fp%2Fjumpserver%3Fhmsr%3Daladdin1e1)

### 软件简介

[开源软件推荐：全球领先的开源向量数据库 Milvus >>>>>> ![img](https://www.oschina.net/img/hot3.png)](https://github.com/milvus-io/milvus)

JumpServer 是全球首款开源的堡垒机，使用 GNU GPL v2.0 开源协议，是符合 4A 规范的运维安全审计系统。

JumpServer 使用 Python / Django 为主进行开发，遵循 Web 2.0 规范，配备了业界领先的 Web Terminal 方案，交互界面美观、用户体验好。

JumpServer 采纳分布式架构，支持多机房跨区域部署，支持横向扩展，无资产数量及并发限制。

改变世界，从一点点开始。

> 注: [KubeOperator](https://github.com/KubeOperator/KubeOperator) 是 JumpServer 团队在 Kubernetes 领域的的又一全新力作，欢迎关注和使用。

## 特色优势

- 开源：零门槛，线上快速获取和安装；
- 分布式：轻松支持大规模并发访问；
- 无插件：仅需浏览器，极致的 Web Terminal 使用体验；
- 多云支持：一套系统，同时管理不同云上面的资产；
- 云端存储：审计录像云端存储，永不丢失；
- 多租户：一套系统，多个子公司和部门同时使用。

## 版本说明

自 v2.0.0 发布后， JumpServer 版本号命名将变更为：v 大版本。功能版本.Bug 修复版本。比如：

```
v2.0.1 是 v2.0.0 之后的Bug修复版本；
v2.1.0 是 v2.0.0 之后的功能版本。
```

像其它优秀开源项目一样，JumpServer 每个月会发布一个功能版本，并同时维护 3 个功能版本。比如：

```
在 v2.4 发布前，我们会同时维护 v2.1、v2.2、v2.3；
在 v2.4 发布后，我们会同时维护 v2.2、v2.3、v2.4；v2.1 会停止维护。
```

## 功能列表

| 分类                    | 子类               | 功能                                                         |      |      |
| ----------------------- | ------------------ | ------------------------------------------------------------ | ---- | ---- |
| 身份认证 Authentication | 登录认证           | 资源统一登录与认证                                           |      |      |
| 身份认证 Authentication |                    | LDAP/AD 认证                                                 |      |      |
|                         |                    | RADIUS 认证                                                  |      |      |
|                         |                    | OpenID 认证（实现单点登录）                                  |      |      |
|                         |                    | CAS 认证 （实现单点登录）                                    |      |      |
|                         | MFA 认证           | MFA 二次认证（Google Authenticator）                         |      |      |
|                         |                    | RADIUS 二次认证                                              |      |      |
|                         | 登录复核（X-PACK） | 用户登录行为受管理员的监管与控制                             |      |      |
| 账号管理 Account        | 集中账号           | 管理用户管理                                                 |      |      |
|                         |                    | 系统用户管理                                                 |      |      |
|                         | 统一密码           | 资产密码托管                                                 |      |      |
|                         |                    | 自动生成密码                                                 |      |      |
|                         |                    | 自动推送密码                                                 |      |      |
|                         |                    | 密码过期设置                                                 |      |      |
|                         | 批量改密（X-PACK） | 定期批量改密                                                 |      |      |
|                         |                    | 多种密码策略                                                 |      |      |
|                         | 多云纳管（X-PACK） | 对私有云、公有云资产自动统一纳管                             |      |      |
|                         | 收集用户（X-PACK） | 自定义任务定期收集主机用户                                   |      |      |
|                         | 密码匣子（X-PACK） | 统一对资产主机的用户密码进行查看、更新、测试操作             |      |      |
| 授权控制 Authorization  | 多维授权           | 对用户、用户组、资产、资产节点、应用以及系统用户进行授权     |      |      |
|                         | 资产授权           | 资产以树状结构进行展示                                       |      |      |
|                         |                    | 资产和节点均可灵活授权                                       |      |      |
|                         |                    | 节点内资产自动继承授权                                       |      |      |
|                         |                    | 子节点自动继承父节点授权                                     |      |      |
|                         | 应用授权           | 实现更细粒度的应用级授权                                     |      |      |
|                         |                    | MySQL 数据库应用、RemoteApp 远程应用（X-PACK）               |      |      |
|                         | 动作授权           | 实现对授权资产的文件上传、下载以及连接动作的控制             |      |      |
|                         | 时间授权           | 实现对授权资源使用时间段的限制                               |      |      |
|                         | 特权指令           | 实现对特权指令的使用（支持黑白名单）                         |      |      |
|                         | 命令过滤           | 实现对授权系统用户所执行的命令进行控制                       |      |      |
|                         | 文件传输           | SFTP 文件上传 / 下载                                         |      |      |
|                         | 文件管理           | 实现 Web SFTP 文件管理                                       |      |      |
|                         | 工单管理（X-PACK） | 支持对用户登录请求行为进行控制                               |      |      |
|                         | 组织管理（X-PACK） | 实现多租户管理与权限隔离                                     |      |      |
| 安全审计 Audit          | 操作审计           | 用户操作行为审计                                             |      |      |
|                         | 会话审计           | 在线会话内容审计                                             |      |      |
|                         |                    | 历史会话内容审计                                             |      |      |
|                         | 录像审计           | 支持对 Linux、Windows 等资产操作的录像进行回放审计           |      |      |
|                         |                    | 支持对 RemoteApp（X-PACK）、MySQL 等应用操作的录像进行回放审计 |      |      |
|                         | 指令审计           | 支持对资产和应用等操作的命令进行审计                         |      |      |
|                         | 文件传输           | 可对文件的上传、下载记录进行审计                             |      |      |

## 快速开始

- [极速安装](https://docs.jumpserver.org/zh/master/install/setup_by_fast/)
- [完整文档](https://docs.jumpserver.org/)
- [演示视频](https://jumpserver.oss-cn-hangzhou.aliyuncs.com/jms-media/【演示视频】Jumpserver 堡垒机 V1.5.0 演示视频 - final.mp4)

## 组件项目

- [Lina](https://github.com/jumpserver/lina) JumpServer Web UI 项目
- [Luna](https://github.com/jumpserver/luna) JumpServer Web Terminal 项目
- [Koko](https://github.com/jumpserver/koko) JumpServer 字符协议 Connector 项目，替代原来 Python 版本的 [Coco](https://github.com/jumpserver/coco)
- [Guacamole](https://github.com/jumpserver/docker-guacamole) JumpServer 图形协议 Connector 项目，依赖 [Apache Guacamole](https://guacamole.apache.org/)

## JumpServer 企业版

- [申请企业版试用](https://jinshuju.net/f/kyOYpi)

> 注：企业版支持离线安装，申请通过后会提供高速下载链接。

## 案例研究

- [JumpServer 堡垒机护航顺丰科技超大规模资产安全运维](https://blog.fit2cloud.com/?p=1147)；
- [JumpServer 堡垒机让 “大智慧” 的混合 IT 运维更智慧](https://blog.fit2cloud.com/?p=882)；
- [携程 JumpServer 堡垒机部署与运营实战](https://blog.fit2cloud.com/?p=851)；
- [小红书的 JumpServer 堡垒机大规模资产跨版本迁移之路](https://blog.fit2cloud.com/?p=516)；
- [JumpServer 堡垒机助力中手游提升多云环境下安全运维能力](https://blog.fit2cloud.com/?p=732)；
- [中通快递：JumpServer 主机安全运维实践](https://blog.fit2cloud.com/?p=708)；
- [东方明珠：JumpServer 高效管控异构化、分布式云端资产](https://blog.fit2cloud.com/?p=687)；
- [江苏农信：JumpServer 堡垒机助力行业云安全运维](https://blog.fit2cloud.com/?p=666)。

## 安全说明

JumpServer 是一款安全产品，请参考 [基本安全建议](https://docs.jumpserver.org/zh/master/install/install_security/) 部署安装.

如果你发现安全问题，可以直接联系我们：

- [ibuler@fit2cloud.com](mailto:ibuler@fit2cloud.com)
- [support@fit2cloud.com](mailto:support@fit2cloud.com)
- 400-052-0755