# 真一文搞定 ingress-nginx 的使用

2020-12-16阅读 4K0

前面我们学习了在 Kubernetes 集群内部使用 kube-dns 实现服务发现的功能，那么我们部署在 Kubernetes 集群中的应用如何暴露给外部的用户使用呢？我们知道可以使用 `NodePort` 和 `LoadBlancer` 类型的 Service 可以把应用暴露给外部用户使用，除此之外，Kubernetes 还为我们提供了一个非常重要的资源对象可以用来暴露服务给外部用户，那就是 `Ingress`。对于小规模的应用我们使用 NodePort 或许能够满足我们的需求，但是当你的应用越来越多的时候，你就会发现对于 NodePort 的管理就非常麻烦了，这个时候使用 Ingress 就非常方便了，可以避免管理大量的端口。

Ingress 其实就是从 Kuberenets 集群外部访问集群的一个入口，将外部的请求转发到集群内不同的 Service 上，其实就相当于 nginx、haproxy 等[负载均衡](https://cloud.tencent.com/product/clb?from=10680)代理[服务器](https://cloud.tencent.com/product/cvm?from=10680)，可能你会觉得我们直接使用 nginx 就实现了，但是只使用 nginx 这种方式有很大缺陷，每次有新服务加入的时候怎么改 Nginx 配置？不可能让我们去手动更改或者滚动更新前端的 Nginx Pod 吧？那我们再加上一个服务发现的工具比如 consul 如何？貌似是可以，对吧？Ingress 实际上就是这样实现的，只是服务发现的功能自己实现了，不需要使用第三方的服务了，然后再加上一个域名规则定义，路由信息的刷新依靠 Ingress Controller 来提供。

Ingress Controller 可以理解为一个监听器，通过不断地监听 kube-apiserver，实时的感知后端 Service、Pod 的变化，当得到这些信息变化后，Ingress Controller 再结合 Ingress 的配置，更新反向代理负载均衡器，达到服务发现的作用。其实这点和服务发现工具 consul、 consul-template 非常类似。

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/ri1tgz7lgj.png?imageView2/2/w/1620)

ingress flow

现在可以供大家使用的 Ingress Controller 有很多，比如 traefik、nginx-controller、Kubernetes Ingress Controller for Kong、HAProxy Ingress controller，当然你也可以自己实现一个 Ingress Controller，现在普遍用得较多的是 traefik 和 nginx-controller，traefik 的性能较 nginx-controller 差，但是配置使用要简单许多，我们这里会重点给大家介绍 nginx-controller 的使用。

## **安装**

NGINX Ingress Controller 是使用 Kubernetes Ingress 资源对象构建的，用 ConfigMap 来存储 Nginx 配置的一种 Ingress Controller 实现。

要使用 Ingress 对外暴露服务，就需要提前安装一个 Ingress Controller，我们这里就先来安装 NGINX Ingress Controller，由于 nginx-ingress 所在的节点需要能够访问外网，这样域名可以解析到这些节点上直接使用，所以需要让 nginx-ingress 绑定节点的 80 和 443 端口，所以可以使用 hostPort 来进行访问，当然对于线上环境来说为了保证高可用，一般是需要运行多个 nginx-ingress 实例的，然后可以用一个 nginx/haproxy 作为入口，通过 keepalived 来访问边缘节点的 vip 地址。

> “所谓的边缘节点即集群内部用来向集群外暴露服务能力的节点，集群外部的服务通过该节点来调用集群内部的服务，边缘节点是集群内外交流的一个 Endpoint。 ”

所以我们这里需要更改下资源清单文件：

```javascript
➜ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
➜ helm repo update
➜ helm fetch ingress-nginx/ingress-nginx
➜ tar -xvf ingress-nginx-3.15.2.tgz
```

复制

我们这里测试环境只有 master1 节点可以访问外网，这里我们就直接讲 ingress-nginx 固定到 master1 节点上，采用 hostNetwork 模式(生产环境可以使用 LB + DaemonSet hostNetwork 模式)。然后新建一个名为 values-prod.yaml 的 Values 文件，用来覆盖 ingress-nginx 默认的 Values 值，对应的数据如下所示：

```javascript
# values-prod.yaml
controller:
  name: controller
  image:
    repository: cnych/ingress-nginx
    tag: "v0.41.2"
    digest: 

  dnsPolicy: ClusterFirstWithHostNet
   
  hostNetwork: true

  publishService:  # hostNetwork 模式下设置为false，通过节点IP地址上报ingress status数据
    enabled: false

  kind: DaemonSet

  nodeSelector: 
    role: lb

  service:  # HostNetwork 模式不需要创建service
    enabled: false

defaultBackend:
  enabled: true
  name: defaultbackend
  image:
    repository: cnych/ingress-nginx-defaultbackend
    tag: "1.5"
```

复制

然后使用如下命令安装 ingress-nginx 应用到 ingress-nginx 的命名空间中：

```javascript
➜ kubectl create ns ingress-nginx
➜ helm install --namespace ingress-nginx ingress-nginx ./ingress-nginx -f ./ingress-nginx/values-prod.yaml 
NAME: ingress-nginx
LAST DEPLOYED: Fri Dec 11 14:19:05 2020
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
Get the application URL by running these commands:
  export POD_NAME=$(kubectl --namespace ingress-nginx get pods -o jsonpath="{.items[0].metadata.name}" -l "app=ingress-nginx,component=controller,release=ingress-nginx")
  kubectl --namespace ingress-nginx port-forward $POD_NAME 8080:80
  echo "Visit http://127.0.0.1:8080 to access your application."

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

复制

部署完成后查看 Pod 的运行状态：

```javascript
➜ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
ingress-nginx-controller-admission   ClusterIP   10.110.143.167   <none>        443/TCP   2m21s
ingress-nginx-defaultbackend         ClusterIP   10.104.156.141   <none>        80/TCP    2m21s
➜ kubectl get pods -n ingress-nginx
NAME                                            READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-596955b554-vfmhq       1/1     Running   0          31s
ingress-nginx-defaultbackend-7bf9445d94-lkgw5   1/1     Running   0          3m52s
➜ POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx -o jsonpath='{.items[0].metadata.name}')
➜ kubectl exec -it $POD_NAME -n ingress-nginx -- /nginx-ingress-controller --version
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v0.41.2
  Build:         d8a93551e6e5798fc4af3eb910cef62ecddc8938
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.4

-------------------------------------------------------------------------------
```

复制

当看到上面的信息证明 ingress-nginx 部署成功了。

## **Ingress**

安装成功后，现在我们来为一个 nginx 应用创建一个 Ingress 资源，如下所示：

```javascript
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    app: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    name: http
  selector:
    app: my-nginx
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: ngdemo.qikqiak.com  # 将域名映射到 my-nginx 服务
    http:
      paths:
      - path: /
        backend:
          serviceName: my-nginx  # 将所有请求发送到 my-nginx 服务的 80 端口
          servicePort: 80     # 不过需要注意大部分Ingress controller都不是直接转发到Service
                            # 而是只是通过Service来获取后端的Endpoints列表，直接转发到Pod，这样可以减少网络跳转，提高性能
```

复制

直接创建上面的资源对象：

```javascript
➜ kubectl apply -f ngdemo.yaml
deployment.apps "my-nginx" created
service "my-nginx" created
ingress.extensions "my-nginx" created
```

复制

注意我们在 Ingress 资源对象中添加了一个 annotations：`kubernetes.io/ingress.class: "nginx"`，这就是指定让这个 Ingress 通过 nginx-ingress 来处理。

上面资源创建成功后，然后我们可以将域名 `ngdemo.qikqiak.com` 解析到 `ingress-nginx` 所在的边缘节点中的任意一个，当然也可以在本地`/etc/hosts`中添加对应的映射也可以，然后就可以通过域名进行访问了。

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/buryat8qa8.jpeg?imageView2/2/w/1620)

ngdemo

下图显示了客户端是如果通过 Ingress Controller 连接到其中一个 Pod 的流程，客户端首先对 `ngdemo.qikqiak.com` 执行 DNS 解析，得到 Ingress Controller 所在节点的 IP，然后客户端向 Ingress Controller 发送 HTTP 请求，然后根据 Ingress 对象里面的描述匹配域名，找到对应的 Service 对象，并获取关联的 Endpoints 列表，将客户端的请求转发给其中一个 Pod。

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/8d7kmkzrx9.png?imageView2/2/w/1620)

ingress controller workflow

## **URL Rewrite**

NGINX Ingress Controller 很多高级的用法可以通过 Ingress 对象的 `annotation` 进行配置，比如常用的 URL Rewrite 功能，比如我们有一个 todo 的前端应用，代码位于 https://github.com/cnych/todo-app，直接部署这个应用进行测试：

```javascript
➜ kubectl apply -f https://github.com/cnych/todo-app/raw/master/k8s/mongo.yaml
➜ kubectl apply -f https://github.com/cnych/todo-app/raw/master/k8s/web.yaml
➜ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
mongo-5c9fd978bb-txn9j      1/1     Running   0          149m
todo-566957d785-tdgs6       1/1     Running   0          3m31s
......
➜ kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP                      54d
mongo                      ClusterIP   10.96.95.11      <none>        27017/TCP                    150m
todo                       ClusterIP   10.111.105.47    <none>        3000/TCP                     145m
......
```

复制

对应的 Ingress 资源对象如下所示：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - path: /
        backend:
          serviceName: todo
          servicePort: 3000
```

复制

就是一个很常规的 `Ingress` 对象，部署该对象后，将域名解析后就可以正常访问到应用：

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/x5dhzt962q.png?imageView2/2/w/1620)

ingress nginx demo

现在我们需要对访问的 URL 路径做一个 Rewrite，比如在 PATH 中添加一个 app 的前缀，关于 Rewrite 的操作在 ingress-nginx 官方文档中也给出对应的说明:

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/7efgbu3pux.png?imageView2/2/w/1620)

ingress nginx rewrite

按照要求我们需要在 `path` 中匹配前缀 `app`，然后通过 `rewrite-target` 指定目标，修改后的 Ingress 对象如下所示：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - backend:
          serviceName: todo
          servicePort: 3000
        path: /app(/|$)(.*)
```

复制

更新后，我们可以遇见到直接访问域名肯定是不行了，因为我们没有匹配 `/` 的 path 路径：

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/azrfpmsz8g.png?imageView2/2/w/1620)

ingress nginx rewrite 404

但是我们带上 `app` 的前缀再去访问:

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/ymllein9un.png?imageView2/2/w/1620)

ingress nginx rewrite 1

我们可以看到已经可以访问到页面内容了，这是因为我们在 `path` 中通过正则表达式 `/app(/|$)(.*)` 将匹配的路径设置成了 `rewrite-target` 的目标路径了，所以我们访问 `todo.qikqiak.com/app` 的时候实际上相当于访问的就是后端服务的 `/` 路径，但是我们也可以发现现在页面的样式没有了：

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/mt4d3tn3b9.png?imageView2/2/w/1620)

ingress nginx rewrite 2

这是因为应用的静态资源路径是在 `/stylesheets` 路径下面的，现在我们做了 url rewrite 过后，要正常访问也需要带上前缀才可以：`http://todo.qikqiak.com/stylesheets/screen.css`，对于图片或者其他静态资源也是如此，当然我们去更改页面引入静态资源的方式为相对路径也是可以的，但是毕竟要修改代码，这个时候我们可以借助 `ingress-nginx` 中的 `configuration-snippet` 来对静态资源做一次跳转，如下所示：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite ^/stylesheets/(.*)$ /app/stylesheets/$1 redirect;  # 添加 /app 前缀
      rewrite ^/images/(.*)$ /app/images/$1 redirect;  # 添加 /app 前缀
spec:
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - backend:
          serviceName: todo
          servicePort: 3000
        path: /app(/|$)(.*)
```

复制

更新 Ingress 对象后，这个时候我们刷新页面可以看到已经正常了：

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/p8lgaskgvo.png?imageView2/2/w/1620)

ingress nginx rewrite 3

要解决我们访问主域名出现 404 的问题，我们可以给应用设置一个 `app-root` 的注解，这样当我们访问主域名的时候会自动跳转到我们指定的 `app-root` 目录下面，如下所示：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/app-root: /app/
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite ^/stylesheets/(.*)$ /app/stylesheets/$1 redirect;  # 添加 /app 前缀
      rewrite ^/images/(.*)$ /app/images/$1 redirect;  # 添加 /app 前缀
spec:
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - backend:
          serviceName: todo
          servicePort: 3000
        path: /app(/|$)(.*)
```

复制

这个时候我们更新应用后访问主域名 `http://todo.qikqiak.com` 就会自动跳转到 `http://todo.qikqiak.com/app/` 路径下面去了。但是还有一个问题是我们的 path 路径其实也匹配了 `/app` 这样的路径，可能我们更加希望我们的应用在最后添加一个 `/` 这样的 slash，同样我们可以通过 `configuration-snippet` 配置来完成，如下 Ingress 对象：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/app-root: /app/
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite ^(/app)$ $1/ redirect;
      rewrite ^/stylesheets/(.*)$ /app/stylesheets/$1 redirect;
      rewrite ^/images/(.*)$ /app/images/$1 redirect;
spec:
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - backend:
          serviceName: todo
          servicePort: 3000
        path: /app(/|$)(.*)
```

复制

更新后我们的应用就都会以 `/` 这样的 slash 结尾了。这样就完成了我们的需求，如果你原本对 nginx 的配置就非常熟悉的话应该可以很快就能理解这种配置方式了。

## **Basic Auth**

同样我们还可以在 Ingress Controller 上面配置一些基本的 Auth 认证，比如 Basic Auth，可以用 htpasswd 生成一个密码文件来验证身份验证。

```javascript
➜ htpasswd -c auth foo
New password:
Re-type new password:
Adding password for user foo
```

复制

然后根据上面的 auth 文件创建一个 secret 对象：

```javascript
➜ kubectl create secret generic basic-auth --from-file=auth
secret/basic-auth created
➜ kubectl get secret basic-auth -o yaml
apiVersion: v1
data:
  auth: Zm9vOiRhcHIxJFNjcVhZcFN6JDc4Nm5ISFNaeDdwN2VscDM2WUo0YS8K
kind: Secret
metadata:
  creationTimestamp: "2019-12-08T06:40:39Z"
  name: basic-auth
  namespace: default
  resourceVersion: "9197951"
  selfLink: /api/v1/namespaces/default/secrets/basic-auth
  uid: 6b2aa299-b511-412e-85ea-d0e91e578af0
type: Opaque
```

复制

然后对上面的 my-nginx 应用创建一个具有 Basic Auth 的 Ingress 对象：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-with-auth
  annotations:
    # 认证类型
    nginx.ingress.kubernetes.io/auth-type: basic
    # 包含 user/password 定义的 secret 对象名
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # 要显示的带有适当上下文的消息，说明需要身份验证的原因
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /
        backend:
          serviceName: my-nginx
          servicePort: 80
```

复制

直接创建上面的资源对象，然后通过下面的命令或者在浏览器中直接打开配置的域名：

```javascript
➜ curl -v http://k8s.qikqiak.com -H 'Host: foo.bar.com'
* Rebuilt URL to: http://k8s.qikqiak.com/
*   Trying 123.59.188.12...
* TCP_NODELAY set
* Connected to k8s.qikqiak.com (123.59.188.12) port 80 (#0)
> GET / HTTP/1.1
> Host: foo.bar.com
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 401 Unauthorized
< Server: openresty/1.15.8.2
< Date: Sun, 08 Dec 2019 06:44:35 GMT
< Content-Type: text/html
< Content-Length: 185
< Connection: keep-alive
< WWW-Authenticate: Basic realm="Authentication Required - foo"
<
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>openresty/1.15.8.2</center>
</body>
</html>
```

复制

我们可以看到出现了 401 认证失败错误，然后带上我们配置的用户名和密码进行认证：

```javascript
➜ curl -v http://k8s.qikqiak.com -H 'Host: foo.bar.com' -u 'foo:foo'
* Rebuilt URL to: http://k8s.qikqiak.com/
*   Trying 123.59.188.12...
* TCP_NODELAY set
* Connected to k8s.qikqiak.com (123.59.188.12) port 80 (#0)
* Server auth using Basic with user 'foo'
> GET / HTTP/1.1
> Host: foo.bar.com
> Authorization: Basic Zm9vOmZvbw==
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: openresty/1.15.8.2
< Date: Sun, 08 Dec 2019 06:46:27 GMT
< Content-Type: text/html
< Content-Length: 612
< Connection: keep-alive
< Vary: Accept-Encoding
< Last-Modified: Tue, 19 Nov 2019 12:50:08 GMT
< ETag: "5dd3e500-264"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

复制

可以看到已经认证成功了。当然出来 Basic Auth 这一种简单的认证方式之外，NGINX Ingress Controller 还支持一些其他高级的认证，比如 OAUTH 认证之类的。

## **灰度发布**

在日常工作中我们经常需要对服务进行版本更新升级，所以我们经常会使用到滚动升级、蓝绿发布、灰度发布等不同的发布操作。而 `ingress-nginx` 支持通过 Annotations 配置来实现不同场景下的灰度发布和测试，可以满足金丝雀发布、蓝绿部署与 A/B 测试等业务场景。ingress-nginx 的 Annotations 支持以下 4 种 Canary 规则：

- `nginx.ingress.kubernetes.io/canary-by-header`：基于 Request Header 的流量切分，适用于灰度发布以及 A/B 测试。当 Request Header 设置为 always 时，请求将会被一直发送到 Canary 版本；当 Request Header 设置为 never时，请求不会被发送到 Canary 入口；对于任何其他 Header 值，将忽略 Header，并通过优先级将请求与其他金丝雀规则进行优先级的比较。
- `nginx.ingress.kubernetes.io/canary-by-header-value`：要匹配的 Request Header 的值，用于通知 Ingress 将请求路由到 Canary Ingress 中指定的服务。当 Request Header 设置为此值时，它将被路由到 Canary 入口。该规则允许用户自定义 Request Header 的值，必须与上一个 annotation (即：canary-by-header) 一起使用。
- `nginx.ingress.kubernetes.io/canary-weight`：基于服务权重的流量切分，适用于蓝绿部署，权重范围 0 - 100 按百分比将请求路由到 Canary Ingress 中指定的服务。权重为 0 意味着该金丝雀规则不会向 Canary 入口的服务发送任何请求，权重为 100 意味着所有请求都将被发送到 Canary 入口。
- `nginx.ingress.kubernetes.io/canary-by-cookie`：基于 cookie 的流量切分，适用于灰度发布与 A/B 测试。用于通知 Ingress 将请求路由到 Canary Ingress 中指定的服务的cookie。当 cookie 值设置为 always 时，它将被路由到 Canary 入口；当 cookie 值设置为 never 时，请求不会被发送到 Canary 入口；对于任何其他值，将忽略 cookie 并将请求与其他金丝雀规则进行优先级的比较。

> “需要注意的是金丝雀规则按优先顺序进行排序：`canary-by-header - > canary-by-cookie - > canary-weight` ”

总的来说可以把以上的四个 annotation 规则划分为以下两类：

- 基于权重的 Canary 规则

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/w4jzoiz1qy.png?imageView2/2/w/1620)

- 基于用户请求的 Canary 规则

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/hsvf8fap2x.png?imageView2/2/w/1620)

下面我们通过一个示例应用来对灰度发布功能进行说明。

第一步. 部署 Production 应用

首先创建一个 production 环境的应用资源清单：

```javascript
# production.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production
  labels:
    app: production
spec:
  selector:
    matchLabels:
      app: production
  template:
    metadata:
      labels:
        app: production
    spec:
      containers:
      - name: production
        image: cnych/echoserver
        ports:
        - containerPort: 8080
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
---
apiVersion: v1
kind: Service
metadata:
  name: production
  labels:
    app: production
spec:
  ports:
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: production
```

复制

然后创建一个用于 production 环境访问的 Ingress 资源对象：

```javascript
# production-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: production
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: echo.qikqiak.com
    http:
      paths:
      - backend:
          serviceName: production
          servicePort: 80
```

复制

直接创建上面的几个资源对象：

```javascript
➜ kubectl apply -f production.yaml
➜ kubectl apply -f production-ingress.yaml
➜ kubectl get pods -l app=production
NAME                         READY   STATUS    RESTARTS   AGE
production-856d5fb99-d6bds   1/1     Running   0          2m50s
➜ kubectl get ingress              
NAME         CLASS    HOSTS                ADDRESS        PORTS   AGE
production   <none>   echo.qikqiak.com     10.151.30.11   80      90s
```

复制

应用部署成功后，将域名 `echo.qikqiak.com` 映射到 master1 节点（ingress-nginx 所在的节点）的外网 IP，然后即可正常访问应用了：

```javascript
➜ curl http://echo.qikqiak.com


Hostname: production-856d5fb99-d6bds

Pod Information:
 node name: node1
 pod name: production-856d5fb99-d6bds
 pod namespace: default
 pod IP: 10.244.1.111

Server values:
 server_version=nginx: 1.13.3 - lua: 10008

Request Information:
 client_address=10.244.0.0
 method=GET
 real path=/
 query=
 request_version=1.1
 request_scheme=http
 request_uri=http://echo.qikqiak.com:8080/

Request Headers:
 accept=*/*
 host=echo.qikqiak.com
 user-agent=curl/7.64.1
 x-forwarded-for=171.223.99.184
 x-forwarded-host=echo.qikqiak.com
 x-forwarded-port=80
 x-forwarded-proto=http
 x-real-ip=171.223.99.184
 x-request-id=e680453640169a7ea21afba8eba9e116
 x-scheme=http

Request Body:
 -no body in request-
```

复制

第二步. 创建 Canary 版本

参考将上述 Production 版本的 `production.yaml` 文件，再创建一个 Canary 版本的应用。

```javascript
# canary.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: canary
  labels:
    app: canary
spec:
  selector:
    matchLabels:
      app: canary
  template:
    metadata:
      labels:
        app: canary
    spec:
      containers:
      - name: canary
        image: cnych/echoserver
        ports:
        - containerPort: 8080
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
---
apiVersion: v1
kind: Service
metadata:
  name: canary
  labels:
    app: canary
spec:
  ports:
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: canary
```

复制

接下来就可以通过配置 Annotation 规则进行流量切分了。

第三步. Annotation 规则配置

\1. 基于权重：基于权重的流量切分的典型应用场景就是蓝绿部署，可通过将权重设置为 0 或 100 来实现。例如，可将 Green 版本设置为主要部分，并将 Blue 版本的入口配置为 Canary。最初，将权重设置为 0，因此不会将流量代理到 Blue 版本。一旦新版本测试和验证都成功后，即可将 Blue 版本的权重设置为 100，即所有流量从 Green 版本转向 Blue。

创建一个基于权重的 Canary 版本的应用路由 Ingress 对象。

```javascript
# canary-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: canary
  annotations:
    kubernetes.io/ingress.class: nginx 
    nginx.ingress.kubernetes.io/canary: "true"   # 要开启灰度发布机制，首先需要启用 Canary
    nginx.ingress.kubernetes.io/canary-weight: "30"  # 分配30%流量到当前Canary版本
spec:
  rules:
  - host: echo.qikqiak.com
    http:
      paths:
      - backend:
          serviceName: canary
          servicePort: 80
```

复制

直接创建上面的资源对象即可：

```javascript
➜ kubectl apply -f canary.yaml
➜ kubectl apply -f canary-ingress.yaml
➜ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
canary-66cb497b7f-48zx4      1/1     Running   0          7m48s
production-856d5fb99-d6bds   1/1     Running   0          21m
......
➜ kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
canary                     ClusterIP   10.106.91.106    <none>        80/TCP                       8m23s
production                 ClusterIP   10.105.182.15    <none>        80/TCP                       22m
......
➜ kubectl get ingress
NAME         CLASS    HOSTS                ADDRESS        PORTS   AGE
canary       <none>   echo.qikqiak.com     10.151.30.11   80      108s
production   <none>   echo.qikqiak.com     10.151.30.11   80      22m
```

复制

Canary 版本应用创建成功后，接下来我们在命令行终端中来不断访问这个应用，观察 Hostname 变化：

```javascript
➜ for i in $(seq 1 10); do curl -s echo.qikqiak.com | grep "Hostname"; done
Hostname: production-856d5fb99-d6bds
Hostname: canary-66cb497b7f-48zx4
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: production-856d5fb99-d6bds
```

复制

由于我们给 Canary 版本应用分配了 30% 左右权重的流量，所以上面我们访问10次有3次访问到了 Canary 版本的应用，符合我们的预期。

\2. 基于 Request Header: 基于 Request Header 进行流量切分的典型应用场景即灰度发布或 A/B 测试场景。

在上面的 Canary 版本的 Ingress 对象中新增一条 annotation 配置 `nginx.ingress.kubernetes.io/canary-by-header: canary`（这里的 value 可以是任意值），使当前的 Ingress 实现基于 Request Header 进行流量切分，由于 `canary-by-header` 的优先级大于 `canary-weight`，所以会忽略原有的 `canary-weight` 的规则。

```javascript
annotations:
  kubernetes.io/ingress.class: nginx 
  nginx.ingress.kubernetes.io/canary: "true"   # 要开启灰度发布机制，首先需要启用 Canary
  nginx.ingress.kubernetes.io/canary-by-header: canary  # 基于header的流量切分
  nginx.ingress.kubernetes.io/canary-weight: "30"  # 会被忽略，因为配置了 canary-by-headerCanary版本
```

复制

更新上面的 Ingress 资源对象后，我们在请求中加入不同的 Header 值，再次访问应用的域名。

> “注意：当 Request Header 设置为 never 或 always 时，请求将不会或一直被发送到 Canary 版本，对于任何其他 Header 值，将忽略 Header，并通过优先级将请求与其他 Canary 规则进行优先级的比较。 ”

```javascript
➜ for i in $(seq 1 10); do curl -s -H "canary: never" echo.qikqiak.com | grep "Hostname"; done
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
```

复制

这里我们在请求的时候设置了 `canary: never` 这个 Header 值，所以请求没有发送到 Canary 应用中去。如果设置为其他值呢：

```javascript
➜ for i in $(seq 1 10); do curl -s -H "canary: other-value" echo.qikqiak.com | grep "Hostname"; done
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: canary-66cb497b7f-48zx4
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: production-856d5fb99-d6bds
Hostname: canary-66cb497b7f-48zx4
Hostname: production-856d5fb99-d6bds
Hostname: canary-66cb497b7f-48zx4
```

复制

由于我们请求设置的 Header 值为 `canary: other-value`，所以 ingress-nginx 会通过优先级将请求与其他 Canary 规则进行优先级的比较，我们这里也就会进入 `canary-weight: "30"` 这个规则去。

这个时候我们可以在上一个 annotation (即 `canary-by-header`）的基础上添加一条 `nginx.ingress.kubernetes.io/canary-by-header-value: user-value` 这样的规则，就可以将请求路由到 Canary Ingress 中指定的服务了。

```javascript
annotations:
  kubernetes.io/ingress.class: nginx 
  nginx.ingress.kubernetes.io/canary: "true"   # 要开启灰度发布机制，首先需要启用 Canary
  nginx.ingress.kubernetes.io/canary-by-header-value: user-value  
  nginx.ingress.kubernetes.io/canary-by-header: canary  # 基于header的流量切分
  nginx.ingress.kubernetes.io/canary-weight: "30"  # 分配30%流量到当前Canary版本
```

复制

同样更新 Ingress 对象后，重新访问应用，当 Request Header 满足 `canary: user-value`时，所有请求就会被路由到 Canary 版本：

```javascript
➜ for i in $(seq 1 10); do curl -s -H "canary: user-value" echo.qikqiak.com | grep "Hostname"; done
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
```

复制

\3. 基于 Cookie：与基于 Request Header 的 annotation 用法规则类似。例如在 A/B 测试场景下，需要让地域为北京的用户访问 Canary 版本。那么当 cookie 的 annotation 设置为 `nginx.ingress.kubernetes.io/canary-by-cookie: "users_from_Beijing"`，此时后台可对登录的用户请求进行检查，如果该用户访问源来自北京则设置 cookie `users_from_Beijing` 的值为 `always`，这样就可以确保北京的用户仅访问 Canary 版本。

同样我们更新 Canary 版本的 Ingress 资源对象，采用基于 Cookie 来进行流量切分，

```javascript
annotations:
  kubernetes.io/ingress.class: nginx 
  nginx.ingress.kubernetes.io/canary: "true"   # 要开启灰度发布机制，首先需要启用 Canary
  nginx.ingress.kubernetes.io/canary-by-cookie: "users_from_Beijing"  # 基于 cookie
  nginx.ingress.kubernetes.io/canary-weight: "30"  # 会被忽略，因为配置了 canary-by-cookie
```

复制

更新上面的 Ingress 资源对象后，我们在请求中设置一个 `users_from_Beijing=always` 的 Cookie 值，再次访问应用的域名。

```javascript
➜ for i in $(seq 1 10); do curl -s -b "users_from_Beijing=always" echo.qikqiak.com | grep "Hostname"; done
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
Hostname: canary-66cb497b7f-48zx4
```

复制

我们可以看到应用都被路由到了 Canary 版本的应用中去了，如果我们将这个 Cookie 值设置为 never，则不会路由到 Canary 应用中。

## **HTTPS**

如果我们需要用 HTTPS 来访问我们这个应用的话，就需要监听 443 端口了，同样用 HTTPS 访问应用必然就需要证书，这里我们用 `openssl` 来创建一个自签名的证书：

```javascript
➜ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=foo.bar.com"
```

复制

然后通过 Secret 对象来引用证书文件：

```javascript
# 要注意证书文件名称必须是 tls.crt 和 tls.key
➜ kubectl create secret tls foo-tls --cert=tls.crt --key=tls.key
secret/who-tls created
```

复制

这个时候我们就可以创建一个 HTTPS 访问应用的：

```javascript
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-with-auth
  annotations:
    # 认证类型
    nginx.ingress.kubernetes.io/auth-type: basic
    # 包含 user/password 定义的 secret 对象名
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # 要显示的带有适当上下文的消息，说明需要身份验证的原因
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /
        backend:
          serviceName: my-nginx
          servicePort: 80
  tls:
  - hosts:
    - foo.bar.com
    secretName: foo-tls
```

复制

除了自签名证书或者购买正规机构的 CA 证书之外，我们还可以通过 `letsencrypt` 来自动生成合法的证书。

## **CertManager 自动 HTTPS**

### **安装配置**

cert-manager 是一个云原生证书管理开源项目，用于在 Kubernetes 集群中提供 HTTPS 证书并自动续期，支持 `Let's Encrypt/HashiCorp/Vault` 这些免费证书的签发。在 Kubernetes 中，可以通过 Kubernetes Ingress 和 Let's Encrypt 实现外部服务的自动化 HTTPS。

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/gn4fxfk4go.png?imageView2/2/w/1620)

cert-manager 架构

上面是官方给出的架构图，可以看到 cert-manager 在 Kubernetes 中定义了两个自定义类型资源：`Issuer(ClusterIssuer)` 和 `Certificate`。

- 其中 `Issuer` 代表的是证书颁发者，可以定义各种提供者的证书颁发者，当前支持基于 `Let's Encrypt/HashiCorp/Vault` 和 CA 的证书颁发者，还可以定义不同环境下的证书颁发者。
- 而 `Certificate` 代表的是生成证书的请求，一般其中存入生成证书的元信息，如域名等等。

一旦在 Kubernetes 中定义了上述两类资源，部署的 cert-manager 则会根据 `Issuer` 和 `Certificate` 生成 TLS 证书，并将证书保存进 Kubernetes 的 Secret 资源中，然后在 Ingress 资源中就可以引用到这些生成的 Secret 资源作为 TLS 证书使用，对于已经生成的证书，还会定期检查证书的有效期，如即将超过有效期，还会自动续期。

要在 Kubernetes 集群上安装 cert-manager 也非常简单，官方提供了一个单一的资源清单文件，包含了所有的资源对象，所以直接安装即可：

```javascript
# Kubernetes 1.16+
➜ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager.yaml
# Kubernetes <1.16
➜ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager-legacy.yaml
```

复制

上面的命令会创建一个名为 cert-manager 的命名空间，安装大量的 CRD 以及 AdmissionWebhook 对象，可以通过如下命令来查看是否安装成功：

```javascript
➜ kubectl get pods -n cert-manager
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-5597cff495-q6rzh             1/1     Running   0          5m31s
cert-manager-cainjector-bd5f9c764-5sc7d   1/1     Running   0          5m31s
cert-manager-webhook-5f57f59fbc-mvcq4     1/1     Running   0          5m30s
```

复制

正常情况下可以看到 `cert-manager`、`cert-manager-cainjector` 以及 `cert-manager-webhook` 这几个 Pod 处于 Running 状态。我们可以通过下面的测试来验证下是否可以签发基本的证书类型，创建一个 Issuer 资源对象来测试 webhook 工作是否正常(在开始签发证书之前，必须在群集中至少配置一个 Issuer 或 ClusterIssuer 资源)：

```javascript
➜ cat <<EOF > test-selfsigned.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}  # 配置自签名的证书机构类型
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
  - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
```

复制

这里我们创建了一个名为 `cert-manager-test` 的命名空间，创建了一个自签名的 Issuer 证书颁发机构，然后使用这个 Issuer 来创建一个证书请求的 Certificate 对象，直接创建上面的资源清单即可：

```javascript
➜ kubectl apply -f test-selfsigned.yaml
namespace/cert-manager-test created
issuer.cert-manager.io/test-selfsigned created
certificate.cert-manager.io/selfsigned-cert created
```

复制

创建完成后可以检查新创建的证书状态，在 cert-manager 处理证书请求之前，可能需要稍微等几秒：

```javascript
➜ kubectl describe certificate -n cert-manager-test
Name:         selfsigned-cert
Namespace:    cert-manager-test
......
Spec:
  Dns Names:
    example.com
  Issuer Ref:
    Name:       test-selfsigned
  Secret Name:  selfsigned-cert-tls
Status:
  Conditions:
    Last Transition Time:  2020-12-12T03:29:07Z
    Message:               Certificate is up to date and has not expired
    Reason:                Ready
    Status:                True
    Type:                  Ready
  Not After:               2021-03-12T03:29:06Z
  Not Before:              2020-12-12T03:29:06Z
  Renewal Time:            2021-02-10T03:29:06Z
  Revision:                1
Events:
  Type    Reason     Age   From          Message
  ----    ------     ----  ----          -------
  Normal  Issuing    6s    cert-manager  Issuing certificate as Secret does not exist
  Normal  Generated  6s    cert-manager  Stored new private key in temporary Secret resource "selfsigned-cert-sppz7"
  Normal  Requested  6s    cert-manager  Created new CertificateRequest resource "selfsigned-cert-z4nvl"
  Normal  Issuing    5s    cert-manager  The certificate has been successfully issued
```

复制

从上面的 Events 事件中我们可以证书已经成功签发了，生成的证书存放在一个名为 `selfsigned-cert-tls` 的 Secret 对象下面：

```javascript
➜ kubectl get secret -n cert-manager-test                            
NAME                  TYPE                                  DATA   AGE
default-token-t928x   kubernetes.io/service-account-token   3      64s
selfsigned-cert-tls   kubernetes.io/tls                     3      63s
➜ kubectl get secret -n cert-manager-test selfsigned-cert-tls -o yaml
apiVersion: v1
data:
  ca.crt: ......
  tls.crt: ......
  tls.key: ......
kind: Secret
......
  name: selfsigned-cert-tls
  namespace: cert-manager-test
  resourceVersion: "13461084"
  selfLink: /api/v1/namespaces/cert-manager-test/secrets/selfsigned-cert-tls
  uid: 42e456dc-6d34-4269-b207-f1f3bd50db8b
type: kubernetes.io/tls
```

复制

到这里证明我们的 cert-manager 已经安装成功了。我们需要注意的是 cert-manager 的功能非常强大，不只是可以支持 ACME 类型的证书签发，还支持其他众多的类型，比如 SelfSigned(自签名)、CA、Vault、Venafi、External、ACME，只是我们一般主要是使用 ACME 来帮我们生成自动化的证书。

下面我们就来使用 cert-manager 结合 ingress-nginx 为 Kubernetes 应用自动签发 `Let's Encrypt` 类型的 HTTPS 证书。

### **自动化 HTTPS**

`Let's Encrypt` 使用 `ACME` 协议来校验域名是否真的属于你，校验成功后就可以自动颁发免费证书，证书有效期只有 90 天，在到期前需要再校验一次来实现续期，而 cert-manager 是可以自动续期的，所以事实上并不用担心证书过期的问题。目前主要有 HTTP 和 DNS 两种校验方式。

HTTP-01 校验

`HTTP-01` 的校验是通过给你域名指向的 HTTP 服务增加一个临时 location，在校验的时候 `Let's Encrypt` 会发送 http 请求到 `http://<YOUR_DOMAIN>/.well-known/acme-challenge/<TOKEN>`，其中 `YOUR_DOMAIN` 就是被校验的域名，`TOKEN` 是 cert-manager 生成的一个路径，它通过修改 Ingress 规则来增加这个临时校验路径并指向提供 TOKEN 的服务。`Let's Encrypt` 会对比 TOKEN 是否符合预期，校验成功后就会颁发证书了，不过这种方法不支持泛域名证书。

使用 HTTP 校验这种方式，首先需要将域名解析配置好，也就是需要保证 ACME 服务端可以正常访问到你的 HTTP 服务。这里我们以上面的 TODO 应用为例，我们已经将 `todo.qikqiak.com` 域名做好了正确的解析。

由于 Let's Encrypt 的生产环境有着严格的接口调用限制，所以一般我们需要先在 staging 环境测试通过后，再切换到生产环境。首先我们创建一个全局范围 staging 环境使用的 HTTP-01 校验方式的证书颁发机构：

```javascript
➜ cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging-http01
spec:
  acme:
    # ACME 服务端地址
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # 用于 ACME 注册的邮箱
    email: icnych@gmail.com
    # 用于存放 ACME 帐号 private key 的 secret
    privateKeySecretRef:
      name: letsencrypt-staging-http01
    solvers:
    - http01: # ACME HTTP-01 类型
        ingress:
          class: nginx  # 指定ingress的名称
EOF
```

复制

同样再创建一个用于生产环境使用的 ClusterIssuer 对象：

```javascript
➜ cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http01
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: icnych@gmail.com
    privateKeySecretRef:
      name: letsencrypt-http01
    solvers:
    - http01: 
        ingress:
          class: nginx
EOF
```

复制

创建完成后可以看到两个 ClusterIssuer 对象：

```javascript
➜ kubectl get clusterissuer
NAME                         READY   AGE
letsencrypt-http01           True    17s
letsencrypt-staging-http01   True    3m12s
```

复制

有了 `Issuer/ClusterIssuer` 证书颁发机构，接下来我们就可以生成免费证书了，cert-manager 给我们提供了 `Certificate` 这个用于生成证书的自定义资源对象，不过这个对象需要在一个具体的命名空间下使用，证书最终会在这个命名空间下以 Secret 的资源[对象存储](https://cloud.tencent.com/product/cos?from=10680)。我们这里是要结合 ingress-nginx 一起使用，实际上我们只需要修改 Ingress 对象，添加上 cert-manager 的相关注解即可，不需要手动创建 Certificate 对象了，修改上面的 todo 应用的 Ingress 资源对象，如下所示：

```javascript
➜ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-staging-http01"  # 使用哪个issuer
spec:
  tls:
  - hosts:
    - todo.qikqiak.com     # TLS 域名
    secretName: todo-tls   # 用于存储证书的 Secret 对象名字 
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - path: /
        backend:
          serviceName: todo
          servicePort: 3000
EOF
```

复制

更新了上面的 Ingress 对象后，正常就会自动为我们创建一个 Certificate 对象：

```javascript
➜ kubectl get certificate                              
NAME       READY   SECRET     AGE
todo-tls   False   todo-tls   34s
```

复制

在校验过程中会自动创建一个 Ingress 对象用于 ACME 服务端访问：

```javascript
➜ kubectl get ingress     
NAME                        CLASS    HOSTS                ADDRESS        PORTS     AGE
cm-acme-http-solver-tgwlb   <none>   todo.qikqiak.com     10.151.30.11   80        25s
my-nginx                    <none>   ngdemo.qikqiak.com   10.151.30.11   80        23h
todo                        <none>   todo.qikqiak.com     10.151.30.11   80, 443   33s
```

复制

校验成功后会将证书保存到 todo-tls 的 Secret 对象中：

```javascript
➜ kubectl get certificate                         
NAME       READY   SECRET     AGE
todo-tls   True    todo-tls   21m
➜ kubectl get secret                                                 
NAME                          TYPE                                  DATA   AGE
default-token-hpd7s           kubernetes.io/service-account-token   3      55d
todo-tls                      kubernetes.io/tls                     2      20m
➜  kubectl describe certificate todo-tls 
Name:         todo-tls
Namespace:    default
......
Events:
  Type    Reason     Age   From          Message
  ----    ------     ----  ----          -------
  Normal  Issuing    22m   cert-manager  Issuing certificate as Secret does not exist
  Normal  Generated  22m   cert-manager  Stored new private key in temporary Secret resource "todo-tls-tr4pq"
  Normal  Requested  22m   cert-manager  Created new CertificateRequest resource "todo-tls-2gchg"
  Normal  Issuing    21m   cert-manager  The certificate has been successfully issued
```

复制

证书自动获取成功后，现在就可以讲 ClusterIssuer 替换成生产环境的了：

```javascript
➜ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: todo
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-http01"  # 使用生产环境的issuer
spec:
  tls:
  - hosts:
    - todo.qikqiak.com     # TLS 域名
    secretName: todo-tls   # 用于存储证书的 Secret 对象名字 
  rules:
  - host: todo.qikqiak.com
    http:
      paths:
      - path: /
        backend:
          serviceName: todo
          servicePort: 3000
EOF
➜ kubectl get certificate                         
NAME       READY   SECRET     AGE
todo-tls   True    todo-tls   25m
```

复制

校验成功后就可以自动获取真正的 HTTPS 证书了，现在在浏览器中访问 https://todo.qikqiak.com 就可以看到证书是有效的了。

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/qj11t1j4r7.png?imageView2/2/w/1620)

DNS-01 校验

`DNS-01` 的校验是通过 DNS 提供商的 API 拿到你的 DNS 控制权限， 在 `Let's Encrypt` 为 cert-manager 提供 TOKEN 后，cert-manager 将创建从该 TOKEN 和你的帐户密钥派生的 `TXT` 记录，并将该记录放在 `_acme-challenge.<YOUR_DOMAIN>`。然后 `Let's Encrypt` 将向 DNS 系统查询该记录，如果找到匹配项，就可以颁发证书，这种方法是支持泛域名证书的。(不知道是不是公众帐号 bug 了，后面内容添加不上了，提示内容太多

![img](https://ask.qcloudimg.com/http-save/yehe-1487868/016byl4s5t.png?imageView2/2/w/1620)

)

## **参考链接**

- https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
- https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/ingress-canary/
- https://cert-manager.io/docs/tutorials/acme/ingress/

本文分享自微信公众号 - k8s技术圈（kube100），作者：阳明

原文出处及转载信息见文内详细说明，如有侵权，请联系 yunjia_community@tencent.com 删除。

原始发表时间：2020-12-13

本文参与[腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。



[真一文搞定 ingress-nginx 的使用 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1761376)