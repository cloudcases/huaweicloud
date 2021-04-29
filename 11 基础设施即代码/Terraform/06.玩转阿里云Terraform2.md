# 玩转阿里云 Terraform(二)：Terraform 的几个关键概念

[阿里云技术](https://www.jianshu.com/u/02e1d2ba8a95)

2019.10.18 14:51:29字数 2,445阅读 640

上一篇《[玩转阿里云Terraform(一)：Terraform 是什么](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Farticles%2F713099)》介绍了 Terraform 的基本定义和特点之后，本文将着重介绍几个Terraform中的关键概念。

## Terraform 关键概念

在使用Terraform的过程中，通常接触到很多名词，如configuration，provider，resource，datasource，state，backend，provisioner等，本文将一一跟大家介绍这些概念。

### 1 Configuration：基础设施的定义和描述

**基础设施即代码（Infrastructure as Code）**，这里的**Code就是对基础设施资源的代码定义和描述，也就是通过代码表达我们想要管理的资源**。

```bash
# VPC 资源
resource "alicloud_vpc" "vpc" {
        name          = "tf_vpc"
        cidr_block  = "172.16.0.0/16"
}
# VSwitch 资源
resource "alicloud_vswitch" "vswitch" {
        vpc_id            = alicloud_vpc.vpc.id
        cidr_block        = "172.16.1.0/24"
        availability_zone = "cn-beijing-a"
}
```

对所有资源的代码描述都需要定义在一个以 `tf` 结尾的文件用于Terraform加载和解析，这个文件我们称之为“Terraform模板”或者“Configuration”。

### 2 Provider：基础设施管理组件

Terraform 通常用于对云上基础设施，如虚拟机，网络资源，容器资源，存储资源等的**创建，更新，查看，删除等**管理动作，也可以实现对物理机的管理，如**安装软件，部署应用**等。

[【Provider】](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fproviders%2Findex.html) 是一个与Open API直接交互的后端驱动，Terraform 就是**通过Provider来完成对基础设施资源的管理**的。不同的基础设施提供商都需要提供一个Provider来实现对自家基础设施的统一管理。目前Terraform目前支持超过160多种的providers，大多数云平台的Provider插件均已经实现了，阿里云对应的Provider为 `alicloud` 。

<img src="https://upload-images.jianshu.io/upload_images/13492518-9a8f41e6631071cf.png?imageMogr2/auto-orient/strip|imageView2/2/w/741/format/webp" alt="img" style="zoom: 80%;" />

在操作环境中，Terraform和Provider是两个独立存在的package，**当运行Terraform时，Terraform会根据用户模板中指定的provider或者resource／datasource的标志自动的下载模板所用到的所有provider，并将其放在执行目录下的一个隐藏目录 `.terraform` 下**。

```bash
provider "alicloud" {
  version              = ">=1.56.0"
  region               = "cn-hangzhou"
  configuration_source = "terraform-alicloud-modules/classic-load-balance"
}
```

模板中显示指定了一个阿里云的Provider，并显示设置了provider的版本为 `1.56.0+` （默认下载最新的版本），指定了需要管理资源的region，指定了当前这个模板的标识。
通常Provider都包含两个主要元素 resource 和 data source。

#### 3 Resource：基础设施资源和服务的管理

在Terraform中，**一个具体的资源或者服务称之为一个resource**，比如一台ECS 实例，一个VPC网络，一个SLB实例。

每个特定的resource包含了若干可用于描述对应资源或者服务的属性字段，通过这些字段来定义一个完整的资源或者服务，比如实例的名称（name），实例的规格（instance_type），VPC或者VSwitch的网段（cidr_block）等。
定义一个Resource的语法非常简单，通过 `resource` 关键字声明，如下：

```bash
# 定义一个ECS实例
resource "alicloud_instance" "default" {
  image_id        = "ubuntu_16_04_64_20G_alibase_20190620.vhd"
  instance_type   = "ecs.sn1ne.large"
  instance_name   = "my-first-vm"
  system_disk_category = "cloud_ssd"
  ...
}
```

- 其中 `alicloud_instance` 为**资源类型（Resource Type)**，定义这个资源的类型，告诉Terraform这个Resource是阿里云的ECS实例还是阿里云的VPC。
- `default` 为**资源名称(Resource Name)**，资源名称在同一个模块中必须唯一，主要用于供其他资源引用该资源。
- 大括号里面的block块为**配置参数(Configuration Arguments)**，定义资源的属性，比如ECS 实例的规格、镜像、名称等。

显然这个Terraform模板的功能为在阿里云上创建一个ECS实例，镜像ID为 `ubuntu_16_04_64_20G_alibase_20190620.vhd` ，规格为 `ecs.sn1ne.large` ，自定义了实例名称和系统盘的类型。

除此之外，在Terraform中，**一个资源与另一个资源的关系也定义为一个资源**，如一块云盘与一台ECS实例的挂载，一个弹性IP（EIP）与一台ECS或者SLB实例的绑定关系。这样定义的好处是，一方面资源架构非常清晰，另一方面，**当模板中有若干个EIP需要与若干台ECS实例绑定时，只需要通过Terraform的 `count` 功能就可以在无需编写大量重复代码的前提下实现绑定功能**。

```swift
resource "alicloud_instance" "default" {
  count = 5
  ...
}
resource "alicloud_eip" "default" {
    count = 5
  ...
}
resource "alicloud_eip_association" "default" {
  count = 5
  instance_id = alicloud_instance.default[count.index].id
  allocation_id = alicloud_eip.default[count.index].id
}
```

显然这个Terraform模板的功能为在阿里云上创建5个ECS实例和5个弹性IP，并将它们一一绑定。

#### 4 Data Source：基础设施资源和服务的查询

对资源的查询是运维人员或者系统最常使用的操作，比如，查看某个region下有哪些可用区，某个可用区下有哪些实例规格，每个region下有哪些镜像，当前账号下有多少机器等，通过对资源及其资源属性的查询可以帮助和引导开发者进行下一步的操作。

除此之外，在编写Terraform模板时，**Resource使用的参数有些是固定的静态变量，但有些情况下可能参数变量不确定或者参数可能随时变化**。比如我们创建ECS 实例时，通常需要指定我们自己的镜像ID和实例规格，但我们的模板可能随时更新，如果在代码中指定ImageID和Instance，则一旦我们更新镜像模板就需要重新修改代码。
在Terraform 中，**Data Source 提供的就是一个查询资源的功能，每个data source实现对一个资源的动态查询，Data Souce的结果可以认为是动态变量，只有在运行时才能知道变量的值**。

Data Sources通过 `data` 关键字声明，如下：

```kotlin
// Images data source for image_id
data "alicloud_images" "default" {
  most_recent = true
  owners      = "system"
  name_regex  = "^ubuntu_18.*_64"
}

data "alicloud_zones" "default" {
  available_resource_creation = "VSwitch"
  enable_details              = true
}

// Instance_types data source for instance_type
data "alicloud_instance_types" "default" {
  availability_zone = data.alicloud_zones.default.zones.0.id
  cpu_core_count    = 2
  memory_size       = 4
}

resource "alicloud_instance" "web" {
  image_id        = data.alicloud_images.default.images[0].id
  instance_type   = data.alicloud_instance_types.default.instance_types[0].id
  instance_name   = "my-first-vm"
  system_disk_category = "cloud_ssd"
  ...
}
```

如上例子中的ECS Instance 没有指定镜像ImageID和实例规格，而是通过 `data`引用，Terraform运行时将首先根据镜像名称前缀选择系统镜像，如果同时有多个镜像满足条件，则选择最新的镜像。实例规格也是类似，在某个可用区下选择2核4G的实例规格进行返回。

### 5 State：保存资源关系及其属性文件的数据库

**Terraform创建和管理的所有资源都会保存到自己的数据库上**，这个数据库不是通常意义上的数据库（MySQL，Redis等），而是一个文件名为 `terraform.tfstate` 的文件，在Terraform 中称之为 [`state`](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fstate%2Findex.html)[ ](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fstate%2Findex.html)，**默认存放在执行Terraform命令的本地目录下**。这个 `state` 文件非常重要，如果该文件损坏，Terraform 将认为已创建的资源被破坏或者需要重建（实际的云资源通常不会受到影响），因为在执行Terraform命令是，**Terraform将会利用该文件与当前目录下的模板做Diff比较，如果出现不一致，Terraform将按照模板中的定义重新创建或者修改已有资源，直到没有Diff，因此可以认为Terraform是一个有状态服务**。

当涉及多人协作时不仅需要拷贝模板，还需要拷贝 `state` 文件，这无形中增加了维护成本。幸运的是，目前Terraform支持把 `state` 文件放到远端的存储服务 `OSS` 上或者 `consul` 上，来实现 `state` 文件和模板代码的分离。具体细节可参考[官方文档Remote State](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fstate%2Fremote.html)或者关注后续文章的详细介绍。

### 6 Backend：存放 State 文件的载体

正如上节提到，**Terraform 在创建完资源后，会将资源的属性存放在一个 `state` 文件中，这个文件可以存放在本地也可以存放在远端**。存放 `state` 文件的载体就是 [`Backend`](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fbackends%2Findex.html) 。
`Backend` 分为本地（local）和远端（remote）两类，默认为本地。远端的类型也非常多，目前官方网站提供的有13种，并且阿里云的[OSS](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fbackends%2Ftypes%2Foss.html)就位列其中。

使用远端的Backend，既可以降低多人协作时对state的维护成本，而且可以将一些敏感的数据存放在远端，保证了数据的安全性。

### 7 Provisioner：在机器上执行操作的组件

[Provisioner](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fprovisioners%2Findex.html) 通常**用来在本地机器或者登陆远程主机执行相关的操作**，如 `local-exec` provisioner 用来执行本地的命令， `chef` provisioner 用来在远程机器安装，配置和执行chef client， `remote-exec` provisioner 用来登录远程主机并在其上执行命令。

Provisioner 通常跟 Provider一起配合使用，provider用来创建和管理资源，provisioner在创建好的机器上执行各种操作。

## 小结

Terraform中涉及到的概念非常多，用法也多种多样，本文只是介绍了其中的几个关键的概念，如果想要了解更多的概念，可以访问Terraform的官方网站。

学习Terraform最好的方式是要多动手，使用terraform去实现和操作一些特定的场景，在不断操作的过程中，持续解决遇到的问题可以帮助大家更快和更好的使用Terraform。下一篇将向大家介绍Terraform的运行原理和基本操作。

最后，欢迎大家关注 Terraform，关注阿里云 Provider，如有任何使用问题，可直接在 [terraform-provider-alicloud](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Fterraform-providers%2Fterraform-provider-alicloud%2Fissues) 提交您的问题，我们将尽快解决。

[阅读原文](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Farticles%2F721188%3Futm_content%3Dg_1000081736)
本文为云栖社区原创内容，未经允许不得转载。