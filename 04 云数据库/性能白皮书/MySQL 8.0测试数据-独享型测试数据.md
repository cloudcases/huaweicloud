# 独享型测试数据

更新时间：2021-11-11 GMT+08:00



#### 关于IOPS

MySQL支持的IOPS取决于云硬盘（Elastic Volume Service，简称EVS）的IO性能，具体请参见《云硬盘产品介绍》中“[磁盘类型及性能介绍](https://support.huaweicloud.com/productdesc-evs/zh-cn_topic_0044524691.html)”的内容。

#### 独享型测试数据

**![img](https://res-img2.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-notice.svg)须知：**

如下表中的“连接数（压力测试值）”是RDS性能压力测试的结果，对于真实运行业务，请设置数据库实例参数**“max_connections”**的值。

| CPU(Core) | 内存(GB) | 连接数（压力测试值） | TPS  | QPS   | IOPS                                                         |
| --------- | -------- | -------------------- | ---- | ----- | ------------------------------------------------------------ |
| 2         | 8        | 2500                 | 590  | 11804 | 请参见[关于IOPS](https://support.huaweicloud.com/pwp-rds/rds_swp_mysql_12.html#rds_swp_mysql_12__section10402103213320) |
| 4         | 16       | 5000                 | 1357 | 27159 |                                                              |
| 8         | 32       | 10000                | 2364 | 47302 |                                                              |
| 16        | 64       | 18000                | 2876 | 57531 |                                                              |
| 32        | 128      | 30000                | 4796 | 95938 |                                                              |
| 64        | 256      | 60000                | 4996 | 97585 |                                                              |

| CPU(Core) | 内存(GB) | 连接数（压力测试值） | TPS  | QPS   | IOPS                                                         |
| --------- | -------- | -------------------- | ---- | ----- | ------------------------------------------------------------ |
| 4         | 32       | 10000                | 1435 | 28701 | 请参见[关于IOPS](https://support.huaweicloud.com/pwp-rds/rds_swp_mysql_12.html#rds_swp_mysql_12__section10402103213320) |
| 8         | 64       | 18000                | 2957 | 59146 |                                                              |
| 16        | 128      | 30000                | 4797 | 95948 |                                                              |
| 64        | 512      | 100000               | 4710 | 99932 |                                                              |

#### 独享型实例测试结果

图1 CPU:内存=1:4
![img](https://support.huaweicloud.com/pwp-rds/zh-cn_image_0000001173488654.png)

图2 CPU:内存=1:8
![img](https://support.huaweicloud.com/pwp-rds/zh-cn_image_0000001219250489.png)

**父主题：** [MySQL 8.0测试数据](https://support.huaweicloud.com/pwp-rds/rds_swp_mysql_04.html)