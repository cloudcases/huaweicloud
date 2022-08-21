# git pull 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

**git pull** 命令用于从远程获取代码并合并本地的版本。

**git pull** 其实就是 **git fetch** 和 **git merge FETCH_HEAD** 的简写。 命令格式如下：

```
git pull <远程主机名> <远程分支名>:<本地分支名>
```

### 实例

更新操作：

```
$ git pull
$ git pull origin
```

将远程主机 origin 的 master 分支拉取过来，与本地的 brantest 分支合并。

```
git pull origin master:brantest
```

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

```
git pull origin master
```

上面命令表示，取回 origin/master 分支，再与本地的 brantest 分支合并。

上面的 pull 操作用 fetch 表示为：

以我的 https://github.com/tianqixin/runoob-git-test 为例，远程载入合并本地分支。

```
$ git remote -v  # 查看信息
origin    https://github.com/tianqixin/runoob-git-test (fetch)
origin    https://github.com/tianqixin/runoob-git-test (push)

$ git pull origin master
From https://github.com/tianqixin/runoob-git-test
 * branch            master     -> FETCH_HEAD
Already up to date.
```

上面命令表示，取回 origin/master 分支，再与本地的 master 分支合并。