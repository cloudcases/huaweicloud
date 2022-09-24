# nginx限流方案的实现(三种方式)

时间:2022-03-29

本文章向大家介绍nginx限流方案的实现(三种方式)，主要包括nginx限流方案的实现(三种方式)使用实例、应用技巧、基本知识点总结和需要注意事项，具有一定的参考价值，需要的朋友可以参考一下。

通过查看nginx官方文档，小弟查看到了三种nginx限流方式。

1、limit_conn_zone

2、limit_req_zone

3、ngx_http_upstream_module

前两种只能对客户端（即单一ip限流），并且文档也很全，但是经过测试发现，还是无法达到官方文档所说的结果（可能小弟的测试方法有问题）。

这里先简单的介绍一下前两种：

### **1、limit_conn_zone**

#### 1.1nginx配置

```powershell
http{ 
 limit_conn_zone $binary_remote_addr zone=one:10m; 
 server 
 { 
   ...... 
  limit_conn one 10; 
  ...... 
 } 
} 
```

其中“limit_conn one 10”既可以放在server层对整个server有效，也可以放在location中只对单独的location有效。
该配置表明：客户端的并发连接数只能是10个。

#### **1.2结果**

```powershell
ab工具20并发去请求nginx，可以看到
Complete requests: 20
Failed requests: 9
```

（由于nginx配置中一个ip并发连接数为10，而结果中成功数为+1的原因未知；nginx的日志中也可以看到有9个请求返回503）

### **2、limit_req_zone**

#### 2.1 nginx配置

```powershell
http{ 
 limit_req_zone $binary_remote_addr zone=req_one:10m rate=1r/s; 
 server 
 { 
   ...... 
  limit_req zone=req_one burst=120; 
  ...... 
 } 
} 
```

其中“limit_req zone=req_one burst=120”既可以放在server层对整个server有效，也可以放在location中只对单独的location有效。

rate=1r/s的意思是每个地址每秒只能请求一次，也就是说令牌桶burst=120一共有120块令牌，并且每秒钟只新增1块令牌，120块令牌发完后，多出来的请求就会返回503.。

### **3、ngx_http_upstream_module**

#### **3.1 介绍**

作为优秀的负载均衡模块，目前是我工作中用到最多的。其实，该模块是提供了我们需要的后端限流功能的。

通过官方文档介绍，该模块有一个参数：max_conns可以对服务端进行限流，可惜在商业版nginx中才能使用。然而，在nginx1.11.5版本以后，官方已经将该参数从商业版中脱离出来了，也就是说只要我们将生产上广泛使用的nginx1.9.12版本和1.10版本升级即可使用（通过测试可以看到，在旧版本的nginx中，如果加上该参数，nginx服务是无法启动的）。

#### **3.2配置**

```powershell
upstream xxxx{ 
 server 127.0.0.1:8080 max_conns=10; 
 server 127.0.0.1:8081 max_conns=10; 
} 
```

#### 3.3结果（不便截图）

用两台机器各自用ab工具向nginx发送20、30、40个并发请求：

可以看到无论并发多少，成功的请求只有12个，成功的次数会多个2个，同时1.2的测试结果中成功次数也是+1，这里是两台机器，基于此种考虑，将机器增加至三台，果然成功的次数为13个。这里得出一个假想，成功的请求数会根据客户端的+1而+1（这里只是假设）

注：还有很重要的几点。

max_conns是针对upstream中的单台server的，不是所有；

nginx有个参数：worker_processes，max_conns是针对每个worker_processes的；



附ab工具安装步骤（转载，来源未知）

```powershell
#ab运行需要依赖apr-util包，安装命令为： 
yum install apr-util 
#安装依赖 yum-utils中的yumdownload 工具，如果没有找到 yumdownload 命令可以 
yum install yum-utils 
cd /opt
mkdir abtmp 
cd abtmp 
yum install yum-utils.noarch 
yumdownloader httpd-tools* 
rpm2cpio httpd-*.rpm | cpio -idmv 
#操作完成后 将会产生一个 usr 目录 ab文件就在这个usr 目录中 
#简单使用说明 
./ab -c 100 -n 10000 http://127.0.0.1/index.html 
#-c 100 即：每次并发100个 
#-n 10000 即： 共发送10000个请求 
```



原文地址：https://www.cnblogs.com/aping-blog/p/16071549.html