# Nginx-Ingress详解

2021-12-30阅读 8460

## 概述

k8s 中所有的资源都有对应的控制器在操控这个资源，管理资源的生命周期，实现”声明式“效果。Deployment、Service、Replicaset等资源的控制器封装在k8s内置的 controller-manager进程中。

对于外部请求如何进入集群内部，K8s 官方定义了 Ingress 这个资源，但是官方并没有提供 Ingress 的控制器，使用时必须手动安装一个 [Ingress controller](https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers/)。云厂商有自己的 ingress controller，开源界也有一下 ingress controller ：

- nginx ingress：基于 nginx
- Traefik：基于 Traefik
- contour：基于 envoy， 参考笔者写的 [envoy-ingress contour源码分析](https://juejin.cn/post/7029286464802258974)
- Ambassador：基于 envoy
- Apache APISIX ingress：基于 apisix
- BFE ingress：基于 BFE
- Gloo: 基于 envoy
- ...

所有的 Ingress 都包括数据面和控制面，控制面负责监听 k8s 资源，生成配置，下发给数据面。

Nginx Ingress 作为使用广泛的 Ingress，底层基于 Nginx，动态生成 nginx.conf 文件，实现将请求重定向到pod内部的目的。

nginx 的优点在于周边生态丰富，和传统[运维](https://cloud.tencent.com/solution/operation?from=10680)无缝集成，不需要更多的学习成本。但是缺点在于热加载的支持不友好，需要重启服务。云原生时代 envoy 逐渐占据风头，出现大量基于 envoy 的 Ingress。

## 安装部署

有两种方式安装 nginx-ingress：helm 和 kubectl apply

### helm 安装

```js
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

复制

### kubectl apply

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/cloud/deploy.yaml
```

复制

### 验证安装是否成功

```js
# 查看 pod 状态
kubectl get pods -n ingress-nginx
# 或者监听 pod 启动
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

复制

## 如何工作的

### 配置

- 动态生成 nginx 需要的配置文件 nginx.conf
- 默认nginx配置修改需要重启才生效。而 nginx-ingress 如果只是更新了 upstream 信息，是不需要重启服务的。这是通过 lua-nginx-model 来实现的

### nginx 模型

- 通过 k8s infomrer 机制监听 k8s 资源：比如 Ingress、Service、Endpoint、Secret、Configmap，以生成配置文件
- 没有办法知道哪次修改会影响最终配置，因此每次变更都会从零重建模型，并和当前模型比较。
  - 如果模型不变，就不会重新生成nginx.conf 配置文件，避免重启。
  - 如果只是 Endpoints 变化，用post请求发送一个 endpoint 列表给运行在 nginx里面的 lua handler。也可以避免重新生成配置文件和服务重启。
  - 如果以上情况都不是，会创建新的模型的配置文件，并触发nginx 重启
- 使用时，应该避免不必要的配置变更和冲突定义，以减少服务重启
- 配置文件是通过 go template 渲染出来的

### 构建 nginx 模型

### 何时 reload 是必须的

- 新的 Ingress 资源创建
- 添加 TLS 到现有的 Ingress 中
- 修改非 upstream_configuration 相关的 annotation（load-balance 注解不会触发 reload）
- Ingress 中的 Path 添加或者修改
- Ingress 或者 Service 资源被移除
- Ingress 中配置的 Service 或 Secret 资源变成可用状态
- Secret 更新 

### 有哪些技术以避免 reload

#### endpoint 修改可以避免 reload

使用 nginx lua 模块，只更新 endpoint 不会reload，在大集群、大量的服务部署更新的情况下可以减少延迟

#### 配置错误可以避免服务中断

- nginx.ingress.kubernetes.io/configuration-snippet 注解中的语法错误，会导致生成的配置文件无效服务中断
- 为避免以上问题，nginx ignress 选择性暴露了一个 admission webhook server 用于确保ingress 的合法性 

## 排错

### 日志和事件

- 检查 Ingress 资源事件

```js

```

复制

  kubectl get ing -n <ns>

  kubectl describe ing <name> -n <ns>

```js

```

复制

- 检查日志

```js

```

复制

  kubectl get pods -n <ns>

  kubectl logs -n <ns> <name>

```js

```

复制

- 检查nginx配置文件

```js

```

复制

  kubectl get pods -n <ns>

  kubectl exec -it -n <ns> <name> -- cat /etc/nginx/nginx.conf

```js

```

复制

- 检查 svc 是否存在

```js

```

复制

  kubectl get svc -A

```js

```

复制

### Debug 日志

使用 --v=XX 参数可以增加日志等级，修改 deployment 文件

```js
kubectl get deploy -n <ns>
kubectl edit deploy -n <ns> <name>
# 增加 --v=X 到 - args
```

复制

- --v=2 显示配置文件差异的细节
- --v=3 显示 svc、ingress、endpoint细节，并且打印出 json 格式的 nginx 配置文件
- --v=5 以 Debug 模型运行 nginx

## Nginx 配置

有三种方式可以自定义 nginx 的配置：

- ConfigMap：使用 configmap 修改全局配置
- Annotations： 针对特定的 Ingress 规则做特定的配置
- 自定义模板：当有多个特殊的配置需要时使用自定义模板，比如修改 open_file_cache、调整 Listen 参数

### 基本配置

- k8s < 1.19 版本
- 支持 配置 ingressClassName
- 支持配置 kubernetes.io/ingress.class: "nginx" 注解

```js

```

复制

  apiVersion: networking.k8s.io/v1

  kind: Ingress

  metadata:

```js
name: ingress-myservicea
```

复制

  spec:

```js
ingressClassName: nginx
```

复制

```js
rules:
```

复制

```js
- host: myservicea.foo.org
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: myservicea
          port:
            number: 80undefined  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: ingress-myserviceb
    annotations:  # use the shared ingress-nginx
  kubernetes.io/ingress.class: "nginx"  spec:
    rules:
    - host: myserviceb.foo.org
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: myserviceb
              port:
                number: 80
         
```

复制

```js

```

复制

- k8s >= 1.19 版本
- 只支持配置nginxClassName，不支持配置注解

```js

```

复制

  apiVersion: networking.k8s.io/v1

  kind: Ingress

  metadata:

```js
name: ingress-myservicea
```

复制

  spec:

```js
rules:
```

复制

```js
- host: myservicea.foo.org
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: myservicea
          port:
            number: 80
ingressClassName: nginxundefined  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: ingress-myserviceb
  spec:
    rules:
    - host: myserviceb.foo.org
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: myserviceb
              port:
                number: 80
    ingressClassName: nginx
```

复制

### ConfigMap

[完整配置参考](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)

### 日志格式

```js
log_format upstreaminfo
    '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
    '$request_length $request_time [$proxy_upstream_name] [$proxy_alternative_upstream_name] $upstream_addr '
    '$upstream_response_length $upstream_response_time $upstream_status $req_id';
```

复制

- $remote_addr: 客户端的源 ip 地址
- $remote_user: 认证的用户名
- $time_local: 时间
- $request: 完整的原始请求行
- $status： 响应状态码
- $body_bytes_sent：客户端发送字节数
- $http_referer：引用的 header 值

## 注解说明

> 注解的 key 和 value 只支持 string，因此 bool 或者 int 必须使用引号。比如 “ture”、“100”、“false”
>
> 注解的前缀默认是 “nginx.ingress.kubernetes.io"，可以通过 --annotations-prefix 命令修改

### 金丝雀

- nginx.ingress.kubernetes.io/canary: "true" ：开启金丝雀发布功能
- nginx.ingress.kubernetes.io/canary-by-header：
- nginx.ingress.kubernetes.io/canary-by-header-value：
- nginx.ingress.kubernetes.io/canary-by-header-pattern：
- `nginx.ingress.kubernetes.io/canary-by-cookie`: 
- `nginx.ingress.kubernetes.io/canary-weight`: 
- `nginx.ingress.kubernetes.io/canary-weight-total`: 总权重，默认100

优先级关系：

canary-by-header > canary-by-cookie -> canary-weight

### 重定向

- nginx.ingress.kubernetes.io/rewrite-target：

### Session 亲和

- nginx.ingress.kubernetes.io/affinity：“cookie",  一个请求将打到同样的 upstream server
- nginx.ingress.kubernetes.io/affinity-mode
- nginx.ingress.kubernetes.io/session-cookie-name
- nginx.ingress.kubernetes.io/session-cookie-path
- nginx.ingress.kubernetes.io/use-regex
- nginx.ingress.kubernetes.io/session-cookie-samesite

### 认证

- nginx.ingress.kubernetes.io/auth-type: basic|digest
- nginx.ingress.kubernetes.io/auth-secret: secretName
- nginx.ingress.kubernetes.io/auth-secret-type: auth-file|auth-map
- nginx.ingress.kubernetes.io/auth-realm: "realm string"

### 自定义 upstream 哈希

Nginx load balance 基于一致性哈希

- nginx.ingress.kubernetes.io/upstream-hash-by
  - "$request_uri"
  - "$request_uri$host"
  - "${request_uri}-text-value"
- `nginx.ingress.kubernetes.io/upstream-hash-by-subset`: "true"

### 自定义 load balance

- nginx.ingress.kubernetes.io/upstream-hash-by
- 

### Configuration 代码段

代码段全部追加到 nginx 的 location 配置中

> 在多住户集群中，这是一个危险操作，会导致其他权限限制之外的人可以获取到集群中所有的 secret
>
> 官方推荐禁用此功能，参考 [configuration snippet](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#configuration-snippet)

- nginx.ingress.kubernetes.io/configuration-snippet: more_set_headers "Request-Id: $req_id";

### 默认后端

- nginx.ingress.kubernetes.io/default-backend: <svc name>

### 跨域

- nginx.ingress.kubernetes.io/enable-cors: "true"

- ```
  nginx.ingress.kubernetes.io/cors-allow-methods
  ```

  : "PUT, GET, POST, OPTIONS"

  - 默认是：GET, PUT, POST, DELETE, PATCH, OPTIONS

- ```
  nginx.ingress.kubernetes.io/cors-allow-headers
  ```

  : 

  - 支持配置多个字段，以‘,’分割
  - 默认是：DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization

- nginx.ingress.kubernetes.io/cors-expose-headers：

  - 默认“”

- ```
  nginx.ingress.kubernetes.io/cors-allow-origin
  ```

  :

  - 默认 ”*"

- ```
  nginx.ingress.kubernetes.io/cors-allow-credentials
  ```

  : 

  - 默认 ”true“

- ```
  nginx.ingress.kubernetes.io/cors-max-age
  ```

  : 

  - 默认 1728000

### Http2

- nginx.ingress.kubernetes.io/http2-push-preload: "true"

### Server Alias

追加 server 配置中的 server_name 字段值

- nginx.ingress.kubernetes.io/server-alias:

### Server 代码段

添加配置代码到 server 配置项中

>  这个注解在每个host 中只能使用一次

- nginx.ingress.kubernetes.io/server-snippet

### 限流

- nginx.ingress.kubernetes.io/limit-connections
- nginx.ingress.kubernetes.io/limit-rps
- nginx.ingress.kubernetes.io/limit-rpm
- nginx.ingress.kubernetes.io/limit-rate-after
- nginx.ingress.kubernetes.io/limit-rate

### 永久重定向

- nginx.ingress.kubernetes.io/permanent-redirect:

### 自定义最大body大小

- nginx.ingress.kubernetes.io/proxy-body-size: 8m

### 后端协议

- nginx.ingress.kubernetes.io/backend-protocol: 
  - "HTTPS"
  - "GRPC"

## 第三方插件

### ModSecurity WAF

### OpenTracing

nginx-ingress 使用  [opentracing-contrib/nginx-opentracing](https://github.com/opentracing-contrib/nginx-opentracing) 模块支持 opentracing，默认该特性的关闭的

#### 全局开启

configmap 中新增如下字段

```js
data:
  enable-opentracing: "true"
```

复制

#### 部分开启

ingress 配置中新增如下注解

```js
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/enable-opentracing: "true"
```

复制

#### configmap中采集端相关配置

```js
opentracing-XXX
zipkin-XXX
jaeger-XXX
datadlog-XXX
```

复制

原创声明，本文系作者授权云+社区发表，未经许可，不得转载。

如有侵权，请联系 yunjia_community@tencent.com 删除。



[Nginx-Ingress详解 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1927201)