# Nginx 反代：$X-Real-Ip和$X-Forwarded-For的区别

[Cyon](https://www.jianshu.com/u/46a8fafb626f)

2017.09.20 15:17:46字数 582阅读 4,800

标签（空格分隔）： nignx 负载均衡 client-ip

------

#### 1.如果只有一层代理，这两个头的值就是一样的

#### 2.多层代理

> - X-Forwarded-For: header包含这样一行

```
X-Forwarded-For: 1.1.1.1, 2.2.2.2, 3.3.3.3
```

> - X-Real-Ip:没有相关标准，上面的例子，如果配置了X-Read-IP，可能会有两种情况

// 最后一跳是正向代理，可能会保留真实客户端IP

```
X-Real-IP: 1.1.1.1
```

// 最后一跳是反向代理，比如Nginx，一般会是与之直接连接的客户端IP

```
X-Real-IP: 3.3.3.3
```

#### 3.CDN情况下：

> - 如果从CDN过来的请求没有设置X-Forwarded-For头（通常这种事情不会发生），而到了我们这里Nginx设置将其设置为$proxy_add_x_forwarded_for的话，X-Forwarded-For的信息应该为CDN的IP，因为相对于Nginx负载均衡来说客户端即为CDN，这样的话，后端的web程序时死活也获得不了真实用户的IP的。

> - CDN设置了X-Forwarded-For，我们这里又设置了一次，且值为$proxy_add_x_forwarded_for的话，那么X-Forwarded-For的内容变成 ”客户端IP,Nginx负载均衡服务器IP“如果是这种情况的话，那后端的程序通过X-Forwarded-For获得客户端IP，则取逗号分隔的第一项即可。

### 4.总结：

1. 他在正向(如squid)反向(如nginx)代理中都是标准用法，而正向代理中是没有x-real-ip相关的标准的，也就是说，如果用户访问你的 nginx反向代理之前，还经过了一层正向代理，你即使在nginx中配置了x-real-ip，取到的也只是正向代理的IP而不是客户端真实IP
2. 大部分nginx反向代理配置文章中都没有推荐加上x-real-ip ，而只有x-forwarded-for，因此更通用的做法自然是取x-forwarded-for
3. 多级代理很少见，只有一级代理的情况下二者是等效的
4. 如果有多级代理，x-forwarded-for效果是大于x-real-ip的，可以记录完整的代理链路