# OpenShift简介

[虹科云科技](https://www.zhihu.com/people/hong-ke-yun-ke-ji)

## 1.OpenShift是什么？

- openshift是红帽的云开发**平台即服务（PaaS），底层以Docker作为容器引擎驱动，以k8s作为容器编排引擎组件**，openshift提供了开发语言、中间件、自动化流程工具及界面等元素，提供了一套完整的基于容器的应用云平台。
- **OKD**是 Kubernetes 的一个发行版，**针对持续应用程序开发和多租户部署进行了优化**。OKD 还充当上游代码库，[红帽 OpenShift Online](https://link.zhihu.com/?target=https%3A//www.openshift.com/products/online/)和[红帽 OpenShift 容器平台](https://link.zhihu.com/?target=https%3A//www.openshift.com/products/container-platform/)基于这些代码库构建。
- 自由和开放的云计算平台使开发人员能够创建、测试和运行他们的应用程序，井且可以把它们部署到云中。 Openshift广泛支持多种程语言和框架，如Java,Ruby和PHIP等。另外它还提供了多种集成开发工具如 Eclipse integration, JBOSS Developer Studio和 Jenkins等， Openshift基于个开源生态系统为移动应用，数据库服务等，提供支持。
- OCP即Openshift Container Platform.
- 构建内部应用市场，提供中间件、数据库自动化的流程，可以快速进行应用的构建、容器化和部署贯通从应用开发测试、上线的测试。**开发、测试、运维都可以同一个平台上协作提高研发效率**。
- 架构：

![img](https://pic2.zhimg.com/80/v2-46541baa00c36a35022e8ebed315d619_1440w.jpg)



### OpenShift与K8S的区别



![img](https://pic2.zhimg.com/80/v2-6a0a0a6194d735f498f10ed542876f89_1440w.jpg)

### 1. 应用部署

K8S的应用程序部署过程较为灵活，用户有很多需要选择的地方；openshift提供的自动化流程多一些，为其提供了DevOps和管道方法，用户只需要创建一个应用程序和一个项目。

### 2. 应用管理

对于大多数运营团队来说，K8S默认的仪表盘界面无法满足他们的需求，也许他们会使用ELK堆栈之类的；OpenShift的web控制台建立在Kubernetes API基础上，并具有许多不同的功能。让SRE和运营团队管理其工作负载。如果想使用istio等功能，可以采用多种方式集成。再利用一些自动安装程序和Ansible playbook，管理应用程序会简单一些。

### 3. 节点配置

Kubernetes将VM加入集群的方式会比较耗时，并且需要开发脚本，OpenShift有Ansible playbook和安装程序，将新的虚拟机引入集群。

### 4. 安全性

通过企业客户合作，从头开始创建最佳安全实践，能够解决客户的一些问题，才能使用Kubernetes，K8S对于团队成员的权限管理不是那么方便。而对于OpenShift，只需要添加用户就可以处理类似命名空间的隔离空间，根据最佳做法创建不同的安全策略，并且某些容器的运行需要root用户权限才可以。

## 2. 为什么要用OpenShift？

通过 Openshift这个平台，企业可以**快速在内部网络中构建出一个多租户的云平台，在这朵云上提供应用开发、测试、部署、运维的各项服务**。 **Openshift在一个平台上贯通开发、测试、部署、运维的流程，实现高度的自动化，满足应用持续集成及持续交付和部署的需求；满足企业及组织对容器管理、容器编排的需求**。通过 Openshift的灵活架构，企业可以以 Openshift作为核心，在其上搭建一个企业的 Devops引擎，推动企业的 Devops变革和转型。

对于开发人员来说，他们要做的就是编写应用程序，更改修改测试并部署到集群之中，任何其他流程的干扰都会减慢程序部署上线的速度。在OpenShift上，开发人员要做的就是创建一个项目和一个应用程序，OpenShift提供Web控制台和CIL，以及各种不同类型的源代码的模板。在开发人员编写好代码后，OpenShift会创建Jenkins和管道进行之后的流程，创建镜像，将镜像推送到Registry，利用Image Stream将更改测试后的应用程序将部署到集群中。

## 3. OpenShift的版本

主要有三个版本：

1. **OpenShift Origin** : 这是OpenShift的社区添加或开源版本.它也被称为其他两个版本的上游项目。
2. **OpenShift Online** : 它是一个公共PaaS作为在AWS上托管的服务。（目前不再用）
3. **OpenShift Enterprise** : 是具有ISV和供应商许可证的OpenShift的强化版本。

## 4. OpenShift架构概览

从技术堆栈的角度分析，作为一个容器云， Openshift自底而上包含了以下几个层次基础架构层、容器引擎层、容器编排层、PaaS服务层、界面及工具层，如下图所示。

![img](https://pic1.zhimg.com/80/v2-af00d92ceeb5f9589ee1fbfe40b61434_1440w.jpg)

### 4.1基础架构层

基础架构层为 Openshift平台的运行提供了基础的运行环境。 OpenShift支持运行在物理机、虚拟机、基础架构云(如 OpenStack、 Amazon Web Service、 Microsoft Azure等)或混合云上。在操作系统层面， Openshift支持多种不同的 Linux操作系统，如企业级的RedHat Enterprise Linux、社区的 Centos。

### 4.2 容器引擎层

目前以Docker（原生）作为容器引擎。

### 4.3 容器编排层

目前以Kubernetes作为容器编排引擎。

### 4.4 PaaS服务层

OpenShift在PaaS服务层默认提供了丰富的开发语言、开发框架、数据库以及中间件的支持。用户可以在OpenShift平台上快速部署和获取一个数据库、分布式缓存或者业务规则引擎的服务。

### 4.5 界面及工具层

- OpenShift提供了自动化流程Source to Image，即S2I，帮助用户容器化用各种编程语言开发的应用源代码。用户可以直接使用S2I或者把现有的流程与S2I整合，从而实现开发流程的持绫集成和持续交付。提升开发、测试和部署的自动化程度，最终提高开发、测试及部署的效率，缩短上市时间。 Openshift提供了多种用户的接入渠道：Web控制台、命令行、IDE集成及 RESTFUL编程接口。这些都是一个完善的企业级平台必不可少的组件。
- 针对容器应用的运维及集群的运维， Openshift提供了性能度量采集、日志聚合模块及运维管理套件，帮助运维用户完成日常的应用及集群运维任务。

## 5. 核心组件详解

OpenShift的核心组件及其之间的关联关系如图所示。Openshift集群可以由一台或多台主机组成。这些主机可以是物理机或虚拟机，同时可以运行在私有云、公有云，或混合云上。在OpenShift的集群成员有两种角色：Master节点和Node节点。

![img](https://pic2.zhimg.com/80/v2-e783cc76f9a4cf9a858ecca3993da225_1440w.jpg)

### 5.1 Master

主控节点。集群内的管理组件均匀性在Master节点上，它主要负责管理和维护OpenShift集群的状态。它的服务组件如下：

### API Server

负责提供集群的Web Console以及RESTful API服务。集群内所有Node节点都会访问API Server更新各节点的状态及其上运行的容器状态。

### 数据源（Data Store）

集群内所有动态的状态信息都会存储在后端一的一个etcd分布式数据库中。默认的etcd实例安装在Master节点上，如有需要，也可以将etcd节点部署在集群之外。

### 调度控制器（Scheduler）

负责按照用户输入的要求合适的计算节点。

### 复制控制器（Replication Controller）

负责监控当前容器实例的数量和用户部署指定的数量是否匹配，保证实际的容器实例数量等于部署定义的数量。

### 5.2 Node节点

计算节点，集群内的容器实例均运行在Node节点之上。负责接受Master结点的指令，运行和维护Docker容器。

### 5.3 Project和Namespace

在同一个命名空间中，某一个对象的名称在其分类中必须是唯一的，但是分布在不同命名空间中的对象则可以同名。每一个Project会和一个Namespace相关联，也可以简单理解为Project就是Namespace，所以在OpenShift中进行操作时，要确认当前执行的上下文是哪一个Project。

查看当前用户所在的Project

```text
[lab-user@bastion ~]$ oc project
Using project "myproject" on server "https://api.cluster-v47h4.dynamic.opentlc.com:6443".
```

### 5.4 Pod

Pod相当于豌豆的豆荚，容器相当于豌豆。和K8S中的概念相同。

### 5.5 Service

为了克服容器变化引发的连接信息变化，K8S提供了Service。当部署某个应用时，我们会为该应用创建一个Service对象。它会与该应用的一个或多个Pod相关联，它会被分配一个相对恒定的IP地址。

- 通过访问这个IP地址以及对应的端口，请求就会被转发到Pod对应的端口上。Service起到了代理作用。
- 可以通过域名访问某一个Service。监听在Master上的集群内置DNS服务器会负责解析这个DSN请求。
- Service域名的格式为：..svc.cluster.local.

### 5.6 Router与Route

Router（路由器）：一个运行在容器内特殊定制的Haproxy。用户可以创建一个Route，一个Route会与一个Service相关联，并且绑定一个域名。Route规则会被Router加载。当用户通过指定域名访问应用时，域名会被解析并指向Router所在的计算节点上。Router 取这个请求，然后根据 Route 规则定义转发给与这个域名对应的 Service 后端所关联的 Pod 容器实例。

*Router与Service的区别：*

*Router是将集群外的请求转发到集群的容器；Service是把集群内的请求转发到指定的容器中。*

### 5.7 Persistent Storage

容器默认非持久化，容器云平台必须为容器提供持久化存储（Persistent Storage）。Openshift不仅支持Docker持久化卷的挂载方式，而且还提供一种持久化供给模型，即Persistent Volume （持久化卷， PV ）及 Persistent Volume Claim （持久化卷请求， PVC ）模型。

在PV 和PVC 模型中，集群 管理员会创建大量不同大小和不同特性的 PV 。用户在部署应用时，显式声明对持久化的需求，创建 PVC 。用户在 PVC 中定义所需存储的大小、访问方式（只读或可读可写；独占或共享） OpenShift 集群会自动寻找符合要求的 PV与PVC 自动对接。 通过 PV和PVC 模型， OpenShift出为用户提供了 种灵活的方式来消费存储资源 。

OpenShift 对持久化后端的支持比较广泛，除了 NFS 以及iSCSI 外，还支持如 Ceph、GluterFS 等的分布式储存，以及 Amazon WebService和Google Compute Engine 云硬盘等。

### 5.8 Registry

OpenShift提供了一个内部的 Docker 镜像仓库（ Registry ），该镜像仓库用于存放用户通过内置的Source to Image 镜像构建流程所产生的镜像Registry组件默认以容器的方式提供。

每当S2I 完成镜像构建，就会向内部的镜像仓库推送构建完成的镜像 。

*是不是 OpenShift用 到的镜像都要存放到内置的仓库？*

*不是的，内部的镜像仓库存放的只是S2I 产生的镜像。其他镜像可以存放在集群外部 的镜像仓库，如企业的镜像仓库或社区的镜像仓库。只要保证 OpenShift的节点可以 访问到这些镜像所在的镜像仓库即可。*

### 5.9 Source to Image

S2I即Source to Image， 是OpenShift的一个重要功能。容器镜像是容器云的应用交付格式。容器镜像中包含了应用及其所依赖的运行环境。可以从社区或者第三方厂商获取基础的操作系统或者中间件的镜像。但是这些外部获取的操作系统或中间件的镜像并不包含企业内部开发制的应用。企业内部的开发人员必须自行基于外部的基础镜像构建包含企业自身开发的应用。这个镜像的构建过程是必须的，要么由企业的 IT人员手工完成，要么使用某种工具实现自动化。

作为一个面向应用的平台， OpenShift 提供了S2I 的流程，使得企业内容器的构建变得标准化和自动化，从而提高了软件从开发到上线的效率。一个典型的S2I 流程包含了以下几个 步骤。

1. 用户输入源代码仓库的地址。
2. 用户选择S2I 构建的基础镜像（又称为 Builder 镜像）。
3. Builder镜像中包含了操作系统、编程语言、框架等应用所需的软件及配置 。OpenShift默认提供了多种编程语言的Builder镜像，如Java、PHP、Ruby、Python Perl 等。用户也可以根据自身需求定制自己的Builder镜像，并发布到服务目录中供用户选用。
4. 用户或系统触发S2I构建。OpenShift将实例化S2I构建执行器。
5. S2I 构建执行器将从用户指定的代码仓库下载源代码。
6. S2I 构建执行器实例化Builder镜像。代码将会被注入Builder镜像中 。
7. Builder 镜像将根据预定义的逻辑执行源代码的编译、构建并完成部署 。
8. S2I构建执行器将完成操作的Builder镜像并生成新的Docker镜像 。
9. S2I构建执行器将新的镜像推送到OpenShift内部的镜像仓库 。
10. S2I构建执行器更新该次构建相关的Image Stream信息 。

S2I构建完成后，根据用户定义的部署逻辑，OpenShift将把镜像实例化部署到集群中 。

除了接受源代码仓库地址作为输入外，S2I还接受Dockerfile以及二进制文件作为构建的输入。 用户甚至可以完全自定义构建逻辑来满足特殊的需求。

### 5.10 开发及管理工具集

OpenShift位提供了不同的工具集为开发和运维的用户提供良好的体验，也为持续集成和打通DevOps流程提供便利。例如，OpenShift提供了 Eclipse插件，开发程师可以在Eclipse中完成应用及服务的创建和部署、远程调试、实时日志查询等功能。

发布于 2021-12-06 17:25

[OpenShift](https://www.zhihu.com/topic/19695802)

[红帽 (Red Hat)](https://www.zhihu.com/topic/19613978)



[OpenShift简介 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/441917116)