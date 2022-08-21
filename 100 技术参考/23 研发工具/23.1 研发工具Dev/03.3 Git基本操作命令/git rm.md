# git rm 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

git rm 命令用于删除文件。

如果只是简单地从工作目录中手工删除文件，运行 **git status** 时就会在 **Changes not staged for commit** 的提示。

git rm 删除文件有以下几种形式：

1、将文件从暂存区和工作区中删除：

```
git rm <file>
```

以下实例从暂存区和工作区中删除 runoob.txt 文件：

```
git rm runoob.txt 
```

如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 **-f**。

强行从暂存区和工作区中删除修改后的 runoob.txt 文件：

```
git rm -f runoob.txt 
```

如果想把文件从暂存区域移除，但仍然希望保留在当前工作目录中，换句话说，仅是从跟踪清单中删除，使用 **--cached** 选项即可：

```
git rm --cached <file>
```

以下实例从暂存区中删除 runoob.txt 文件：

```
git rm --cached runoob.txt
```

### 实例

删除 hello.php 文件：

```
$ git rm hello.php 
rm 'hello.php'
$ ls
README
```

文件从暂存区域移除，但工作区保留：

```
$ git rm --cached README 
rm 'README'
$ ls
README
```

可以递归删除，即如果后面跟的是一个目录做为参数，则会递归删除整个目录中的所有子目录和文件：

```
git rm –r * 
```

进入某个目录中，执行此语句，会删除该目录下的所有文件和子目录。