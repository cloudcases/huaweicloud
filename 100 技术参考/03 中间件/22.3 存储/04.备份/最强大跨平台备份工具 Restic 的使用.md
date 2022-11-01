# 最强大跨平台备份工具 Restic 的使用

[天兵公园](https://www.jianshu.com/u/163012fd7793)

2018.10.15 19:34:40字数 644阅读 3,930

为什么选择 restic，主要基于以下几点考虑：

- rclone、sync 只支持本地向远程无条件的同步，远程的永远会被覆盖。
- Brog是纯 C 开发的，只在*nix系统下运行，而 restic 可以在任何 CPU架构，任何系统上运行，因为是由 go 开发的，运行效率接近于 C 。
- rsync 对于新手并不友好，而且只有 *nix 版本，虽然也有 windows 版本，但似乎不是官方出品。

首先，创建一个备份仓库，如果结合 rclone 使用，可以指定一个挂在为远程服务器的位置，没有的话，可以使用本地路径作为备份仓库。

```kotlin
restic init --repo ./backup
```

执行此命令后，会让你输入备份仓库密码，注意如它所说，记住此密码不要丢失。这个命令的执行可能需要等待1分钟左右，对于 NFS 文件系统，可能需要的时间更长，等就是了。

```csharp
enter password for new repository:
enter password again:
created restic repository a6801fab57 at ./backup

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

添加一个本地文件夹到备份仓库，你也可以继续添加其它的文件夹。

```undefined
restic --repo ./backup backup ./mywork
```

同样，对于访问备份仓库，需要密码访问，然后这个过程也会比较长，因为都是基于文件哈希值作为备份版本的依据，此时命令会提示：

```csharp
enter password for repository:
repository a6801fab opened successfully, password is correct

Files:           1 new,     0 changed,     0 unmodified
Dirs:            0 new,     0 changed,     0 unmodified
Added to the repo: 319 B

processed 1 files, 19 B in 0:08
snapshot 8c4b2b4a saved
```

最后的一行，snapshot 8c4b2b4a saved 中的8位字符就是本次的备份版本号，这和 Git 十分类似，用过的大家都知道，以后无论是删除备份还是还原备份，都是基于这个版本号。

查看备份库中的所有备份快照：

```undefined
restic -r ./backup snapshots
```

在恢复备份之前，我们都会查看一下快照，防止恢复了错误的版本，可以使用上面的命令。

```css
enter password for repository:
repository a6801fab opened successfully, password is correct
ID        Date                 Host        Tags        Directory
----------------------------------------------------------------------
8c4b2b4a  2018-10-15 16:59:48  DELL-PC               E:\Temps\mywork
----------------------------------------------------------------------
1 snapshots
```

在以上的表格中，ID 就是备份的版本号，Date 是备份快照创建日期， HOST 是备份客户端的主机名，Tags 是标签，在我们这个演示中没有用到，Directory 是客户端原始备份目录，这是因为它可以支持多个客户端，多个仓库的备份， restic 更像是一个集中式的版本备份系统。

接下来是如何还原一个备份，十分简单，指定 restore 哪一个版本号，以及 target 指向一个恢复路径，就会完成备份的还原。

```undefined
restic -r ./backup restore 8c4b2b4a --target ./mywork_restore
```



[最强大跨平台备份工具 Restic 的使用 - 简书 (jianshu.com)](https://www.jianshu.com/p/083edbbb426f)