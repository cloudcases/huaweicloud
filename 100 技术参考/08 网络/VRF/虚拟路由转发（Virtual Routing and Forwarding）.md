#### 「VRF」- 虚拟路由转发（Virtual Routing and Forwarding）

 2022-07-01  [CREATED BY JENKINSBOT](https://blog.k4nz.com/category/from-jenkins-automation/)

## 问题描述

![img](https://wiki.k4nz.com/03.NETWORKING-OF-DATA-COMMUNICATION/2.DATA-COMMUNICATION-TECHNIQUE/VPN,_Virtual_Private_Network/VRF,_Virtual_Routing_and_Forwarding/pasted_image.png)

某企业网络内有生产和管理两张网络，这两张网络独占接入和汇聚层交换机，共享核心交换机。
但是，核心交换机上同时连接了生产网络和管理网络的服务器群，两个网段均为192.168.100.0/24网段。
需求：实现生产和管理网络内部的数据通信，同时隔离两张网络之间的通信。

## 解决方案

**通过 ACL 实现**：在核心交换机部署 ACL 策略，禁止生产和管理网络之间的互访流量。
缺陷：（1）配置繁琐，拓展性差。（2）无法解决两张网络使用重叠网段的问题，需要在部署时规避重叠网段。

**通过增加核心交换机实现**：增加核心交换机，从物理上隔离两张网络。
缺陷：（1）增加额外的设备成本投入；

VRF（Virtual Routing and Rorwarding，虚拟路由转发）技术，华为 VRF 又称 VPN Instance（VPN 实例），是种**虚拟化技术**。

## 原理简述

在物理设备上，创建多个 VRF-Instance（VRF 实例，VRF-INST），然后**将物理接口划入不同的 VRF-INST 中**。

简单理解是，**将整个设备“分割”成多个小设备**。

## 特性说明

每个 VPN Instance 拥有**独立的接口、路由表、路由协议进程、路由转发表项等**。

## 应用场景

常用于 MPLS VPN、Firewall 等一些需要实现隔离的应用场景。**VRF 能够实现网络层的隔离**。

### Firewall

![img](https://wiki.k4nz.com/03.NETWORKING-OF-DATA-COMMUNICATION/2.DATA-COMMUNICATION-TECHNIQUE/VPN,_Virtual_Private_Network/VRF,_Virtual_Routing_and_Forwarding/pasted_image003.png)

防火墙虚拟系统，虚拟系统（Virtual System）是**在一台物理设备上划分出的多台相互独立的逻辑设备**，其中路由虚拟化依靠创建 VPN Instance 来实现。

虚拟系统主要具有以下特点：

1）**资源虚拟化**：每个虚拟系统都有独享的资源，包括接口、VLAN、策略和会话等。

2）**路由虚拟化**：每个虚拟系统都拥有各自的路由表，相互独立隔离。

### BGP/MPLS IP VPN

![img](https://wiki.k4nz.com/03.NETWORKING-OF-DATA-COMMUNICATION/2.DATA-COMMUNICATION-TECHNIQUE/VPN,_Virtual_Private_Network/VRF,_Virtual_Routing_and_Forwarding/pasted_image002.png)

BGP/MPLS IP VPN 是种基于 PE 的 L3VPN 技术。它使用BGP在服务提供商骨干网上发布VPN路由，使用MPLS在服务提供商骨干网上转发VPN报文。
通过创建 VPN Instance 的方式在PE上区别不同VPN的路由。



[**↑**「VRF」- 概念术语](https://blog.k4nz.com/f73d3f85894c05ad49c8dbd376c2e00f/)[**↓**「VPN」- SSL VPN](https://blog.k4nz.com/1984bcb00691c32a08e159881e79f2a9/)