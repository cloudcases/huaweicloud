# 步骤一：创建终端节点服务

更新时间：2021/02/09 GMT+08:00

#### 操作场景

为实现跨VPC通信，您需要将VPC内的云资源（即后端资源）创建为终端节点服务，以便于同一区域其他VPC的终端节点通过私网IP访问该终端节点服务。

本节以VPC2中，属于帐号B的“弹性负载均衡”作为后端资源为例，指导您创建终端节点服务。

#### 前提条件

在同一VPC内，已经完成后端资源的创建。

#### 操作步骤

1. 登录管理控制台。
2. 在管理控制台左上角单击“![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945877.png)”图标，选择区域和项目。
3. 单击“服务列表”中的“网络 > VPC终端节点”，进入“终端节点”页面。

1. 在左侧导航栏选择“VPC终端节点 > 终端节点服务”，单击“创建终端节点服务”。

   进入“创建终端节点服务”页面。

   图1 创建终端节点服务
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945969.png)

   

2. 根据界面提示配置参数。

   | 参数         | 说明                                                         |
   | ------------ | ------------------------------------------------------------ |
   | 区域         | 终端节点服务所在区域。不同区域的资源之间内网不互通。请选择靠近您的区域，可以降低网络时延、提高访问速度。 |
   | 名称         | 可选参数。终端节点服务的名称。长度不大于16，支持大小写字母、数字、下划线、中划线。如果您不填写该参数，系统生成的终端节点服务的名称为{region}.{service_id}。如果您填写该参数，系统生成的终端节点服务的名称为{region}.{Name}.{service_id}。 |
   | 虚拟私有云   | 终端节点服务所属虚拟私有云。                                 |
   | 服务类型     | 终端节点服务的类型，此处仅支持设置为“接口”类型。             |
   | 连接审批     | 连接审批控制的是终端节点与终端节点服务的连接是否需要审批，审批权由终端节点服务控制。可选择开启或关闭连接审批。若选择开启连接审批，则与本终端节点服务连接的终端节点需要进行审批，详细操作请查看[连接审批](https://support.huaweicloud.com/qs-vpcep/vpcep_02_02023.html#vpcep_02_02023__li1979812511478)。 |
   | 端口映射     | 终端节点服务与终端节点建立连接关系，进行通信，支持TCP协议。服务端口：终端节点服务绑定了后端资源，作为提供服务的端口。终端端口：终端节点提供给用户，作为访问终端节点服务的端口。服务端口和终端端口取值范围1～65535，单次操作最多添加50条端口映射。**说明：**通过“终端端口 → 服务端口”的方式进行访问。 |
   | 后端资源类型 | 实际提供服务的后端资源。可创建为终端节点服务的后端资源包括：弹性负载均衡：适用于高访问量业务和对可靠性和容灾性要求较高的业务。云服务器：作为服务器使用。裸金属服务器：作为服务器使用。此处选择“弹性负载均衡”。**说明：**安全组添加的规则是白名单。终端节点服务配置的后端资源所在安全组，需要添加源地址为198.19.128.0/20的白名单入方向规则，详细操作请参考《虚拟私有云用户指南》中的[添加安全组规则](https://support.huaweicloud.com/usermanual-vpc/zh-cn_topic_0030969470.html)。 |
   | 选择负载均衡 | “后端资源类型”选择为“弹性负载均衡”时，会出现该参数，在下拉列表中选择需要提供服务的负载均衡，只支持弹性负载均衡。**说明：**弹性负载均衡作为终端节点服务的后端资源后，不支持获取真实访问客户端的地址。 |
   | 标签         | 可选参数。终端节点服务的标识，包括键和值。可以为终端节点服务创建10个标签。标签的命名规则请参考[表2](https://support.huaweicloud.com/qs-vpcep/vpcep_02_02032.html#vpcep_02_02032__vpcep_02_02022_table539113432713)。**说明：**如果已经通过TMS的预定义标签功能预先创建了标签，则可以直接选择对应的标签键和值。预定义标签的详细内容，请参见[预定义标签简介](https://support.huaweicloud.com/usermanual-tms/zh-cn_topic_0056266269.html)。 |

   | 参数 | 规则                                                         |
   | ---- | ------------------------------------------------------------ |
   | 键   | 不能为空。对于同一资源键值唯一。长度不超过36个字符。取值为不包含“=”、“*”、“<”、“>”、“\”、“,”、“\|”、“/”的所有Unicode字符，且首尾字符不能为空格。 |
   | 值   | 不能为空。长度不超过43个字符。取值为不包含“=”、“*”、“<”、“>”、“\”、“,”、“\|”、“/”的所有Unicode字符，且首尾字符不能为空格。 |

3. 单击“立即创建”。

4. 返回终端节点服务列表可查看创建的终端节点服务。

5. 单击终端节点服务的“名称”，即可查看终端节点服务的详细信息。

   图2 终端节点服务详情
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945817.png)





# 步骤二：添加白名单

更新时间：2021/02/09 GMT+08:00

#### 操作场景

终端节点服务的**权限管理用于控制是否允许跨租户的终端节点进行访问**。

创建完终端节点服务后，可以**设置允许连接该终端节点服务的授权帐号ID，将授权帐号ID添加至终端节点服务的白名单中**。

本操作指导您获取帐号ID，并添加帐号ID到终端节点服务的白名单中。

#### 前提条件

终端节点待连接的终端节点服务已经存在。

#### 获取被授权的帐号ID

1. 登录管理控制台。

2. 单击

   帐号

   

   下的“我的凭证”。

   图1 我的凭证
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945972.png)

   进入“我的凭证”页面，即可查看到VPC1所属租户的“帐号ID”，如[图2](https://support.huaweicloud.com/qs-vpcep/vpcep_02_02034.html#vpcep_02_02034__fig7602125554116)所示。

   

   图2 帐号ID
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0293090375.png)

#### 添加被授权的帐号ID至终端节点服务的白名单中

1. 登录管理控制台。
2. 在管理控制台左上角单击“![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945877.png)”图标，选择区域和项目。

1. 单击“服务列表”中的“网络 > VPC终端节点”，进入“终端节点”页面。

1. 在左侧导航栏选择“VPC终端节点 > 终端节点服务”。

2. 在“终端节点服务”页面，单击需要添加白名单的终端节点服务名称。

3. 在该终端节点服务的“权限管理”页签，单击“添加白名单记录”。

4. 根据提示配置参数，输入授权用户的

   帐号

   

   ID，添加白名单。

   图3 添加白名单记录
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945871.png)

   ![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

   - 本帐号默认在自身帐号的终端节点服务的白名单中。
   - 添加“*”到白名单，表示所有用户可访问。

5. 单击“确定”，完成白名单的设置。



# 步骤三：购买终端节点

更新时间：2021/02/09 GMT+08:00

#### 操作场景

在VPC2中完成终端节点服务的创建，并设置允许连接该终端节点服务的白名单之后，您可以在VPC1中购买连接终端节点服务的终端节点。

![img](https://res-img3.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

终端节点需要选择与终端节点服务相同的区域和项目。

#### 操作步骤

1. 登录管理控制台。

2. 在管理控制台左上角单击“![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0289945877.png)”图标，选择区域和项目。

3. 单击“服务列表”中的“网络 > VPC终端节点”，进入“终端节点”页面。

4. 在“终端节点”页面，单击“购买终端节点”。

   进入“购买终端节点”页面。

   图1 购买终端节点（按名称查找服务-接口型）
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0298514623.png)

5. 根据界面提示配置参数。

   | 参数       | 说明                                                         |
   | ---------- | ------------------------------------------------------------ |
   | 区域       | 终端节点所在区域，与终端节点服务所在区域保持一致。           |
   | 计费方式   | 按需计费是后付费模式，按终端节点服务的实际使用时长计费，可以随时开通/删除终端节点。仅支持按需计费。 |
   | 服务类别   | 可选择“云服务”或“按名称查找服务”。云服务：当您要连接的终端节点服务为云服务时，需要选择“云服务”。按名称查找服务：当您要连接的终端节点服务为用户私有服务时，需要选择“按名称查找服务”。此处选择“按名称查找服务”。 |
   | 服务名称   | 若“服务类别”选择“按名称查找服务”，则会出现该参数。输入[查看终端节点服务详情](https://support.huaweicloud.com/qs-vpcep/vpcep_02_02022.html#vpcep_02_02022__li837613314320)中记录的终端节点服务名称，单击“验证”：若显示“已找到服务”，继续后续操作。若显示“未找到服务”，请检查“区域”是否和终端节点服务所在区域一致或输入的“服务名称”是否正确。 |
   | 内网域名   | 如果您想要以域名的方式访问终端节点，则选择“创建内网域名”，终端节点创建完成后，即可通过内网域名直接访问终端节点。终端节点服务的类型为“网关”时，该参数不可见；终端节点服务的类型为“接口”时，可选择是否创建内网域名。 |
   | 虚拟私有云 | 选择终端节点所属的虚拟私有云。                               |
   | 子网       | 选择终端节点所属的子网。                                     |
   | 节点IP     | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。终端节点的私网IP。可选择“自动分配”或“手动分配”。 |
   | 访问控制   | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。用于设置允许访问终端节点的IP。开启：只允许白名单列表中的IP访问终端节点。关闭：允许任何IP访问终端节点。 |
   | 白名单     | 当创建连接“接口”类型终端节点服务的终端节点时，则会出现该参数。用于设置允许访问的IP地址或网段，最多支持添加20个记录。 |
   | 标签       | 可选参数。终端节点的标识，包括键和值。可以为终端节点创建10个标签。标签的命名规则请参考[表2](https://support.huaweicloud.com/qs-vpcep/vpcep_02_02035.html#vpcep_02_02035__vpcep_02_02023_table1349117521628)。**说明：**如果已经通过TMS的预定义标签功能预先创建了标签，则可以直接选择对应的标签键和值。预定义标签的详细内容，请参见[预定义标签简介](https://support.huaweicloud.com/usermanual-tms/zh-cn_topic_0056266269.html)。 |

   | 参数 | 规则                                                         |
   | ---- | ------------------------------------------------------------ |
   | 键   | 不能为空。对于同一资源键值唯一。长度不超过36个字符。取值为不包含“=”、“*”、“<”、“>”、“\”、“,”、“\|”、“/”的所有Unicode字符，且首尾字符不能为空格。 |
   | 值   | 不能为空。长度不超过43个字符。取值为不包含“=”、“*”、“<”、“>”、“\”、“,”、“\|”、“/”的所有Unicode字符，且首尾字符不能为空格。 |

6. 参数配置完成，单击“立即购买”，进行规格确认。

   - 规格确认无误，单击“提交”，任务提交成功。
   - 参数信息配置有误，需要修改，单击“上一步”，修改参数，然后单击“提交”。

7. 连接管理。

   如果终端节点状态为“已接受”，表示终端节点已成功连接至终端节点服务；如果终端节点状态为“待接受”，表示要连接的终端节点服务开启了“连接审批”功能，需要先进行审批，操作如下：

   1. 在左侧导航栏选择“VPC终端节点>终端节点服务”。
   2. 单击对应的终端节点服务名称，进入终端节点服务详情页面。
   3. 在终端节点服务详情页面，单击“连接管理”
      - 如果同意终端节点的连接，在连接管理页面的“操作”栏下，单击“接受”。
      - 如果不同意终端节点的连接，在连接管理页面的“操作”栏下，单击“拒绝”。
   4. 再返回终端节点列表查看终端节点状态变为“已接受”，表示终端节点已成功连接至终端节点服务。

8. 单击终端节点ID，即可查看终端节点的详细信息。

   终端节点创建成功后，会生成一个“节点IP”（就是私有IP）和“内网域名”（如果在创建终端节点时您勾选了“创建内网域名”）。

   图2 终端节点详情（接口）
   ![img](https://support.huaweicloud.com/qs-vpcep/zh-cn_image_0000001079739408.png)

   您可以使用节点IP或内网域名访问终端节点服务，进行跨VPC资源通信。