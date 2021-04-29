# 玩转阿里云 Terraform(一)：Terraform 是什么

[阿里云技术](https://www.jianshu.com/u/02e1d2ba8a95)

2019.10.18 13:45:39字数 1,623阅读 377

从本文起，我将陆续推出一系列有关 Terraform 的文章，从概念，特点，工作机制，用法以及最佳实践等多个方面由浅入深的向大家介绍如何在阿里云上玩转 Terraform。同时也希望借此机会，与感兴趣的开发者相互探讨和学习。

作为本系列的引子，本文将从 Terraform 是什么开始介绍。

### Terraform 是什么

如果两年前问 Terraform 是什么，国内熟悉它的人可能并不是很多，毕竟它是2014年的新生儿，提到资源编排，大家能优先想到的也是 **AWS 的 CloudFormation，Azure 的 ARM 以及[阿里云的 ROS](https://links.jianshu.com/go?to=https%3A%2F%2Fhelp.aliyun.com%2Fdocument_detail%2F28852.html)**。但是如果今天再问“Terraform 是什么”，相信很大一部分人对它不再陌生，并且一些公司或者感兴趣的开发者已经开始使用 Terraform 来管理他们的基础设施。不论是 Hashicorp 的官方宣传，还是[阿里云云栖社区](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fsearch%3Fq%3DTerraform%26type%3DARTICLE)的布道；不论是 Devops的持续发展，还是云原生的火热传播；不论是国内最初只有阿里云支持，还是后来国内其他云厂商的陆续支持，Terraform 近两年在国内被越来越多的提及。

在此之前，不止一篇云栖文章介绍过Terraform是什么，本文将结合Terraform的官方文档，对这些文章中的介绍做一个整理和总结。

2014年，[Hashicorp](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.hashicorp.com%2F) 公司推出了一款产品 [Terraform](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2F)

<img src="https://upload-images.jianshu.io/upload_images/13492518-d66687c54d803d9a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp" alt="img" style="zoom:25%;" />

[Terraform 官方的定义](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fintro%2Findex.html%23what-is-terraform-)如下：

> Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.
>
> Configuration files describe to Terraform the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.
>
> The infrastructure Terraform can manage includes low-level components such as compute instances, storage, and networking, as well as high-level components such as DNS entries, SaaS features, etc.

总结起来，三句话：

- Terraform 是一个可以**安全、高效地建立，变更以及版本化管理基础设施的工具**，并可在主流的服务提供商上提供自定义的解决方案。
- Terraform **以配置文件为驱动，在文件中定义所要管理的组件（基础设施资源）**，以此生成一个可执行的计划（如果不可执行，会提示报错），通过执行这个计划来完成所定义组件的创建，增量式的变更和持续的管理。
- Terraform**不仅可以管理IaaS层的资源，如计算实例，网络实例，存储实例等，也可以管理更上层的服务，如DNS 域名和解析记录，SaaS 应用的功能等**

更通俗的讲，Terraform 就是运行在客户端的一个开源的，**用于资源编排的自动化运维工具**。**以代码的形式将所要管理的资源定义在模板中，通过解析并执行模板来自动化完成所定义资源的创建，变更和管理，进而达到自动化运维的目标**。

### Terraform 主要特点

Terraform 具备以下几个主要特点：

- **基础设施即代码（IaC, Infrastructure as Code）**
  Terraform **基于一种特定的配置语言（HCL, Hashicorp Configuration Language）来描述基础设施资源**。由此，可以像对待任何其他代码一样，实现对所描述的解决方案或者基础架构的版本控制和管理。同时，**通用的解决方案和基础架构可以以模板的形式进行便捷的共享和重用**。

```csharp
resource "alicloud_instance" "instance" {
  count = 5
  instance_name   = "my-first-tf-vm"
  image_id        = "ubuntu_140405_64_40G_cloudinit_20161115.vhd"
  instance_type   = "ecs.sn1ne.small"
  security_groups = ["sg-abc12345"]
  vswitch_id      = "vsw-abc12345"

  tags = {
    from = "terraform"
  }
}
```

- **执行计划（Execution Plans）**
  Terraform 在执行模板前，运行 `terraform plan` 命令会先**通过解析模板生成一个可执行的计划，这个计划展示了当前模板所要创建或变更的资源及其属性**。操作人员可以预览这个计划，在确认无误后执行 `terraform apply` 命令，即可完成对所定义资源的快速创建和变更，以免发生一些超预期的问题。

```python
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # alicloud_instance.instance[0] will be created
  + resource "alicloud_instance" "instance" {
      + availability_zone          = (known after apply)
      + id                         = (known after apply)
      + image_id                   = "ubuntu_140405_64_40G_cloudinit_20161115.vhd"
      + instance_name              = "my-first-tf-vm"
      + instance_type              = "ecs.sn1ne.small"
      + security_groups            = [
          + "sg-abc12345",
        ]
      + tags                       = {
          + "from" = "terraform"
        }
      + vswitch_id                 = "vsw-abc12345"
            ...
    }

  # alicloud_instance.instance[1] will be created
  + resource "alicloud_instance" "instance" {
      ...
  }
  # alicloud_instance.instance[2] will be created
  + resource "alicloud_instance" "instance" {
      ...
  }
    ...

Plan: 5 to add, 0 to change, 0 to destroy.
```

- **资源拓扑图（Resource Graph）**
  Terraform 会根据模板中的定义，构建所有资源的图形，并且**以并行的方式创建和修改那些没有任何依赖资源的资源**，以保证执行的高效性。**对于有依赖资源的资源，被依赖的资源优先执行**。
- **自动化变更（Change Automation）**
  不论多复杂的资源，当模板中定义的资源内容发生变更时，Terraform 都会基于新的资源拓扑图将变更的内容plan 出来，在确认无误后，只需一个命令即可完成数个变更操作，避免了人为操作带来的错误。

### Terraform 活跃的社区

Terraform **作为 DevOps 的一个利器，大大降低了基础设施的管理成本**，并且随着业务架构的不断复杂，基础设施资源及其资源关系的不断复杂，Terraform 所带来的便利性和优势将越来越明显。

正因为如此，云计算火热发展的今天，Terraform 受到越来越多的云厂商和其他软件服务商的青睐。目前，Terraform 累计提供了超过100个 [Providers](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fproviders%2Findex.html) 和 [Provisioners](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fprovisioners%2Findex.html)，支持15个 [Backend Types](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fbackends%2Ftypes%2Findex.html)。

阿里云作为国内第一家与 Terraform 集成的云厂商，目前已经提供了超过 [138 个 resource 和 92 个 datasource](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fproviders%2Falicloud%2Findex.html)，覆盖计算，存储，网络，负载均衡，CDN，中间件，访问控制，数据库等多个领域。并且从 Terraform 0.12.2 版本开始，阿里云 OSS 作为[标准的 Backend](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fwww.terraform.io%2Fdocs%2Fbackends%2Ftypes%2Foss.html) 开始提供远端存储 State 的能力，持续提升开发者的使用体验，降低使用成本。

欢迎大家关注 Terraform，关注阿里云 Provider，如有任何使用问题，可直接在 [terraform-provider-alicloud](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Fgo%2FarticleRenderRedirect%3Furl%3Dhttps%3A%2F%2Fgithub.com%2Fterraform-providers%2Fterraform-provider-alicloud%2Fissues) 提交您的问题，我们将尽快解决。

[阅读原文](https://links.jianshu.com/go?to=https%3A%2F%2Fyq.aliyun.com%2Farticles%2F713099%3Futm_content%3Dg_1000081735)
本文为云栖社区原创内容，未经允许不得转载。