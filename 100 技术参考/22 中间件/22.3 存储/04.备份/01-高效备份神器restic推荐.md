# 高效备份神器restic推荐!

#### Tue Sep 10, 2019

##### 400 Words|Read in about 2 Min

##### Tags: [Devops](https://www.sklinux.com/tags/devops/) 

作为一个DevOps、SaOPS、NetOps、系统运维、网络运维、DBA人员来说，资料数据的备份是必不可少的。我们经常在各个软件产品维护过程备份各类数据库、程序文件、配置文件等。

经常使用的工具有 cp、scp、rsync、ssh、tar、git等等。然鹅，Restic是集大成于一身的超级备份神器！Restic是为何物?

### 介绍

restic 是一个 Go 语言编写的备份工具，特点是快速、高效而且安全。

### 支持备份介质

- 本地磁盘
- Sftp-异地SSH主机磁盘
- HTTP rest文件存储
- AWS S3
- B2
- Azure BS
- Google CloudStorage等

### 特点

```
1.按版本备份  
2.多版本存储  
3.差异化对比备份  
4.老旧版本可以清理
```

#### 简单

备份操作，应该非常容易操作。如果太过于复杂，可能运维人员就想跳过备份的情况。所以restic的备份操作非常简单，恢复也是一样的简单。只需要一个backup或者restore的关键字即可。比如：

```bash
备份
restic -r /srv/restic-repo backup /www 
restic -r /srv/restic-repo backup /conf 
恢复
restic -r /srv/restic-repo restore 79766175 --target /tmp/restore-work
```

#### 速度

备份数据的速度，取决于你的磁盘IO性能或者网络吞吐性能，可以随时激活备份机制进行备份。还原备份也尽可能快速，并快速得到需要还原所需要的数据。否则就没有人去做还原或者备份了。

#### 安全

Restic使用加密技术来保证数据的机密性和完整性。假设存储备份数据的位置不是可信环境，restic的备份是加密的。其他人获取到备份文件也无济于事。

#### 可校验

比备份还要重要的是恢复，如果备份都不能恢复，再高效、再安全的备份都无济于事。所以restic支持快速、简单地检测备份是否完整。

#### 高效

随着数据的增长，各个版本的快照备份只是增量数据的容量实际空间增长。重复数据的管理以保证宝贵的存储空间得以高效利用。

### 操作演练

代码获取或安装

```
仓库 https://github.com/restic/restic
初始化仓库
restic -r /backup/ init
备份
restic -r /backup backup /wwww
restic -r /backup backup /opt/nginx/conf
检查快照
#restic -r /backup/ check
using temporary cache in /tmp/restic-check-cache-137737689
enter password for repository:
repository 6db5d370 opened successfully, password is correct
created new cache in /tmp/restic-check-cache-137737689
create exclusive lock for repository
load indexes
check all packs
check snapshots, trees and blobs
no errors were found
#检查快照备份情况
# restic -r /backup/ snapshots
```

![备份](https://www.sklinux.com/img/restic0.png)

```bash
挂载备份
restic -r /backup mount /r
enter password for repository:
repository 6db5d370 opened successfully, password is correct
Now serving the repository at /r
When finished, quit with Ctrl-c or umount the mountpoint.
清理快照，只保留最新的2个
# restic -r /backup/ forget --keep-last 2 --prune
repository 6db5d370 opened successfully, password is correct
Applying Policy: keep the last 2 snapshots snapshots
keep 2 snapshots:
ID        Time                 Host           Tags        Reasons        Paths
----------------------------------------------------------------------------------------
492a4515  2019-09-10 06:08:48  ip-10-10-1-91              last snapshot  /opt/nginx/conf
ad2490ad  2019-09-10 06:29:02  ip-10-10-1-91              last snapshot  /opt/nginx/conf
----------------------------------------------------------------------------------------
2 snapshots

keep 2 snapshots:
ID        Time                 Host           Tags        Reasons        Paths
--------------------------------------------------------------------------------------
ad58caaa  2019-09-10 06:25:17  ip-10-10-1-91              last snapshot  /data/wwwroot
5c166686  2019-09-10 06:28:52  ip-10-10-1-91              last snapshot  /data/wwwroot
--------------------------------------------------------------------------------------
2 snapshots

remove 1 snapshots:
ID        Time                 Host           Tags        Paths
-----------------------------------------------------------------------
1b9f9482  2019-09-10 06:03:32  ip-10-10-1-91              /data/wwwroot
-----------------------------------------------------------------------
1 snapshots

1 snapshots have been removed, running prune
counting files in repo
building new index for repo
[0:04] 100.00%  5075 / 5075 packs
repository contains 5075 packs (74841 blobs) with 21.764 GiB
processed 74841 blobs: 0 duplicate blobs, 0 B duplicate
load all snapshots
find data that is still in use for 4 snapshots
[0:02] 100.00%  4 / 4 snapshots
found 74813 of 74841 data blobs still in use, removing 28 blobs
will remove 0 invalid files
will delete 2 packs and rewrite 1 packs, this frees 4.516 MiB
[0:00] 100.00%  1 / 1 packs rewritten
counting files in repo
[0:05] 100.00%  5073 / 5073 packs
finding old index files
saved new indexes as [f0e9bfae 41f4180c]
remove 3 old index files
[0:00] 100.00%  3 / 3 packs deleted
done
删除某个快照
restic -r /srv/restic-repo forget bdbd3439
清理磁盘文件
restic -r /srv/restic-repo prune
指定密码文件或者密码,自动化备份过程通过自定义环境变量或者密码文件进行无声备份
RESTIC_PASSWORD or RESTIC_PASSWORD_FILE
export RESTIC_PASSWORD="xxxpass"
```

- [← 前一篇](https://www.sklinux.com/posts/devops/aws-vpc安全组/)
- [后一篇 →](https://www.sklinux.com/posts/devops/以太坊挖矿教程/)

## See Also

- [VPC安全组](https://www.sklinux.com/posts/devops/aws-vpc安全组/)
- [CoreDNS自定义域名](https://www.sklinux.com/posts/devops/coredns定义自定义主机/)
- [Metrics插件部署](https://www.sklinux.com/posts/devops/metrics插件/)
- [SSH转发VPC内部服务](https://www.sklinux.com/posts/devops/ssh端口转发vpc服务/)
- [k8s集群中部署apollo配置中心](https://www.sklinux.com/posts/devops/apollo在k8s集群中的部署/)

#### Tue Sep 10, 2019