# 小马智行基础设施自动化（IaC）实践

更新时间：2022-09-22 11:46

## 背景

小马智行（pony.ai）成立于2016年，在硅谷、广州、北京、上海、深圳设立研发中心，并获得中美多地自动驾驶测试、运营资质与牌照。凭借人工智能技术领域的最新突破，已与丰田、现代、一汽、广汽等车厂建立合作。目前估值85亿美元。 小马智行中国内地业务使用阿里云作为基础架构的重要部分，承载了包括小马智行自研的Data Labeling、Robotaxi、Robotruck等业务。这些业务使用了包括ECS、RDS、SLB、堡垒机、云安全中心在内的众多产品。如何管理好这些基础组建给DevOps团队带来了不小的挑战。从业务需求出发，我们定义了三个目标：

1. 「**组件部署可评审**」：保障小马智行整体运维活动从需求收集、 架构设计、 代码编写最终到部署不会出现任何偏差。同时也能够保证代码编写符合我们的要求。
2. 「**组件部署版本化**」：保障小马智行任何基础设施生产的迭代都有迹可循。当出现极端情况的时候， 可以快速恢复到指定版本， 避免影响到业务。
3. 「**组件部署多环境一致**」：不同环境的一致性则能够保证小马智行的基础设施部署不会因为环境部署差异导致故障。

## 技术选型

从业内角度来看， 我们看到主要有三种主流公有云组件部署和管理方案：

1. 云服务商的控制台管理能力
2. 使用管理系统（可能自研或者购买）调用公有云API做操作
3. 基础设施即代码（IaC）框架

![img](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9076078461/p425323.png)

IaC方案，在国际的主流社区已经成为基础设施自动化的既定标准，也是最广泛使用的多云管理框架。 但是中国内地相对来说比较滞后，知道并使用的人还是比较少。 在这方面，拥有海外背景的小马智行具备技术的先发优势。我们发现，结合Git等代码管理工具，可以很好的解决部署可追溯以及版本化的问题。 所有运维团队部署的过程和变更细节都可以通过代码很好的管理起来。 当需要回退的时候，可通过Git的分支对基础设施进行回滚。

在社区生态方面，Terraform作为最优秀的、开源的IaC工具之一，已经被大量企业应用于生产环境。作为DevOps管理团队，我们则把主要精力投入到对应的基础架构逻辑代码的编写中去。

使用Terraform比调用各个云厂商API做二次开发不论是从开发量、复杂程度和运维难度都是极具优势的；帮助我们做到更好、更敏捷的部署。

最后，考虑到多云战略以及小马智行混合云的现状，结合Terraform的标准性、便捷性、易用性以及社区繁荣的特点，最终我们选择使用Terraform作为企业IaC落地的工具。

## 架构设计

小马智行团队选用以Terraform为技术核心的IaC解决方案，并通过如下所示的架构图最终落地到业务生产中：

![img](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9076078461/p425324.png)

在Terraform配置文件格式上，技术团队综合考虑小马智行已经使用众多的JSON应用的现状，为了保持一致以及方便代码Review，没有选择直接使用HCL格式， 而是选择了JSON格式。

在代码组织方面，小马智行选择了按业务维度来组织Terraform代码。比如：业务1使用了SLB、证书、ECS三种资源，那么在代码编写的时候会把三种资源都统一定义在一个Terraform文件上，类似如下：

```json
{
  "output": {
    "ecs_instance_1-private-ip": {
      "value": "${alicloud_instance.ecs_instance_1.private_ip}"
    }，
    "ecs_instance_2-private-ip": {
      "value": "${alicloud_instance.ecs_instance_2.private_ip}"
    }，
    "ponyai_business_1-slb-address": {
      "value": "${alicloud_slb.ponyai_business_1-slb.address}"
    }
  }，
  "provider": {
    "alicloud": {
      "region": "alicloud_region"
    }
  }，
  "resource": {
    "alicloud_instance": {
      "ecs_instance_1": {
        "availability_zone": "availability_zone_1"，
        "data_disks": [
          {
            "category": "cloud_essd"，
            "name": "data_volume"，
            "size": "xx"
          }
        ]，
        "host_name": "ecs_instance_1"，
        "image_id": "image_id_1"，
        "instance_name": "ecs_instance_1"，
        "instance_type": "ecs_instance_type"，
        "internet_charge_type": "PayByTraffic"，
        "internet_max_bandwidth_out": 10，
        "key_name": "key_name_1"，
        "security_groups": [
          "security_groups_1"
        ]，
        "system_disk_category": "cloud_essd"，
        "system_disk_size": "xx"，
        "tags": {
          "host_name": "ecs_instance_1"
        }，
        "vswitch_id": "vswitch_id_1"
      }，
      "ecs_instance_2": {
        "availability_zone": "availability_zone_2"，
        "data_disks": [
          {
            "category": "cloud_essd"，
            "name": "data_volume"，
            "size": "xx"
          }
        ]，
        "host_name": "availability_zone_2"，
        "image_id": "image_id_1"，
        "instance_name": "availability_zone_2"，
        "instance_type": "ecs_instance_type"，
        "internet_charge_type": "PayByTraffic"，
        "internet_max_bandwidth_out": 10，
        "key_name": "key_name_1"，
        "security_groups": [
          "security_groups_1"
        ]，
        "system_disk_category": "cloud_essd"，
        "system_disk_size": "xx"，
        "tags": {
          "host_name": "availability_zone_2"
        }，
        "vswitch_id": "vswitch_id_2"
      }
    }，
    "alicloud_slb": {
      "slb-1": {
        "address_type": "internet"，
        "internet_charge_type": "PayByTraffic"，
        "name": "slb_name"，
        "specification": "slb_specification"
      }
    }，
    "alicloud_slb_listener": {
      "slb-listener-1": {
        "backend_port": "xx"，
        "bandwidth": -1，
        "frontend_port": "xx"，
        "health_check": "on"，
        "health_check_connect_port": "xx"，
        "health_check_domain": "domain_name"，
        "health_check_type": "check_type"，
        "health_check_uri": "uri_1"，
        "load_balancer_id": "${alicloud_slb.slb-1.id}"，
        "protocol": "protocol_1"，
        "scheduler": "scheduler_1"，
        "server_certificate_id":
        "${alicloud_slb_server_certificate.slb-certificate-1.id}"，
        "server_group_id":
        "${alicloud_slb_server_group.slb-server-group-1.id}"
      }
    }，
    "alicloud_slb_server_certificate": {
      "slb-certificate-1": {
        "alicloud_certificate_id": "xx"，
        "alicloud_certificate_name": "xx"，
        "name": "certificate_1"
      }
    }，
    "alicloud_slb_server_group": {
      "slb-server-group-1": {
        "load_balancer_id": "${alicloud_slb.slb-1.id}"，
        "name": "slb-server-group"，
        "servers": {
          "port": "xx"，
          "server_ids": [
            "${alicloud_instance.ecs_instance_1.id}"，
            "${alicloud_instance.ecs_instance_2.id}"
          ]
        }
      }
    }
  }，
  "terraform": {
    "backend": {
      "s3": {
        "bucket": "bucket_name"，
        "dynamodb_table": "table"，
        "key": "key_1"，
        "profile": "profile_1"，
        "region": "region_1"
      }
    }，
    "required_providers": {
      "alicloud": {
        "source": "aliyun/alicloud"，
        "version": "xx"
      }
    }
  }
}
```

## 业务挑战

在Terraform代码实际编写中，我们逐步发现，对一些资源的需求，我们更关心其中的一些参数和特性。比如：

- 阿里云的ECS，我们更多的是关心ECS创建时的instance_type，instance_name， availability_zone等。
- 同一个业务在不同的部署环境仅仅只是一些资源的规格不太一样， 比如业务1在正式的生产环境中的SLB使用的规格为slb.s2.medium，在测试环境中的规格为slb.s1.small，如果每次部署仅仅只是不同参数的相同组件时，代码上需要重新写一遍，无疑代码的可读性和重用性会非常差。

## 解决方案

考虑到Terraform使用的是JSON格式的文件， 为了解决其代码重用和可读性问题， 我们引入开源的JSONnet模板语言来生成Terraform使用的JSON文件，同时封装了丰富的Utils。比如：我们对于创建ECS，封装了如下的Function：

```javascript
generateEcs（instance_name，
            availability_zone，
            vswitch_id，
            security_groups，
            instance_type，
            host_name，
            data_volume_size=null，
            system_disk_size=null，
            internet_charge_type="PayByTraffic"，
            image_id="ubuntu_18_04_x64_20G_alibase_20200914.vhd"，
            key_name="bootstrap-bot"，
            system_disk_category="cloud_essd"，
            internet_max_bandwidth_out=10，
            data_disk_category="cloud_essd"）: {
  instance_name: instance_name，
  availability_zone: availability_zone，
  vswitch_id: vswitch_id，
  security_groups: security_groups，
  instance_type: instance_type，
  internet_charge_type: internet_charge_type，
  image_id: image_id，
  system_disk_category: system_disk_category，
  [if system_disk_size != null then "system_disk_size"]:
    system_disk_size，
    key_name: key_name，
    internet_max_bandwidth_out: internet_max_bandwidth_out，
    host_name: host_name，
    data_disks: if data_volume_size != null then [
      {
        name: "data_volume"，
        size: data_volume_size，
        category: data_disk_category，
      }，
    ] else []，
      tags: {
        host_name: host_name，
      }，
}
```

这样上层在使用的时候直接调用该function就能生成对应的JSON部分。如下所示：

```json
alicloud_instance: {
  [host_config.host_name]:
    ecsUtils.generateEcs（
      instance_name=host_config.host_name，
      availability_zone=host_config.az，
      security_groups=$.ecs_security_groups，
      host_name=host_config.host_name，
      instance_type=$.ecs_instance_type，
      vswitch_id=VPC_output["vswitch-public-" + host_config.az].value，
      data_volume_size=$.ecs_data_volume_size，
      system_disk_size=$.ecs_system_disk_size
    ）
  for host_config in host_configs
}，
```

同时如果要调整的话直接调整对应的Utils函数即可， 不需要每个基础架构组件去调整。

对于不同的环境（正式， 预发布， 测试）只有少量组件参数不同的情况，我们也仅仅先定义好一个基础的模板，然后不同的环境import之后对需要调整的参数赋予不同的值即可。

通过这样的方式，可以做到对于小马智行的某项业务在不同环境的部署，我们仅需要遵循如下的代码路径即可。

备注：“generated/main.tf.JSON”是文件“main.tf.JSON.JSONnet”用JSONnet工具生成出来的JSON文件，是Terraform 最终执行的文件，“main.tf.JSON.JSONnet.output”则是Terraform生效之后产生的一些字段和字段值输出）：

```json
├── alicloud-region
│   ├── dev
│   │   ├── generated
│   │   │   └── main.tf.JSON
│   │   ├── main.tf.JSON.JSONnet
│   │   └── main.tf.JSON.JSONnet.output
│   ├── prod
│   │   ├── generated
│   │   │   └── main.tf.JSON
│   │   ├── main.tf.JSON.JSONnet
│   │   └── main.tf.JSON.JSONnet.output
│   └── staging
│       ├── generated
│       │   └── main.tf.JSON
│       ├── main.tf.JSON.JSONnet
│       └── main.tf.JSON.JSONnet.output
└── ponyai_business_1_base.libsonnet
```

以不同环境的SLB规格为例，在正式环境， 测试环境中， 我们仅仅需要把基础模板引入之后， 调整不同的参数即可。比如正式环境我们用如下的代码：

```json
local base = import "../../ponyai_business_1_base.libsonnet";
base {
  name: "ponyai_business_1_prod"，
  environment: "prod"，
  region: "alicloud_region"，
  slb_specification: "slb.s2.medium"
}
```

测试环境我们仅仅需要用如下的代码：

```json
local base = import "../../ponyai_business_1_base.libsonnet";
base {
  name: "ponyai_business_1_dev"，
  environment: "dev"，
  region: "alicloud_region"，
  slb_specification: "slb.s1.small"
}
```

这样可以实现较好的代码可读性和重用性。JSONnet也能够很方便的解决基础组件互相依赖导致在编写Terraform代码需要互相引用的问题。 比如阿里云上创建ECS， 需要提供VPC的ID，但是VPC一般是单独写在一个独立的Terraform文件中并单独生效，这个时候在代码层面创建ECS的Terrform代码则需要引用创建VPC生成出来的VPC id。以小马智行为例子， 我们用“main.tf.JSON”文件创建了一个VPC， 其ID按照我们的要求输出到了“main.tf.JSON.JSONnet.output”文件中：

```json
├── ali-cloud-region
│   ├── dev
│   │   ├── generated
│   │   │   └── main.tf.JSON
│   │   ├── main.tf.JSON.JSONnet
│   │   └── main.tf.JSON.JSONnet.output
│   └── prod
│       ├── generated
│       │   └── main.tf.JSON
│       ├── main.tf.JSON.JSONnet
│       └── main.tf.JSON.JSONnet.output
```

此时其"main.tf.JSON.JSONnet.output"文件输出如下：

```json
{
  "VPC_id": {
    "sensitive": false，
    "type": "string"，
    "value": "VPC_id_for_ponyai"
  }，
"vswitch-id": {
    "sensitive": false，
    "type": "string"，
    "value": "vswitch_public_id_for_ponyai"
  }
}
```

我们在其他需要引用的地方用如下的语法就能很方便的进行引用，从而避免我们在代码库里面直接放入生成出来的值：

```json
{
"ali-cloud-region": {
  prod: import "./ali-cloud-region/prod/main.tf.JSON.JSONnet.output"，
  }
}
```

通过上述一层层的技术封装，小马智行DevOps团队即使用了Terraform的技术及生态能力，也解决了业务调用的复杂性问题。让运维效率整体的到很大提升。

## 业务收益

通过使用IaC的方式， 我们对各个业务使用的各个阿里云的组件的参数都定义得非常明确，比如某业务使用了两台ECS（每一台的规格都是s6-c1m1.small且带有一块80G的系统盘和20G的数据盘）。结合Git管理，可以非常方便的进行Code Review；从而对整个部署过程进行把控。

如下图所示，我们团队在每次实际部署之前，都会通过Git进行代码Review整个PR，并在PR内进行讨论，最终确认好部署各个细节。并且，在未来的某一天，我们都有能力回溯到今天来看云基础设施发生了什么变更。

正是由于能够做到参数级别的Review， 我们可以保证最终的部署和最初的设计不会出现偏差。如果在最终部署的时候发现和原先设计的时候会有偏差， 我们也能够在Terraform代码编写的时候发现并及时调整原先设计。 同时我们在提交PR进行Review前， 都要求进行自测， 尽量避免出现Review多次， 结果发现连运行都运行不了的情况出现。

![img](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9076078461/p425325.png)

回顾整个使用阿里云Terraform落地IaC的过程，我们总结了四个我们认为的主要业务收益：

- 「更快」：同样的基础设施生产不需要再去控制台重复低效操作，生产周期大大缩短。为企业的业务决策和市场机遇响应提供了强有力的基础支撑。
- 「更可控」：基础设施代码化，历史任何时间点都可以回溯。将组织从控制台操作升级至更优雅、可控、可信、可回溯的体系化管理阶段。
- 「更高效」：多团队、跨团队同尤其是跨国协同效率提高。解决了时差、工作模式带来的效率降低的问题。
- 「更安全」：大大减少了人为误操作导致的生产事故。针对不同的环境采取不同的审批链路，做到技术+人的双重结合。为小马智行的业务保驾护航。

## 总结

- 管理模式升级

小马智行整体建设都是基于IaC的整体理念之下。目前企业内部已经将众多的阿里云组件抽象成内部自己的函数， 包括ECS，VPC， OSS， PUBLIC DNS， RAM， SLS， USERS等。

- 业务模式升级

DevOps团队或者业务团队需要使用阿里云组件的时候都会使用这些封装好的函数去拉取对应的阿里云组件，DevOps团队则专心维护并迭代好这些函数即可。运维效率大大提升。

- 运维模式升级

小马智行目前内部20+业务都是按照IaC思路通过Terraform100%落地，我们能够非常清晰的看到各个业务使用的阿里云组件的迭代史；也能够很好的进行Review，及时拒绝不合理的部署，保证线上环境的干净、整洁、高可靠的同时兼顾优秀的可扩展性。

## 作者介绍

小马智行DevOps团队