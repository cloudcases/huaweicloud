# Ingress-nginx工作原理和实践

[![精益码农](https://pic3.zhimg.com/v2-7ac85c3b05b69296ab95ccb67253cbc5_xs.jpg?source=172ae18b)](https://www.zhihu.com/people/no1coder)

[精益码农](https://www.zhihu.com/people/no1coder)[](https://www.zhihu.com/question/48510028)

同程艺龙成都 后端开发工程师

2 人赞同了该文章

本文记录/分享 目前项目的 K8s 部署结构和请求追踪改造方案

![img](https://pic4.zhimg.com/80/v2-495d3a805a3805948bea1152f0ad6e9b_1440w.jpg)



这个图算是一个通用的前后端分离的 k8s 部署结构:
Nginx Ingress 负责暴露服务(nginx前端静态资源服务)， 根据十二要素应用的原 则，将后端 api 作为 nginx 服务的附加动态资源。

## **Ingress vs Ingress-nginx**

Ingress 是一种向 k8s 集群外部的客户端公开服务的方法， **Ingress 在网络协议栈的应用层工作**,
根据请求的主机名 host 和路径 path 决定请求转发到的服务。

![img](https://pic2.zhimg.com/80/v2-433ad7fba7ba38345f560ff4ee4a2465_1440w.jpg)



在应用 Ingress对象提供的功能之前，必须强调集群中存在 Ingress Controller， Ingress 资源才能正常工作。

我这里的 web 项目使用的是常见的 Ingress-nginx (官方还有其他用途的 Ingress)，Ingress-nginx 是使用 nginx 作为反向代理和负载均衡器的 K8s Ingress 控制器， 作为 Pod 运行在`kube-system` 命名空间。

了解 Ingress 工作原理，有利于我们如何与运维人员打交道。

![img](https://pic2.zhimg.com/80/v2-d99c13076f5e13881b6e68c7c2efb381_1440w.jpg)

下面通过 Ingress-nginx 暴露 Kibana 服务：

```text
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kibana
  labels:
    app: kibana
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"
    nginx.ingress.kubernetes.io/proxy-body-size: "8m"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
    - hosts:
      - 'https://logging.internal.gridsum.com/'
      secretName: tls-cert
  rules:
    - host: 'https://logging.internal.gridsum.com'
      http:
        paths:
          - path: /
            backend:
              serviceName: kibana
              servicePort: 5601
```

Ingress-nginx 中最让我困惑的是它的`Paths分流`与`rewrite-target`注解。

- Paths 分流 一般用于 根据特定的 Path，将请求转发到特定的后端服务 Pod，后端服务 Pod 能接收到 Path 这个信息。 一般后端服务是作为 api。
- rewrite-target 将请求重定向到后端服务， 那有什么用处呢？

答： 以上面暴露的 kibana 为例， 我们已经可以在`https://logging.internal.gridsum.com/` 访问完整的 Kibana， 如果我想利用这个域名暴露 ElasticSearch 站点，怎么操作？ 这时就可以利用`rewrite-target`，

```text
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: elasticsearch
  labels:
    app: kibana
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"
    nginx.ingress.kubernetes.io/proxy-body-size: "8m"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: "/$2"
spec:
  tls:
    - hosts:
      - 'logging.internal.gridsum.com'
      secretName: tls-cert
  rules:
    - host: 'logging.internal.gridsum.com'
      http:
        paths:
          - path: /es(/|$)(.*)
            backend:
              serviceName: elasticsearch
              servicePort: 9200
```

在此 Ingress 定义中，由`(.*)`捕获的所有字符都将分配给占位符$2，然后将其用作重写目标注解中的参数。 这样的话：`https://logging.internal.gridsum.com/es` 将会重定向到后端 elasticsearch 站点，并且忽略了 es 这个 path

![img](https://pic1.zhimg.com/80/v2-15f93393aeb28886c5f93ad5d82f4df0_1440w.jpg)

## **Ingress-nginx 到 webapp 的日志追踪**

熟悉我的朋友知道， 我写了《**[一套标准的ASP.NET Core容器化应用日志收集分析方案](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/JulianHuang/p/14049455.html)**》，这里面主要是 BackEnd App 的日志，从我上面的结构图看，

Ingress-nginx----> Nginx FrontEnd App--->BackEnd App 需要一个串联的追踪 Id， 便于观察运维网络和业务应用。

幸好 Ingress-nginx， Nginx 强大的配置能力帮助我们做了很多事情：

- 客户端请求到达 Ingress-Nginx Controllerr，Ingress-Nginx Controller 会自动添加一个`X-Request-ID`的请求 Header, 随机值---- 这个配置是默认的
- 请求达到 Nginx FrontEnd App， Nginx 有默认配置`proxy_pass_request_headers on;`， 自动将请求头都传递到上游的 Backend App

这样跨越整个结构图的 request_id 思路已经清楚了，最后一步只需要我们在 Backend App 中提取请求中携带的`X-Request-ID`， 并作为日志的关键输出字段。

这就涉及到怎么从**自定义日志的 LayoutRender**。

下面为 NLog 自定义名为`x_request_id`的 Render，该 Render 从请求的 X-Request-ID 标头中提取值。

① 定义 NLog Render

```text
    /// <summary>
    /// Represent a unique identifier to represent a request from the request HTTP header X-Request-Id.
    /// </summary>
    [LayoutRenderer("x_request_id")]
    public class XRequestIdLayoutRender : HttpContextLayoutRendererBase
    {
        protected override void Append(StringBuilder builder, LogEventInfo logEvent)
        {
            var identityName = HttpContextAccessor.HttpContext?.Request?.Headers?["X-Request-Id"].FirstOrDefault();
            builder.Append(identityName);
        }
    }

    /// <summary>
    /// Represent a http context layout renderer to access the current http context.
    /// </summary>
    public abstract class HttpContextLayoutRendererBase : LayoutRenderer
    {
        private IHttpContextAccessor _httpContextAccessor;

        /// <summary>
        /// Gets the <see cref="IHttpContextAccessor"/>.
        /// </summary>
        protected IHttpContextAccessor HttpContextAccessor { get { return _httpContextAccessor ?? (_httpContextAccessor = ServiceLocator.ServiceProvider.GetService<IHttpContextAccessor>()); } }
    }

    internal sealed class ServiceLocator
    {
        public static IServiceProvider ServiceProvider { get; set; }
    }
```

② 从请求中获取 X-Request-Id 依赖 IHttpContextAccessor 组件
这里使用 依赖查找的方式获取该组件， 故请在 Startup ConfigureService 中生成服务

```text
 public void ConfigureServices(IServiceCollection services)
 {
     // ......
     ServiceLocator.ServiceProvider = services.BuildServiceProvider();
 }
```

③ 最后在 Program 中注册这个 NLog Render：

```text
 public static void Main(string[] args)
{
     LayoutRenderer.Register<XRequestIdLayoutRender>("x_request_id");
     CreateHostBuilder(args).Build().Run();
}
```

这样从 Ingress-Nginx 产生的`request_id`，将会流转到 Backend App， 并在日志分析中起到巨大作用，也便于划清运维/开发的故障责任。

- [https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#generate-request-id](https://link.zhihu.com/?target=https%3A//kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/%23generate-request-id)
- [http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_headers](https://link.zhihu.com/?target=http%3A//nginx.org/en/docs/http/ngx_http_proxy_module.html%23proxy_pass_request_headers)

### **总结**

1. 了解了Ingress在应用层工作，根据Host和Path暴露k8s服务
2. 本文梳理了Ingress和常见的Ingress-nginx的关系
3. 对于应用了Ingress的应用，梳理了从Ingress-Nginx到WebApp的日志追踪id， 便于排查网络/业务故障

发布于 2021-03-18 10:53



[Ingress-nginx工作原理和实践 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/358002081)