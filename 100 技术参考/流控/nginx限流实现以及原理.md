# nginx限流实现以及原理

[第9号当铺](https://www.jianshu.com/u/6c2e5ec28bc7)

2022.03.23 11:21:01字数 120阅读 321

1、限流两个算法漏桶(Leaky Bucket)算法和令牌桶算法(Token Bucket)。
2、nginx限流有两种思路

> 1、控制速率
> 2、控制并发数

3、nginx接入层限流可以使用Nginx自带的两个模块：

- 漏桶算法实现的请求限流模块ngx_http_limit_req_module控制速率。

> limit_req_zone $binary_remote_addr zone=contentRateLimit:10m rate=10r/s;
> limit_req zone=contentRateLimit burst=50 nodelay;

- 连接数限流模块ngx_http_limit_conn_module控制并发数。
   limit_conn_zone $binary_remote_addr zone=one:10m; 
  limit_conn one 10; 