# git mv 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

git mv 命令用于移动或重命名一个文件、目录或软连接。

```
git mv [file] [newfile]
```

如果新但文件名已经存在，但还是要重命名它，可以使用 **-f** 参数：

```
git mv -f [file] [newfile]
```

我们可以添加一个 README 文件（如果没有的话）：

```
$ git add README 
```

然后对其重命名:

```
$ git mv README  README.md
$ ls
README.md
```