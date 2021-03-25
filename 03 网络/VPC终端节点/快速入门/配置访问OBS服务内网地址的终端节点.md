# 简介

更新时间：2021/02/09 GMT+08:00

#### 操作场景

如果您希望**本地数据中心通过VPN或者云专线以内网方式访问OBS服务，则可以通过终端节点连接终端节点服务实现**。

本节介绍线下节点（即本地数据中心）通过内网方式访问云上OBS服务的配置指导。

![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

仅“拉美-墨西哥城一”、“拉美-圣保罗一”和“拉美-圣地亚哥”区域支持将OBS配置为终端节点服务，因此本场景仅适用于这些区域。

图1 本地数据中心访问OBS（内网）
![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0298583614.png)

如[图1](https://support.huaweicloud.com/qs-vpcep/vpcep_02_0301.html#vpcep_02_0301__fig1589611845912)所示，**线下节点（即本地数据中心）通过VPN或者云专线与VPC连通**。在VPC内购买终端节点，**与云上的OBS和DNS类型的终端节点服务连接，实现线下节点（即本地数据中心）通过内网访问云上服务**。

终端节点不能脱离终端节点服务单独存在，购买终端节点的前提是要连接的终端节点服务已存在。

本操作场景涉及两个系统创建的终端节点服务：

- **终端节点服务（DNS）**：提供域名解析服务，**用于线下本地数据中心解析OBS域名**。

  以“拉美-墨西哥城一”为例：com.myhuaweicloud.na-mexico-1.dns

- **终端节点服务（OBS）**：提供OBS服务，供线下本地数据中心访问。

  以“拉美-墨西哥城一”为例：com.myhuaweicloud.na-mexico-1.obs

#### 操作流程

配置本地数据中心通过内网访问OBS，具体操作流程如[图2](https://support.huaweicloud.com/qs-vpcep/vpcep_02_0301.html#vpcep_02_0301__fig121641538567)所示。

图2 操作流程
![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0298562045.png)





# 步骤一：购买连接DNS的终端节点

更新时间：2021/02/09 GMT+08:00

#### 操作场景

**为了将解析OBS域名的请求转发到终端节点，您需要购买连接DNS服务的终端节点**。

#### 前提条件

终端节点要连接的终端节点服务已经存在。

#### 操作步骤

1. 登录管理控制台。
2. 在管理控制台左上角单击“![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945877.png)”图标，选择区域和项目。
3. 单击“服务列表”中的“网络 > VPC终端节点”，进入“终端节点”页面。

1. 在“终端节点”页面，单击“购买终端节点”。

   进入“购买终端节点”页面。

   图1 购买终端节点（云服务-接口型）
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0000001079290158.png)

   

2. 根据界面提示配置参数。

   | 参数       | 说明                                                         |
   | ---------- | ------------------------------------------------------------ |
   | 区域       | 终端节点所在区域。**不同区域的资源之间内网不互通**。请选择靠近您的区域，可以降低网络时延、提高访问速度。 |
   | 计费方式   | 按需计费是后付费模式，按终端节点服务的实际使用时长计费，可以随时开通/删除终端节点。仅支持按需计费。 |
   | 服务类别   | 可选择“云服务”或“按名称查找服务”。云服务：当您要连接的终端节点服务为云服务时，需要选择“云服务”。按名称查找服务：当您要连接的终端节点服务为用户私有服务时，需要选择“按名称查找服务”。此处选择“云服务”。 |
   | 选择服务   | 若“服务类别”选择“云服务”，则会出现该参数。**终端节点服务实例已由运维人员预先创建完成，您可以直接使用**。此处选择DNS服务实例，即“com.myhuaweicloud.na-mexico-1.dns”。 |
   | 内网域名   | 如果您想要以域名的方式访问终端节点，则**选择“创建内网域名”，终端节点创建完成后，即可通过内网域名直接访问终端节点**。接口终端节点才需在页面设置此选项。终端节点服务的类型为“网关”时，该参数不可见；终端节点服务的类型为“接口”时，可选择是否创建内网域名。 |
   | 虚拟私有云 | 选择终端节点所属的虚拟私有云。                               |
   | 子网       | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。选择终端节点所属的子网。 |
   | 节点IP     | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。终端节点的私网IP。可选择“自动分配”或“手动分配”。 |
   | 访问控制   | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。用于设置允许访问终端节点的IP。开启：只允许白名单列表中的IP访问终端节点。关闭：允许任何IP访问终端节点。 |
   | 白名单     | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。用于设置允许访问的IP地址或网段，最多支持添加20个记录。 |

3. 参数配置完成，单击“立即购买”，进行规格确认。

   - 规格确认无误，单击“提交”，任务提交成功。
   - 参数信息配置有误，需要修改，单击“上一步”，修改参数，然后单击“提交”。

4. 提交成功后，返回终端节点列表。

   当新创建的终端节点状态为“已接受”时，表示连接“com.myhuaweicloud.na-mexico-1.dns”的终端节点创建成功。

5. 单击终端节点ID前面的“

   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0210877115.png)

   ”，即可查看终端节点的详细信息。

   接口终端节点创建成功后，会生成一个“节点IP”（就是私有IP）和“内网域名”（如果在创建终端节点时您勾选了“内网域名”）。

   图2 终端节点详情
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0000001079578256.png)





# 步骤二：购买连接OBS的终端节点

更新时间：2021/02/09 GMT+08:00

#### 操作场景

为了实现用户本地数据中心节点通过终端节点访问OBS服务，需要购买连接OBS服务的终端节点。

#### 前提条件

终端节点要连接的终端节点服务已经存在。

#### 操作步骤

1. 登录管理控制台。

2. 在管理控制台左上角单击“![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945877.png)”图标，选择区域和项目。

3. 单击“服务列表”中的“网络 > VPC终端节点”，进入“终端节点”页面。

4. 在“终端节点”页面，单击“购买终端节点”。

   进入“购买终端节点”页面。

   图1 购买终端节点（云服务-网关型）
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0000001126477771.png)

   

5. 根据界面提示配置参数。

   | 参数       | 说明                                                         |
   | ---------- | ------------------------------------------------------------ |
   | 区域       | 终端节点所在区域。不同区域的资源之间内网不互通。请选择靠近您的区域，可以降低网络时延、提高访问速度。 |
   | 计费方式   | 按需计费是后付费模式，按终端节点服务的实际使用时长计费，可以随时开通/删除终端节点。仅支持按需计费。 |
   | 服务类别   | 可选择“云服务”或“按名称查找服务”。云服务：当您要连接的终端节点服务为云服务时，需要选择“云服务”。按名称查找服务：当您要连接的终端节点服务为用户私有服务时，需要选择“按名称查找服务”。此处选择“云服务”。 |
   | 选择服务   | 若“服务类别”选择“云服务”，则会出现该参数。终端节点服务实例已由运维人员预先创建完成，您可以直接使用。此处选择OBS服务实例，即“com.myhuaweicloud.na-mexico-1.obs”。 |
   | 虚拟私有云 | 选择终端节点所属的虚拟私有云。                               |

6. 参数配置完成，单击“立即购买”，进行规格确认。

   - 规格确认无误，单击“提交”，任务提交成功。
   - 参数信息配置有误，需要修改，单击“上一步”，修改参数，然后单击“提交”。

7. 任务提交成功，返回终端节点列表。

   当新创建的终端节点状态由“创建中”变为“已接受”时，表示连接“com.myhuaweicloud.na-mexico-1.obs”的终端节点创建成功。

8. 单击终端节点ID前面的“

   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0210877115.png)

   ”，即可查看终端节点的详细信息。

   图2 终端节点详情
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0000001079578242.png)





# 步骤三：访问OBS服务

更新时间：2021/02/09 GMT+08:00

#### 操作场景

本节介绍如何通过VPN或者云专线方式访问OBS服务。

#### 前提条件

您的本地数据中心**已通过VPN或者云专线与VPC连通**。

- VPN连接对应的需要与本地数据中心互通的VPC子网网段，需包含OBS的网段100.125.0.0/16。

  创建虚拟专用网络，请参考[创建VPN网关](https://support.huaweicloud.com/qs-vpn/zh-cn_topic_0133627788.html)。

- 专线虚拟网关允许访问的VPC子网网段需包含OBS的网段100.125.0.0/16。

  开通云专线，请参考[开通云专线](https://support.huaweicloud.com/qs-dc/zh-cn_topic_0145790541.html)。

#### 操作步骤

1. 在“终端节点”列表，单击创建的连接DNS服务的终端节点ID，查看该终端节点的“节点IP”。

2. 在**用户本地数据中心的DNS服务器配置相应的DNS转发规则，将解析OBS域名的请求转发到连接DNS服务的终端节点**。

   不同操作系统中配置DNS转发规则的方法不同，具体操作请参考对应DNS软件的操作指导。

   本步骤以Unix操作系统，常见的DNS软件Bind为例介绍：

   **方式1**：在/etc/named.conf内，增加DNS转发器的配置，“forwarders”为连接DNS服务的终端节点的IP地址。

   **options {**

   **forward only；**

   **forwarders{ xx.xx.xx.xx;};**

   **};**

   **方式2**：在/etc/named.rfc1912.zones文件，增加如下内容，“forwarders”为连接DNS服务的终端节点的IP地址。

   以华南OBS Endpoint为例：

   **zone "com.myhuaweicloud.na-mexico-1.obs" {**

   **type forward;**

   **forward only;**

   **forwarders{ xx.xx.xx.xx;};**

   **};**

   ![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

   - **用户本地数据中心若无DNS服务器，需要将连接DNS服务的终端节点的节点IP增加到用户本地数据中心节点的/etc/resolv.conf文件中**。
   - xx.xx.xx.xx为[查看终端节点详情](https://support.huaweicloud.com/qs-vpcep/vpcep_02_0304.html#vpcep_02_0304__li146041853163516)中的节点IP。

3. 配置用户本地数据中心节点到VPN网关或者专线网关的DNS路由。

   连接DNS服务的终端节点的节点IP地址为xx.xx.xx.xx，为了通过VPN或者云专线访问DNS，需要**将用户本地数据中心节点访问DNS的流量指向用户本地数据中心节点的专线网关或者VPN网关**。

   在用户本地数据中心节点配置永久路由，指定访问DNS的流量下一跳为用户本地数据中心节点专线网关或者VPN网关的IP地址。

   **route -p add xx.xx.xx.xx mask 255.255.255.255 xxx.xxx.xxx.xxx**

   ![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

   - xx.xx.xx.xx为[查看终端节点详情](https://support.huaweicloud.com/qs-vpcep/vpcep_02_0304.html#vpcep_02_0304__li146041853163516)中的节点IP。
   - xxx.xxx.xxx.xxx为用户本地数据中心节点专线网关或者VPN网关的IP地址。

4. 配置用户本地数据中心节点到VPN网关或者专线网关的OBS路由。

   连接OBS服务的终端节点的IP地址网段为100.125.0.0/16，为了通过VPN或者云专线访问OBS，需要**将用户本地数据中心节点访问OBS服务的流量指向用户本地数据中心节点的专线网关或者VPN网关**。

   在用户本地数据中心节点配置永久路由，指定访问OBS的流量下一跳为用户本地数据中心节点专线网关或者VPN网关的IP地址。

   **route -p add 100.125.0.0 mask 255.255.0.0 xxx.xxx.xxx.xxx**

   ![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

   xxx.xxx.xxx.xxx为用户本地数据中心节点专线网关或者VPN网关的IP地址。

5. 在本地数据中心，通过以下命令验证本地数据中心与OBS的连通性：

   **telnet** ***bucket.\******endpoint\***

   其中：

   - bucket：表示OBS的桶名。
   - endpoint：表示OBS的Endpoint信息。

   例如，**telnet** **bucket.****obs.na-mexico-1.myhuaweicloud.com**

   ![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

   您可以从[地区和终端节点](https://developer.huaweicloud.com/endpoint?OBS)中查询不同区域OBS的Endpoint信息。