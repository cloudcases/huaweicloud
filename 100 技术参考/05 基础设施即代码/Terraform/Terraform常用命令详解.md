# Terraform 常用命令详解

## 1 资源管理常用命令

Terraform 对资源的管理主要是**对资源生命周期的管理，即通过命令实现对Terraform模板中所定义资源的创建，修改，查看和删除**。

### 1.1 terraform plan：资源预览

`plan` 命令用于对模板中所定义资源的预览，主要用于以下几个场景：

- **预览当前模板中定义的资源**是否符合管理预期
- 如果当前模板已经存在对应的state文件，那么 `plan` 命令将会**展示模板定义与state文件内容的diff结果**，如果有变更，将会展示结果并在下方显示出来
- 对DataSource而言，执行 `plan` 命令，即可直接**获取并输出所要查询的资源及其属性**

```bash
# 资源预览
terraform plan
terraform plan -var-file="./vars/alpha.tfvars"
```

### 1.2 terraform apply：资源新建和变更

`apply` 命令用于实际资源的新建和变更操作，为了安全起见，在命令运行过程中增加了人工交互的过程，即需要手动确认是否继续，当然也可以通过 `--auto-approve` 参数来跳过人工确认的过程。

`apply` 命令适用于以下几种场景：

- **创建新的资源**
- 通过修改模板参数来**修改资源的属性**
- 如果从当前模板中删除某个资源的定义， `apply` 命令会**将该资源彻底删除**。可以理解为“资源的移除也是一种变更”。

```bash
# 资源新建和变更
terraform apply
terraform apply -var-file="./vars/alpha.tfvars"
```

### 1.3 terraform show：资源展示

`show` 命令用于**展示当前state中所有被管理的资源及其所有属性值**。

```bash
# 资源展示
terraform show
```

### 1.4 terraform destroy：资源释放

`destroy` 命令用于对资源的释放操作，为了安全起见，在命令执行过程中，也增加了人工交互的过程，如果想要跳过手动确认操作，可以通过 `--force` 参数来跳过。

`terraform destroy` 默认会**释放当前模板中定义的所有资源**，如果只想释放其中某个特定的资源，可以通过参数 `-target=<资源类型>.<资源名称>` 来指定。

```bash
# 资源释放
terraform destroy
terraform destroy -target=<资源类型>.<资源名称>
```

### 1.5 terraform import：资源导入

`import` 命令用于**将存量的云资源导入到terraform state中**，进而加入到Terraform的管理体系中，适用的场景包含但不限于以下几种：

- 从来没有使用Terraform管控过任何资源，当前所有的**存量云资源都是通过控制台，阿里云CLI，ROS或者直接调用API创建和管理的，现在想要切换为Terraform管理**
- 在不影响资源正常使用的前提下，重构资源模板中的资源定义
- 阿里云的Provider进行了兼容性升级，新版Provider对原有模板中所定义的资源支持了更多的参数，需要把最新的参数同步进来

```bash
# 资源导入
terraform import
```

### 1.6 terraform taint: 标记资源为“被污染”

`taint` 命令用于**把某个资源标记为“被污染”状态，当再次执行 `apply` 命令时，这个被污染的资源将会被先释放，然后再创建一个新的**，相当于对这个特定资源做了先删除后新建的操作。
命令的详细格式为： `terraform taint <资源类型>.<资源名称>` ，如：

```bash
# 标记资源为“被污染”
命令格式：terraform taint <资源类型>.<资源名称>

$ terraform taint alicloud_vswitch.this

Resource instance alicloud_vswitch.this has been marked as tainted.
```

### 1.7 terraform untaint：取消“被污染”标记

`untaint` 命令是 `taint` 的逆向操作，**用于取消“被污染”标记，使其恢复到正常的状态**。命令的详细格式和 `taint` 类似为： `terraform untaint <资源类型>.<资源名称>` ，如：

```bash
# 取消“被污染”标记
命令格式: terraform untaint <资源类型>.<资源名称>

$ terraform untaint alicloud_vswitch.this

Resource instance alicloud_vswitch.this has been successfully untainted.
```

### 1.8 terraform output：打印出参及其值

如果在模板中显示定义了 `output` 参数，那么这个output的值将在 `apply` 命令之后展示，但 `plan` 命令并不会展示，如果想随时随地快速查看output的值，可以直接运行命令 `terraform output` :

```bash
# 打印出参及其值
$ terraform output

vswitchId = vsw-gw8gl31wz********
```



## 2 状态管理常用命令

Terraform 对资源状态的管理，实际上是**对State文件中数据的管理**。

**State文件保存了当前Terraform管理的所有资源及其属性**，内容都是由Terraform自动存储的，为了保证数据的完整性，不建议手动修改State内容。

对State数据的操作可以通过 `terraform state` 命令来完成。

### 2.1 terraform state list：列出当前state中的所有资源

`state list` 按照 `<资源类型>.<资源名称>` 的格式列出当前state中存在的所有资源（包括datasource），如：

```bash
# 列出当前state中的所有资源
$ terraform state list

data.alicloud_slbs.default
alicloud_vpc.default
alicloud_vswitch.this
```

### 2.2 terraform state show：展示某一个资源的属性

`state show` 命令按照Key-Value的格式展示出特定资源的所有属性及其值，命令的完整格式为 `terraform state show <资源类型>.<资源名称>` ，如：

```bash
# 展示某一个资源的属性
命令格式: terraform state show <资源类型>.<资源名称>

$ terraform state show alicloud_vswitch.this

# alicloud_vswitch.this:
resource "alicloud_vswitch" "this" {
    availability_zone = "eu-central-1a"
    cidr_block        = "172.16.0.0/24"
    id                = "vsw-gw8gl31wz******"
    vpc_id            = "vpc-gw8calnzt*******"
}
```

### 2.3 terraform state pull：获取当前state内容并展示

 `state pull` 命令用于**原样展示当前state文件数据**，类似与Shell下的cat命令，如：

```bash
# 原样展示当前state文件数据
$ terraform state pull
{
  "version": 4,
  "terraform_version": "0.12.8",
  "serial": 615,
  "lineage": "39aeeee2-b3bd-8130-c897-2cb8595cf8ec",
  "outputs": {
    ***
    }
  },
  "resources": [
    {
      "mode": "data",
      "type": "alicloud_slbs",
      "name": "default",
      "provider": "provider.alicloud",
      ***
    },
    {
      "mode": "managed",
      "type": "alicloud_vpc",
      "name": "default",
      "provider": "provider.alicloud",
      ***
    }
  ]
}
```

### 2.4 terraform state rm：移除特定的资源

`state rm` 命令用于**将state中的某个资源移除，但是实际上并不会真正删除这个资源**，命令格式为： `terraform state rm <资源类型>.<资源名称>` ，如：

```bash
# 移除特定的资源
命令格式为： terraform state rm <资源类型>.<资源名称>

$ terraform state rm alicloud_vswitch.this

Removed alicloud_vswitch.this
Successfully removed 1 resource instance(s).
```

移除后，如果模板内容不变并且再次执行 `apply` 命令，将会新增一个同样的资源。

移除后的资源可以再次通过 `import` 命令再次加入，针对这部分的介绍，同样可以移步文章《[一文揭秘存量云资源的管理难题](https://yq.aliyun.com/articles/726955)》详细了解。

### 2.5 terraform state mv：变更特定资源的存放地址

如果想调整某个资源所在的state文件，可以通过 `state mv` 命令来完成，类似于Shell下的mv命令，这个命令的使用有多种选项，可以通过命令 `terraform state mv --help` 来详细了解。

本文只介绍最常用的一种： `terraform state mv --state=./terraform.tfstate --state-out=<target path>/terraform-target.tfstate <资源类型>.<资源名称A> <资源类型>.<资源名称B>` ，如：

```bash
#变更特定资源的存放地址
$ terraform state mv -state-out=../tf.tfstate alicloud_vswitch.this alicloud_vswitch.default
Move "alicloud_vswitch.this" to "alicloud_vswitch.default"
Successfully moved 1 object(s).
```

如上命令省略了默认的 `--state=./terraform.tfstate` 选项，命令最终的结果是将当前State中的VSwitch 资源移动到了上层目录下名为 `tf.tfstate` 的State中，并且将VSwitch的资源名称由"this"改为了"default"。

### 2.6 terraform refresh：刷新当前state

`refresh` 命令可以用来**刷新当前State的内容，即再次调用API并拉取最新的数据写入到state文件中**。

```bash
# 刷新当前state
$ terraform refresh
```



## 3 其他常用命令

除了资源和state的管理命令外，还有一些常用的应用在模板，provider等多种场景下的命令。

### 3.1 terraform init：初始化加载模块

`init` 用来初始化加载所需的模块，包括Provider，Provisioner，Module等。

```bash
# 初始化加载模块
# 包括Provider，Provisioner，Module等
$ terraform init
```

### 3.2 terraform graph：输出当前模板定义的资源关系图

每个模板定义的资源之间都存在不同程度的关系，如果想看**资源关系大图**，可以使用命令 `terraform graph` :

```bash
# 输出当前模板定义的资源关系图
$ terraform graph
digraph {
        compound = "true"
        newrank = "true"
        subgraph "root" {
                "[root] alicloud_vpc.default" [label = "alicloud_vpc.default", shape = "box"]
                "[root] alicloud_vswitch.this" [label = "alicloud_vswitch.this", shape = "box"]
                ******
                "[root] output.vswitchId" -> "[root] alicloud_vswitch.this"
                "[root] provider.alicloud (close)" -> "[root] alicloud_vswitch.this"
                                ******
                "[root] root" -> "[root] provider.alicloud (close)"
        }
}
```

该命令的结果还可以通过命令 `terraform graph | dot -Tsvg > graph.svg` 直接导出为一张图片（需要提前安装graphviz： `brew install graphviz` ）：

```bash
# 提前安装graphviz： 
$ brew install graphviz

# 输出当前模板定义的资源关系图
# 导出为一张图片
$ terraform graph | dot -Tsvg > graph.svg
```

![_2019_11_14_9_52_35](https://yqfile.alicdn.com/06e85f3c3b8d527d6684870bbf69113989cff48c.png)



### 3.3 terraform validate：验证模板语法是否正确

Terraform 模板的编写需要遵循其自身定义的一套简单的语法规范，编写完成后，如果想要检查模板是否存在语法错误或者在运行 `plan` 和 `apply` 命令的时候报语法错误，可以通过执行命令 `terraform validate` 来检查和定位错误出现的详细位置和原因。

```bash
# 验证模板语法是否正确
# 检查和定位错误出现的详细位置和原因
terraform validate
```



## 4 写在最后

本文主要介绍了一些在使用Terraform过程经常会遇到的一些命令，这些命令覆盖了模块下载，模板的检查，资源的管理，资源状态的管理等几个方面。

看得出这些命令使用起来并不复杂，不同命令的组合使用可以满足不同复杂的使用场景。本文只介绍了所有命令中的一部分，更多命令可以直接运行 `terraform` 或者 `terraform --help` 详细查看。

Terraform是面向客户端的工具，并且主要以命令驱动，因此对Terraform的学习最好的方法便是勤动手，多尝试，熟能生巧。