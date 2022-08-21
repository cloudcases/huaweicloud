# 代理协议Proxy protocol 

2018-05-24 17:30

**前言**

Nginx获取用户真实IP是个常见的需求，基本上现在所有HTTP的项目都会使用Nginx做反向代理、负载均衡等。

但是在某些复杂的网络及架构下，Nginx获取用户IP就不是那么容易了。

一般在做7层代理时，我们会通过添加XFF头来传递用户真实IP，这还比较方便。

但是如果Nginx前面还有一个做4层代理的HAProxy时，获取用户真实IP就不是那么容易了。

这时用代理协议会特别的爽。

**一、代理协议**

**代理协议(Proxy protocol)**，是HAProxy的作者Willy Tarreau于2010年开发和设计的一个Internet协议，**通过为tcp添加一个很小的头信息，来方便的传递客户端信息（协议栈、源IP、目的IP、源端口、目的端口等)，在网络情况复杂又需要获取用户真实IP时非常有用**。

代理协议分为V1和V2两个版本，V1是人类易读的，V2是二进制格式的。

由于**Nginx目前只支持V1版本**，所以这里只先简单介绍下V1版本的格式。

Proxy protocol V1的格式如下:

PROXY 协议栈 源IP 目的IP 源端口 目的端口rn

例如：

PROXY TCP4 213.103.23.88 10.0.0.2 49863 8080rn

**二、配置示例**

下面介绍一个简单场景下的配置示例：

HAProxy(4层tcp代理) --> Nginx(7层http代理)

HAProxy配置示例

HAProxy作为sender时配置非常简单，在server段添加 send-proxy，例如：

server lb1 10.0.0.2:8080 maxconn 10000 check send-proxy

server lb1 10.0.0.2:8443 maxconn 10000 check send-proxy

Nginx配置示例

Nginx作为reciver时配置也非常简单，在listen段添加 proxy_protocol，例如：

listen 8080 proxy_protocol;

listen 8443 ssl proxy_protocol;

Nginx启用proxy_protocol后，可通过real_ip模块从proxy_protocol头中获取用户真实IP并替换remote_addr，配置如下：

set_real_ip_from 0.0.0.0/16;

real_ip_header proxy_protocol;

**三、坑在最后**

要使用Proxy protocol需要两个角色sender和receiver,sender在与receiver之间建立连接后，会先发送一个带有客户信息的tcp header,因为更改了tcp协议，需receiver也支持Proxy protocol，否则不能识别tcp包头，导致无法成功建立连接。

而对于Nginx来说listen段中的proxy_protocol配置是针对监听端口生效的，所以虽然是在server的listen段中配置，实际上对于端口来说算是全局配置。

所以对于同一个Nginx上如果有部分server需要代理协议，部分server不需要代理协议。他们所监听的端口一定要分开，例如：

listen 80;

listen 8080 proxy_protocol;