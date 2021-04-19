# 为什么IaC是个好主意？

[![img](https://upload.jianshu.io/users/upload_avatars/44480/1352c0bc-467e-490b-b9a9-e76c2ac2346f.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/fb255ad4b067)

[wu_sphinx](https://www.jianshu.com/u/fb255ad4b067)关注

2020.11.29 19:55:37字数 1,165阅读 62

# IaC是什么？

IaC即Infrastructure as Code的简称，通常提到基础设施，会想到虚拟机、服务器、数据库、防火墙等词汇，在k8s一统江湖、云原生大行其道的当下，基础设施这一层已悄然发生的变化，在我的理解里k8s已隐隐成为未来的基础设施了。

- 应用
- 容器
- k8s
- 虚拟机
- 物理机

# IaC能带来什么好处？

前面提到k8s是基础设施，那基础设施始终是为应用服务的。当一个应用开发人员开发完毕以后，部署这一步在以前是由运维完成的，运维的操作步骤无外乎以下几点：

1. 拉取代码
2. 编译打包
3. 配置部署
4. 交付测试

这是以前套路，即使放到现在，也有不少人在用，就笔者的从业经验来说，不少开发人员并不知道自己的应用在生产是如何部署运行的。这在一定情况下并不是一件坏事，因为各尽其职，开发完成功能，运维负责部署，测试负责验证，每一个环节都是一个gap。可以试想以下场景：

1. 传统部署或者可以说是非容器化部署，因为生产某服务demo负载过高，需要进行水平扩容，增加实例，将实例注册到负载均衡服务器，然后使扩容生效。规范一点，需要提工单，有操作日志/记录，比较糙或者是紧急的情况下可能只是口头描述，然后线上操，这是完全有可能的，当然，如果要缩容，需要进行反向操作。
2. 如果服务部署是容器化的，容器化解决的是环境一致的问题，既然是解决环境一致的问题，那在开发环境当然也应该容器化，比较奇怪的是，因为职能的不同，不少开发并不懂容器化，服务项目Repo里面自然也不包括`Dockerfile`，当然更加不会有k8s的资源描述文件（或者是Helm的Chart包）了（这到底是不是个问题呢，因人而异，以我的角度来讲，这是个问题）。
   对于场景1，我认为问题在于变动没有追溯或不易追溯，come on, 明明git就是一个非常好的追溯工具好不好，当然，如果基础设施是k8s，定义好HPA就可以了，如下所示：



```bash
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demo
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: AverageUtilization
        averageUtilization: 70
……
```

对于场景2的问题，个人觉得是一个认知问题，容器化时代，开发人员需要知道自己的服务是如何容器化（个人认为这是容器化时代开发人员的基本修养），试想：作为开发人员的你，你定义好了`Dockerfile`以及k8s资源文件，有没有一种一切尽在掌握的感觉。

# IaC推进的障碍？

基础设施代码代码化这是一个趋势，**可描述**与**可追溯**这是一大优势，所以才会诞生“DevOps工程师”（一家之言），用于弥补开发的运维短板和运维的开发短板。但是没有让开发、运维、测试都参与进来，这其中一部分原因是由于工作职能划分的过于清晰，当然，也可能是部门墙，还有习惯的改变、新知识和概念的学习成本等，这些都是问题，以笔者浅见，解决这些问题的方法不是规避，新技术实施和推广，大家不免会有疑惑，或许让愿意接受的人来拥抱与参与会是一个好方法，用脚投票，只要是真正能带来效率提升的方法，相信没有人会拒绝的。

ps:
这是一个*后端开发*，曾在质量团队呆过，做过PaaS平台，给运维团队做过乙方后，对IaC的一点浅见，真的是浅见，但也因此爱上了IaC

参考：

- [https://insights.thoughtworks.cn/nfrastructure-as-code/](https://links.jianshu.com/go?to=https%3A%2F%2Finsights.thoughtworks.cn%2Fnfrastructure-as-code%2F)
- [https://en.wikipedia.org/wiki/Infrastructure_as_code](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FInfrastructure_as_code)
- [https://kubernetes.io/zh/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/](https://links.jianshu.com/go?to=https%3A%2F%2Fkubernetes.io%2Fzh%2Fdocs%2Ftasks%2Frun-application%2Fhorizontal-pod-autoscale-walkthrough%2F)