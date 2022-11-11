# ES、IEC、IEF有什么区别和关联？

更新时间：2022-10-26 GMT+08:00

本节将从多方面对比[智能边缘小站](https://www.huaweicloud.com/zh-cn/product/ies.html)（Intelligent EdgeSite，IES）、[智能边缘云](https://www.huaweicloud.com/zh-cn/product/iec.html)（Intelligent EdgeCloud，IEC）和[智能边缘平台](https://www.huaweicloud.com/zh-cn/product/ief.html)（Intelligent EdgeFabric，IEF）三款产品，旨在为您推荐最优的边缘解决方案，满足您的多样化业务需求。

#### 边缘计算

在传统的集中式云计算场景中，所有数据都集中存储在大型数据中心。由于**地理位置和网络传输的限制，无法满足新型业务（如增强现实AR、虚拟现实VR、互动直播等）的低时延、高带宽等要求**。

**边缘计算通过在靠近终端应用的位置建立站点，最大限度的将集中式云计算的能力延伸到边缘侧，有效解决以上的时延和带宽问题**。更多详细介绍请参见[边缘计算](https://support.huaweicloud.com/zh-cn/productdesc-ies/ies_01_0100.html#section0)。

IES、IEC和IEF是华为云推出的面向边缘计算场景的三款产品，旨在让云更贴近您的业务，成为您在边缘位置部署应用的必备利器。

- 从业务面的部署位置来看：**IES部署于用户数据中心（即本地机房）**；**IEC部署于距离企业和热点用户区域更近的城域位置**；**IEF纳管的边缘节点部署于靠近用户业务的任意位置**，而**华为云上的其他云服务一般部署于华为云的中心区域（简称中心云）**。对于您而言，使用这三款产品，如同使用更靠近您实际业务的华为云。
- 从服务体验来看：三款产品与中心云统一架构，使用体验与中心云一致。

[图1](https://support.huaweicloud.com/ies_faq/ies_04_0102.html#ies_04_0102__fig1754112221615)展示了三款产品在华为云上的布局。

图1 IES、IEC、IEF在华为云上的布局
![img](https://support.huaweicloud.com/ies_faq/zh-cn_image_0000001164392775.png)



[表1](https://support.huaweicloud.com/ies_faq/ies_04_0102.html#ies_04_0102__table112151172713)详细介绍IES、IEC、IEF的区别点。

| 产品名称                | IES                                                          | IEC                                                          | IEF                                                          |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 产品定位                | 部署于**用户数据中心**的华为云边缘小站。详细介绍请参见[什么是智能边缘小站](https://support.huaweicloud.com/productdesc-ies/ies_01_0100.html)。 | 广域覆盖的分布式边缘云。详细介绍请参见[什么是智能边缘云](https://support.huaweicloud.com/productdesc-iec/iec_01_0100.html)。 | 基于云原生技术构建的**边云协同操作系统**。详细介绍请参见[什么是智能边缘平台](https://support.huaweicloud.com/productdesc-ief/ief_productdesc_0001.html)。 |
| 产品特点                | 访问**低时延资源专属（稳定/自主性强）数据合规/本地化**       | 访问低时延资源共享（按需/弹性）                              | 访问低时延智能化**应用管理**                                 |
| 产品能力                | 将华为云可用区部署在用户数据中心，在本地提供丰富的公有云必选和可选云服务，满足数据本地化需求，降低网络时延。 | 提供多元算力，满足多种业务需求，用户通过就近部署业务，有效降低网络时延。 | 从云端下发应用到边缘，帮助用户在云端对边缘应用进行管理，解决应用“推送/简化部署”到边缘的问题。 |
| 典型应用场景            | IES主要**面向政企/低时延、数据合规、AIoT等业务**，典型应用场景包含：传统应用云化大数据/数据治理SaaS本地化部署智慧园区详细介绍请参见[IES应用场景](https://support.huaweicloud.com/productdesc-ies/ies_01_0300.html)。 | IEC主要面向互联网/广覆盖、大流量、低时延等业务，典型应用场景包含：互动直播在线教育应用加速自建CDN详细介绍请参见[IEC应用场景](https://support.huaweicloud.com/productdesc-iec/iec_01_0300.html)。 | IEF主要面向**政企/轻量级、AIoT**等业务，典型应用场景包含：安平监控工业视觉文字识别CDN边缘站点详细介绍请参见[IEF应用场景](https://support.huaweicloud.com/productdesc-ief/ief_productdesc_0004.html)。 |
| 计费说明                | 提供全预付和半预付的计费模式。计费项包括IES必选服务，IES可选服务，中心region服务，华为云支持计划等。详细介绍请参见[IES计费说明](https://support.huaweicloud.com/productdesc-ies/ies_01_0900.html)。 | 提供按需计费模式，由边缘虚拟机和边缘硬盘使用时长，以及边缘带宽流量叠加计费。您注册华为云帐号和完成充值后，即可使用。详细介绍请参见[IEC计费说明](https://support.huaweicloud.com/productdesc-iec/iec_01_0700.html)。 | 提供按需计费和套餐包计费两种计费模式，系统按照使用的边缘应用实例数进行收费。详细介绍请参见[IEF计费说明](https://support.huaweicloud.com/productdesc-ief/ief_productdesc_0007.html)。 |
| 部署位置                | 边缘小站**部署于用户数据中心**，管理控制台部署于华为中心云。 | 边缘站点部署于距离企业和热点用户区域更近的城域位置，管理控制台部署于华为中心云。 | 纳管的**边缘节点部署于靠近用户业务的任意位置**，管理控制台部署于华为中心云。 |
| 安装和运维责任方        | 边缘小站**由华为云负责安装在您的本地数据中心**。后续也主要由华为负责其运维工作，您只需负责机房基础运维即可。 | 边缘站点由华为云负责安装和运维。                             | **边缘节点由您进行安装和负责日常运维**，以确保IEF可以正常下发应用。 |
| 在哪些区域/站点提供服务 | [IES在华为云哪些区域提供服务？](https://support.huaweicloud.com/ies_faq/ies_04_0403.html) | [IEC在哪些站点提供服务？](https://support.huaweicloud.com/iec_faq/iec_04_0405.html) | [IEF支持的华为云区域？](https://developer.huaweicloud.com/endpoint?IEF) |
| 支持的云服务/应用       | IES运行有必选云服务，同时您可以根据需求将一些可选的云服务和应用部署在IES上，实现在您本地使用各类华为云服务，以及边云协同的场景，以满足数据本地化和低时延访问的需求。支持**必选服务ECS、EVS、VPC、EIP部署至边缘小站，为您提供在本地使用华为云基础云服务资源的机会**。支持丰富的可选云服务（如MRS、DWS、IEF、ROMA Connect、SFS Turbo等）在边缘小站本地运行，与云上服务形成协同关系，共同支撑业务需求。在IES上部署云市场相关应用，将华为云生态无缝拓展到边缘（即将上线）。除了上述提到的必选服务和可选服务，IES还支持各类管理、监控、安全和迁移服务。详细介绍请参见[与IES有业务交互的云服务](https://support.huaweicloud.com/productdesc-ies/ies_01_1100.html)。 | IEC提供基础云资源，同时您可以将一些可选的云服务部署在IEC上，实现在更贴近您业务的热点区域位置使用各类华为云服务的场景。独立提供计算、存储、网络等基础服务能力，多样化的算力能够支撑多种边缘业务场景。支持丰富的可选云服务（如CVR等）部署至边缘实例，为云上时延敏感型业务提供便捷的场景化解决方案。详细介绍请参见[与IEC有业务交互的云服务](https://support.huaweicloud.com/productdesc-iec/iec_01_0900.html)。 | IEF支持纳管多样化的边缘节点，同时支持与IEC/IES协同使用。通过将IEC边缘实例或IES边缘小站作为边缘节点进行纳管，与相关可选服务深度融合与协同。详细介绍请参见[注册边缘节点](https://support.huaweicloud.com/usermanual-ief/ief_01_0002.html)。 |

**父主题：** [产品咨询](https://support.huaweicloud.com/ies_faq/ies_04_0100.html)