# git clone 命令

[![Git 基本操作](https://www.runoob.com/images/up.gif)Git 基本操作](https://www.runoob.com/git/git-basic-operations.html)

------

**git clone** 拷贝一个 Git 仓库到本地，让自己能够查看该项目，或者进行修改。

拷贝项目命令格式如下：

```
 git clone [url]
```

**[url]** 是你要拷贝的项目。

例如我们拷贝 Github 上的项目：

```
$ **git clone** https:**//**github.com**/**tianqixin**/**runoob-git-test
Cloning into 'runoob-git-test'...
remote: Enumerating objects: 12, done.
remote: Total 12 **(**delta 0**)**, reused 0 **(**delta 0**)**, pack-reused 12
Unpacking objects: 100**%** **(**12**/**12**)**, done.
```

拷贝完成后，在当前目录下会生成一个 runoob-git-test 目录：

```
$ cd simplegit/
$ ls
README.md    runoob-test.txt    test.txt
```

上述操作将复制该项目的全部记录。

```
$ ls -a
.        ..        .git        README.md    runoob-test.txt    test.txt
$ cd .git 
$ ls
HEAD        description    index        logs        packed-refs
config        hooks        info        objects        refs
```

默认情况下，Git 会按照你提供的 URL 所指向的项目的名称创建你的本地项目目录。 通常就是该 URL 最后一个 / 之后的项目名称。如果你想要一个不一样的名字， 你可以在该命令后加上你想要的名称。

例如，以下实例拷贝远程 git 项目，本地项目名为 **another-runoob-name**：

$ **git clone** https:**//**github.com**/**tianqixin**/**runoob-git-test another-runoob-name
Cloning into 'another-runoob-name'...
remote: Enumerating objects: 12, done.
remote: Total 12 **(**delta 0**)**, reused 0 **(**delta 0**)**, pack-reused 12
Unpacking objects: 100**%** **(**12**/**12**)**, done.