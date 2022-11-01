# Nginx如何限流

AiJinJin [对线JAVA面试](javascript:void(0);) *2022-05-27 00:00* *发表于广东*

流量限制(rate-limiting)，是 Nginx 中一个非常实用，却经常被错误理解和错误配置的功能。我们可以用来限制用户在给定时间内 HTTP 请求的数量。请求，可以是一个简单网站首页的 GET 请求，也可以是登录表单的 POST 请求。

流量限制可以用作安全目的，比如可以减慢暴力密码破解的速率。通过将传入请求的速率限制为真实用户的典型值，并标识目标URL地址(通过日志)，还可以用来抵御 DDOS 攻击。更常见的情况，该功能被用来保护上游应用服务器不被同时太多用户请求所压垮。

本篇文章将会介绍 Nginx 的流量限制 的基础知识和高级配置，”流量限制”在 Nginx Plus 中也适用。

### Nginx如何限流

Nginx 的”流量限制”使用漏桶算法(leaky bucket algorithm)，该算法在通讯和分组交换计算机网络中广泛使用，用以处理带宽有限时的突发情况。

就好比，一个桶口在倒水，桶底在漏水的水桶。如果桶口倒水的速率大于桶底的漏水速率，桶里面的水将会溢出；同样，在请求处理方面，水代表来自客户端的请求，水桶代表根据”先进先出调度算法”(FIFO)等待被处理的请求队列，桶底漏出的水代表离开缓冲区被服务器处理的请求，桶口溢出的水代表被丢弃和不被处理的请求。

### 配置基本的限流 

“流量限制”配置两个主要的指令，`limit_req_zone`和`limit_req`，如下所示：

```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
server {
    location /login/ {
        limit_req zone=mylimit;

        proxy_pass http://my_upstream;
    }
}
```

`limit_req_zone`指令定义了流量限制相关的参数，而`limit_req`指令在出现的上下文中启用流量限制(示例中，对于”/login/”的所有请求)。`limit_req_zone`指令通常在 HTTP 块中定义，使其可在多个上下文中使用，它需要以下三个参数：

- Key - 定义应用限制的请求特性。示例中的 Nginx 变量`remote_addr`，占用更少的空间)
- Zone - 定义用于存储每个 IP 地址状态以及被限制请求 URL 访问频率的共享内存区域。保存在内存共享区域的信息，意味着可以在 Nginx 的 worker 进程之间共享。定义分为两个部分：通过`zone=keyword`标识区域的名字，以及冒号后面跟区域大小。16000 个 IP 地址的状态信息，大约需要 1MB，所以示例中区域可以存储 160000 个 IP 地址。
- Rate - 定义最大请求速率。在示例中，速率不能超过每秒 10 个请求。Nginx 实际上以毫秒的粒度来跟踪请求，所以速率限制相当于每 100 毫秒 1 个请求。因为不允许”突发情况”，这意味着在前一个请求 100 毫秒内到达的请求将被拒绝。

> ❝
>
> 当 Nginx 需要添加新条目时存储空间不足，将会删除旧条目。如果释放的空间仍不够容纳新记录，Nginx 将会返回 503 状态码(Service Temporarily Unavailable)。另外，为了防止内存被耗尽，Nginx 每次创建新条目时，最多删除两条 60 秒内未使用的条目。
>
> ❞

`limit_req_zone`指令设置流量限制和共享内存区域的参数，但实际上并不限制请求速率。所以需要通过添加`limit_re`q指令，将流量限制应用在特定的 location 或者 server 块。在上面示例中，我们对`/login/`请求进行流量限制。

![图片](https://mmbiz.qpic.cn/mmbiz_png/TLH3CicPVibrcZopLF7Qh5qm7nMCxIhjCfmPZ3KxDTeDJbI9zby5rkJEZ77RQtMOzWI27WZvg63KKcjp4UrsVibgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

现在每个 IP 地址被限制为每秒只能请求 10 次 `/login/`，更准确地说，在前一个请求的 100 毫秒内不能请求该 URL。

### 处理突发

如果我们在 100 毫秒内接收到 2 个请求，怎么办？对于第二个请求，Nginx 将给客户端返回状态码 503。这可能并不是我们想要的结果，因为**应用本质上趋向于突发性**。相反地，我们希望缓冲任何超额的请求，然后及时地处理它们。我们更新下配置，在`limit_req`中使用 burst 参数：

```
location /login/ {
    limit_req zone=mylimit burst=20;
    proxy_pass http://my_upstream;
}
```

**burst 参数定义了超出 zone 指定速率的情况下(示例中的 mylimit 区域，速率限制在每秒 10 个请求，或每 100 毫秒一个请求)，客户端还能发起多少请求**。上一个请求 100 毫秒内到达的请求将会被放入队列，我们将队列大小设置为 20。

这意味着，如果从一个给定 IP 地址发送 21 个请求，Nginx 会立即将第一个请求发送到上游服务器群，然后将余下 20 个请求放在队列中。然后每 100 毫秒转发一个排队的请求，只有当传入请求使队列中排队的请求数超过 20 时，Nginx 才会向客户端返回 503。

### 无延迟的排队

配置 burst 参数将会使通讯更流畅，但是可能会不太实用，因为该配置会使站点看起来很慢。在上面的示例中，队列中的第 20 个包需要等待 2 秒才能被转发，此时返回给客户端的响应可能不再有用。要解决这个情况，可以在 burst 参数后添加 nodelay 参数：

```
location /login/ {
    limit_req zone=mylimit burst=20 nodelay;

    proxy_pass http://my_upstream;
}
```

**使用 nodelay 参数，Nginx 仍将根据 burst 参数分配队列中的位置，并应用已配置的速率限制，而不是清理队列中等待转发的请求**。相反地，当一个请求到达“太早”时，只要在队列中能分配位置，Nginx 将立即转发这个请求。将队列中的该位置标记为”taken”(占据)，并且不会被释放以供另一个请求使用，直到一段时间后才会被释放(在这个示例中是，100 毫秒后)。

假设如前所述，队列中有 20 个空位，从给定的 IP 地址发出的 21 个请求同时到达。Ngin x会立即转发这个 21 个请求，并且标记队列中占据的 20 个位置，然后每 100 毫秒释放一个位置。如果是25个请求同时到达，Nginx 将会立即转发其中的 21 个请求，标记队列中占据的 20 个位置，并且返回 503 状态码来拒绝剩下的 4 个请求。

现在假设，第一组请求被转发后 101 毫秒，另 20 个请求同时到达。队列中只会有一个位置被释放，所以 Nginx 转发一个请求并返回503状态码来拒绝其他 19 个请求。如果在 20 个新请求到达之前已经过去了 501 毫秒，5 个位置被释放，所以 Nginx 立即转发 5 个请求并拒绝另外 15 个。

效果相当于每秒 10 个请求的“流量限制”。如果希望不限制两个请求间允许间隔的情况下实施“流量限制”，nodelay 参数是很实用的。

注意：**对于大部分部署，我们建议使用 burst 和 nodelay 参数来配置`limit_req`指令**。

### 高级配置示例

通过将基本的“流量限制”与其他 Nginx 功能配合使用，我们可以实现更**细粒度的流量限制**。

**「白名单」**

下面这个例子将展示，如何**对任何不在白名单内的请求强制执行“流量限制”**：

```
geo $limit {
    default         1;
    10.0.0.0/8         0;
    192.168.0.0/64     0;
}

map $limit $limit_key {
    0 "";
    1 $binary_remote_addr;
}

limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;

server {
    location / {
        limit_req zone=req_zone burst=10 nodelay;

        # ...
    }
}
```

这个例子同时使用了 geo 和 map 指令。

**geo 块将给在白名单中的 IP 地址对应的 $limit 变量分配一个值 0，给其它不在白名单中的分配一个值 1**。

然后我们使用一个映射将这些值转为 key，如下：

- 如果变量的值是，`limit_key`变量将被赋值为空字符串
- 如果变量的值是，`limit_key`变量将被赋值为客户端二进制形式的 IP 地址 两个指令配合使用，白名单内 IP 地址的`$limit_key`变量被赋值为空字符串，不在白名单内的被赋值为客户端的 IP 地址。当`limit_req_zone`后的第一个参数是空字符串时，不会应用“流量限制”，所以白名单内的 IP 地址(10.0.0.0/8 和192.168.0.0/24 网段内)不会被限制。其它所有 IP 地址都会被限制到每秒 5 个请求。

`limit_req`指令将限制应用到 `/` 的`location`块，允许在配置的限制上最多超过 10 个数据包的突发，并且不会延迟转发。

### location包含多limit_req指令

我们可以在一个 location 块中配置多个`limit_req`指令。**符合给定请求的所有限制都被应用时，意味着将采用最严格的那个限制**。例如，多个指令都制定了延迟，将采用最长的那个延迟。同样，请求受部分指令影响被拒绝，即使其他指令允许通过也无济于事。

扩展前面将“流量限制”应用到白名单内 IP 地址的例子：

```
http {
    # ...

    limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;
    limit_req_zone $binary_remote_addr zone=req_zone_wl:10m rate=15r/s;
    
    server {
        # ...
        location / {
            limit_req zone=req_zone burst=10 nodelay;
            limit_req zone=req_zone_wl burst=20 nodelay;
            # ...
        }
    }
}
```

白名单内的 IP 地址不会匹配到第一个“流量限制”，而是会匹配到第二个`req_zone_wl`，并且被限制到每秒 15 个请求。不在白名单内的 IP 地址两个限制能匹配到，所以应用限制更强的那个：每秒 5 个请求。

### 配置相关功能

日志记录 默认情况下，**Nginx 会在日志中记录由于流量限制而延迟或丢弃的请求**，如下所示：

```
2015/06/13 04:20:00 [error] 120315#0: *32086 limiting requests, excess: 1.000 by zone "mylimit", client: 192.168.1.2, server: nginx.com, <br>request: "GET / HTTP/1.0", host: "nginx.com"
```

日志条目中包含的字段：

- limiting requests - 表明日志条目记录的是被“流量限制”请求
- excess - 每毫秒超过对应“流量限制”配置的请求数量
- zone - 定义实施“流量限制”的区域
- client - 发起请求的客户端 IP 地址
- server - 服务器 IP 地址或主机名
- request - 客户端发起的实际 HTTP 请求
- host - HTTP 报头中 host 的值

默**认情况下，Nginx 以 error 级别来记录被拒绝的请求**，如上面示例中的`[error]`所示(Ngin 以较低级别记录延时请求，一般是 info 级别)。

如要更改 Nginx 的日志记录级别，需要使用`limit_req_log_level`指令。这里，我们将被拒绝请求的日志记录级别设置为 warn：

```
location /login/ {
    limit_req zone=mylimit burst=20 nodelay;
    limit_req_log_level warn;
    
    proxy_pass http://my_upstream;
}
```

### 发送到客户端的错误代码

一般情况下，**客户端超过配置的流量限制时，Nginx 响应状态码为 503(Service Temporarily Unavailable)**。

可以使用`limit_req_status`指令来设置为其它状态码(例如下面的 444 状态码):

```
location /login/ {
    limit_req zone=mylimit burst=20 nodelay;
    limit_req_status 444;
}
```

### 指定location拒绝所有请求

如果你想拒绝某个指定 URL 地址的所有请求，而不是仅仅对其限速，只需要在 location 块中配置 deny all 指令：

```
location /foo.php {
    deny all;
}
```

### 总结

前文已经涵盖了 Nginx 和 Nginx Plus 提供的“流量限制”的很多功能，包括为 HTTP 请求的不同 loation 设置请求速率，给“流量限制”配置 burst 和 nodelay 参数。还涵盖了针对客户端 IP 地址的白名单和黑名单应用不同“流量限制”的高级配置，阐述了如何去日志记录被拒绝和延时的请求。



https://mp.weixin.qq.com/s?__biz=MzI4NjE4NTUwNQ==&mid=2247494915&idx=2&sn=b37a2aca654a9b345a8dadd84e0d8b9b&chksm=ebe26c4ddc95e55b99c1b3b1c2d2969e33ecb49215f29716e6cd3e7f3101fb7019170fc647f7&scene=27