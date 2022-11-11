# 玩转阿里云 Terraform(三)：Terraform 的安装和加速

2019-11-09 12528

**简介：** 本文以Mac OS为例，详细介绍如何在本地安装Terraform，并在文章最后介绍一种可以加速Terraform安装的方法。

上一篇《[玩转阿里云 Terraform(二)：Terraform 的几个关键概念](https://yq.aliyun.com/articles/721188)》介绍了 Terraform 运行的几个关键概念，掌握了这些名词后，本文将详细介绍如何安装Terraform，为后续的动手实践最准备。

由于Terraform是用Golang语言编写的，并且是一个面向客户端的工具，因此对Terraform的安装等同于安装一个可执行的二进制文件，过程非常简单。本文以Mac OS为例，详细介绍如何在本地安装Terraform，并在文章最后介绍一种可以加速Terraform安装的方法。

## 1 Terraform 的安装

Terraform 的安装包含Terraform运行主体的安装，Provider和Provisioner等插件的安装。

### 1.1 Terraform 主体的安装

本文以Mac OS为例，通过三步完成对Terraform的安装：

**第一步：下载Terraform版本到本地特定目录并解压**
访问[Terraform Release](https://releases.hashicorp.com/terraform/)页面，点击想要下载的版本（强烈建议下载0.12.x版本，其对Terraform命令和语法规则做了很大的升级）目录，找到Mac系统的 `darwin_amd64` 包，点击直接下载到特定目录，如 `workspace` ，或者右键“复制链接地址”手动下载和解压：

```
$ mkdir workspace && cd workspace
$ wget https://releases.hashicorp.com/terraform/0.12.13/terraform_0.12.13_darwin_amd64.zip
$ unzip terraform_0.12.13_darwin_amd64.zip
```

**第二步：配置安装目录到PATH，以便系统可以识别**

```
$ export PATH=$(PWD):$PATH

# 如果想要永久生效，可以将其配置到Profile变量文件 ~/.bash_profile
$ echo export PATH=$(PWD):$PATH >> ~/.bash_profile
$ source ~/.bash_profile # 执行 source 命令使其生效
```

**第三步：验证是否安装成功**

```
$ terraform version
Terraform v0.12.13
```

如果想要在其他操作系统上安装Terraform，可以参考官方展示的安装步骤：https://learn.hashicorp.com/terraform/getting-started/install.html#installing-terraform



### 1.2 插件的安装

插件的安装包括对Provider和Provisioner的安装。插件的安装有两种方式：自动安装和手动安装。

**init命令自动安装插件**

编写了Terraform模板之后，在模板所在的目录下执行 `terraform init` , terraform 将会根据模板中指定的Provider和Provisioner的类型或者Provider中的资源类型，自动加载最新的或者指定的Provider版本：

```
provider "alicloud" {
  profile = "terraform"
  region  = "eu-central-1"
  version = "1.60.0"
}
```

显示指定Provider及其版本，`init` 命令将自动下载阿里云Provider 1.60.0 版本：

```
$ terraform version
Terraform v0.12.8
+ provider.alicloud v1.60.0
```

如果在模板中不指定Provider，那么定义跟Provider版本相关的resource或者data source， `init` 命令同样可以完成对应Provider最新版本的下载：

```
resource "alicloud_vpc" "default" {
  cidr_block = "172.16.0.0/16"
  ...
}
```

**下载插件Package，手动安装插件**

除了 `init` 命令自动下载外，还可以手动完成插件的安装。具体的操作步骤跟安装Terraform主体的方法一样，即访问Provider的下载页面：https://releases.hashicorp.com/terraform-provider-alicloud/ 下载特定版本的Package，并将其解压到Terraform主体所在的目录即可。感兴趣的同学可以尝试以下，本文不再赘述。

值得注意的是，如果模板中定义的Provider的版本与本地安装版本不一致，那么再次运行 `init` 命令时，将以模板优先。



## 2 Terraform 安装加速

相信大部分同学在上述的下载和安装过程中，经常会遇到下载失败的问题。这因为Terraform及其Provider等插件的所有发布的Release都存放在海外服务上，国内网络访问时会遇到下载速度慢甚至无法访问的情况。面对在这种情况，Hashicorp目前并未给出好的解决方案。

阿里云为了不影响大家对Terraform以及阿里云Provider的使用，在一款在线工具 [CloudShell](https://shell.aliyun.com/) 上对Terraform进行了预安装，同时对 `terraform init` 做了加速处理，支持对包括阿里云Provider `alicloud` 在内的更多Provider，如 `docker` , `helm` , `kubernetes` , `local` , `null` , `random` 等20多种。CloudShell的加速过程可以详见云栖博客《[加速Terraform init](https://yq.aliyun.com/articles/723935)》。

加速后的CloudShell将 `terraform init` 初始化的时间从 3min+ 降低到 3-6s ，大大降低了Terraform Provider安装和使用的成本，欢迎大家使用。

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092?spm=a2c6h.12873639.article-detail.10.288721f3W3tAq8)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。



[玩转阿里云 Terraform(三)：Terraform 的安装和加速-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/726467?spm=a2c6h.13262185.profile.40.799751ccoCDFMz)