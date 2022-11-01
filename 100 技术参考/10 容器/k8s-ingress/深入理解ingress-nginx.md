# [深入理解ingress-nginx](https://www.cnblogs.com/zhaobin-diray/p/13473813.html)

### 8.1 Ingress为弥补NodePort不足而生

NodePort存在的不足：

- 一个端口只能一个服务使用，端口需提前规划
- 只支持4层负载均衡

### 8.2 Pod与Ingress的关系

- 通过Service相关联
- 通过Ingress Controller实现Pod的负载均衡
  - 支持TCP/UDP 4层和HTTP 7层

![img](https://k8s-1252881505.cos.ap-beijing.myqcloud.com/k8s-1/pod-ingress.png)

 

### 8.3 Ingress Controller

为了使Ingress资源正常工作，集群必须运行一个Ingress Controller（负载均衡实现）。

所以要想通过ingress暴露你的应用，大致分为两步：

1. 部署Ingress Controller
2. 创建Ingress规则

整体流程如下：

![img](https://k8s-1252881505.cos.ap-beijing.myqcloud.com/k8s-1/ingress-controller.png)

Ingress Controller有很多实现，我们这里采用官方维护的Nginx控制器。

部署文档：[https](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)[://](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)[github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

 

**注意事项：**

- 镜像地址修改成国内的：lizhenliang/nginx-ingress-controller:0.20.0
- 使用宿主机网络：hostNetwork: true

```
# kubectl apply -f ingress-controller.yaml``# kubectl get pods -n ingress-nginx``NAME               READY  STATUS  RESTARTS  AGE``nginx-ingress-controller-5r4wg  1``/1`   `Running  0     13s``nginx-ingress-controller-x7xdf  1``/1`   `Running  0     13s
```

　此时在任意Node上就可以看到该控制监听的80和443端口：

```
# netstat -natp |egrep ":80|:443"``tcp    0   0 0.0.0.0:80       0.0.0.0:*        LISTEN   104750``/nginx``: maste``tcp    0   0 0.0.0.0:443       0.0.0.0:*        LISTEN   104750``/nginx``: maste
```

　　

80和443端口就是接收来自外部访问集群中应用流量，转发对应的Pod上。

其他主流控制器：

Traefik： HTTP反向代理、负载均衡工具

Istio：服务治理，控制入口流量

### 8.4 Ingress 规则

接下来，就可以创建ingress规则了。

在ingress里有三个必要字段：

- host：访问该应用的域名，也就是域名解析
- serverName：应用的service名称
- serverPort：service端口

　**1、HTTP访问**

```
apiVersion: networking.k8s.io``/v1beta1``kind: Ingress``metadata:`` ``name: example-ingress``spec:`` ``rules:`` ``- host: example.ctnrs.com``  ``http:``   ``paths:``   ``- path: /``    ``backend:``     ``serviceName: web``     ``servicePort: 80
```

生产环境：example.ctnrs.com 域名是在你购买域名的运营商上进行解析，A记录值为K8S Node的公网IP（该Node必须运行了Ingress controller）。

测试环境：可以绑定hosts模拟域名解析（"C:\Windows\System32\drivers\etc\hosts"），对应IP是K8S Node的内网IP。例如：

192.168.31.62 example.ctnrs.com

　　

 

 **2、HTTPS访问**

四。创建https方式访问网站

1.创建cfssl.sh，并执行此脚本，默认加载到bin目录变成可执行文件

```
wget https:``//pkg``.cfssl.org``/R1``.2``/cfssl_linux-amd64``wget https:``//pkg``.cfssl.org``/R1``.2``/cfssljson_linux-amd64``wget https:``//pkg``.cfssl.org``/R1``.2``/cfssl-certinfo_linux-amd64``chmod` `+x cfssl*``mv` `cfssl_linux-amd64 ``/usr/bin/cfssl``mv` `cfssljson_linux-amd64 ``/usr/bin/cfssljson``mv` `cfssl-certinfo_linux-amd64 ``/usr/bin/cfssl-certinfo
```

2.创建certs.sh

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) 脚本

3.创建ingress.yaml 证书里的域名和ingress里的域名要一样

```
apiVersion: networking.k8s.io``/v1beta1``kind: Ingress``metadata:`` ``name: tls-example-ingress``spec:`` ``tls:`` ``- hosts:``  ``- blog.ctnrs.com``  ``secretName: example-ctnrs-com`` ``rules:``  ``- host: blog.ctnrs.com``   ``http:``    ``paths:``    ``- path: /``     ``backend:``      ``serviceName: web``      ``servicePort: 80
```

4.直接网页访问即可

 

**3、根据URL路由到多个服务**

```
apiVersion: networking.k8s.io``/v1beta1``kind: Ingress``metadata:`` ``name: url-ingress`` ``annotations:``  ``nginx.ingress.kubernetes.io``/rewrite-target``: /``spec:`` ``rules:`` ``- host: foobar.ctnrs.com``  ``http:``   ``paths:``   ``- path: ``/foo``    ``backend:``     ``serviceName: service1``     ``servicePort: 80`` ``- host: foobar.ctnrs.com``  ``http:``   ``paths:``   ``- path: ``/bar``    ``backend:``     ``serviceName: service2``     ``servicePort: 80
```

　工作流程：

```
foobar.ctnrs.com -> 178.91.123.132 -> / foo  service1:80``                   ``/ bar  service2:80
```

　**4、基于名称的虚拟主机**

```
apiVersion: networking.k8s.io``/v1beta1``kind: Ingress``metadata:`` ``name: name-virtual-host-ingress``spec:`` ``rules:`` ``- host: foo.ctnrs.com``  ``http:``   ``paths:``   ``- backend:``     ``serviceName: service1``     ``servicePort: 80`` ``- host: bar.ctnrs.com``  ``http:``   ``paths:``   ``- backend:``     ``serviceName: service2``     ``servicePort: 80
```

　工作流程:

```
foo.bar.com --|         |-> service1:80``       ``| 178.91.123.132 |``bar.foo.com --|         |-> service2:80
```

### 8.5 Annotations对Ingress个性化配置

参考文档 ：https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md

**HTTP：配置Nginx常用参数**

```
apiVersion: networking.k8s.io``/v1beta1``kind: Ingress``metadata:`` ``name: example-ingress`` ``annotations:``   ``kubernetes.io``/ingress``.class: "nginx“``   ``nginx.ingress.kubernetes.io``/proxy-connect-timeout``: ``"600"``   ``nginx.ingress.kubernetes.io``/proxy-send-timeout``: ``"600"``   ``nginx.ingress.kubernetes.io``/proxy-read-timeout``: ``"600"``   ``nginx.ingress.kubernetes.io``/proxy-body-size``: ``"10m"``spec:`` ``rules:`` ``- host: example.ctnrs.com``  ``http:``   ``paths:``   ``- path: /``    ``backend:``     ``serviceName: web``     ``servicePort: 80
```

　**HTTPS：禁止访问HTTP强制跳转到HTTPS（默认开启）**

```
apiVersion: networking.k8s.io``/v1beta1``kind: Ingress``metadata:`` ``name: tls-example-ingress`` ``annotations:``  ``kubernetes.io``/ingress``.class: "nginx“``  ``nginx.ingress.kubernetes.io``/ssl-redirect``: ``'false'``spec:`` ``tls:`` ``- hosts:``  ``- sslexample.ctnrs.com``  ``secretName: secret-tls`` ``rules:``  ``- host: sslexample.ctnrs.com``   ``http:``    ``paths:``    ``- path: /``     ``backend:``      ``serviceName: web``      ``servicePort: 80
```

　

### 8.6 Ingress Controller高可用方案

 

如果域名只解析到一台Ingress controller，是存在单点的，挂了就不能提供服务了。这就需要具备高可用，有两种常见方案：

![img](https://k8s-1252881505.cos.ap-beijing.myqcloud.com/k8s-1/ingress-controller-ha.png)

**左边：双机热备**，选择两台Node专门跑Ingress controller，然后通过keepalived对其做主备。用户通过VIP访问。

**右边：高可用集群（推荐）**，前面加一个负载均衡器，转发请求到后端多台Ingress controller。