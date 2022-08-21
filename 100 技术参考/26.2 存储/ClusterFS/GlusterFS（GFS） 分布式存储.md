# GFS 分布式文件系统

目录

- 一： GlusterFS 概述
  - [1.1 GlusterFS 简介](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#11-glusterfs-简介)
  - 1.2 GlusterFS特点
    - [1.2.1 扩展性和高性能](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#121-扩展性和高性能)
    - [1.2.2 高可用性](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#122-高可用性)
    - [1.2.3 全局统一命名空间](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#123-全局统一命名空间)
    - [1.2.4 弹性卷管理](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#124-弹性卷管理)
    - [1.2.5 基于标准协议](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#125-基于标准协议)
  - [1.3 GlusterFS 术语](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#13-glusterfs-术语)
  - [1.4 模块化堆栈式架构](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#14-模块化堆栈式架构)
  - [1.5 GlusterFS的工作流程](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#15-glusterfs的工作流程)
  - [1.6 弹性HASH 算法](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#16-弹性hash-算法)
- 二: GlusterFS 的卷类型
  - 2.1 分布式卷（Distribute volume）
    - [2.1.1 分布式卷示例原理](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#211-分布式卷示例原理)
    - [2.1.2 分布式卷特点 ](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#212-分布式卷特点)
    - [ 2.1.3 创建分布式卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#213-创建分布式卷)
  - 2.2 条带卷 （Stripe volume）
    - [2.2.1 条带卷示例原理](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#221-条带卷示例原理)
    - [2.2.2 条带卷特点](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#222-条带卷特点)
    - [2.2.3创建条带卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#223创建条带卷)
  - 2.3 复制卷（Replica volume）
    - [2.3.1 复制卷示例原理](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#231-复制卷示例原理)
    - [2.3.2 复制卷特点](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#232-复制卷特点)
    - [2.3.3 创建复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#233-创建复制卷)
  - 2.4 分布式条带卷（Distribute Stripe volume）
    - [2.4.1 分布式条带卷示例原理 ](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#241-分布式条带卷示例原理)
    - [2.4.2 创建分布式条带卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#242-创建分布式条带卷)
  - 2.5 分布式复制卷（Distribute Replica volume）
    - [2.5.1 分布式条带卷的示例原理](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#251-分布式条带卷的示例原理)
    - [ 2.5.2 创建分布式复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#252-创建分布式复制卷)
  - [2.6 条带复制卷（Stripe Replica volume）和分布式条带复制卷（Distribute Stripe Replicavolume）](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#26-条带复制卷stripe-replica-volume和分布式条带复制卷distribute-stripe-replicavolume)
- 三： 部署GlusterFS
  - 3.1 准备环境(所有node节点)
    - [3.1.1 关闭防火墙](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#311-关闭防火墙)
    - [3.1.2 磁盘分区，并挂载](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#312-磁盘分区并挂载)
    - [3.1.3 修改主机名，配置/etc/hosts文件](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#313-修改主机名配置etchosts文件)
  - [3.2 安装GFS](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#32-安装gfs)
  - [3.3 任意node上，添加节点到存储信任池中](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#33-任意node上添加节点到存储信任池中)
- 四 创建卷
  - 4.1 分布式卷
    - [4.1.1 创建分布式卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#411-创建分布式卷)
    - [4.1.2 查看卷列表](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#412-查看卷列表)
    - [4.1.3 启动新建分布式卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#413-启动新建分布式卷)
    - [4.14 查看创建分布式卷信息](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#414-查看创建分布式卷信息)
  - [4.2 创建条带卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#42-创建条带卷)
  - [4.3 创建复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#43-创建复制卷)
  - [4.4 创建分布式条带卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#44-创建分布式条带卷)
  - [4.5 创建分布式复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#45-创建分布式复制卷)
  - [4.6 查看当前所有卷列表](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#46-查看当前所有卷列表)
- 五： 部署Gluster 客户端
  - [5.1 安装部客户端软件](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#51-安装部客户端软件)
  - [5.2 创建挂载点目录，配置主机名映射,挂载卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#52-创建挂载点目录配置主机名映射挂载卷)
- 六：测试GlusterFS 文件系统
  - [6.1 卷中写入文件，客户端操作](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#61-卷中写入文件客户端操作)
  - 6.2 查看文件分布
    - [6.2.1 node1,node2 查看分布式文件分布](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#621-node1node2-查看分布式文件分布)
    - [6.2.2 node1，node2 查看条带卷文件分布](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#622-node1node2-查看条带卷文件分布)
    - [6.2.3 node3，node4查看复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#623-node3node4查看复制卷)
    - [6.2.4 node1,node2,node3,node4 查看分布式条带卷 ](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#624-node1node2node3node4-查看分布式条带卷)
    - [6.2.5 node1,node2,node3,node4 查看分布式复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#625-node1node2node3node4-查看分布式复制卷-)
  - 6.3 破坏性测试
    - [6.3.1 客户端查看分布式卷数据](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#631-客户端查看分布式卷数据)
    - [6.3.2 客户端查看条带卷数据](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#632-客户端查看条带卷数据)
  - 6.3.3 客户端查看分布式条带卷
    - [6.3.4 客户端查看分布式复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#634-客户端查看分布式复制卷)
    - [6.3.5 宕机node3.客户端查看复制卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#635-宕机node3客户端查看复制卷)
    - [6.3.6 小结](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#636-小结)
  - 6.4 其他维护命令
    - [6.4.1 查看 GlusterFS卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#641-查看-glusterfs卷)
    - [6.4.2 查看卷信息](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#642-查看卷信息)
    - [6.4.3 查看卷状态](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#643-查看卷状态)
    - [6.2.4 停止一个卷](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#624-停止一个卷)
    - [6.2.5 删除一个卷 ](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#625-删除一个卷)
    - [6.2.6设置卷的访问控制值](https://www.cnblogs.com/zhijiyiyu/p/15339674.html#626设置卷的访问控制值)



# 一： GlusterFS 概述

## 1.1 GlusterFS 简介

**GlusterFS 是一个开源的分布式文件系统。**由存储服务器、客户端以及NFS/Samba 存储网关（可选，根据需要选择使用）组成。没有元数据服务器组件，这有助于提升整个系统的性能、可靠性和稳定性。
GlusterFS同时也是Scale-Out（横向扩展）存储解决方案Gluster的核心，在存储数据方面具有强大的横向扩展能力，通过扩展能够支持数PB存储容量和处理数千客户端。
GlusterFS支持借助TCP/IP或InfiniBandRDMA网络（一种支持多并发链接的技术，具有高带宽、低时延、高扩展性的特点）将物理分散分布的存储资源汇聚在一起，统一提供存储服务，并使用统一全局命名空间来管理数据。
**传统的分布式文件系统MFS介绍：**MFS传统的分布式文件系统大多通过元服务器来存储元数据，元数据包含存储节点上的目录信息、目录结构等。这样的设计在浏览目录时效率高，但是也存在一些缺陷，例如单点故障。一旦元数据服务器出现故障，即使节点具备再高的冗余性，整个存储系统也将崩溃。而 GlusterFS 分布式文件系统是基于无元服务器的设计，数据横向扩展能力强，具备较高的可靠性及存储效率。

## 1.2 GlusterFS特点

扩展性和高性能高可用性全局统一命名空间弹性卷管理基于标准协议

### 1.2.1 扩展性和高性能

GlusterFS利用双重特性来提供高容量存储解决方案。
（1）Scale-Out架构允许通过简单地增加存储节点的方式来提高存储容量和性能（磁盘、计算和I/O资源都可以独立增加），支持10GbE和 InfiniBand等高速网络互联。（2）Gluster弹性哈希（ElasticHash）解决了GlusterFS对元数据服务器的依赖，改善了单点故障和性能瓶颈，真正实现了并行化数据访问。GlusterFS采用弹性哈希算法在存储池中可以智能地定位任意数据分片（将数据分片存储在不同节点上），不需要查看索引或者向元数据服务器查询。

### 1.2.2 高可用性

GlusterFS可以对文件进行自动复制，如镜像或多次复制，从而确保数据总是可以访问，甚至是在硬件故障的情况下也能正常访问。
当数据出现不一致时，自我修复功能能够把数据恢复到正确的状态，数据的修复是以增量的方式在后台执行，几乎不会产生性能负载。
GlusterFS可以支持所有的存储，因为它没有设计自己的私有数据文件格式，而是采用操作系统中主流标准的磁盘文件系统（如EXT3、XFS等）来存储文件，因此数据可以使用传统访问磁盘的方式被访问。





### 1.2.3 全局统一命名空间

分布式存储中，将所有节点的命名空间整合为统一命名空间，将整个系统的所有节点的存储容量组成一个大的虚拟存储池，供前端主机访问这些节点完成数据读写操作。

### 1.2.4 弹性卷管理

GlusterFS通过将数据储存在逻辑卷中，逻辑卷从逻辑存储池进行独立逻辑划分而得到。
逻辑存储池可以在线进行增加和移除，不会导致业务中断。逻辑卷可以根据需求在线增长和缩减，并可以在多个节点中实现负载均衡。
文件系统配置也可以实时在线进行更改并应用，从而可以适应工作负载条件变化或在线性能调优

### 1.2.5 基于标准协议

Gluster 存储服务支持 NFS、CIFS、HTTP、FTP、SMB 及 Gluster原生协议，完全与 POSIX 标准（可移植操作系统接口）兼容。
现有应用程序不需要做任何修改就可以对Gluster 中的数据进行访问，也可以使用专用 API 进行访问。

## 1.3 GlusterFS 术语

Brick（存储块）：指可信主机池中由主机提供的用于物理存储的专用分区，是GlusterFS中的基本存储单元，同时也是可信存储池中服务器上对外提供的存储目录。存储目录的格式由服务器和目录的绝对路径构成，表示方法为 SERVER:EXPORT，如 192.168.80.10:/data/mydir/。Volume（逻辑卷）：一个逻辑卷是一组 Brick 的集合。卷是数据存储的逻辑设备，类似于 LVM 中的逻辑卷。大部分 Gluster 管理操作是在卷上进行的。FUSE：是一个内核模块，允许用户创建自己的文件系统，无须修改内核代码。VFS：内核空间对用户空间提供的访问磁盘的接口。Glusterd（后台管理进程）：在存储群集中的每个节点上都要运行。

## 1.4 模块化堆栈式架构

GlusterFS 采用模块化、堆栈式的架构。

通过对模块进行各种组合，即可实现复杂的功能。例如 Replicate 模块可实现 RAID1，Stripe 模块可实现 RAID0， 通过两者的组合可实现 RAID10 和 RAID01，同时获得更高的性能及可靠性。

## 1.5 GlusterFS的工作流程

（1）客户端或应用程序通过 GlusterFS 的挂载点访问数据。（2）linux系统内核通过 VFS API 收到请求并处理。（3）VFS 将数据递交给 FUSE 内核文件系统，并向系统注册一个实际的文件系统 FUSE，而 FUSE 文件系统则是将数据通过 /dev/fuse 设备文件递交给了 GlusterFS client 端。可以将 FUSE 文件系统理解为一个代理。（4）GlusterFS client 收到数据后，client 根据配置文件的配置对数据进行处理。（5）经过 GlusterFS client 处理后，通过网络将数据传递至远端的 GlusterFS Server，并且将数据写入到服务器存储设备上。

![image-20210924145545124](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916996.png)

## 1.6 弹性HASH 算法

弹性 HASH 算法是 Davies-Meyer 算法的具体实现，通过 HASH 算法可以得到一个 32 位的整数范围的 hash 值，假设逻辑卷中有 N 个存储单位 Brick，则 32 位的整数范围将被划分为 N 个连续的子空间，每个空间对应一个 Brick。当用户或应用程序访问某一个命名空间时，通过对该命名空间计算 HASH 值，根据该 HASH 值所对应的 32 位整数空间定位数据所在的 Brick。
**弹性 HASH 算法的优点：**保证数据平均分布在每一个 Brick 中。解决了对元数据服务器的依赖，进而解决了单点故障以及访问瓶颈。

# 二: GlusterFS 的卷类型

**GlusterFS 支持七种卷类型**分布式卷条带卷复制卷分布式条带卷分布式复制卷条带复制卷分布式条带复制卷

## 2.1 分布式卷（Distribute volume）

**文件通过 HASH 算法分布到所有 Brick Server 上**，这种卷是 GlusterFS 的默认卷；以文件为单位根据 HASH 算法散列到不同的 Brick，其实只是扩大了磁盘空间，如果有一块磁盘损坏，数据也将丢失，属于文件级的 RAID0， 不具有容错能力。在该模式下，并**没有对文件进行分块处理**，文件直接存储在某个 Server 节点上。 由于直接使用本地文件系统进行文件存储，所以存取效率并没有提高，反而会因为网络通信的原因而有所降低。

### 2.1.1 分布式卷示例原理

File1 和 File2 存放在 Server1，而 File3 存放在 Server2，文件都是随机存储，一个文件（如 File1）要么在 Server1 上，要么在 Server2 上，不能分块同时存放在 Server1和 Server2 上。

![img](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916002.png)

### 2.1.2 分布式卷特点 

文件分布在不同的服务器，不具备冗余性更容易和廉价地扩展卷的大小单点故障会造成数据丢失依赖底层的数据保护

###  2.1.3 创建分布式卷

创建一个名为dis-volume的分布式卷，文件将根据HASH分布在server1:/dir1、server2:/dir2和server3:/dir3中`gluster volume create dis-volume server1:/dir1 server2:/dir2 server3:/dir3`

## 2.2 条带卷 （Stripe volume）

类似 RAID0，文件被分成数据块并以轮询的方式分布到多个 Brick Server 上，文件存储以数据块为单位，支持大文件存储， 文件越大，读取效率越高，但是不具备冗余性。

### 2.2.1 条带卷示例原理

File 被分割为 6 段，1、3、5 放在 Server1，2、4、6 放在 Server2。

### 2.2.2 条带卷特点

根据偏移量将文件分成N块（N个条带点），轮询的存储在每个Brick Serve 节点.分布减少了负载,在存储大文件时，性能尤为突出.没有数据冗余,类似于Raid 0

![image-20210924151703550](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916004.png)

### 2.2.3创建条带卷

创建了一个名为stripe-volume的条带卷，文件将被分块轮询的存储在Server1:/dir1和Server2:/dir2两个Brick中`gluster volume create stripe-volume stripe 2 transport tcp server1:/dir1 server2:/dir2`

## 2.3 复制卷（Replica volume）

将文件同步到多个 Brick 上，使其具备多个文件副本，属于文件级 RAID 1，具有容错能力。因为数据分散在多个 Brick 中，所以读性能得到很大提升，但写性能下降。复制卷具备冗余性，即使一个节点损坏，也不影响数据的正常使用。但因为要保存副本，所以磁盘利用率较低。

### 2.3.1 复制卷示例原理

File1 同时存在 Server1 和 Server2，File2 也是如此，相当于 Server2 中的文件是 Server1 中文件的副本。

![image-20210924152742271](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916005.png)

### 2.3.2 复制卷特点

卷中所有的服务器均保存一个完整的副本。具备冗余性.卷的副本数量可由客户创建的时候决定，但复制数必须等于卷中 Brick 所包含的存储服务器数。至少由两个块服务器或更多服务器。若多个节点上的存储空间不一致，将按照木桶效应取最低节点的容量作为改卷的总容量。

### 2.3.3 创建复制卷

创建名为rep-volume的复制卷，文件将同时存储两个副本，分别在Server1:/dir1和Server2:/dir2两个Brick中`gluster volume create rep-volume replica 2 transport tcp server1:/dir1 server2:/dir2`





## 2.4 分布式条带卷（Distribute Stripe volume）

Brick Server 数量是条带数（数据块分布的 Brick 数量）的倍数，兼具分布式卷和条带卷的特点。 主要用于大文件访问处理**，创建一个分布式条带卷最少需要 4 台服务器。**



### 2.4.1 分布式条带卷示例原理 

File1 和 File2 通过分布式卷的功能分别定位到Server1和 Server2。在 Server1 中，File1 被分割成 4 段，其中 1、3 在 Server1 中的 exp1 目录中，2、4 在 Server1 中的 exp2 目录中。在 Server2 中，File2 也被分割成 4 段，其中 1、3 在 Server2 中的 exp3 目录中，2、4 在 Server2 中的 exp4 目录中。

![img](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916007.png)



### 2.4.2 创建分布式条带卷

创建一个名为dis-stripe的分布式条带卷，配置分布式的条带卷时，卷中Brick所包含的存储服务器数必须是条带数的倍数（>=2倍）。Brick 的数量是 4（Server1:/dir1、Server2:/dir2、Server3:/dir3 和 Server4:/dir4），条带数为 2（stripe 2）`gluster volume create dis-stripe stripe 2 transport tcp server1:/dir1 server2:/dir2 server3:/dir3 server4:/dir4`
**创建卷时，存储服务器的数量如果等于条带或复制数，那么创建的是条带卷或者复制卷；如果存储服务器的数量是条带或复制数的 2 倍甚至更多，那么将创建的是分布式条带卷或分布式复制卷。**

## 2.5 分布式复制卷（Distribute Replica volume）

●分布式复制卷（Distribute Replica volume）：Brick Server 数量是镜像数（数据副本数量）的倍数，兼具分布式卷和复制卷的特点。主要用于需要冗余的情况下。

### 2.5.1 分布式条带卷的示例原理

File1 和 File2 通过分布式卷的功能分别定位到 Server1 和 Server2。在存放 File1 时，File1 根据复制卷的特性，将存在两个相同的副本，分别是 Server1 中的exp1 目录和 Server2 中的 exp2 目录。在存放 File2 时，File2 根据复制卷的特性，也将存在两个相同的副本，分别是 Server3 中的 exp3 目录和 Server4 中的 exp4 目录。

![img](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916008.png)

###  2.5.2 创建分布式复制卷

建一个名为dis-rep的分布式复制卷，配置分布式的复制卷时，卷中Brick所包含的存储服务器数必须是复制数的倍数（>=2倍）。Brick 的数量是 4（Server1:/dir1、Server2:/dir2、Server3:/dir3 和 Server4:/dir4），复制数为 2（replica 2）`gluster volume create dis-rep replica 2 transport tcp server1:/dir1 server2:/dir2 server3:/dir3 server4:/dir4`

## 2.6 条带复制卷（Stripe Replica volume）和分布式条带复制卷（Distribute Stripe Replicavolume）


条带复制卷（Stripe Replica volume）类似 RAID 10，同时具有条带卷和复制卷的特点。分布式条带复制卷（Distribute Stripe Replicavolume）三种基本卷的复合卷，通常用于类 Map Reduce 应用





# 三： 部署GlusterFS


GFS安装包 链接：https://pan.baidu.com/s/1beOcxaZwSqsZU-OysgUITA?pwd=fjot 提取码：fjot



![image-20210924161715580](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916009.png)



## 3.1 准备环境(所有node节点)

### 3.1.1 关闭防火墙

```
复制[root@host103 ~]# systemctl  stop firewalld [root@host103 ~]# setenforce 0 
```

### 3.1.2 磁盘分区，并挂载

```
复制vim /opt/fdisk.sh #!/bin/bash #获取sdb,sdc,sdd,sde  NEWDEV=`ls /dev/sd* | grep -o 'sd[b-z]' | uniq` for VAR in $NEWDEV do   #免交互分区   echo -e "n\np\n\n\n\nw\n" | fdisk /dev/$VAR &> /dev/null      #刷新分区表   partprobe &> /dev/null    #格式化   mkfs.xfs /dev/${VAR}"1" &> /dev/null    #创建挂载点   mkdir -p /data/${VAR}"1" &> /dev/null   #在 /etc/fstab配置开机自动挂载   echo "/dev/${VAR}"1" /data/${VAR}"1" xfs defaults 0 0" >> /etc/fstab done #检测并挂载 mount -a  &> /dev/null #授予脚本执行权限，运行脚本,查看挂载情况 [root@host103 ~]# chmod  +x /opt/fdisk.sh  [root@host103 ~]# /opt/fdisk.sh  [root@host103 ~]# df -h 
```

![image-20210924163514533](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916010.png)

![image-20210924163737718](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916011.png)

![image-20210924164550616](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916012.png)





### 3.1.3 修改主机名，配置/etc/hosts文件

```
复制#其他节点修改为相应的主机名 #修改完主机名可以重新连接，或者使用 su 切换 [root@host11 ~]# hostnamectl  set-hostname node1 #每台node 节点都将 所有node 节点的主机名映射写入到 /etc/hosts文件 [root@node1 ~]# echo "192.168.23.11 node1" >> /etc/hosts [root@node1 ~]# echo "192.168.23.12 node2" >> /etc/hosts [root@node1 ~]# echo "192.168.23.13 node3" >> /etc/hosts [root@node1 ~]# echo "192.168.23.103 node4" >> /etc/hosts 
```





## 3.2 安装GFS

```
复制#将gfsrepo 软件上传到/opt目录下。可以在windows里将包压缩成 zip 格式上传到linux [root@node3 ~]# cd  /opt/ [root@node3 opt]# unzip gfsrepo.zip [root@node1 ~]#cd /etc/yum.repos.d/ [root@node1 yum.repos.d]# mkdir bak2 [root@node1 yum.repos.d]# mv *.repo bak2  [root@node1 yum.repos.d]# vim glfs.repo [glfs] name=glfs baseurl=file:///opt/gfsrepo gpgcheck=0 enabled=1 [root@node1 ~]# yum -y install glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma  #启动服务 [root@node1 ~]# systemctl start glusterd.service  [root@node1 ~]# systemctl  enable glusterd.service  
```

![image-20210924173355826](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916013.png)

![image-20210924173404702](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916014.png)





## 3.3 任意node上，添加节点到存储信任池中

```
复制#在任意node节点上，将节点添加到存储信任池中 [root@node1 ~]# gluster peer probe node1 peer probe: success. Probe on localhost not needed [root@node1 ~]# gluster peer probe node2 peer probe: success.  [root@node1 ~]# gluster peer probe node3 peer probe: success.  [root@node1 ~]# gluster peer probe node4 peer probe: success.  [root@node1 ~]#  
```

在任意Node节点查看集群状态

```
复制[root@node1 ~]# gluster peer  status 
```

![image-20210926153124258](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916015.png)

![image-20210926153144499](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916016.png)





# 四 创建卷

**根据规划创建如下卷：**卷名卷类型Brickdis-volume分布式卷node1(/data/sdb1)， node2(/data/sdb1)stripe-volume条带卷node1(/data/sdc1)、node2(/data/sdc1)rep-volume复制卷node3(/data/sdb1)、node4(/data/sdb1)dis-stripe分布式条带卷node1(/data/sdd1)、node2(/data/sdd1)、node3(/data/sdd1)、node4(/data/sdd1)dis-rep分布式复制卷node1(/data/sde1)、node2(/data/sde1)、node3(/data/sde1)、node4(/data/sde1)**在任意node主机都可以操作。因为都关联起来了.**

## 4.1 分布式卷

### 4.1.1 创建分布式卷

创建分布式卷，没有指定类型，默认创建的是分布式卷.。在没有指传输协议时，默认使用tcp

```
复制[root@node1 ~]# gluster volume create dis-volume node1:/data/sdb1 node2:/data/sdb1 force 
```

![image-20210926154318446](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916017.png)



### 4.1.2 查看卷列表

```bash
复制[root@node1 ~]# gluster volume list
dis-volume
```

![image-20210926154443910](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916018.png)



### 4.1.3 启动新建分布式卷

```bash
复制[root@node1 ~]# gluster volume start dis-volume 
volume start: dis-volume: success
```

![image-20210926154606427](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916019.png)



### 4.14 查看创建分布式卷信息

```bash
复制[root@node1 ~]# gluster volume info  dis-volume 
```

![image-20210926155029406](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916020.png)





## 4.2 创建条带卷

**指定类型为 stripe ，数值为2，且后面跟了2 个 Brick Server，所以创建的是条带卷**

```
复制[root@node1 ~]# gluster volume create stripe-volume stripe 2 node1:/data/sdc1 node2:/data/sdc1 force [root@node1 ~]# gluster volume start stripe-volume [root@node1 ~]# gluster volume info stripe-volume 
```

l

![image-20210926155436086](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916021.png)





## 4.3 创建复制卷

**指定类型为replica ，数值为2 ，且后面跟了2 个 Brick Server,所以创建的是复制卷**

```
复制[root@node3 ~]# gluster volume create rep-volume replica 2 node3:/data/sdb1 node4:/data/sdb1 force [root@node3 ~]# gluster volume start rep-volume [root@node3 ~]# gluster volume info rep-volume 
```

![image-20210926155850456](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916022.png)





## 4.4 创建分布式条带卷

**指定类型为stripe ，数值为2，后面跟了4个brick Server，是2的两倍。所以创建的是分布式条带卷**

```
复制[root@node1 ~]# gluster volume create dis-stripe stripe 2 node1:/data/sdd1 node2:/data/sdd1 node3:/data/sdd1 node4:/data/sdd1 force [root@node1 ~]# gluster volume start dis-stripe [root@node1 ~]# gluster volume info dis-stripe  
```

![image-20210926160311839](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916023.png)





## 4.5 创建分布式复制卷

**指定类型为 replica，数值为 2，而且后面跟了 4 个 Brick Server，是 2 的两倍，所以创建的是分布式复制**

```
复制[root@node1 ~]# gluster volume create dis-rep replica 2 node1:/data/sde1 node2:/data/sde1 node3:/data/sde1 node4:/data/sde1 force [root@node1 ~]# gluster volume start dis-rep [root@node1 ~]# gluster volume info dis-rep  
```

![image-20210926160657397](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916024.png)





## 4.6 查看当前所有卷列表

```
复制[root@node1 ~]# gluster volume list dis-rep dis-stripe dis-volume rep-volume stripe-volume 
```

![image-20210926160858981](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916025.png)





# 五： 部署Gluster 客户端

## 5.1 安装部客户端软件

将gfsrepo 软件上传到 /opt 目录下(可以解压后压缩为zip 格式上传到linux中)

```
复制[root@host104 opt]# unzip gfsrepo.zip [root@host104 opt]# cd /etc/yum.repos.d/ [root@host104 yum.repos.d]# mkdir bak02 [root@host104 yum.repos.d]# mv *.repo bak02 [root@host104 yum.repos.d]# vim glfs.repo [glfs] name=glfs baseurl=file:///opt/gfsrepo gpgcheck=0 enabled=1 [root@host104 yum.repos.d]# yum clean all && yum makecache [root@host104 yum.repos.d]# yum -y install glusterfs glusterfs-fuse.x86_64 
```





## 5.2 创建挂载点目录，配置主机名映射,挂载卷

```
复制[root@host104 ~]# mkdir -p /test/{dis,stripe,rep,dis_stripe,dis_rep} [root@host104 ~]# ls /test/ dis  dis_rep  dis_stripe  rep  stripe [root@host104 ~]# echo "192.168.23.11 node1" >> /etc/hosts [root@host104 ~]# echo "192.168.23.12 node2" >> /etc/hosts [root@host104 ~]# echo "192.168.23.13 node3" >> /etc/hosts [root@host104 ~]# echo "192.168.23.103 node4" >> /etc/hosts  [root@host104 ~]# ping node1 [root@host104 ~]# ping node2 [root@host104 ~]# ping node3 [root@host104 ~]# ping node4 [root@host104 ~]# vim /etc/fstab node1:dis-volume /test/dis glusterfs defaults,_netdev 0 0 node1:stripe-volume /test/stripe glusterfs defaults,_netdev 0 0node1:rep-volume /test/rep glusterfs defaults,_netdev 0 0 node1:dis-stripe /test/dis_stripe glusterfs defaults,_netdev 0 0node1:dis-rep /test/dis_rep glusterfs defaults,_netdev 0 0 [root@host104 ~]# mount -a [root@host104 ~]# df -h 
```

![image-20210926173229991](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916026.png)









# 六：测试GlusterFS 文件系统

## 6.1 卷中写入文件，客户端操作

```
复制[root@host104 opt]# dd if=/dev/zero of=/opt/test1.txt bs=1M count=40 [root@host104 opt]# dd if=/dev/zero of=/opt/test2.txt bs=1M count=40 [root@host104 opt]# dd if=/dev/zero of=/opt/test3.txt bs=1M count=40 [root@host104 opt]# dd if=/dev/zero of=/opt/test4.txt bs=1M count=40 [root@host104 opt]# dd if=/dev/zero of=/opt/test5.txt bs=1M count=40  [root@host104 opt]# cp test* /test/dis [root@host104 opt]# cp test* /test/stripe/ [root@host104 opt]# cp test* /test/rep/ [root@host104 opt]# cp test* /test/dis_stripe/ [root@host104 opt]# cp test* /test/dis_rep/ 
```





## 6.2 查看文件分布

### 6.2.1 node1,node2 查看分布式文件分布

```
复制[root@node1 ~]# ls -lh /data/sdb1 [root@node2 ~]# ls -lh  /data/sdb1 
```

![image-20210926174251359](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916027.png)



### 6.2.2 node1，node2 查看条带卷文件分布

```bash
复制[root@node1 ~]# ls -lh /data/sdc1
[root@node2 ~]# ls -lh /data/sdc1
```

![image-20210926174521004](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916028.png)



### 6.2.3 node3，node4查看复制卷

```bash
复制[root@node3 ~]# ls -lh /data/sdb1
[root@node4 ~]# ls -lh /data/sdb1
```

![image-20210926174928364](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916029.png)



### 6.2.4 node1,node2,node3,node4 查看分布式条带卷 

```bash
复制[root@node1 ~]# ls -lh /data/sdd1
[root@node2 ~]# ls -lh /data/sdd1
[root@node3 ~]# ls -lh /data/sdd1
[root@node4 ~]# ls -lh /data/sdd1
```

![image-20210926175511690](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916030.png)



### 6.2.5 node1,node2,node3,node4 查看分布式复制卷

```bash
复制[root@node1 ~]# ls -lh /data/sde1
[root@node2 ~]# ls -lh /data/sde1
[root@node3 ~]# ls -lh /data/sde1
[root@node4 ~]# ls -lh /data/sde1
```

![image-20210926175912767](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916031.png)





## 6.3 破坏性测试

（虚拟机）挂起node2节点或者关闭glusterd服务来模拟故障

```
复制虚拟机挂起node2或者停止glusterd服务 [root@node2 ~]# systemctl stop glusterd.service 
```

![image-20210926180501681](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916032.png)



### 6.3.1 客户端查看分布式卷数据

```bash
复制[root@host104 ~]# ls /test/dis/
test1.txt  test3.txt  test4.txt
```

![image-20210926180732479](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916033.png)



### 6.3.2 客户端查看条带卷数据

```bash
复制[root@host104 ~]# ls  /test/stripe/
[root@host104 ~]#
```

![image-20210926180955421](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916034.png)



## 6.3.3 客户端查看分布式条带卷

```bash
复制[root@host104 ~]# ls /test/dis_stripe/
test2.txt  test5.txt
```

![image-20210926181614973](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916035.png)



### 6.3.4 客户端查看分布式复制卷

```bash
复制[root@host104 ~]# ls /test/dis_rep/
test1.txt  test2.txt  test3.txt  test4.txt  test5.txt
```

![image-20210926182014757](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916036.png)



### 6.3.5 宕机node3.客户端查看复制卷

```bash
复制[root@host104 ~]# ls /test/rep/
test1.txt  test2.txt  test3.txt  test4.txt  test5.txt
```

![image-20210926182349039](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916037.png)



### 6.3.6 小结

所以，凡是带复制数据的，相比而言数据比较安全。在写入数据时，条带式卷比复制卷和分布式卷速度更快





## 6.4 其他维护命令

### 6.4.1 查看 GlusterFS卷

```bash
复制[root@node1 ~]# gluster volume list
```

![image-20210926183107839](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916038.png)



### 6.4.2 查看卷信息

```bash
复制#查看卷dis-volume 的信息
[root@node1 ~]# gluster volume  info dis-volume

#查看所有卷的信息
[root@node1 ~]# gluster volume info
```

![image-20210926183427809](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916039.png)



### 6.4.3 查看卷状态

```bash
复制#查看 卷 dis—volume的状态
[root@node1 ~]# gluster volume status dis-volume

#查看所有卷的状态
[root@node1 ~]# gluster volume status
```

![image-20210926183747266](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916040.png)



### 6.2.4 停止一个卷

```bash
复制[root@node1 ~]# gluster volume stop dis-stripe 
Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y 
```

![image-20210926184138449](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916041.png)



### 6.2.5 删除一个卷 

删除卷时，需要先停止卷，且信任池中不能有主机处于宕机状态，否则删除不成功

```bash
复制[root@node1 ~]# gluster volume delete dis-stripe 
Deleting volume will erase all information about the volume. Do you want to continue? (y/n) y
```

![image-20210926184528698](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916042.png)



**当没有停止卷而尝试删除卷，则报错**

```bash
复制[root@node1 ~]# gluster volume delete dis-rep 
Deleting volume will erase all information about the volume. Do you want to continue? (y/n) y
volume delete: dis-rep: failed: Volume dis-rep has been started.Volume needs to be stopped before deletion.
```

![image-20210926185133657](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916043.png)



**当信任池中有处于宕机状态的主机,删除时则报错**

```bash
复制#宕机 node3.然后停止卷 rep-volume，并尝试删除
[root@node1 ~]# gluster volume stop  rep-volume
[root@node1 ~]# gluster volume delete  rep-volume 
#报错
volume delete: rep-volume: failed: Some of the peers are down
```

![image-20210926185636238](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916044.png)



### 6.2.6设置卷的访问控制值

**设置白名单（仅允许）**

```bash
复制[root@node1 ~]# gluster volume  set dis-rep auth.allow 192.168.23.104
volume set: success
```

![image-20210926190316004](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916045.png)



**设置黑名单（仅拒绝）**

```bash
复制[root@node1 ~]# gluster volume set rep-volume auth.reject 192.168.23.104
volume set: success
```

![image-20210926191539086](https://gitee.com/zhijiyiyu/bloglmage/raw/master/img/202109261916046.png)

分类: [分布式文件系统与企业级应用](https://www.cnblogs.com/zhijiyiyu/category/2036946.html)



[GlusterFS（GFS） 分布式存储 - 知己一语 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhijiyiyu/p/15339674.html)