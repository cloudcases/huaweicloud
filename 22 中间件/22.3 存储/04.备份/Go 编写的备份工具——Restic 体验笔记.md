# Go 编写的备份工具——Restic 体验笔记

[![img](https://upload.jianshu.io/users/upload_avatars/137499/3de68ca8-5a91-400a-a4f5-4fd365205c1a.png?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/e213f00c7c35)

[左蓝](https://www.jianshu.com/u/e213f00c7c35)关注

0.3122017.02.09 23:17:21字数 748阅读 3,609

> [restic](https://link.jianshu.com/?t=https://github.com/restic/restic) 是一个 Go 语言编写的备份工具，特点是快速、高效而且安全。

下载最新版本：[release page](https://link.jianshu.com/?t=https://github.com/restic/restic/releases/latest)

目前最新版本是 0.0.4，你可以下载回来解压构建使用。但是，我习惯使用 Docker，毕竟我懒得装 Golang 到服务器编译，所以我还是用 Docker 运行吧。

> 上面 release 有已经编译好的版本，我写完全文才发现，打脸。
> (￣ε(#￣)☆╰╮(￣▽￣///)

官方给了一份 [Dockerfile](https://link.jianshu.com/?t=https://github.com/restic/restic/blob/master/Dockerfile)，官方的镜像自然是比较全面的，但我基本用不上那些功能，所以还是自己写一份吧：



```ruby
FROM alpine:edge
ENV RESTIC_VERSION=0.4.0

RUN apk add --no-cache go git musl-dev openssl ca-certificates && \
    wget https://github.com/restic/restic/releases/download/v${RESTIC_VERSION}/restic-${RESTIC_VERSION}.tar.gz && \
    tar -xzvf restic-${RESTIC_VERSION}.tar.gz && \
    cd restic-${RESTIC_VERSION} && \
    go build build.go && \
    ./build && \
    mv restic /bin/restic && \
    cd ../ && \
    rm -rf restic-* && \
    apk del -q go git musl-dev openssl && \
    rm -rf /var/cache/apk/*
```

上面从源代码构建整个程序，和官方那目测几百 MB 的镜像体积，我这个 Dockerfile 仅有 14 MB。

> 如果你需要的话可以使用 `docker pull zuolan/restic` 拉取镜像，只有 6MB 的传输体积。

先运行看一下说明，顺便测试一下镜像运行时是否正常：



```bash
docker run --rm -it zuolan/restic restic --help
```

一般来说没什么问题，毕竟都构建成功了。

这个备份工具有点类似于 Git 这种版本控制工具，有着仓库（repository）的概念，所以我们先初始化仓库：



```kotlin
$ docker run --rm -it -v ~/backup:/tmp/backup zuolan/restic restic init --repo /tmp/backup
enter password for new backend:
enter password again:
created restic backend 085b3c76b9 at /tmp/backup

Please note that knowledge of your password is required to access the repository.
Losing your password means that your data is irrecoverably lost.
```

记住密码，不然你数据就无法恢复了。

然后创建一个快照：



```ruby
$ docker run --rm -it \
    -v ~/backup:/tmp/backup \
    -v ~/nginx:/tmp/nginx \
    zuolan/restic \
    restic -r /tmp/backup backup /tmp/nginx

enter password for repository:
scan [/tmp/work]
scanned 764 directories, 1816 files in 0:00
[0:29] 100.00%  54.732 MiB/s  1.582 GiB / 1.582 GiB  2580 / 2580 items  0 errors  ETA 0:00
duration: 0:29, 54.47MiB/s
snapshot 40dc1520 saved
```

这里面第一次索引可能会比较耗时间，但实际上也是挺快的，几GB的数据也是喝一杯水的功夫就搞定了。

查看一下仓库的快照：



```ruby
$ docker run --rm -it \
    -v ~/backup:/tmp/backup \
    zuolan/restic \
    restic -r /tmp/backup snapshots
enter password for repository: 
ID        Date                 Host        Tags            Directory
----------------------------------------------------------------------
9236eead  2017-02-09 14:51:10  e1c6f61cb5e0                  /tmp/nginx
```

注意的是，仓库和快照不是一对一的，你可以在一个仓库中存放不同目录的多个快照，所以一般只用一个备份仓库就可以了，除非你数据很多很复杂。

例如给其他目录也备份：



```ruby
$ docker run --rm -it \
    -v ~/backup:/tmp/backup \
    -v ~/apache:/tmp/apache \
    zuolan/restic \
    restic -r /tmp/backup backup /tmp/apache
```

注意 -v 参数中原来的 nginx 改为了 apache，这样在数据卷中也要相应修改名称，避免程序认为是同一个目录的变动而修改了原有的快照。

排除文件/夹自然也是老办法：--exclude 或者 --exclude-file 之类的参数。

------

恢复快照：



```ruby
$ docker run --rm -it \
    -v ~/backup:/tmp/backup \
    -v ~/nginx:/tmp/nginx \
    zuolan/restic \
    restic -r /tmp/backup restore 9236eead --target /tmp/nginx

restoring <Snapshot 9236eead of [/tmp/nginx] at 2017-02-09 14:51:10.376787233 +0000 UTC by @e1c6f61cb5e0> to /tmp/nginx
```

完美恢复！～

------

官方教程地址：
[https://restic.readthedocs.io/en/stable/Manual/](https://link.jianshu.com/?t=https://restic.readthedocs.io/en/stable/Manual/)

详细的我还要再看看，我才刚接触这个软件两个小时，有什么问题可以留言，笑。

------

写在最后，为了使用愉快，你最好用 alias 把整个 Docker 命令丢到 .bashrc 中：



```bash
alias restic="docker run --rm -v ~/data:/tmp/data zuolan/restic restic"
```

~/data 是你备份数据的地方，因为我是“万物皆容器”的原则，所以服务器全部数据都统一放置在一个文件中，管理备份很轻松。

这样你使用 restic 命令时实际上就是调用这个镜像运行容器了。总之使用方式随机应变。



[Go 编写的备份工具——Restic 体验笔记 - 简书 (jianshu.com)](https://www.jianshu.com/p/83e374f4c9a0)