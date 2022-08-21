# git add 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

**git add** 命令可将该文件添加到暂存区。

添加一个或多个文件到暂存区：

```
git add [file1] [file2] ...
```

添加指定目录到暂存区，包括子目录：

```
git add [dir]
```

添加当前目录下的所有文件到暂存区：

```
git add .
```

以下实例我们添加两个文件：

```
$ touch README                # 创建文件
$ touch hello.php             # 创建文件
$ ls
README        hello.php
$ git status -s
?? README
?? hello.php
$ 
```

git status 命令用于查看项目的当前状态。

接下来我们执行 git add 命令来添加文件：

```
$ git add README hello.php 
```

现在我们再执行 git status，就可以看到这两个文件已经加上去了。

```
$ git status -s
A  README
A  hello.php
$ 
```

![img](https://www.runoob.com/wp-content/uploads/2020/08/C87C909B-94F9-496B-B942-1FE9DD464843.jpg)

新项目中，添加所有文件很普遍，我们可以使用 **git add .** 命令来添加当前项目的所有文件。

现在我们修改 README 文件：

```
$ vim README
```

在 README 添加以下内容：**# Runoob Git 测试**，然后保存退出。

再执行一下 git status：

```
$ git status -s
AM README
A  hello.php
```

![img](https://www.runoob.com/wp-content/uploads/2020/08/F3B705BE-7D52-46E7-BF64-EA0EFCC52373.jpg)

**AM** 状态的意思是这个文件在我们将它添加到缓存之后又有改动。改动后我们再执行 **git add .** 命令将其添加到缓存中：

```
$ git add .
$ git status -s
A  README
A  hello.php
```

文件修改后，我们一般都需要进行 git add 操作，从而保存历史版本。