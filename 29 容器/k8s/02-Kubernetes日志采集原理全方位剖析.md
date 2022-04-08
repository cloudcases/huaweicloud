# Kubernetes日志采集原理全方位剖析

来自：[阿里云容器服务 ACK](https://developer.aliyun.com/group/kubernetes) 2021-11-17 670

**简介：** 本文将主要介绍SLS对于Kubernetes日志采集的基本原理，便于大家在实践中能够更好的规划使用方式。

## 简介

作为容器编排领域的实施标准，Kubernetes（K8s）应用的场景也越来越广。日志作为可观测性建设中的重要一环，可以记录详细的访问请求以及错误信息，非常利于问题的定位。

Kubernetes上的应用、Kubernetes组件本身、宿主机等都会产生各类日志数据，SLS全面支持这些数据的采集和分析。

本文将主要介绍SLS对于Kubernetes日志采集的基本原理，便于大家在实践中能够更好的规划使用方式。

## Kubernetes日志采集方式

在K8s中，日志采集一般分为Sidecar和DaemonSet两种方式。一般建议**DaemonSet在中小型集群中使用**；**Sidecar推荐在超大型的集群中使用**（为多个业务方提供服务，每个业务方有明确的自定义日志采集需求，采集配置数会超过500）。

- DaemonSet方式**在每个node节点上只运行一个日志agent，采集这个节点上所有的日志**。DaemonSet相对**资源占用要小很多，但扩展性、租户隔离性受限，比较适用于功能单一或业务不是很多的集群**。
- Sidecar方式**为每个POD单独部署日志agent，这个agent只负责一个业务应用的日志采集**。Sidecar相对**资源占用较多，但灵活性以及多租户隔离性较强，建议大型的K8S集群或作为PAAS平台为多个业务方服务的集群使用该方式**。

|              | **DaemonSet方式**                                            | **Sidecar方式**                                              |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 采集日志类型 | 标准输出+部分文件                                            | 文件                                                         |
| 部署运维     | 一般，需维护DaemonSet                                        | 较高，每个需要采集日志的POD都需要部署sidecar容器             |
| 日志分类存储 | 一般，可通过容器/路径等映射                                  | 每个POD可单独配置，灵活性高                                  |
| 多租户隔离   | 一般，只能通过配置间隔离                                     | 强，通过容器进行隔离，可单独分配资源                         |
| 支持集群规模 | 取决于采集配置数（由于每个节点的Agent都需要加载所有配置并工作，所以集群的采集配置数有上限，一般不建议超过500个采集配置） | 无限制                                                       |
| 资源占用     | 较低，每个节点运行一个容器                                   | 较高，每个POD运行一个容器                                    |
| 查询便捷性   | 较高，可进行自定义的查询、统计                               | 高，可根据业务特点进行定制                                   |
| 可定制性     | 低                                                           | 高，每个POD单独配置                                          |
| 耦合度       | 低，Agent可独立升级                                          | 一般，默认采集Agent升级对应Sidecar业务也会重启（有一些扩展包可以支持Sidecar热升级） |
| 适用场景     | 日志分类明确、功能较单一的集群                               | 大型、混合型、PAAS型集群                                     |

## SLS日志采集原理

日志采集流程通常包括3个部分：**部署Agent、配置Agent、Agent按照配置工作**。

SLS的日志采集流程也基本类似，相比开源采集软件提供了更多的部署、配置和采集方式。在K8s中，部署方式主要分为DaemonSet和Sidecar两种；配置方式包括CRD配置、环境变量配置、控制台配置、API配置；采集方式支持容器文件采集、容器标准输出采集和标准文件采集。整个流程包括：

1. 部署Logtail：支持DaemonSet和Sidecar两种部署方式（第一部分将介绍这两种方式的采集原理）。
2. 创建采集配置：支持CRD配置、环境变量配置、控制台配置、API配置等方式（第二部分将介绍这几种配置的优劣以及适用场景）。
3. Logtail按照配置采集数据：Logtail获取步骤2中创建的采集配置并根据配置内容进行工作。

注意：默认DaemonSet和Sidecar模式全部使用自定义标识的机器组，适用于节点/容器自动扩容场景，所有的采集配置必须挂载到机器组中才能生效。

注意：Logtail的采集配置全部从服务端获取，当Logtail连通服务端后，会将所在的机器组关联的采集配置自动同步到本地并开始工作。

## DaemonSet日志采集原理

<img src="https://cdn.nlark.com/yuque/0/2021/png/347081/1637153797475-1c1815ce-b61d-48d6-8b19-0759c481635e.png" alt="img" style="zoom:33%;" />

DaemonSet（相关文档可以参考：[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)）和Deployment、StatefullSet都是Kubernetes中对于Pod的一种高级编排方式（Controller），**Deployment和StatefullSet都会定义一个副本数，K8s根据副本数进行调度**。而**DaemonSet不需要指定副本数，默认会在每个节点上启动一个Node，一般用于采集日志、监控、磁盘清理等运维相关的工作**。因此Logtail的日志采集默认推荐的就是DaemonSet采集方式。

DaemonSet模式下，默认安装的Logtail会在kube-system命名空间下，DaemonSet名为logtail-ds，每个节点的Logtail的Pod负责采集这个节点上所有运行的Pod的数据（包括标准输出和文件）。

下述命令可以查看到logtail-ds运行的Pod的状态：`kubectl get pod -n kube-system | grep logtail-ds`

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153797639-a3c5014f-1e18-4b87-afec-cf8efd7a78fd.png)



<img src="https://cdn.nlark.com/yuque/0/2021/png/347081/1637153797790-c0049e4c-db42-4706-888a-952ee41063ef.png" alt="img" style="zoom: 33%;" />

### 前提条件

Logtail采集其他Pod/容器的前提是需要能够访问到宿主机上的容器运行时并且可以访问到其他容器的数据：

- 访问容器运行时：Logtail容器会将宿主机上容器运行时（Docker Engine/ContainerD）的sock挂载到容器目录中，因此Logtail容器就可以访问到这台节点的Docker Engine/ContainerD
- 访问其他容器的数据：Logtail容器会将宿主机的根目录（'/'目录）挂载到容器的/logtail_host挂载点上，因此可以通过/logtail_host目录访问其他容器的数据（前提是容器运行时中的文件系统以普通文件系统的形式存储在宿主机上，一般overlayfs即可，或者容器的日志目录挂载到宿主机上，例如以hostPath或emptyDir方式挂载）

### 工作流程

在具备这两个前提条件后，Logtail会从SLS服务端加载采集配置并开始工作，针对容器的日志采集，Logtail启动后的工作流程主要分为两个部分：

1. 发现需要被采集容器，该流程主要包括：

1. 从容器运行时（Docker Engine/ContainerD）中获取到所有的容器及其配置信息，例如容器名、ID、挂载点、环境变量、Label等
2. 根据采集配置中的IncludeEnv、ExcludeEnv、IncludeLabel、ExcludeLabel定位需要采集的容器。通过这种采集配置的匹配关系，可以针对性的定位需要采集的目标，方式采集全部容器而造成资源浪费、数据难以拆分等问题。如下图的示例：配置1只采集Env为Pre的容器，配置2只采集APP为APP1的容器，配置3只采集非ENV为Pre的容器，这三个配置会把对应不同的数据采集到不同的Logstore。

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153800728-3a8c7049-6958-45db-abec-a4e649217b44.png?x-oss-process=image%2Fresize%2Cw_2202)

1. 采集容器数据，该流程主要包括：

1. 确定容器被采集的数据地址，包括标准输出和文件的采集地址。这些信息主要在容器的配置中，例如下图中就标识了标准输出的LogPath以及容器文件的存储路径。需要注意的是：

1. 标准输出：容器的标准输出最终必须保存成文件才能被采集到，对于DockerEngine和ContainerD都需要配置[LogDriver](https://docs.docker.com/config/containers/logging/configure/)，配置为json-file、local即可（一般默认配置都会保存到文件，所以大部分情况不需要关心）。
2. 容器文件：容器文件为的系统为overlay时，会自动根据UpperDir查找到所有容器中的文件；但是ContainerD默认的配置为devicemapper，这种情况下必须把日志挂载到HostPath或者EmptyDir才可以查找到对应的路径。

1. 根据对应的地址采集数据，其中对于标准输出比较特殊，需要去解析标准输出文件，从而得到用户实际的标准输出
2. 根据配置的解析规则解析原始的日志，解析规则包括：行首正则、字段提取（正则、分隔符、JSON、Anchor等）、过滤/丢弃、脱敏等
3. 上传数据到SLS

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153797587-500133a8-a2b8-496a-87ca-a116359da219.png?x-oss-process=image%2Fresize%2Cw_2202)

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153800407-d400fad7-488b-47f7-afc2-72fd0954f1ca.png?x-oss-process=image%2Fresize%2Cw_2202)

## Sidecar日志采集原理

<img src="https://cdn.nlark.com/yuque/0/2021/png/347081/1637153799380-cb127271-f377-4c0e-a436-50a116a1d1b7.png" alt="img" style="zoom: 50%;" />

**在K8s中，一个Pod可以运行多个容器，这些容器会共享一个命名空间**，其中我们一般称核心的工作容器为主容器，其他容器为[Sidecar容器](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)。

**Sidecar容器一般用作辅助作用，通过共享的Volume，实现例如同步文件、采集监控/日志、文件清理等功能**。

Logtail的Sidecar采集也是同样的原理，Sidecar模式下，除了业务主容器外，还会运行Logtail的Sidecar容器，Logtail容器和主容器共享日志的Volume。采集的流程如下：

1. 业务容器将日志打印到共享Volume（只能是文件，标准输出无法写入到共享volume）
2. Logtail通过共享Volume监控日志，发现日志变更后，采集到SLS

关于Sidecar模式的使用最佳实践，请参考：《[通过Sidecar方式采集日志](https://help.aliyun.com/document_detail/157315.html)》[《Kubernetes 文件采集实践：Sidecar + hostPath 卷》](https://developer.aliyun.com/article/801162)。

## 采集配置原理

SLS的日志采集Logtail支持CRD配置、环境变量配置、控制台配置、API配置等方式，针对不同的配置场景，我们的建议是：

1. 如果对于CICD自动化部署和运维程度要求较高的用户，使用CRD配置方式
2. 环境较为静态（发布不是很频繁，日志采集策略变化不频繁）的场景，建议使用控制台配置方式
3. 高端玩家可以考虑API自定义配置
4. 环境变量配置方式限制性和能力较弱，一般不推荐使用

每种配置方式的优缺点对比如下：

|                    | CRD配置                                                      | 环境变量配置                                                 | 控制台配置                                                   | API配置                                                      |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 适用场景           | CICD自动化日志采集配置，复杂日志采集配置和管理               | 日志采集定制化需求低，应用简单、日志种类少、复杂度较低（一般情况下不推荐此种配置） | 手动管理日志采集配置                                         | 高级定制化需求，有较高开发能力的用户                         |
| 配置难度           | 一般，需要了解SLS的配置格式                                  | 低，只需要配置环境变量即可                                   | 低，控制台引导，配置简单                                     | 高，需要使用SLS的SDK且需要了解SLS的Logtail配置格式           |
| 采集配置定制化程度 | 高，支持SLS所有的配置参数                                    | 低，只支持配置文件地址，其他解析方式不支持配置               | 高，支持SLS所有的配置参数                                    | 高，支持SLS所有的配置参数                                    |
| 可运维程度         | 较高，通过K8s的CRD接口进行管理                               | 低，只支持创建配置，不支持修改、删除                         | 一般，需要手动管理                                           | 高，可根据SLS接口定制适用于自己业务场景的管理方式            |
| 与CICD场景结合能力 | 高，CRD本质上是K8s的接口，因此支持K8s CICD自动化的流程均支持 | 高，环境变量配置在Pod上，可以无缝集成                        | 低，需要手动处理                                             | 高，适用于自己开发CICD流程的场景                             |
| 使用说明           | [DaemonSet模式下使用CRD](https://help.aliyun.com/document_detail/74878.html)；[Sidecar模式下使用CRD](https://help.aliyun.com/document_detail/100575.html) | [环境变量配置方式](https://help.aliyun.com/document_detail/87540.html) | [DaemonSet模式下使用控制台配置](https://help.aliyun.com/document_detail/66658.html)；[Sidecar模式下使用控制台配置](https://help.aliyun.com/document_detail/157315.html) | [Logtail配置接口](https://help.aliyun.com/document_detail/29041.html)；[Logtail配置格式](https://help.aliyun.com/document_detail/29058.html)；[各类SDK参考](https://help.aliyun.com/document_detail/29063.html) |

## CRD（Operator）配置方式

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153800247-617e4260-f6c4-4135-8766-71b234abeb32.png)

日志服务为K8s新增了一个[CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)扩展，名为AliyunLogConfig。同时开发了alibaba-log-controller用于监听AliyunLogConfig事件并自动创建Logtail的采集配置。当用户创建/删除/修改AliyunLogConfig资源时，alibaba-log-controller会监听到资源变化，并对应的在日志服务上创建/删除/修改相应的采集配置。以此实现K8S内部AliyunLogConfig与日志服务中采集配置的关联关系。

### CRD AliyunLogConfig实现方式

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153800301-90125a6c-d267-4f16-b8d5-196076585830.png)

如上图所示，日志服务为K8S新增了一个[CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)扩展，名为AliyunLogConfig。同时开发了alibaba-log-controller用于监听AliyunLogConfig事件。

当用户创建/删除/修改AliyunLogConfig资源时，alibaba-log-controller会监听到资源变化，并对应的在日志服务上创建/删除/修改相应的采集配置。以此实现K8S内部AliyunLogConfig与日志服务中采集配置的关联关系。

### alibaba-log-controller内部实现

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153801839-204d8f83-dadd-46aa-90e0-cba2ced60ec9.png)

alibaba-log-controller主要由6个模块组成，各个模块的功能以及依赖关系如上图所示：

- EventListener：负责监听AliyunLogConfig的CRD资源。这个EventListener是广义上的listener，主要功能有

- 初始化时会list所有的AliyunLogConfig资源
- 注册AliyunLogConfig监听变化的事件
- 定期再扫描全量的AliyunLogConfig资源防止事件出现遗漏或处理失效
- 将事件打包，交由EventHandler处理

- EventHandler：负责处理对应的Create/Update/Delete事件，作为Controller的核心模块，主要功能如下：

- 首先检查ConfigMapManager中对应的checkpoint，如该事件已经被处理（版本号相同且状态为200），则直接跳过
- 为防止历史事件干扰处理结果，从服务端拉取最新的资源状态，检查是否为同一版本，若版本不一致，使用服务端版本替换
- 对事件进行一定的预处理，使之符合LogSDK的基本格式需求
- 调用LogSDKWrapper，创建日志服务Logstore，Create/Update/Delete对应的配置
- 根据上述处理结果，更新对应AliyunLogConfig资源的状态

- ConfigMapManager：依赖于K8S的ConfigMap机制实现Controller的checkpoint管理，包括：

- 维护checkpoint到ConfigMap的映射关系
- 提供基础的checkpoint增删改查接口

- LogSDKWrapper：基于阿里云LOG golang sdk的二次封装，功能包括：

- 初始化创建日志服务资源，包括Project、MachineGroup、Operation Logstore等
- 将CRD资源转换为对应的日志服务资源操作，为1对多关系
- 包装SDK接口，自动处理网络异常、服务器异常、权限异常
- 负责权限管理，包括自动获取role，更新sts token等

- ScheduledSyner：后台的定期同步模块，防止进程/节点失效期间配置改动而遗漏事件，保证配置管理的最终一致性：

- 定期刷新所有的checkpoint和AliyunLogConfig
- 检查checkpoint和AliyunLogConfig资源的映射关系，如果checkpoint中出现不存在的配置，则删除对应的资源

- Monitor：alibaba-log-controller除了将本地运行日志输出到stdout外，还会将日志直接采集到日志服务，便于远程排查问题。采集日志种类如下：

- k8s api内部异常日志
- alibaba-log-controller运行日志
- alibaba-log-controller内部异常数据（自动聚合）

## 环境变量配置方式

![img](https://cdn.nlark.com/yuque/0/2021/png/347081/1637153803895-1f2d3c2f-e497-40b2-bae2-08085b0d4123.png?x-oss-process=image%2Fresize%2Cw_1476)

环境变量的配置相对是最简单的配置方式，用户在配置Pod的时候只需要加上特殊字段`aliyun_logs`开头的环境变量即可完成配置的定义和数据的采集。该种方式是通过Logtail来实现：

1. Logtail从容器运行时（Docker Engine/ContainerD）获取所有的容器列表
2. 对于运行的容器，查看环境变量中是否包含`aliyun_logs`开头的环境变量
3. 对于`aliyun_logs`开头的环境变量，映射成SLS的Logtail采集配置，调用SLS的接口创建采集配置
4. Logtail获取服务端的采集配置并开始工作

## 推荐使用方式

K8s日志采集方案有多种实施方式，复杂度和效果也各不相同，通常我们需要选择采集方式以及配置方式，这里我们推荐的使用方式如下：

采集方式

- DaemonSet：比较适用于**功能单一或业务不是很多的集群**，需要控制整个集群的采集配置数量不超过500个，否则Logtail的资源占用会较多。
- Sidecar：建议**大型的K8S集群或作为PAAS平台为多个业务方服务的集群**使用该方式，典型的衡量标准是有超过500个以上的采集配置存在。
- 混用方式：容器的标准输出、系统日志、部分业务日志使用DaemonSet方式；对于某些日志采集可靠性要求较高的Pod使用Sidecar方式。

配置方式
- 如果对于CICD自动化部署和运维程度要求较高的用户，使用CRD配置方式
- 环境较为静态（发布不是很频繁，日志采集策略变化不频繁）的场景，建议使用控制台配置方式
- 高端玩家可以考虑API自定义配置

# 参考：

1. https://help.aliyun.com/document_detail/157315.html
2. https://developer.aliyun.com/article/801162
3. https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers
4. https://docs.docker.com/config/containers/logging/configure/
5. https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/



- 获得第一手资料与支持：

![img](https://cdn.nlark.com/yuque/0/2021/png/668175/1635946581334-0d9c7204-708b-4983-b455-a167d1938873.png)

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。



[Kubernetes日志采集原理全方位剖析-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/806369?spm=a2c6h.12873639.0.0.5e5b5681EPdMh1)