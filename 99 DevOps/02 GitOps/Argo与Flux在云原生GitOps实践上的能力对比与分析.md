# Argo与Flux在云原生GitOps实践上的能力对比与分析

2020-09-04 2173

**简介：** 随着云原生技术的普及和落地，越来越多的云原生应用被部署到生产环境中，由于云原生应用通常都是基于云的分布式部署模式，且每个应用可能是由多个功能组件互相调用来一起提供完整的服务的，每个组件都有自己独立的迭代流程和计划。在这种情况下，功能组件越多，意味着应用的发布管理越复杂，如果没有一个好的方案或者系统来管理复杂应用的发布上线的话，业务面临的风险也是非常大的。开源社区在复杂应用发布管理方面逐渐开始发力，

随着云原生技术的普及和落地，越来越多的云原生应用被部署到生产环境中，由于云原生应用通常都是基于云的分布式部署模式，且每个应用可能是由多个功能组件互相调用来一起提供完整的服务的，每个组件都有自己独立的迭代流程和计划。在这种情况下，功能组件越多，意味着应用的发布管理越复杂，如果没有一个好的方案或者系统来管理复杂应用的发布上线的话，业务面临的风险也是非常大的。开源社区在复杂应用发布管理方面逐渐开始发力，本文对其中2种针对上层应用发布管理的方案进行对比和分析，它们就是Intuit的[ArgoCD](https://github.com/argoproj/argo-cd)和[ArgoRollouts](https://github.com/argoproj/argo-rollouts)结合的方案以及Weaveworks的[Flux](https://github.com/fluxcd/flux)和[Flagger](https://github.com/weaveworks/flagger)结合的方案。

ArgoCD和Flux（或者Flux CD）的主要职责都是监听Git Repository源中的应用编排变化，并与当前环境中应用运行状态进行对比，自动化同步拉取应用变更并部署到进群中。

ArgoRollouts和Flagger的主要职责都是执行更复杂的应用发布策略，比如蓝绿发布、金丝雀发布、AB Testing等。

# 1. ArgoCD与Flux CD

ArgoCD与Flux CD的主要职责是监听Git Repositories变化，对比当前应用运行状态与期望运行状态的差异，然后自动拉取变更并同步部署到集群环境中。但架构设计与功能支持上有很多差异点。本文从以下几个方面对其进行对比分析。

## 1.1 ArgoCD

### 1.1.1 架构

ArgoCD包括3个主要组件：

###### API Server

ArgoCD API server是一个gRPC/REST风格的server，提供API给Web UI，CLI以及其他CI/CD做系统调用或集成，包括以下职责：

- 应用管理和状态上报
- 执行应用变更，如同步、回滚等
- Git Repository和集群凭证的管理（存储为k8s secret）
- 对外部身份提供者进行身份验证（添加外部集群）
- RBAC增强
- 监听和转发Git webhook事件

###### Repository Server

repossitory server是一个内部服务，维护一个Git Repo中应用编排文件的本地缓存。支持以下参数设置：

- repository URL
- revision (commit, tag, branch)
- application path，Git Repo中的subpath
- 模板化的参数设置，如parameters, ksonnet environments, helm values.yaml

#### Application Controller

Application Controller是一个Kubernetes Controller，主要工作是持续监听应用当前运行状态与期望状态（Git Repo中描述的状态）的不同。自动检测应用OutOfSync状态并根据Sync测试执行下一步动作。

![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/cea81e5429a3c93895587404e8eb8102.png)

### 1.1.2 Web UI

Argo UI之前是Argo组织下一个独立的项目，现在已经合并到Argo CD项目里。Argo CD的UI基本已经实现了Argcd CLI的大部分功能，用户可以通过UI做以下事情：

- 配置和连接远端Git Repositories
- 配置用于连接私有Git Repositories的凭证
- 添加管理不同的k8s集群
- 配置project
  ![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/105b4482a5b5d10385a8a44b23ea239c.png)
- 创建应用
- 动态展示应用当前运行状态
- 应用发布
- 应用历史版本回滚
  ![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/fdd7ba5f98d06bb7e9abb9000c8bddd7.png)

### 1.1.3 多集群管理

ArgoCD支持管理多集群。 ArgoCD可以添加管理多个集群，它以secret的方式存储外部集群的凭证，secret中包括一个与被托管集群kube-system命名空间下名为argocd-manager的ServiceAccount相关联的k8s API bearer token，和连接被托管集群API Server方式信息。支持revoke。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/bb0668a3f9dfdca67f3f61351010f1f5.png)

### 1.1.4 应用管理能力

ArgoCD下有明确的应用的概念，基本上覆盖了一个应用生命周期内所有需要的操作；此外，ArgoCD支持引用单一Git Repository在不同集群中创建不同应用，实际上我们还可以利用这个能力配合应用的定制化能力，实现在不同集群中创建相同应用的场景。
应用生命周期管理：

```
$ argocd app -h
Available Commands:
  actions        Manage Resource actions
  create         Create an application
  delete         Delete an application
  diff           Perform a diff against the target and live state.
  edit           Edit application
  get            Get application details
  history        Show application deployment history
  list           List applications
  manifests      Print manifests of an application
  patch          Patch application
  patch-resource Patch resource in an application
  rollback       Rollback application to a previous deployed version by History ID
  set            Set application parameters
  sync           Sync an application to its target state
  terminate-op   Terminate running operation of an application
  unset          Unset application parameters
  wait           Wait for an application to reach a synced and healthy state
```

引用单一Git Repository在不同集群中创建应用，下面是一个引用`https://github.com/haoshuwei/argocd-samples.git`在ack-pre、ack-pro和gke-pro 3个k8s集群中创建应用的示例：

```
$ argocd cluster list
SERVER                          NAME     VERSION  STATUS      MESSAGE
https://xxx.xx.xxx.xxx:6443     ack-pro  1.14+    Successful
https://xx.xx.xxx.xxx           gke-pro  1.14+    Successful
https://xxx.xx.xxx.xxx:6443     ack-pre  1.14+    Successful
https://kubernetes.default.svc           1.14+    Successful
```

创建应用`ack-pre`，部署`argocd-samples`项目下`overlays/pre`子目录里的编排文件，分支为`latest`，应用部署到api server为`https://xx.xx.xxx.xxx:6443`的集群，命名空间为`argocd-samples`， 同步策略为`automated`

```
$ argocd app create --project default --name ack-pre --repo https://github.com/haoshuwei/argocd-samples.git --path overlays/pre --dest-server https://xx.xx.xxx.xxx:6443 --dest-namespace  argocd-samples --revision latest --sync-policy automated
```

创建应用`ack-pro`，部署`argocd-samples`项目下`overlays/pro`子目录里的编排文件，分支为`master`，应用部署到api server为`https://xx.xx.xxx.xxx:6443`的集群，命名空间为`argocd-samples`， 同步策略为`automated`

```
$ argocd app create --project default --name ack-pro --repo https://github.com/haoshuwei/argocd-samples.git --path overlays/pro --dest-server https://xx.xx.xxx.xxx:6443 --dest-namespace  argocd-samples --revision master --sync-policy automated
```

创建应用`gke-pro`，部署`argocd-samples`项目下`overlays/gke`子目录里的编排文件，分支为`master`，应用部署到api server为`https://xx.xx.xxx.xxx`的集群，命名空间为`argocd-samples`， 同步策略为`automated`

```
$ argocd app create --project default --name gke-pro --repo https://github.com/haoshuwei/argocd-samples.git --path overlays/gke --dest-server https://xx.xx.xxx.xxx --dest-namespace  argocd-samples --revision master
```

![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/a01a04b87a63f2dd67543e40237d4516.png)

### 1.1.5 kubernetes应用编排工具支持

ArgoCD支持Kustomize应用、Helm Charts、Ksonnet应用、Jsonnet文件，并且可以通过管理和配置插件的方式支持其他你想要使用的编排工具。

### 1.1.6 安全

ArgoCD只设置了一个内置的admin用户，只能登录用户才可以进行下一步操作。ArgoCD认为一个内置的admin用户主要用于管理和配置比App资源更底层的集群资源，App资源的变更历史记录都需要在Git Provider端进行审计。尽管如此，ArgoCD也支持了其他用户使用SSO的方式进行登录，比如通过OAuth2接入阿里云RAM服务或GitHub：
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/8035e80ef1b7c1901a05eb19f7d9f4dd.png)
另外，ArgoCD在App资源的上层又抽象出来一个Project的概念，Project是应用管理的一个逻辑上的组，每一个应用都要属于且只能属于一个组，针对多团队合作的情景而设计的一个概念。它的主要作用包括：限制指定的Git Repository才可以被部署；限制应用只能被部署到指定clusters的指定namespace下；限制指定的资源类型可以被部署或不能被部署；定义project roles来提供对应用的角色访问控制RBAC。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/c4d80add57633a32589dc08671d58f56.png)

### 1.1.7 应用的自动化同步能力

ArgoCD只监听Git Repository源中的代码变更，不关心容器镜像仓库中镜像tag的变换。应用的同步策略有两种：`automated`和`none`

### 1.1.8 Git Repositories支持

支持ssh、git、http/https协议；
支持自签发证书添加、ssh known hosts添加
私有仓库的设置支持private token和ssh private key

## 1.2 Flux CD

### 1.2.1 架构

Flux是Weaveworks公司在2016年发起的开源项目，2019年8月成为CNCF基金会的一个孵化项目。

###### Fluxd

Flux daemon， 主要职责是持续监听用户配置的Git Repo中Kubernetes资源编排文件的变化，然后同步部署到集群中； 它还可以检测容器镜像仓库中image更新，提交更新到用户Git Repo，然后同步部署到集群中。 以上同步部署动作均基于用户设置的策略。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/695af07a37a654e479ddeecac863013a.png)

### 1.2.2 Web UI

Flux CD目前不提供UI。

### 1.2.3 多集群管理

Flux CD不支持多集群管理，需要在每个集群中部署Flux CD组件。

### 1.2.4 应用管理能力

Flux CD 下需要配置flux连接你的目标Git Repository并将应用同步部署到k8s集群中。 如果你想使用单一Git Repository部署应用到不同集群，则需要在每个集群中部署flux并为每个集群设置唯一的git tag。

```
$ fluxctl -h
Available Commands:
  automate       Turn on automatic deployment for a workload.
  deautomate     Turn off automatic deployment for a workload.
  help           Help about any command
  identity       Display SSH public key
  install        Print and tweak Kubernetes manifests needed to install Flux in a Cluster
  list-images    Show deployed and available images.
  list-workloads List workloads currently running in the cluster.
  lock           Lock a workload, so it cannot be deployed.
  policy         Manage policies for a workload.
  release        Release a new version of a workload.
  save           save workload definitions to local files in cluster-native format
  sync           synchronize the cluster with the git repository, now
  unlock         Unlock a workload, so it can be deployed.
  version        Output the version of fluxctl
```

### 1.2.5 kubernetes应用编排工具支持

Flux CD支持配置使用Helm Operator部署Helm Charts，支持Kustomize应用。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/865cbe3dead6f0f452d689b33f25e60a.png)

### 1.2.6 安全

Flux CD支持管理和使用户k8s集群多租特性，分为集群管理员和普通开发测试团队两种角色，团队成员只能修改自己所以命名空间下的应用，对集群级别的应用或者其他命名空间下的应用无任何操作权限。此外，对于每个团队或者命名空间来说，只能指定的一个Git Repository，对应地需要启动一个Flux实例。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/dc2d56602a3bc50fa3308c2713a8c7b9.png)

### 1.2.7 应用的自动化同步能力

FluxCD 除了监听Git Repository源中的代码变更之外，还可以监听docker registry中与当前运行应用相同的镜像的tag变化，可以根据不同策略决定是否把镜像tag变化的信息自动commit到Git Repository源中，然后再同步部署到集群中。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/e77cf06fae9980a5a308fd7be4634f0f.png)

### 1.2.8 Git Repositories支持

私有仓库的设置只支持ssh private key

# 2.ArgoRollouts与Flagger

ArgoRollouts与Flagger的主要职责都是执行更复杂的应用发布策略，比如蓝绿发布、金丝雀发布、AB Testing等。目前ArgoRollouts部署并不依赖istio环境，Flagger则必须在istio环境下才能正常工作。

## 2.1 ArgoRollouts

### 2.1.1 Istio、Service Mesh或App Mesh的支持

在流量管理方面，ArgoRollouts只支持istio和ingress，目前还不支持Service Mesh或App Mesh，不过社区下一阶段的主要工作就是做这部分的支持。
一个ArgoRollouts结合istio对流量进行管理的编排示例：

```
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollout-example
spec:
  ...
  strategy:
    canary:
      steps:
      - setWeight: 5
      - pause:
          duration: 600
      canaryService: canary-svc # required
      stableService: stable-svc  # required
      trafficRouting:
        istio:
           virtualService: 
            name: rollout-vsvc  # required
            routes:
            - primary # At least one route is required
```

其中名为`rollout-vsvc`的istio自定义资源`VirtualService`的编排为：

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: rollout-vsvc
spec:
  gateways:
    - istio-rollout-gateway
  hosts:
    - istio-rollout.dev.argoproj.io
  http:
    - name: primary
      route:
        - destination:
            host: stable-svc
          weight: 100
        - destination:
            host: canary-svc
          weight: 0
```

在发布应用的时候，ArgoRollouts会根据`spec.strategy.steps`的设置来动态修改`rollout-vsvc`中`stable-svc`和`canary-svc`的权重比例，直到应用发布完毕。

### 2.1.2 全自动渐进式应用发布支持

全自动渐进式应用发布是指能使运行在k8s体系上的应用发布流程全自动化(无人参与), 它能减少发布的人为关注时间, 并且在发布过程中能自动识别一些风险(例如:RT,成功率,自定义metrics)并执行回滚操作。
ArgoRollouts使用`AnalysisTemplate` `AnalysisRun` 和`Experiment`3种crd资源来分析从Prometheus查询到的监控指标，然后进行决策决定应用是否继续进一步发布。
示例：

```
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: guestbook
spec:
...
  strategy:
    canary: 
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 2 # delay starting analysis run
                        # until setWeight: 40%
        args:
        - name: service-name
          value: guestbook-svc.default.svc.cluster.local
      steps:
      - setWeight: 20
      - pause: {duration: 600}
      - setWeight: 40
      - pause: {duration: 600}
      - setWeight: 60
      - pause: {duration: 600}
      - setWeight: 80
      - pause: {duration: 600}
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: service-name
  metrics:
  - name: success-rate
    interval: 5m
    successCondition: result >= 0.95
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.example.com:9090
        query: |
          sum(irate(
            istio_requests_total{reporter="source",destination_service=~"{{args.service-name}}",response_code!~"5.*"}[5m]
          )) / 
          sum(irate(
            istio_requests_total{reporter="source",destination_service=~"{{args.service-name}}"}[5m]
          ))
```

在上面的例子中，应用的发布策略为每`600s`增加20%的路由权重到新版本应用上，这个动作有`AnalysisRun`执行，依赖于`AnalysisTemplate `中定义的`success-rate`，`success-rate`是根据从Prometheus系统中查询到的数据进行计算得出的，如果某一次`success-rate`小于95%，则ArgoRollouts会进行回滚操作，否则逐渐增加权重到100%完成应用发布。

## 2.2 Flagger

### 2.2.1 Istio、Service Mesh或App Mesh的支持

Flagger需要结合Istio, Linkerd, App Mesh, NGINX, Contour or Gloo等的流量管理能力以及Prometheus的指标收集与分析来完成应用的全自动化、渐进式的金丝雀发布。
Flagger通过spec.provider来指定使用哪种流量管理方案来发布应用：

```
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: podinfo
  namespace: test
spec:
  # service mesh provider (optional)
  # can be: kubernetes, istio, linkerd, appmesh, nginx, contour, gloo, supergloo
  provider: istio
  # deployment reference
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: podinfo
```

Flagger已经与GKE Istio以及EKS App Mesh做了很好的集成并开放给用户使用：
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/c6456deaaff48084713e13604062f8db.png)
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/9878d13d54181df47b1c09e387ff3199.png)

### 2.2.2 全自动渐进式应用发布支持

Flagger的canary analysis资源中定义了应用发布策略、用于验证新版本的指标、webhook扩展测试验证能力和告警设置：

```
analysis:
    # schedule interval (default 60s)
    interval:
    # max number of failed metric checks before rollback
    threshold:
    # max traffic percentage routed to canary
    # percentage (0-100)
    maxWeight:
    # canary increment step
    # percentage (0-100)
    stepWeight:
    # total number of iterations
    # used for A/B Testing and Blue/Green
    iterations:
    # canary match conditions
    # used for A/B Testing
    match:
      - # HTTP header
    # key performance indicators
    metrics:
      - # metric check
    # alerting
    alerts:
      - # alert provider
    # external checks
    webhooks:
      - # hook
```

一个示例如下：

```
analysis:
    # schedule interval (default 60s)
    interval: 1m
    # max number of failed metric checks before rollback
    threshold: 10
    # max traffic percentage routed to canary
    # percentage (0-100)
    maxWeight: 50
    # canary increment step
    # percentage (0-100)
    stepWeight: 5
    # validation (optional)
    metrics:
    - name: request-success-rate
      # builtin Prometheus check
      # minimum req success rate (non 5xx responses)
      # percentage (0-100)
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      # builtin Prometheus check
      # maximum req duration P99
      # milliseconds
      thresholdRange:
        max: 500
      interval: 30s
    - name: "database connections"
      # custom Prometheus check
      templateRef:
        name: db-connections
      thresholdRange:
        min: 2
        max: 100
      interval: 1m
    # testing (optional)
    webhooks:
      - name: "conformance test"
        type: pre-rollout
        url: http://flagger-helmtester.test/
        timeout: 5m
        metadata:
          type: "helmv3"
          cmd: "test run podinfo -n test"
      - name: "load test"
        type: rollout
        url: http://flagger-loadtester.test/
        metadata:
          cmd: "hey -z 1m -q 10 -c 2 http://podinfo.test:9898/"
    # alerting (optional)
    alerts:
      - name: "dev team Slack"
        severity: error
        providerRef:
          name: dev-slack
          namespace: flagger
      - name: "qa team Discord"
        severity: warn
        providerRef:
          name: qa-discord
      - name: "on-call MS Teams"
        severity: info
        providerRef:
          name: on-call-msteams
```

对以上示例进行分解分析，

```
analysis:
    # schedule interval (default 60s)
    interval: 1m
    # max number of failed metric checks before rollback
    threshold: 10
    # max traffic percentage routed to canary
    # percentage (0-100)
    maxWeight: 50
    # canary increment step
    # percentage (0-100)
    stepWeight: 5
```

以上编排字段设置定义了应用发布过程中，流量最大只能切换到50%（maxWeight：50），总共执行10次（maxWeight/stepWeight），每次流量切换的增量为5%（stepWeight：5），每次执行间隔为1m(interval: 1m)，期间允许10次metrics验证失败（threshold: 10），若超过10次则进行回滚操作。

```
metrics:
    - name: request-success-rate
      # builtin Prometheus check
      # minimum req success rate (non 5xx responses)
      # percentage (0-100)
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      # builtin Prometheus check
      # maximum req duration P99
      # milliseconds
      thresholdRange:
        max: 500
      interval: 30s
    - name: "database connections"
      # custom Prometheus check
      templateRef:
        name: db-connections
      thresholdRange:
        min: 2
        max: 100
      interval: 1m
```

以上编排字段设置定义了3种metrics检查：request-success-rate（请求成功率）不能超过99%，request-duration（RT均值）不能超过500ms，然后还有一个自定义metrics检查，即database connections最小不能小于2，最大不能大于100。

```
    webhooks:
      - name: "conformance test"
        type: pre-rollout
        url: http://flagger-helmtester.test/
        timeout: 5m
        metadata:
          type: "helmv3"
          cmd: "test run podinfo -n test"
      - name: "load test"
        type: rollout
        url: http://flagger-loadtester.test/
        metadata:
          cmd: "hey -z 1m -q 10 -c 2 http://podinfo.test:9898/"
```

以上编排字段设置定义了2种类型的webhook：名为"conformance test"类型为pre-rollout的webhooks需要在流量权重增加之前执行，名为"load test"类型为rollout的webhooks在metrics检查执行期间运行。

```
 alerts:
      - name: "dev team Slack"
        severity: error
        providerRef:
          name: dev-slack
          namespace: flagger
      - name: "qa team Discord"
        severity: warn
        providerRef:
          name: qa-discord
      - name: "on-call MS Teams"
        severity: info
        providerRef:
          name: on-call-msteams
```

以上编排字段设置定义了发布过程中的通知策略，severity表示通知信息的等级，如error、warn、info等，providerRef引用了不同AlterProvider的详细信息，如slack，msteams、dingding等。

# 3. GitOps Engine

2019年11月，Weaveworks宣布和Intuit合作共建Argo Flux，主要针对Kubernetes应用发布的GitOps解决方案，AWS作为活跃开发者也加入其中，AWS的BlackRock会是第一个使用此方案的企业级应用服务。这个项目就是[GitOps Engine ](https://github.com/argoproj/gitops-engine)。Argo组织正在以一个整体加入CNCF开源基金会，Flux和Flagger已经是CNCF基金会的孵化项目，从基金会的角度是希望他们能寻找一些可以合作的方式的，另外两者在解决方案上确实有很多相似的地方，两家公司也希望能集合2个社区的开源力量做出一个更优秀的GitOps解决方案来。不过从目前来看，GitOps Engine还没有特别大的动作。
![image.png](https://ata2-img.cn-hangzhou.oss-pub.aliyun-inc.com/80965a33b74e3e3ad4e0aa7ca6042d33.png)

GitOps Engine第一步动作是整合ArgoCD和Flux当前已具备的核心能力，未来社区还会考虑ArgoRollouts与Flagger的结合。 我们对齐保持持续关注。

持续更新，如有偏差，欢迎指正。

# 参考资料

https://argoproj.github.io/argo-cd/
https://argoproj.github.io/argo-rollouts/
[https://github.com/fluxcd/flux]()https://github.com/fluxcd/flux
https://docs.fluxcd.io/en/1.18.0/introduction.html
https://github.com/weaveworks/flagger
https://www.weave.works/blog/flux-joins-the-cncf-sandbox
https://docs.flagger.app/
https://www.weave.works/blog/argo-flux-join-forces

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。

[Prometheus](https://developer.aliyun.com/label/sc/article_de-3-100271) [Kubernetes](https://developer.aliyun.com/label/sc/article_de-3-100209) [Cloud Native](https://developer.aliyun.com/label/sc/article_de-3-100024) [应用服务中间件](https://developer.aliyun.com/label/sc/article_de-3-100033) [API](https://developer.aliyun.com/label/sc/article_de-3-100252) [网络安全](https://developer.aliyun.com/label/sc/article_de-3-100098) [开发工具](https://developer.aliyun.com/label/sc/article_de-3-100016) [nginx](https://developer.aliyun.com/label/sc/article_de-3-100224) [git](https://developer.aliyun.com/label/sc/article_de-3-100234) [容器](https://developer.aliyun.com/label/sc/article_de-3-100018)

[kubernetes分钟](https://www.aliyun.com/sswc/12627.html) [kubernetes开启](https://www.aliyun.com/sswc/232924.html) [kubernetes指定](https://www.aliyun.com/sswc/212380.html) [kubernetes不同](https://www.aliyun.com/sswc/202892.html) [kubernetes解决](https://www.aliyun.com/sswc/412371.html)

[开发者社区](https://developer.aliyun.com/) > [开发与运维](https://developer.aliyun.com/group/othertech/) > [文章](https://developer.aliyun.com/group/othertech/article/)



[Argo与Flux在云原生GitOps实践上的能力对比与分析-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/771574?spm=a2c6h.13262185.0.0.150a24e2LyOzRy)