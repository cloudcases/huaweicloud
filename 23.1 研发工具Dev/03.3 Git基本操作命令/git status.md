# git status 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

git status 命令用于查看在你上次提交之后是否有对文件进行再次修改。

```bash
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

    new file:   README
    new file:   hello.php
```

通常我们使用 **-s** 参数来获得简短的输出结果：

```bash
$ git status -s
AM README
A  hello.php
```

**AM** 状态的意思是这个文件在我们将它添加到缓存之后又有改动。