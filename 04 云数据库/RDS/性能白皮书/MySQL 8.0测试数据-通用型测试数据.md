# 通用型测试数据

更新时间：2021-11-11 GMT+08:00



#### 关于IOPS

MySQL支持的IOPS取决于云硬盘（Elastic Volume Service，简称EVS）的IO性能，具体请参见《云硬盘产品介绍》中“[磁盘类型及性能介绍](https://support.huaweicloud.com/productdesc-evs/zh-cn_topic_0044524691.html)”的内容。

**![img](https://res-img2.huaweicloud.com/content/dam/cloudbu-site/archive/china/zh-cn/support/resource/framework/v3/images/support-doc-new-notice.svg)须知：**

如下表中的“连接数（压力测试值）”是RDS性能压力测试的结果，对于真实运行业务，请设置数据库实例参数**“max_connections”**的值。

#### 通用型实例测试列表

| CPU(Core) | 内存(GB) | 连接数（压力测试值） | TPS  | QPS   | IOPS                                                         |
| --------- | -------- | -------------------- | ---- | ----- | ------------------------------------------------------------ |
| 2         | 4        | 1500                 | 395  | 7914  | 请参见[关于IOPS](https://support.huaweicloud.com/pwp-rds/rds_swp_mysql_11.html#rds_swp_mysql_11__section10402103213320) |
| 4         | 8        | 2500                 | 1013 | 20276 |                                                              |
| 8         | 16       | 5000                 | 1591 | 31829 |                                                              |

| CPU(Core) | 内存(GB) | 连接数（压力测试值） | TPS  | QPS   | IOPS                                                         |
| --------- | -------- | -------------------- | ---- | ----- | ------------------------------------------------------------ |
| 2         | 8        | 2500                 | 571  | 11437 | 请参见[关于IOPS](https://support.huaweicloud.com/pwp-rds/rds_swp_mysql_11.html#rds_swp_mysql_11__section10402103213320) |
| 4         | 16       | 5000                 | 1349 | 26996 |                                                              |
| 8         | 32       | 10000                | 2308 | 46176 |                                                              |

#### 通用型实例测试结果

图1 CPU:内存=1:2
![img](https://support.huaweicloud.com/pwp-rds/zh-cn_image_0000001219243811.png)

图2 CPU:内存=1:4
![img](https://support.huaweicloud.com/pwp-rds/zh-cn_image_0000001173803776.png)

**父主题：** [MySQL 8.0测试数据](https://support.huaweicloud.com/pwp-rds/rds_swp_mysql_04.html)