# 通过专线或VPN连接IEF

更新时间：2022-08-04 GMT+08:00

#### 操作场景

线下边缘节点无法通过公网访问IEF时，可以选择通过[云专线（DC）](https://www.huaweicloud.com/product/dc.html)或[VPN](https://www.huaweicloud.com/product/vpn.html)连接华为云VPC，然后通过[VPC终端节点](https://www.huaweicloud.com/product/vpcep.html)在VPC提供私密安全的通道连接IEF，从而使得线下边缘节点在无法访问公网时连接IEF。

#### 连接方案说明

纳管边缘节点部署应用时，**需要能够与IEF、SWR、OBS通信**，在无法通过公网连接的情况下，可以先**通过VPN或专线（DC）与华为云VPC连接，然后通过VPC终端节点服务，让VPC能够在内网访问IEF、SWR和OBS**，具体连接方案如[图1](https://support.huaweicloud.com/bestpractice-ief/ief_04_0006.html#ief_04_0006__fig887344831319)所示。

与IEF连接需要创建三个终端节点，分别为如下三个。

- ief-placement：用于边缘节点的纳管和升级。
- ief-edgeaccess：用于边缘节点与IEF发送边云消息。
- ief-telemetry：边缘节点上传监控和日志数据。

与SWR连接需要创建一个终端节点，与OBS通信需要创建OBS和DNS两个终端节点（OBS只能通过域名访问，需要通过DNS动态解析OBS的地址才能访问到）。

图1 通过专线或VPN连接IEF
![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0277719024.png)

#### 操作步骤

1. 创建VPC。

   创建VPC的方法请参见[创建虚拟私有云和子网](https://support.huaweicloud.com/usermanual-vpc/zh-cn_topic_0013935842.html)。

   您也可以使用已有VPC。

   **![img](https://res-static.hc-cdn.cn/aem/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-notice.svg)须知：**

   VPC网段不能与IDC的网段重复。

2. 使用DC或VPN连接VPC。

   具体连接方法请参见如下链接。

   - VPN：https://support.huaweicloud.com/qs-vpn/zh-cn_topic_0133627788.html
   - DC：https://support.huaweicloud.com/qs-dc/zh-cn_topic_0145790541.html

3. 创建IEF终端节点，使得边缘节点能够与IEF连接。

   共需要创建三个终端节点，分别为ief-placement、ief-edgeaccess和ief-telemetry。具体创建步骤如下。

   1. 登录[VPCEP控制台](https://console.huaweicloud.com/vpc/#/vpcep/vpceps)，单击右上角的“购买终端节点”。

   2. 选择IEF的终端节点和虚拟私有云。

      图2 创建IEF终端节点
      ![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0000001160267533.png)

   3. 单击“立即购买”，确认信息无误后单击“提交”，完成创建。

   

4. 创建SWR终端节点，使得边缘节点能够从SWR拉取容器镜像。

   创建方法与[创建IEF终端节点](https://support.huaweicloud.com/bestpractice-ief/ief_04_0006.html#ief_04_0006__li29971714134813)相同。

   图3 创建SWR终端节点
   ![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0000001113587732.png)

   

5. 创建DNS和OBS终端节点，使得边缘节点能够访问OBS。

   具体方法请参见[访问OBS](https://support.huaweicloud.com/usermanual-vpcep/vpcep_03_0300.html)。

6. 给边缘节点添加hosts配置。

   查询IEF和SWR的终端节点IP地址，共4个IP地址，配置到边缘节点的“/etc/hosts”文件中。

   图4 查询终端节点IP地址
   ![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0277472006.png)

   打开“/etc/hosts”文件，在文件末尾加入如下配置，使得访问IEF和SWR的域名指向终端节点的IP地址。

   **![img](https://res-static.hc-cdn.cn/aem/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-notice.svg)须知：**

   此处IP地址和域名需要根据实际情况修改，IP地址为上面步骤查询到的地址，不同区域的域名不相同，具体请参见[域名地址](https://support.huaweicloud.com/bestpractice-ief/ief_04_0006.html#ief_04_0006__section113248194294)。

   ```
   192.168.2.20	ief2-placement.cn-north-1.myhuaweicloud.com
   192.168.2.142	ief2-edgeaccess.cn-north-1.myhuaweicloud.com
   192.168.2.106   ief2-telemetry.cn-north-1.myhuaweicloud.com
   192.168.2.118   swr.cn-north-1.myhuaweicloud.com
   ```

7. 注册并纳管边缘节点，具体步骤请参见[边缘节点概述](https://support.huaweicloud.com/usermanual-ief/ief_01_0003.html)。



#### 域名地址

![img](https://res-static.hc-cdn.cn/aem/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-note.svg)说明：

铂金版ief-edgeaccess有单独的地址，请在IEF控制台“总览”页面查询，云端接入域名的取值即为edgeaccess域名。

| 区域           | 名称                                         | 域名                                        |
| -------------- | -------------------------------------------- | ------------------------------------------- |
| 华北-北京一    | ief-placement                                | ief2-placement.cn-north-1.myhuaweicloud.com |
| ief-edgeaccess | ief2-edgeaccess.cn-north-1.myhuaweicloud.com |                                             |
| ief-telemetry  | ief2-telemetry.cn-north-1.myhuaweicloud.com  |                                             |
| swr            | swr.cn-north-1.myhuaweicloud.com             |                                             |
| 华北-北京四    | ief-placement                                | ief2-placement.cn-north-4.myhuaweicloud.com |
| ief-edgeaccess | ief2-edgeaccess.cn-north-4.myhuaweicloud.com |                                             |
| ief-telemetry  | ief2-telemetry.cn-north-4.myhuaweicloud.com  |                                             |
| swr            | swr.cn-north-4.myhuaweicloud.com             |                                             |
| 华南-广州      | ief-placement                                | ief-placement.cn-south-1.myhuaweicloud.com  |
| ief-edgeaccess | ief-edgeaccess.cn-south-1.myhuaweicloud.com  |                                             |
| ief-telemetry  | ief-telemetry.cn-south-1.myhuaweicloud.com   |                                             |
| swr            | swr.cn-south-1.myhuaweicloud.com             |                                             |
| 华东-上海一    | ief-placement                                | ief-placement.cn-east-3.myhuaweicloud.com   |
| ief-edgeaccess | ief-edgeaccess.cn-east-3.myhuaweicloud.com   |                                             |
| ief-telemetry  | ief-telemetry.cn-east-3.myhuaweicloud.com    |                                             |
| swr            | swr.cn-east-3.myhuaweicloud.com              |                                             |
| 华东-上海二    | ief-placement                                | ief2-placement.cn-east-2.myhuaweicloud.com  |
| ief-edgeaccess | ief2-edgeaccess.cn-east-2.myhuaweicloud.com  |                                             |
| ief-telemetry  | ief2-telemetry.cn-east-2.myhuaweicloud.com   |                                             |
| swr            | swr.cn-east-2.myhuaweicloud.com              |                                             |

[上一篇：工业IoT边缘实时流分析](https://support.huaweicloud.com/bestpractice-ief/ief_04_0005.html)

[下一篇：使用开源C语言库连接MQTT Broker](https://support.huaweicloud.com/bestpractice-ief/ief_04_0007.html)