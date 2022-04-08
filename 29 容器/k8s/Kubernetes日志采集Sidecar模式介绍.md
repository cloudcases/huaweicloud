# Kubernetes日志采集Sidecar模式介绍

来自：[阿里云存储服务](https://developer.aliyun.com/group/storage) 2018-10-10 18574

**简介：** DaemonSet和Sidecar模式各有优缺点，目前没有哪种方式可以适用于所有场景。因此我们阿里云日志服务同时支持了DaemonSet以及Sidecar两种方式，并对每种方式进行了一些额外的改进，更加适用于K8S下的动态场景。

Kubernetes（K8S）作为CNCF（cloud native computing foundation）的一个核心项目，背靠Google和Redhat的强大社区，近两年发展十分迅速，在成为容器编排领域中领导者的同时，也正在朝着PAAS底座标配的方向发展。



## 日志采集方式

日志作为任一系统不可或缺的部分，在K8S的官方文档中也介绍了多种的[日志采集形式](https://kubernetes.io/docs/concepts/cluster-administration/logging/)，总结起来主要有下述3种：原生方式、DaemonSet方式和Sidecar方式。

1. 原生方式：使用 `kubectl logs` 直接在查看本地保留的日志，或者通过docker engine的 `log driver` 把日志重定向到文件、syslog、fluentd等系统中。
2. DaemonSet方式：在K8S的每个node上部署日志agent，由agent采集所有容器的日志到服务端。
3. Sidecar方式：一个POD中运行一个sidecar的日志agent容器，用于采集该POD主容器产生的日志。

![e469affc-e2ac-4c45-8283-e15c8d0357be.png](https://ucc.alicdn.com/pic/developer-ecology/b70e7fec6a164dcebf508c25ec338c47.png)



## 采集方式对比

每种采集方式都有一定的优劣势，这里我们进行简单的对比：

|              | 原生方式                                          | DaemonSet方式                    | Sidecar方式                                      |
| ------------ | ------------------------------------------------- | -------------------------------- | ------------------------------------------------ |
| 采集日志类型 | 标准输出                                          | 标准输出+部分文件                | 文件                                             |
| 部署运维     | 低，原生支持                                      | 一般，需维护DaemonSet            | 较高，每个需要采集日志的POD都需要部署sidecar容器 |
| 日志分类存储 | 无法实现                                          | 一般，可通过容器/路径等映射      | 每个POD可单独配置，灵活性高                      |
| 多租户隔离   | 弱                                                | 一般，只能通过配置间隔离         | 强，通过容器进行隔离，可单独分配资源             |
| 支持集群规模 | 本地存储无限制，若使用syslog、fluentd会有单点限制 | 中小型规模，业务数最多支持百级别 | 无限制                                           |
| 资源占用     | 低，docker engine提供                             | 较低，每个节点运行一个容器       | 较高，每个POD运行一个容器                        |
| 查询便捷性   | 低                                                | 较高，可进行自定义的查询、统计   | 高，可根据业务特点进行定制                       |
| 可定制性     | 低                                                | 低                               | 高，每个POD单独配置                              |
| 适用场景     | 测试、POC等非生产场景                             | 功能单一型的集群                 | 大型、混合型、PAAS型集群                         |

从上述表格中可以看出：

1. **原生方式相对功能太弱，一般不建议在生产系统中使用**，否则问题调查、数据统计等工作很难完成；
2. **DaemonSet方式在每个节点只允许一个日志agent，相对资源占用要小很多，但扩展性、租户隔离性受限**，比较适用于**功能单一或业务不是很多的集群**；
3. **Sidecar方式为每个POD单独部署日志agent，相对资源占用较多，但灵活性以及多租户隔离性较强**，建议**大型的K8S集群或作为PAAS平台为多个业务方服务的集群**使用该方式。



## 日志服务K8S采集方式

DaemonSet和Sidecar模式各有优缺点，目前没有哪种方式可以适用于所有场景。因此我们阿里云日志服务同时支持了DaemonSet以及Sidecar两种方式，并对每种方式进行了一些额外的改进，更加适用于K8S下的动态场景。

这两种模式均基于Logtail实现，日志服务客户端Logtail目前已有百万级部署，每天采集上万应用、数PB的数据，历经多次双11、双12考验。相关技术分享可以参见文章：[多租户隔离技术+双十一实战效果](https://yq.aliyun.com/articles/251629)，[Polling + Inotify 组合下的日志保序采集方案](https://yq.aliyun.com/articles/204554)。



## DaemonSet采集方式

DaemonSet方式下Logtail做了非常多的适配工作，包括：

- 一条命令一个参数即可实现部署，资源自动初始化。
- 支持CRD方式配置，支持K8S控制台、kubectl、kube api等，与K8S发布、部署无缝集成。
- K8S RBAC鉴权，日志服务STS鉴权管理。



![609a6e61-fafc-48e1-b9ed-a7b072a2e84d.png](https://ucc.alicdn.com/pic/developer-ecology/546b09244f844351ae0e323a0d43bd8c.png)



详细的介绍文章可以参考：
[再次升级！阿里云Kubernetes日志解决方案](https://yq.aliyun.com/articles/596310)
[LC3视角：Kubernetes下日志采集、存储与处理技术实践](https://yq.aliyun.com/articles/606248)



## Sidecar采集方式

sidecar方式的配置以及使用相对在虚拟机/物理机上采集数据区别不大，从Logtail容器视角来看：Logtail工作在一个“虚拟机”上，需要采集这个机器上某个/某些日志文件。



![5900c61e-7bf6-4b15-97c3-67cb467f5f6f.png](https://ucc.alicdn.com/pic/developer-ecology/e14cca50f4744166a4e3c4dfcc526986.png)



但在容器场景下还需解决两个问题：

1. 配置：使用编排的方式配置agent容器。
2. 动态性：需适应POD的IP地址和hostname的变化。
   目前Logtail的容器支持通过环境变量配置相关参数，并支持自定义标识的机器组进行工作，可以完美解决上述两个问题。



## Sidecar配置示例

Sidecar模式下日志组件安装、配置方式如下：

#### 步骤一： 部署Logtail容器

1. 在部署POD时将日志路径挂载到本地，并将对应的volume也挂载到Logtail容器。
2. Logtail容器需配置 `ALIYUN_LOGTAIL_USER_ID` 、 `ALIYUN_LOGTAIL_CONFIG` 、 `ALIYUN_LOGTAIL_USER_DEFINED_ID` ，参数含义以及值的选取参见：[标准Docker日志采集](https://help.aliyun.com/document_detail/66659.html)。

tips：

1. Logtail容器建议配置健康检查，在运行环境、内核等出现异常时可自动恢复。
2. 示例中使用的Logtail镜像访问的是阿里云hangzhou公网镜像仓库，您可根据需求替换成本Region的镜像，并使用内网方式。

```
apiVersion: batch/v1
kind: Job
metadata:
  name: nginx-log-sidecar-demo
  namespace: kube-system
spec:
  template:
    metadata:
      name: nginx-log-sidecar-demo
    spec:
      # volumes配置
      volumes:
      - name: nginx-log
        emptyDir: {}
      containers:
      # 主容器配置
      - name: nginx-log-demo
        image: registry.cn-hangzhou.aliyuncs.com/log-service/docker-log-test:latest
        command: ["/bin/mock_log"]
        args: ["--log-type=nginx", "--stdout=false", "--stderr=true", "--path=/var/log/nginx/access.log", "--total-count=1000000000", "--logs-per-sec=100"]
        volumeMounts:
        - name: nginx-log
          mountPath: /var/log/ngin
      # Logtail的Sidecar容器配置
      - name: logtail
        image: registry.cn-hangzhou.aliyuncs.com/log-service/logtail:latest
        env:
          # aliuid
          - name: "ALIYUN_LOGTAIL_USER_ID"
            value: "165421******3050"
          # 自定义标识机器组配置
          - name: "ALIYUN_LOGTAIL_USER_DEFINED_ID"
            value: "nginx-log-sidecar"
          # 启动配置（用于选择Logtail所在Region）
          - name: "ALIYUN_LOGTAIL_CONFIG"
            value: "/etc/ilogtail/conf/cn-hangzhou/ilogtail_config.json"
        # 和主容器共享volume
        volumeMounts:
        - name: nginx-log
          mountPath: /var/log/nginx
        # 健康检查
        livenessProbe:
          exec:
            command:
            - /etc/init.d/ilogtaild
            - status
          initialDelaySeconds: 30
          periodSeconds: 30
```



#### 步骤二： 配置机器组

如下图所示，在日志服务控制台创建一个Logtail的机器组，机器组选择自定义标识，可以动态适应POD ip地址的改变。具体操作步骤如下：

1. 开通日志服务并创建Project、Logstore，详细步骤请参考[准备流程](https://help.aliyun.com/document_detail/68225.html?spm=a2c4g.11186623.2.23.b57536deEetKyp)。
2. 在日志服务控制台的机器组列表页面单击[创建机器组](https://help.aliyun.com/document_detail/28966.html?spm=a2c4g.11186623.2.24.b57536deEetKyp)。
3. 选择用户自定义标识，将您上一步配置的 `ALIYUN_LOGTAIL_USER_DEFINED_ID`填入用户自定义标识内容框中。

![6a3f17e6-b4ce-4de2-941b-5e5d16daf28f.png](https://ucc.alicdn.com/pic/developer-ecology/39f04df982154a108523060009d11aa6.png)

#### 步骤三：配置采集方式

机器组创建完成后，即可配置对应文件的采集配置，目前支持极简、Nginx访问日志、分隔符日志、JSON日志、正则日志等格式，具体可参考：[文本日志配置方式](https://help.aliyun.com/document_detail/28967.html)。本示例中配置如下：

![img](https://cdn.nlark.com/lark/0/2018/png/33393/1539177113823-11daa00d-685e-44c2-a460-2a3880e72361.png)

#### 步骤四：查询日志

采集配置完成并应用到机器组后，1分钟内日志即可采集上来，进入对应logstore的查询页面即可查询到采集上来的日志。

![img](https://cdn.nlark.com/lark/0/2018/png/33393/1539177246344-16ef22ad-d226-48ce-98b1-44d1847da876.png)







## 日志进阶

阿里云日志服务针对日志提供了完整的解决方案，日志采集只是其中的第一步，以下相关功能是日志进阶的必备良药：

1. 日志上下文查询：https://help.aliyun.com/document_detail/48148.html
2. 快速查询：https://help.aliyun.com/document_detail/88985.html
3. 实时分析：https://help.aliyun.com/document_detail/53608.html
4. 快速分析：https://help.aliyun.com/document_detail/66275.html
5. 基于日志设置告警：https://help.aliyun.com/document_detail/48162.html
6. 配置大盘：https://help.aliyun.com/document_detail/69313.html
   更多日志进阶内容可以参考：[日志服务学习路径](https://help.aliyun.com/learn/learningpath/log.html)。

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。



[Kubernetes日志采集Sidecar模式介绍-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/650939)