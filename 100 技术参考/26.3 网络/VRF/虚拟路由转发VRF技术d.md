# 思博网络分享：虚拟路由转发VRF技术

[思博SPOTO](https://author.baidu.com/home?from=bjh_article&app_id=1600429866806160)

2021-06-17 11:22福州思博网络科技有限公司官方帐号

关注

**网络需求**

某企业网络内有生产和管理两张网络，这两张网络**独占接入和汇聚层交换机**，**共享核心交换机**。

核心交换机上同时连接了生产网络和管理网络的服务器群，两个网段均为192.168.100.0/24网段。

需求：**实现生产和管理网络内部的数据通信，同时隔离两张网络之间的通信**。

![img](https://pics1.baidu.com/feed/cc11728b4710b912d5a4aeff571a560b934522bd.jpeg@f_auto?token=0a4031e08c30e53106bf3363febd0ab6)

**1. 通过部署ACL实现**

在核心交换机部署ACL，禁止生产和管理网络之间的互访流量。

缺陷：

- 配置繁琐，拓展性差。
- 无法解决两张网络使用重叠网段的问题，需要在部署时规避重叠网段。

**2. 通过增加核心交换机实现**

![img](https://pics2.baidu.com/feed/8d5494eef01f3a29b14c5d630ac216395e607c83.jpeg@f_auto?token=b328087cb3e499348731ce0b1ba23446)

增加核心交换机，从物理上隔离两张网络。

缺陷：增加额外的设备成本投入。

**3. 通过VRF技术实现**

VRF又称VPN实例（VPN Instance），是一种**虚拟化技术**。在物理设备上创建多个VPN实例，每个VPN实例拥有独立的接口、路由表和路由协议进程等。

![img](https://pics6.baidu.com/feed/b58f8c5494eef01f1ee417826d19332dbe317df0.jpeg@f_auto?token=2b2754f400802a7143dfbb80471218f7)

![img](https://pics4.baidu.com/feed/dc54564e9258d109a783c8e742bf66b76d814d76.jpeg@f_auto?token=1cf4623c23391c412543c90712db0da4)

**VRF的实现过程**

VRF是**对物理设备的一个逻辑划分**，每个逻辑单元都被称为一个VPN实例，**实例之间在路由层面是隔离的**。

VRF实现过程如下：

1. 创建实例，并将三层接口（可以是路由器的物理接口或者子接口，也可以是VLANIF接口）绑定到实例；·

   2.（可选）配置与实例绑定的路由协议或静态路由；

3. 基于与实例绑定的接口和路由协议等建立实例路由表并基于实例路由表转发数据，实现实例间隔离。

![img](https://pics6.baidu.com/feed/5366d0160924ab18cf409e29a61d4cc57a890b4d.jpeg@f_auto?token=6ae341a261762c67f87b95056f51b846)

**举例：部署VRF前**

背景：PC1、R1及R1所连的1.1.1.0/24网段属于业务网络。

PC2、R2及R2所连的2.2.2.0/24网段属于管理网络。核心交换机上所有的直连路由，以及设备所发现的、到达远端网络的路由，均被存储在设备的路由表中（我们将该路由表称为全局路由表），业务及管理网络可以通过核心交换机实现互通。

需求：通过配置实现业务与管理网络完全隔离。

![img](https://pics3.baidu.com/feed/f2deb48f8c5494ee97dbd539b9124af698257e01.jpeg@f_auto?token=7e5ecc8152f34a31d6405e0546f2ca7a)

![img](https://pics6.baidu.com/feed/d833c895d143ad4b0db3bf2518e5f0a7a60f06a2.jpeg@f_auto?token=914eb8099f6a2715b01a93a1a5a843af)

**创建VPN实例**

![img](https://pics3.baidu.com/feed/c9fcc3cec3fdfc0362a1771545d82d9ca6c226b6.jpeg@f_auto?token=0c16ee8e20d56375da6b437b3b0c1c00)

**部署动态路由协议**

![img](https://pics4.baidu.com/feed/242dd42a2834349ba0582fe55a0dbfc637d3be60.jpeg@f_auto?token=380b516bf78b35c6bb3a312873e8bddf)

**VRF基本配置命令**

1.创建VPN实例/进入VPN实例视图

[Huawei] ip vpn-instance vpn-instance-name

ip vpn-instance命令用来创建VPN实例，并进入VPN实例视图。缺省情况下，未配置VPN实例。

\2. 使能VPN实例的IPv4类型的路由通告和数据转发功能

. [Huawei-vpn-instance-InstanceName] ipv4-family

ipv4-family命令用来使能VPN实例的IPv4地址族，并进入VPN实例IPv4地址族视图。缺省情况下，未使能VPN实例的IPv4地址族。接口不能与未使能任何地址族的VPN实例绑定。

\3. 将接口绑定到VPN实例

[Huawei-GigabitEthernet0/0/0]ip binding vpn-instance vpn-instance-name

ip binding vpn-instance命令用来将PE上的接口与VPN实例绑定。缺省情况下，接口不与任何VPN实例绑定，属于根实例。配置接口与VPN实例绑定后，或取消接口与VPN实例的绑定，都会清除该接口的IP地址、三层特性和IP相关的路由协议，如果需要应重新配置。

\4. 向VPN实例的路由表中添加静态路由

[Huawei] ip route-static vpn-instance vpn-instance-name ip-address { mask | mask-length } { nexthop-address | interface-type interface-number }

\5. 创建与VPN实例绑定的动态路由协议进程（以OSPF为例）

[Huawei] ospf [ process-id | router-id router-id ] vpn-instance vpn-instance-name

\6. VPN实例维护命令

[Huawei] display ip routing-table vpn-instance vpn-instance-name

[Huawei] ping -vpn-instance vpn-instance-name host

注：如果执行ping时没有携带vpn-instance关键字及参数，则默认在根设备上执行ping操作，并且所产生的ICMP报文根据全局路由表进行转发。tracert操作同理。



https://baijiahao.baidu.com/s?id=1702782710105988010&wfr=spider&for=pc