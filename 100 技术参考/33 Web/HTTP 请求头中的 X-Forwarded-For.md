# HTTP 请求头中的 X-Forwarded-For

[大富帅](https://www.jianshu.com/u/8a717ed3921d)

2018.05.22 11:40:47字数 1,848阅读 137,863

> 原创文章出自公众号：「码农富哥」，如需转载请请注明出处！
> 文章如果对你有收获，可以收藏转发，这会给我一个大大鼓励哟！另外可以关注我公众号**「码农富哥」** (搜索id：coder2025)，我会持续输出Python，算法，计算机基础的 **原创** 文章

#### X-Forwarded-For和相关几个头部的理解

- **$remote_addr**
  是nginx与客户端进行TCP连接过程中，获得的客户端真实地址. Remote Address 无法伪造，因为建立 TCP 连接需要三次握手，如果伪造了源 IP，无法建立 TCP 连接，更不会有后面的 HTTP 请求
- **X-Real-IP**
  是一个自定义头。X-Real-Ip 通常被 HTTP 代理用来表示与它产生 TCP 连接的设备 IP，这个设备可能是其他代理，也可能是真正的请求端。需要注意的是，X-Real-Ip 目前并不属于任何标准，代理和 Web 应用之间可以约定用任何自定义头来传递这个信息
- **X-Forwarded-For**
  X-Forwarded-For 是一个扩展头。HTTP/1.1（RFC 2616）协议并没有对它的定义，它最开始是由 Squid 这个缓存代理软件引入，用来表示 HTTP 请求端真实 IP，现在已经成为事实上的标准，被各大 HTTP 代理、负载均衡等转发服务广泛使用，并被写入 RFC 7239（Forwarded HTTP Extension）标准之中.

X-Forwarded-For请求头格式非常简单，就这样：



```css
  X-Forwarded-For:client, proxy1, proxy2
```

可以看到，XFF 的内容由「英文逗号 + 空格」隔开的多个部分组成，最开始的是离服务端最远的设备 IP，然后是每一级代理设备的 IP。

如果一个 HTTP 请求到达服务器之前，经过了三个代理 Proxy1、Proxy2、Proxy3，IP 分别为 IP1、IP2、IP3，用户真实 IP 为 IP0，那么按照 XFF 标准，服务端最终会收到以下信息：



```undefined
X-Forwarded-For: IP0, IP1, IP2
```

Proxy3 直连服务器，它会给 XFF 追加 IP2，表示它是在帮 Proxy2 转发请求。列表中并没有 IP3，IP3 可以在服务端通过 ![remote_address 字段获得。我们知道 HTTP 连接基于 TCP 连接，HTTP 协议中没有 IP 的概念，](https://math.jianshu.com/math?formula=remote_address%20%E5%AD%97%E6%AE%B5%E8%8E%B7%E5%BE%97%E3%80%82%E6%88%91%E4%BB%AC%E7%9F%A5%E9%81%93%20HTTP%20%E8%BF%9E%E6%8E%A5%E5%9F%BA%E4%BA%8E%20TCP%20%E8%BF%9E%E6%8E%A5%EF%BC%8CHTTP%20%E5%8D%8F%E8%AE%AE%E4%B8%AD%E6%B2%A1%E6%9C%89%20IP%20%E7%9A%84%E6%A6%82%E5%BF%B5%EF%BC%8C)remote_address 来自 TCP 连接，表示与服务端建立 TCP 连接的设备 IP，在这个例子里就是 IP3。

详细分析一下，这样的结果是经过这样的流程而形成的：

1. 用户IP0---> 代理Proxy1（IP1），Proxy1记录用户IP0，并将请求转发个Proxy2时，带上一个Http Header
   `X-Forwarded-For: IP0`
2. Proxy2收到请求后读取到请求有 `X-Forwarded-For: IP0`，然后proxy2 继续把链接上来的proxy1 ip**追加**到 X-Forwarded-For 上面，构造出`X-Forwarded-For: IP0, IP1`，继续转发请求给Proxy 3
3. 同理，Proxy3 按照第二部构造出 `X-Forwarded-For: IP0, IP1, IP2`,转发给真正的服务器，比如NGINX，nginx收到了http请求，里面就是 `X-Forwarded-For: IP0, IP1, IP2` 这样的结果。所以Proxy 3 的IP3，不会出现在这里。
4. nginx 获取proxy3的IP 能通过![remote_address获取到，因为这个](https://math.jianshu.com/math?formula=remote_address%E8%8E%B7%E5%8F%96%E5%88%B0%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%BF%99%E4%B8%AA)remote_address就是真正建立TCP链接的IP，这个不能伪造，是直接产生链接的IP。**$remote_address 无法伪造，因为建立 TCP 连接需要三次握手，如果伪造了源 IP，无法建立 TCP 连接，更不会有后面的 HTTP 请求。**

#### x-forwarded-for 实践研究：

1. uwsgi_pass的情况下，nginx 没有设置proxy_pass x-forwarded-for: $proxy_add_x_forwarded_for;
   如果请求头传了XFF，在flask里面能正常读取请求头里面的XFF,就是当是一个普通的头读出；如果header不传这个XFF的话，就读不到
2. proxy_pass 情况下

- 没有传 # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for 的话，跟上面的uwsgi_pass 一样，都是在没有设置header XFF情况下，读不到。
- 如果传了 proxy_set_header X-Forwarded-For ![proxy_add_x_forwarded_for， header 不传xff 的话，也是可以在程序里面读到Xff 头： X-Forwarded-For: 10.0.2.2 （这个IP就是真正连上nginx 的IP， 也就是](https://math.jianshu.com/math?formula=proxy_add_x_forwarded_for%EF%BC%8C%20header%20%E4%B8%8D%E4%BC%A0xff%20%E7%9A%84%E8%AF%9D%EF%BC%8C%E4%B9%9F%E6%98%AF%E5%8F%AF%E4%BB%A5%E5%9C%A8%E7%A8%8B%E5%BA%8F%E9%87%8C%E9%9D%A2%E8%AF%BB%E5%88%B0Xff%20%E5%A4%B4%EF%BC%9A%20X-Forwarded-For%3A%2010.0.2.2%20%EF%BC%88%E8%BF%99%E4%B8%AAIP%E5%B0%B1%E6%98%AF%E7%9C%9F%E6%AD%A3%E8%BF%9E%E4%B8%8Anginx%20%E7%9A%84IP%EF%BC%8C%20%E4%B9%9F%E5%B0%B1%E6%98%AF)remote_address），因为这句proxy_set_header 会让nginx追加一个$remote_address到XFF。
- header 传xff的话， 程序里面可以读到Xff 头： X-Forwarded-For: 188.103.19.120, 10.0.2.2 （第一个是我自己编的，第二个是![remote_address），nginx还是会因为proxy_set_header X-Forwarded-For](https://math.jianshu.com/math?formula=remote_address%EF%BC%89%EF%BC%8Cnginx%E8%BF%98%E6%98%AF%E4%BC%9A%E5%9B%A0%E4%B8%BAproxy_set_header%20X-Forwarded-For)proxy_add_x_forwarded_for 这句而追加$remote_addr到XFF。

总结：

1. 只要nginx前端（例如lvs， varnish）转发请求给nginx的时候，带了x-forwarded-for ,那么程序就一定能读到这个字段，如果nginx还设置了proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for， 那么程序能读到XFF是：ip0, ip1 (客户端Ip，lvs或者varnishIP)。 如果nginx没有设置，那么nginx还是会原样把http头传给程序，也就是说程序也能读到XFF，而且XFF就是ip0 客户端IP。
2. proxy_pass 设置这个头 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 是站在一个作为代理的角度把。能继续传输多级代理的头。
3. nginx的日志格式写了$http_x_forwared_for 说明前端（lvs）确实传了这个头过来。所以是程序是读取到的
4. uwsgi_pass 不能设置 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 这个头，是因为这个头是对http代理来说，用来传递IP的，uwsgi 不可能充当一个代理。
5. nginx->程序，这里其实有两个链接过程，其他IP与nginx的TCP链接， nginx与程序的TCP链接。所以$remote_addr都是对各自来说的。
   程序的remote_addr: remote_addr 127.0.0.1 (跟它链接的是nginx 内网127.0.0.1)
   nginx的remote_addr : X-Real-Ip: 10.0.2.2 （跟它链接的是我的电脑，IP 10.0.2.2）

1. 对程序来说，读取的request.remote_addr 也永远是直接跟他链接的ip， 也就是反向代理nginx
2. The access_route attribute uses the [X-Forwarded-For](https://links.jianshu.com/go?to=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FX-Forwarded-For)[ ](https://links.jianshu.com/go?to=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FX-Forwarded-For)[header](https://links.jianshu.com/go?to=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FX-Forwarded-For), falling back to the REMOTE_ADDRWSGI variable; 也就是说access_route默认读取XFF头，如果没有，降级读取WSGI的REMOTE_ADDR变量,这个 WSGI的REMOTE_ADDR变量 就是 $remote_addr
3. request.envron 是WSGI的变量，都是wsgi server转过来的，普通的头都是加了HTTP_前缀的 ，包括proxy_set_header Host ![host:8000; proxy_set_header X-Forwarded-For](https://math.jianshu.com/math?formula=host%3A8000%3B%20proxy_set_header%20X-Forwarded-For)proxy_add_x_forwarded_for;
   添加的头都会出现在处理，因为他们就是普通的http头
4. LVS->nginx的情况下， 请求的时候主动加XFF，程序读取的时候没显示。因为LVS设置XFF的时候，直接把直连的IP赋值给LVS，忽略掉所有本来有的XFF，要从LVS这里开始。 所以程序读到的XFF是 ：XFF headers 218.107.55.254, 10.120.214.252
   前面的是我的IP， 后面的是LVS的IP



```json
{
  "wsgi.multiprocess": "False",
  "SERVER_SOFTWARE": "Werkzeug/0.11.10",
  "SCRIPT_NAME": "",
  "REQUEST_METHOD": "GET",
  "PATH_INFO": "/api/get_agreement_url/",
  "SERVER_PROTOCOL": "HTTP/1.0",
  "QUERY_STRING": "",
  "werkzeug.server.shutdown": "<function shutdown_server at 0x7f4a2f4e5488>",
  "CONTENT_LENGTH": "",
  "SERVER_NAME": "127.0.0.1",
  "REMOTE_PORT": 58284,
  "werkzeug.request": "",
  "wsgi.url_scheme": "http",
  "SERVER_PORT": "6000",
  "HTTP_POSTMAN_TOKEN": "666cfd97-585b-c342-f0bd-5c785dfff27d",
  "wsgi.input": "",
  "wsgi.multithread": "False",
  "HTTP_CACHE_CONTROL": "no-cache",
  "HTTP_ACCEPT": "*/*",
  "wsgi.version": "(1, 0)",
  "wsgi.run_once": "False",
  "wsgi.errors": "",
  "CONTENT_TYPE": "",
  "REMOTE_ADDR": "127.0.0.1",

  "HTTP_CONNECTION": "close",
  "HTTP_USER_AGENT": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36",
  "HTTP_ACCEPT_LANGUAGE": "zh-CN,zh;q=0.8,en;q=0.6",
  "HTTP_X_FORWARDED_FOR": "10.0.2.2",
  "HTTP_ACCEPT_ENCODING": "gzip, deflate, sdch",
  "HTTP_HOST": "[test.mumu.nie.netease.com:8000](http://test.mumu.nie.netease.com:8000/)",

}
```

![proxy_add_x_forwarded_for; nginx的这个变量含义就是，每次都追加](https://math.jianshu.com/math?formula=proxy_add_x_forwarded_for%3B%20nginx%E7%9A%84%E8%BF%99%E4%B8%AA%E5%8F%98%E9%87%8F%E5%90%AB%E4%B9%89%E5%B0%B1%E6%98%AF%EF%BC%8C%E6%AF%8F%E6%AC%A1%E9%83%BD%E8%BF%BD%E5%8A%A0)remote_address 到 xff头，如果xff头不存在，那么xff就被设置成跟$remote_address 一样了。如果本来就存在，就追加了 ip1, ip2这样的形式

问题：

1. 为什么lvs-nginx ， nginx的日志记录$http_x_forwarded_for还是可以的？

### 最后

> 原创文章出自公众号：「码农富哥」，如需转载请请注明出处！
> 文章如果对你有收获，可以收藏转发，这会给我一个大大鼓励哟！另外可以关注我公众号**「码农富哥」** (搜索id：coder2025)，我会持续输出Python，算法，计算机基础的 **原创** 文章