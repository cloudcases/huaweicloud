# 反向代理与 Real-IP 和 X-Forwarded-For

2020-06-11阅读 9980

> **开篇语：** 开涛新作**《亿级流量网站架构核心技术》**出版计划公布以来，博文视点遭受到一波又一波读者询问面世时间的DDos攻击。面对亿级流量的热情，感激之余，我们也很庆幸——这部作品质量的确过硬，不会辜负拥趸厚望，更有开涛的高度负责和体贴周到加持，让她绝对物超所值、长久流芳。下面，看一篇来自这位技术暖男的售前支持。 ——本书策划编辑  侠少

![img](https://ask.qcloudimg.com/http-save/yehe-3927631/03infe961r.jpeg?imageView2/2/w/1620)

本文作者张开涛。为保障《亿级流量网站架构核心技术》一书内容的连续性，有些需要读者了解的内容，或者书的补充和引申内容，会通过二维码嵌入的方式引导读者阅读学习。大家可以关注作者公众号“**开涛的博客**”，并从菜单栏“我的新书”中查阅相关内容。

============================

本文是「4.4 接入层限流」节中的「按照IP限制并发连接数配置示例」部分需要了解的内容。

当我们访问互联网上的服务时，大多数时，客户端并不是直接访问到服务端的，而是客户端首先请求到反向代理，反向代理再转发到服务端实现服务访问，通过反向代理实现路由/[负载均衡](https://cloud.tencent.com/product/clb?from=10680)等策略。这样一来在服务端拿到的客户端IP将是反向代理IP，而不是真实客户端IP，因此需要一种办法来获取到真实客户端IP。

如下图所示，客户端通过Nginx Proxy1 和 Nginx Proxy2 两层反向代理才访问到具体服务Nginx Backend（或如Tomcat服务）。那Nginx Backend如何才能拿到真实客户端IP呢？

![img](https://ask.qcloudimg.com/http-save/yehe-3927631/jtrb2q1tcp.jpeg?imageView2/2/w/1620)

接下来我们来看看如何才能获取到客户端真实IP。

- **场景1**

场景1是很简单的场景，Nginx Proxy直接把请求往后转发，没有做任何处理。

**Nginx Proxy**

**192.168.107.107 nginx.conf**

location /test {

​    proxy_pass http://192.168.107.112:8080;

}

**192.168.107.112 nginx.conf**

location /test {

​    proxy_pass http://192.168.107.114:8080;

}

Nginx Proxy就是简单的把请求往后转发。

**Nginx Backend**

**192.168.107.114 nginx.conf**

location /test {

​    default_type text/html;

​    charset gbk;

​    echo "

}

Nginx Backend输出客户端IP（remoteaddr）和X−Forwarded−For请求头（http_x_forwarded_for），当访问服务时输出结果如下所示：

192.168.107.112 ||

- **分析**

1. $remote_addr代表客户端IP，当前配置的输出结果为最后一个代理服务器的IP，并不是真实客户端IP；
2. 在没有特殊配置情况下，X-Forwarded-For请求头不会自动添加到请求头中，即Nginx Backend的$http_x_forwarded_for输出为空。

- **场景2**

场景2通过添加X-Real-IP和X-Forwarded-For捕获客户端真实IP。

**Nginx Proxy**

**192.168.107.107 nginx.conf**

location /test {

​    proxy_set_header X-Real-IP $remote_addr;

​    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

​    proxy_pass http://192.168.107.112:8080;

}

**192.168.107.112 nginx.conf**

location /test {

​    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

​    proxy_pass http://192.168.107.114:8080;

}

**Nginx Backend**

**192.168.107.114 nginx.conf**

location /test {

​    default_type text/html;

​    charset gbk;

​    echo "

}

当访问服务时，输出结果为：

192.168.107.112 || 192.168.162.16 || 192.168.162.16, 192.168.107.107

- **分析**

1. 在离用户最近的反向代理NginxProxy 1，通过“proxy_set_header X-Real-IP remoteaddr”把真实客户端IP写入到请求头X−Real−IP，在NginxBackend输出http_x_real_ip获取到了真实客户端IP；而Nginx Backend的“
2. “proxy_set_headerX-Forwarded-For Double subscripts: use braces to clarifyremote_addr用逗号合起来，如果请求头中没有X-Forwarded-For则Double subscripts: use braces to clarifyremote_addr。

X-Forwarded-For代表了客户端IP，反向代理如Nginx通过$proxy_add_x_forwarded_for添加此项，X-Forwarded-For的格式为X-Forwarded-For:real client ip, proxy ip 1, proxy ip N，每经过一个反向代理就在请求头X-Forwarded-For后追加反向代理IP。

到此我们可以使用请求头X-Real-IP和X-Forwarded-For来获取客户端IP及客户端到服务端经过的反向代理IP了。这种方式还是很麻烦，$remote_addr并不是真实客户端IP。

- **场景3**

为了更方便的获取真实客户端IP，可以使用nginx http_realip_module模块解决，在安装nginx时通过--with-http_realip_module安装该模块。NginxProxy配置和场景2一样。

**Nginx Backend**

**192.168.107.114 nginx.conf**

real_ip_header X-Forwarded-For; 

set_real_ip_from 192.168.0.0/16; 

real_ip_recursive on; 

location /test {

​    default_type text/html;

​    charset gbk;

​    echo "

}

当访问服务时，输出结果为：

192.168.162.16 || 192.168.162.16 || 192.168.162.16, 192.168.107.107

- **分析**

1. X-Real-IP和X-Forwarded-For和场景2一样；
2. 使用realip模块后，remoteaddr输出结果为真实客户端IP，可以使用realip_remote_addr获取最后一个反向代理的IP；
3. real_ip_headerX-Forwarded-For：告知Nginx真实客户端IP从哪个请求头获取，如X-Forwarded-For；
4. set_real_ip_from192.168.0.0/16：告知Nginx哪些是反向代理IP，即排除后剩下的就是真实客户端IP，支持配置具体IP地址、CIDR地址；
5. real_ip_recursive on：是否递归解析，当real_ip_recursive配置为off时，Nginx会把real_ip_header指定的请求头中的最后一个IP作为真实客户端IP；当real_ip_recursive配置为on时，Nginx会递归解析real_ip_header指定的请求头，最后一个不匹配set_real_ip_from的IP作为真实客户端IP。

如果配置“real_ip_recursive off; ”，则输出结果为：

192.168.107.107 || 192.168.162.16 ||192.168.162.16, 192.168.107.107

- **总结**

1. 通过“proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for” 把从真实客户端IP和反向代理IP通过逗号分隔，添加到请求头中；
2. 可以在第一个反向代理上配置“proxy_set_header X-Real-IP $remote_addr” 获取真实客户端IP；
3. 配合realip模块从X-Forwarded-For也可以获取到真实客户端IP。

在整个反向代理链条的第一个反向代理可以不配置“proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for”，否则客户端可以伪造X-Forwarded-For从而伪造客户端真实IP，如果服务端使用X-Forwarded-For第一个IP作为真实客户端IP，则就出问题了。如果通过配置X-Real-IP请求头或者配合realip模块也不会出现该问题。如果自己解析X-Forwarded-For的话，记得使用realip算法解析，而不是取第一个。

当我们进行限流时一定注意限制的是真实客户端IP，而不是反向代理IP，我遇到过很多同事在这块出问题的。

本文分享自微信公众号 - 博文视点Broadview（bvbooks），作者：张开涛

原文出处及转载信息见文内详细说明，如有侵权，请联系 yunjia_community@tencent.com 删除。

原始发表时间：2017-01-04

本文参与[腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。



[反向代理与 Real-IP 和 X-Forwarded-For - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1643277)