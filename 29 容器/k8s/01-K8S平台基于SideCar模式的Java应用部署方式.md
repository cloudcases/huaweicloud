# K8S平台基于SideCar模式的Java应用部署方式

2019年11月16日 15:03 · 阅读 3624

------

## SideCar模式介绍

SideCar中文译为边车，是附着在摩托车旁的小型车辆，用于载客。在编程世界中，其主要功能是将主应用与外围辅助服务进行解耦，提供更灵活的应用部署方式。其理念符合设计模式中的单一职责原则，让主应用和辅助服务分离，更专注自身功能。

------

## K8S环境中SideCar模式几种用法

### 共享存储

基于K8S Pod特性，同一个POD可以共享根容器中挂载的Volume。基于该特性，我们可以想到以下SideCar应用方式：

1.日志收集上传

> 我们可以应用日志挂载到共享的Volume上，业务容器写日志，SideCar容器读日志，并上传日志分析平台，以生产者消费者方式进行解耦。

2.应用Jar包挂载

> 因为Java应用需要依赖拥有Java运行环境，因此大多使用open-jdk等镜像作为基础镜像。而这类镜像大多上百M。
>
> 通过共享存储，我们可以利用busybox这类体积只有几M的镜像作为基础镜像，然后将jar包拷贝到共享Volume下。并将这个承载jar的镜像作为InitContainer，主业务容器使用该共享Volume下的jar包启动业务。后续应用版本更新，只需要更新jar包镜像。这个jar包镜像便是一个SideCar。

### 共享网络

K8S中同一个POD同时也共享一个IP。基于该特性，我们可以这样使用SideCar模式：

1.容器代理

> 通过SideCar容器代理应用容器。

2.容器适配

> 当该容器需要提供给多个已有业务访问，但不同业务数据交互格式不一致时，可以借鉴适配器模式，把SideCar容器作为一个适配器，在不修改原有业务代码的同时对外提供服务。

------

## 基于SideCar模式的Java应用部署示例

将InitContainer作为我们的SideCar容器，通过共享存储方式将jar包挂载到主业务容器，主页容器提供java运行环境。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-depl
  labels:
    app: demo
spec:
  selector:
    matchLabels:
      app: demo
  replicas: 1
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: business-container
        image: java:8-jdk
        imagePullPolicy: IfNotPresent
        command:
        - java
        - -Djava.security.egd=file:/dev/./urandom
        - -Dspring.profiles.active=k8s
        - -jar
        - /temp/demo-1.0.0-SNAPSHOT.jar
        ports:
        - containerPort: 8062
        volumeMounts:
        - mountPath: /temp
          name: jar-volume
        - name: TZ
          value: Asia/Shanghai
      initContainers:
      - image: registry.demo.com/demo:1.0.0
        imagePullPolicy: IfNotPresent
        name: demo-jar
        command:
        - cp
        - /demo-1.0.0-SNAPSHOT.jar
        - /app/demo-1.0.0-SNAPSHOT.jar
        volumeMounts:
        - mountPath: /app
          name: jar-volume
      volumes:
      - emptyDir: {}
        name: jar-volume
复制代码
```

------

## 参考资料

1.CNCF X 阿里巴巴云原生技术公开课 [gitbook.cn/gitchat/col…](https://link.juejin.cn/?target=https%3A%2F%2Fgitbook.cn%2Fgitchat%2Fcolumn%2F5d68b823de93ed72d6eca1bc%2Ftopic%2F5d6dcee2de93ed72d6ed64ae)



[K8S平台基于SideCar模式的Java应用部署方式 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903998382669832)