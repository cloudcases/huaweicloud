# Terraform 的复杂 Variable 类型

[大可不加冰](https://www.zhihu.com/people/he-zi-jie)

最近在研究可复用的 Terraform Module，在如何设计复杂的 Variable 类型这个问题上遇到了一些取舍困难，所以把思考写在这里供参考。

## **什么是复杂 Variable 类型**

设计 Terraform 的 Variable 时，有时你会想要传入一个复杂类型的对象，例如我们在 Module 中创建一个 Subnet 时，会需要一组有关 Virtual Network 的信息。

我们可以选择通过一个 Variable 让调用者**传入一个对象**来传递这些信息，比如这样：

```Terraform
variable "virtual_network" {
  type = object({
    id       = string
    name     = string
    location = string
  })
    
  description = <<-EOT
    The virtual network which to attach the subnet. Changing this forces some new resources to be created.
    id: The virtual NetworkConfiguration ID.
    name: The name of the virtual network.
    location: The location/region where the virtual network is created.
  EOT
  nullable    = false
  validation {
    condition     = var.virtual_network.id != null
    error_message = "`virtual_network.id` is required."
  }
  validation {
    condition     = var.virtual_network.name != null
    error_message = "`virtual_network.name` is required."
  }
  validation {
    condition     = var.virtual_network.location != null
    error_message = "`virtual_network.location` is required."
  }
}
```

当然我们也可以将之分拆为三个 Variable，例如：

```Terraform
variable "virtual_network_id" {
  type        = string
  description = "The virtual NetworkConfiguration ID."
  nullable    = false
}

variable "virtual_network_name" {
  type        = string
  description = "The name of the virtual network."
  nullable    = false
}

variable "virtual_network_location" {
  type        = string
  description = "The location/region where the virtual network is created."
  nullable    = false
}
```

两者相比，后者看起来更加简洁，但存在如下的两个劣势：

1. 无法进行涉及到其他 `variable` 的验证

让我们假设如果 `var.virtual_network_id` 也可以为 `null`，其他两个参数的验证条件为：假如 `var.virtual_network_id` 为 `null`，那么他们俩就不做验证，因为我们不会使用这组参数；假如 `var.virtual_network_id` 不为 `null`，那么其他两个参数就不可以为 `null`。

如果是使用 `object` 类型，那么 `validation` 可以是：

```Terraform
 validation {
    condition     = var.virtual_network == null ? true : (var.virtual_network.id == null || var.virtual_network.name != null)
    error_message = "`virtual_network.name` is required when `virtual_network.id` is presented."
}
```

由于 `name` 和 `id` 属于同一个 `variable`，所以验证条件里可以同时引用他们俩进行验证，而拆分成独立的 `variable` 后则不行。

另外为什么上述表达式的 `condition` 要写得这么复杂？这里需要解释一下。

Terraform 的表达式设计中有一个重大缺陷，布尔计算是不短路的。在普通的编程语言里，如果我们写这样的表达式：

```Terraform
validation {
    condition     = var.virtual_network == null || (var.virtual_network.id == null || var.virtual_network.name != null)
    error_message = "`virtual_network.name` is required when `virtual_network.id` is presented."
}
```

如果 `var.virtual_network` 为 `null`，那么 `||` 后的表达式将不会被计算，也就不会触发空引用错误。

只可惜，Terraform 的布尔计算是不短路的，即使 `||` 前的表达式足以返回，它还是会坚持计算后者，从而引发空引用，所以需要我们用三目运算符进行人工短路。

1. 作为创建资源的条件时可能会引发问题

让我们看另一个例子。例如这样一组 `variable`：

```Terraform
variable "route_table_id" {
  type        = string
  description = "The ID of the Route Table which should be associated with the Subnet. Changing this forces a new route table association to be created."
  default     = null
}

variable "route_table_name" {
  type        = string
  description = "The name of the route table. Changing this forces a new route table association to be created."
  default     = null
}
```

假如我们的 Terraform 代码中依赖该参数来判断是否创建某种资源，例如：

```Terraform
resource "azurerm_route_table" "this" {
  count               = var.route_table_id == null ? 1 : 0
  location            = local.location
  name                = coalesce(var.new_route_table_name, "${var.subnet_name}-rt")
  resource_group_name = var.resource_group_name
}
```

那么如果我们用如下代码调用该模块时将会遇到问题：

```Terraform
resource "azurerm_route_table" "rt" {
  location            = azurerm_resource_group.rg.location
  name                = "${module.label.id}-rt"
  resource_group_name = azurerm_resource_group.rg.name
  tags                = module.label.tags
}

module "public" {
  source              = "../../"
  address_prefixes    = ["10.0.1.0/24"]
  ......
  route_table_id      = azurerm_route_table.rt.id
  route_table_name    = azurerm_route_table.rt.name
}
```

这里的问题在于，`module.public` 的 `var.route_table_id` 直接使用了另一个 `resource`，也就是 `azurerm_route_table.rt` 的输出，并将其用在了 `azurerm_route_table.this` 资源的 `count` 表达式内。

Terraform 要求在 Plan 阶段必须得到确定的执行计划，也就是说，计算 `count` 或是 `for_each` 表达式的时候，不可以引用任何其他 `resource` 的输出，除非使用 `-target` 参数首先将涉及到的 `resource` 创建出来。

这个问题可以通过将其包装为一个 `object` 来解决。

```Terraform
variable "route_table" {
  type = object({
    id   = string
    name = string
  })
  default     = null
  description = <<-EOT
    Leave this parameter null would create a new route table.
    id: The ID of the Route Table which should be associated with the Subnet. Changing this forces a new route table association to be created.
    name: The name of the route table. Changing this forces a new route table association to be created.
  EOT
  validation {
    condition     = var.route_table == null ? true : var.route_table.id != null
    error_message = "`route_table.id` is required when `route_table` is not null."
  }
}
```

之前使得我们陷入困境的是因为 `var.route_table_id` 为另一个 `resource` 的输出时 Terraform 无法在 Plan 阶段判断该参数的值是否为 `null`，虽然我们都清楚这是不可能的；使用 `object` 将参数封装一下以后，即使是在 Plan 阶段我们不知道 `object` 内部的值，但我们仍然可以轻松地判断 `object` 本身是否为 `null`：

```Terraform
module "public" {
  source              = "../../"
  address_prefixes    = ["10.0.1.0/24"]
  ......
  route_table = {
    id   = azurerm_route_table.rt.id
    name = azurerm_route_table.rt.name
  }
}
```

## **`object` 的缺点**

使用 `object` 类型也有一些明显的缺点，首先是 `description` 会很难写。

`variable` 的 `description` 看起来是为原生数据类型（`number`、`string`等）设计的，用来描述一个带有层次结构的复杂类型时就显得捉襟见肘。

其次是 `object` 目前还没有正式支持 `optional`。例如这样一段代码：

```Terraform
variable "network_rules" {
  default = null
  type = object({
    bypass = list(string)
    ip_rules = list(string)
    virtual_network_subnet_ids = list(string)
  })
}

resource "azurerm_storage_account" "sa" {
  name = random_string.name.result
  location = var.location
  resource_group_name = var.resource_group_name
  account_replication_type = var.account_replication_type
  account_tier = var.account_tier

  dynamic "network_rules" {
    for_each = var.network_rules == null ? [] : list(var.network_rules)

    content {
      bypass = network_rules.value.bypass
      ip_rules = network_rules.value.ip_rules
      virtual_network_subnet_ids = network_rules.value.virtual_network_subnet_ids
    }
  }
```

如果我们允许 `bypass`、`ip_rules` 和 `virtual_network_subnet_ids` 为 `null`，并且我们**不打算**设置 `bypass`，我们仍然必须显式设置它：

```Terraform
resource "azurerm_storage_account" "sa" {
  name = random_string.name.result
  location = var.location
  resource_group_name = var.resource_group_name
  account_replication_type = var.account_replication_type
  account_tier = var.account_tier

  dynamic "network_rules" {
    for_each = var.network_rules == null ? [] : list(var.network_rules)

    content {
      bypass = null
      ip_rules = network_rules.value.ip_rules
      virtual_network_subnet_ids = network_rules.value.virtual_network_subnet_ids
    }
  }
```

因为 Terraform 中没有 Class 的设计，完全是依靠匹配对象的类型结构是否完全相同来判断类型是否匹配，所以如果省略了 `bypass`，会被认为类型不符。

在 Terraform 的 [https://github.com/hashicorp/terraform/issues/19898](https://link.zhihu.com/?target=https%3A//github.com/hashicorp/terraform/issues/19898) 草案中提出了 `optional` 的设计：

```Terraform
variable "network_rules" {
  default = null
  type = object({
    bypass = optional(list(string))
    ip_rules = optional(list(string))
    virtual_network_subnet_ids = optional(list(string))
  })
}
```

设置为 `optional` 的字段将允许在传值时省略。

但很遗憾的是，目前该功能仍然处于试验阶段，必须显式启用实验特性标记才可以使用。

由于 `optional` 的缺失，又带来了 `object` 类型的下一个问题，很难在保证向前兼容的前提下进行升级。假设我们在后续的开发中需要扩展该 `object`，添加新的字段，这种做法会使得引用旧版本 Module 的代码在更新引用的 Module 版本后惊讶地发现类型错误，因为 `object` 添加了新字段，使得 Terrform 会抱怨类型不匹配。

## **使用 `map`？**

相比起 `object` 的这些缺陷来说，使用 `map` 可以解决部分的问题。还是刚才的例子，如果使用 `map`，那么大概会是这样：

```Terraform
variable "network_rules" {
  default = null
  type = map(string)
}

resource "azurerm_storage_account" "sa" {
  name = random_string.name.result
  location = var.location
  resource_group_name = var.resource_group_name
  account_replication_type = var.account_replication_type
  account_tier = var.account_tier

  dynamic "network_rules" {
    for_each = var.network_rules == null ? [] : list(var.network_rules)

    content {
      bypass = lookup(network_rules, "bypass", null) == null ? null : split(",", lookup(network_rules, "bypass"))
      ip_rules = lookup(network_rules, "ip_rules", null) == null ? null : split(",", lookup(network_rules, "ip_rules"))
      virtual_network_subnet_ids = lookup(network_rules, "virtual_network_subnet_ids", null) == null ? null : split(",", lookup(network_rules, "virtual_network_subnet_ids"))
    }
  }
}
```

在没有使用 `optional` 的情况下，手工编写了大量的 `lookup` 调用实现了类似 `optional` 的逻辑。

但是 `map` 也有一些缺陷。首先我们可以从上面的代码中看到，`variable` 参数的 `type` 只有简单的 `map(string)`。Module 的调用者假如不仔细阅读 `description` 或是样例代码，就无法了解到该参数的具体结构。

其次是 `object` 与 `map` 类型的一个根本性的区别。这两个类型在使用上几乎是相同的，很多场合可以通过隐式类型转换直接互转，但只有一种场景例外：

- `map` 的值必须是同一种类型，而 `object` 的值可以是不同类型。

例如这样一个 `variable`：

```Terraform
variable "subnet_delegations" {
  type = list(object({
    name               = string
    service_delegation = object({
      name    = string
      actions = list(string)
    })
  }))
  default     = null
  description = <<-EOT
    Details: https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet#delegation
    name: A name for this delegation.
    service_delegation:
      name: The name of service to delegate to. Possible values include
      actions: The name of service to delegate to. Possible values include
  EOT
  validation {
    condition     = var.subnet_delegations == null ? true : alltrue([for d in var.subnet_delegations : (d != null)])
    error_message = "`subnet_delegations`'s element cannot be null."
  }
  validation {
    condition     = var.subnet_delegations == null ? true : alltrue([for n in var.subnet_delegations.*.name : (n != null)])
    error_message = "`subnet_delegation.name` is required when `subnet_delegation` is not null."
  }
  validation {
    condition     = var.subnet_delegations == null ? true : alltrue([for d in var.subnet_delegations.*.service_delegation : (d != null)])
    error_message = "`subnet_delegation.service_delegation` is required when `subnet_delegation` is not null."
  }
  validation {
    condition     = var.subnet_delegations == null ? true : alltrue([for n in var.subnet_delegations.*.service_delegation.name : (n != null)])
    error_message = "`subnet_delegation.service_delegation.name` is required when `subnet_delegation` is not null."
  }
}
```

`name` 的类型是 `string` 而 `service_delegation` 的类型是另一个 `object`，像这样的结构就不是 `map` 可以描述的了。



## **小结**

由于 Terraform 自身的一些设计问题，导致在设计 Module 的 Variable 类型时可能没有一个可以遵循的统一的标准。

假如多个独立的 `variable` 那么 `descritpion` 的可读性最好，但如果涉及到作为是否创建某个资源的判断条件时，就要考虑使用 `object` 或是 `map` 进行封装；`object` 可以包含不同类型的成员，但很难在不破坏向前兼容的情况下进行扩展；`map` 的扩展性稍好，但无法包含不同类型的成员。

更加麻烦的事如果想要从其中一种用法转到另一种用法，很可能会破坏向前兼容，使得调用者必须修改自己的调用代码才可以升级 Module 版本。

我们可能遵循的一些原则可以是：

- 尽量**避免根据某种条件创建资源**，而是使用依赖注入，**避免被迫使用 `object` 封装一个参数**
- 如果能**确定 `object` 的结构将保持长时间的稳定，并且所有成员都是必填项时，可以考虑优先使用 `object`**
- 如果第二条不满足，或是同一 Module 中包含有其他复杂类型参数是使用 `map` 类型更为合理的，那为了风格的统一，似乎应该全部使用 `map` 类型
- 使用 `map` 作为 `variable` 的类型时，请确保在样例代码中有完善的调用示范，这样用户在阅读 `description` 后仍然感到困惑的情况下，可以通过阅读样例代码来学习该参数的各种字段的含义
- 如果 `variable` 的目的是作为 `dynamic` 块的来源被使用于 `resource` 内部的，由于 `provider` 结构的相对稳定以及这种 `dynamic` 块一般使用的场景里会涉及到类型不同的成员，所以 `object` 似乎更合适。

发布于 2022-03-21 08:21



[Terraform 的复杂 Variable 类型 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/484464792)