# git remote 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

**git remote** 命用于在远程仓库的操作。

本章节内容我们将以 Github 作为远程仓库来操作，所以阅读本章节前需要先阅读关于 Github 的相关内容：[Git 远程仓库(Github)](https://www.runoob.com/git/git-remote-repo.html)。

显示所有远程仓库：

```
git remote -v
```

以下我们先载入远程仓库，然后查看信息：

$ **git clone** https:**//**github.com**/**tianqixin**/**runoob-git-test
$ **cd** runoob-git-test
$ **git remote** -v
origin https:**//**github.com**/**tianqixin**/**runoob-git-test **(**fetch**)**
origin https:**//**github.com**/**tianqixin**/**runoob-git-test **(**push**)**

**origin** 为远程地址的别名。

显示某个远程仓库的信息：

```
git remote show [remote]
```

例如：

```
$ git remote show https://github.com/tianqixin/runoob-git-test
* remote https://github.com/tianqixin/runoob-git-test
  Fetch URL: https://github.com/tianqixin/runoob-git-test
  Push  URL: https://github.com/tianqixin/runoob-git-test
  HEAD branch: master
  Local ref configured for 'git push':
    master pushes to master (local out of date)
```

添加远程版本库：

```
git remote add [shortname] [url]
```

shortname 为本地的版本库，例如：

```
# 提交到 Github
$ git remote add origin git@github.com:tianqixin/runoob-git-test.git
$ git push -u origin master
```

其他相关命令：

```
git remote rm name  # 删除远程仓库
git remote rename old_name new_name  # 修改仓库名
```

更多内容可以查看：[Git 远程仓库(Github)](https://www.runoob.com/git/git-remote-repo.html)。