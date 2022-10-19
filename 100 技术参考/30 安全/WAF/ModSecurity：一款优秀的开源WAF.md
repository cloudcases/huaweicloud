# ModSecurity：一款优秀的开源WAF

[owensky ](https://www.freebuf.com/author/owensky)2019-09-10 15:00:36 1158437 17

![img](https://image.3001.net/images/20190814/1565774841_5d53d3f9c2643.png!small)

## **一、ModSecurity3.0介绍**

**ModSecurity是一个开源的跨平台Web应用程序防火墙（WAF）引擎，用于Apache，IIS和Nginx，由Trustwave的SpiderLabs开发。作为WAF产品，ModSecurity专门关注HTTP流量，当发出HTTP请求时，ModSecurity检查请求的所有部分，如果请求是恶意的，它会被阻止和记录。**

优势：

> 完美兼容nginx，是nginx官方推荐的WAF
>
> 支持OWASP规则
>
> 3.0版本比老版本更新更快，更加稳定，并且得到了nginx、Inc和Trustwave等团队的积极支持
>
> 免费

ModSecurity的功能：

> SQL Injection (SQLi)：阻止SQL注入
>
> Cross Site Scripting (XSS)：阻止跨站脚本攻击
>
> Local File Inclusion (LFI)：阻止利用本地文件包含漏洞进行攻击
>
> Remote File Inclusione(RFI)：阻止利用远程文件包含漏洞进行攻击
>
> Remote Code Execution (RCE)：阻止利用远程命令执行漏洞进行攻击
>
> PHP Code Injectiod：阻止PHP代码注入
>
> HTTP Protocol Violations：阻止违反HTTP协议的恶意访问
>
> HTTPoxy：阻止利用远程代理感染漏洞进行攻击
>
> Shellshock：阻止利用Shellshock漏洞进行攻击
>
> Session Fixation：阻止利用Session会话ID不变的漏洞进行攻击
>
> Scanner Detection：阻止黑客扫描网站
>
> Metadata/Error Leakages：阻止源代码/错误信息泄露
>
> Project Honey Pot Blacklist：蜜罐项目黑名单
>
> GeoIP Country Blocking：根据判断IP地址归属地来进行IP阻断

劣势：

> 不支持检查响应体的规则，如果配置中包含这些规则，则会被忽略，nginx的的sub_filter指令可以用来检查状语从句：重写响应数据，OWASP中相关规则是95X。
>
> 不支持OWASP核心规则集DDoS规则REQUEST-912-DOS- PROTECTION.conf,nginx本身支持配置DDoS限制
>
> 不支持在审计日志中包含请求和响应主体

## **二、安装部署**

测试环境：centOS7.6阿里云镜像

升级软件和内核

```
yum update
```

安装nginx： http://nginx.org/en/linux_packages.html#mainline

```
yum install yum-utilsvim /etc/yum.repos.d/nginx.repo[nginx-stable]name=nginx stable repobaseurl=http://nginx.org/packages/centos/$releasever/$basearch/gpgcheck=1enabled=1gpgkey=https://nginx.org/keys/nginx_signing.key[nginx-mainline]name=nginx mainline repobaseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/gpgcheck=1enabled=0gpgkey=https://nginx.org/keys/nginx_signing.keyyum install nginxyum install epel-releaseyum install gcc-c++ flex bison yajl yajl-devel curl-devel curl GeoIP-devel doxygen zlib-devel pcre pcre-devel libxml2 libxml2-devel autoconf automake lmdb-devel ssdeep-devel ssdeep-libs lua-devel libmaxminddb-devel git apt-utils autoconf automake build-essential git libcurl4-openssl-dev libgeoip-dev liblmdb-dev ibpcre++-dev libtool libxml2-dev libyajl-dev pkgconf wget zlib1g-dev
```

报错解决：Error: Cannot retrieve metalink for repository: epel. Please verify its path and try again

解决办法：一句话：把/etc/yum.repos.d/epel.repo，文件第3行注释去掉，把第四行注释掉，修改为

> \1. [epel]
>
> \2. name=Extra Packages for Enterprise Linux 6 - $basearch
>
> \3. baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
>
> \4. #mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch

克隆GitHub存储库:

```
git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
```



编译源代码：

```
$ cd ModSecurity$ git submodule init$ git submodule update$ ./build.sh$ ./configure$ make$ make install
```

注意：安装中有报错fatal: No names found, cannot describe anything.是正常现象

下载用于ModSecurity的NGINX连接器：

```
git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```



确定哪个版本的NGINX是运行在主机上的ModSecurity模块将加载:

```
[root@guigu ModSecurity]# nginx -vnginx version: nginx/1.17.3
```

下载与安装版本对应的源代码：

```
wget http://nginx.org/download/nginx-1.17.3.tar.gztar zxvf nginx-1.17.3.tar.gz
```

编译动态模块，复制到模块标准目录:

```
cd nginx-1.17.3#./configure --with-compat --add-dynamic-module=../ModSecurity-nginx$ make modulescp objs/ngx_http_modsecurity_module.so /etc/nginx/modules/将以下load_module指令添加到/etc/nginx/nginx.conf的main中：load_module modules/ngx_http_modsecurity_module.so;
```

确定nginx模块加载成功：

```
nginx -t
```

## **三、防护效果测试**

ModSecurity 3简单示例

创建Demo web应用vim /etc/nginx/nginx.conf

```
server {listen 8085;
location / {

    default_type text/plain;

    return 200 "Thank you for requesting ${request_uri}\n";

    }
}
```

重新加载nginx:nginx -s reload

确认nginx正常工作:curl -D - [http://localhost](http://localhost/)

保护Demo web应用

创建/etc/nginx/modsec文件夹：mkdir /etc/nginx/modsec

下载推荐的ModSecurity配置文件

```
wget https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommendedmv modsecurity.conf-recommended modsecurity.conf
```

vim modsecurity.conf #在些文件中编辑以下配置

### SecRuleEngine DetectionOnly

SecRuleEngine On

创建ModSecurity的主配置文件

```
vim /etc/nginx/modsec/main.conf
```

### Include the recommended configuration

```
Include /etc/nginx/modsec/modsecurity.conf
```

### A test rule

```
SecRule ARGS:testparam "@contains test" "id:1234,deny,log,status:403"
```

报错解决：[emerg] "modsecurity_rules_file" directive Rules error.

```
vim /etc/nginx/modsec/modsecurity.conf#SecUnicodeMapFile unicode.mapping 20127
```

配置nginx反向代理，vim /etc/nginx/conf.d/proxy.conf

\#include /etc/nginx/conf.d/*.conf; #把这一行注释掉，不然80端口会有冲突

```
server {    listen 80;    modsecurity on;

    modsecurity_rules_file /etc/nginx/modsec/main.conf;

    location / {

    proxy_pass [http://0.0.0.0:8085;](http://0.0.0.0:8085/)

    proxy_set_header Host $host;

    }
nginx -s reload    #重新加载nginxcurl -D - http://localhost/foo?testparam=123    #能正常返回“Thank you for requesting /foo?testparam=123”curl -D - http://localhost/foo?testparam=test    #则返回"403 Forbidden"，说明前面配置的那条modsecuriy规则生效了，并阻拦了testparam参数中带test的请求
```

在/var/log/nginx/error.log中可以看到拦截的详细日志

部署OWASP规则--CRS（Core Rule Set）

安装运行nikto漏洞扫描工具，用于测试CRS的防御效果

```
git clone https://github.com/sullo/nikto    #下载niktocd nikto perl program/nikto.pl -h localhost    #用nikto扫描nginx搭建的web系统（反向代理）扫描结果是+ 7687 requests: 0 error(s) and 308 item(s) reported on remote host    #扫描出308个问题
```

启用OWASP CRS

```
cd /etc/nginx/modsec/wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.0.2.tar.gz    #下载OWASP CRScd owasp-modsecurity-crs-3.0.2/cp crs-setup.conf.example crs-setup.conf
```

在modsecurity主配置文件中include CRS的配置和规则

```
vim /etc/nginx/modsec/main.conf
```

### Include the recommended configuration

```
Include /etc/nginx/modsec/modsecurity.conf
```

### OWASP CRS v3 rules

```
Include /usr/local/owasp-modsecurity-crs-3.0.2/crs-setup.confInclude /usr/local/owasp-modsecurity-crs-3.0.2/rules/*.conf
```

测试CRS

```
nginx -s reload    #重新加载nginx配置curl http://localhost    #返回Thank you for requesting /curl -H "User-Agent: Nikto" http://localhost    #返回403 Forbidden，说明WAF防护已经生效，此处匹配的规则是user-agent中不能包含漏洞扫描器名字perl nikto/program/nikto.pl -h localhost    #再次用nikto扫描nginx搭建的web系统扫描结果是+ 7687 requests: 0 error(s) and 83 item(s) reported on remote host    #扫描出83个问题，比308个少了很多
```

在安装ModSecurity时，我们将演示应用程序配置为为每个请求返回状态代码200,但实际上并没有返回这些文件,Nikto将这200个状态码解释为它请求的文件确实存在,所以报告出83个问题，为了优化nikto，去除误报，我们做如下配置

```
cp nikto/program/nikto.conf.default nikto/program/nikto.conf
```

vim nikto/program/nikto.conf #在第76行最后加上;-sitefiles，如下所示

```
@@DEFAULT=@@ALL;-@@EXTRAS;tests(report:500);-sitefiles
```

之后再次用nikto扫描

```
perl program/nikto.pl -h localhost
```

扫描结果是+ 7583 requests: 0 error(s) and 7 item(s) reported on remote host

可以看出问题只有7个问题，由于ModSecurity不支持响应（response）的检查，所以涉及此类的漏洞无法防御。但总体还是抵御了绝大部分的nikto的漏洞扫描。

![img](https://image.3001.net/images/20190814/1565774866_5d53d412b3b33.png!small)

## 参考链接：

> https://www.nginx.com/resources/library/modsecurity-3-nginx-quick-start-guide/
>
> https://github.com/SpiderLabs/ModSecurity
>
> https://github.com/SpiderLabs/ModSecurity/tree/v3/master

***本文作者：owensky，转载请注明来自FreeBuf.COM**



https://www.freebuf.com/news/211354.html