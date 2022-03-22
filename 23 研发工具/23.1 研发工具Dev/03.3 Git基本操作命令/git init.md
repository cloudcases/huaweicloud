# git init 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

**git init** 命令用于在目录中创建新的 Git 仓库。

在目录中执行 **git init** 就可以创建一个 Git 仓库了。

例如我们在当前目录下创建一个名为 runoob 的项目：

## 实例

$ **mkdir** runoob
$ **cd** runoob**/**
$ **git init**
Initialized empty Git repository **in** **/**Users**/**tianqixin**/**www**/**runoob**/**.git**/**
*# 初始化空 Git 仓库完毕。*

现在你可以看到在你的项目中生成了 **.git** 这个子目录，这就是你的 Git 仓库了，所有有关你的此项目的快照数据都存放在这里。

**.git** 默认是隐藏的，可以用 ls -a 命令查看：

```
ls -a
.    ..    .git
```