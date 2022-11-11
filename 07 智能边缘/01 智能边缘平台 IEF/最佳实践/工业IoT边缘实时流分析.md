# 工业IoT边缘实时流分析

更新时间：2022-08-04 GMT+08:00

终端设备可以产生大量的数据，为了减少数据上云的数据量或提高数据处理实时性，有时需要**在靠近数据产生的地方对其进行分析处理**。智能边缘平台可以和[数据湖探索服务（DLI）](https://support.huaweicloud.com/dli/index.html)结合，通过**在边缘节点上部署系统提供的流计算应用，将实时流计算能力从云端延伸到边缘**。然后**通过数据湖探索服务编辑流处理作业并下发到边缘执行**，可以帮助您在边缘快速实现对流数据的实时、快速、准确地分析处理。

图1 边缘实时流分析
![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0294509513.png)

#### 前提条件

- 已开通IEF和DLI服务。
- 已成功注册并纳管边缘节点。

#### 部署应用

1. 登录[边缘应用中心](https://console.huaweicloud.com/ief2.0/#/app/edgeMarket/application/list)，选择“边缘Flink”应用，单击“部署应用”**。**

2. 输入应用名称，配置容器规格（注意：配额不得小于默认值，否则会部署失败），关联边缘节点，单击“创建”。

3. 登录[DLI管理控制台](https://console.huaweicloud.com/dli)，选择左侧导航栏的“作业管理 > Flink作业”。

4. 单击右上角“创建作业”。

   图2 创建作业
   ![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0294506361.png)

   

5. 配置作业信息。

   类型选择“Flink Edge SQL”，填写名称，描述、模板和标签均为可选填内容，若使用已存在的模板创建作业，可帮助您快速完成新建。

6. 单击“确定”，进入“编辑”页面。

   图3 编辑作业
   ![点击放大](https://support.huaweicloud.com/bestpractice-ief/zh-cn_image_0294507082.png)

   

7. 根据需要编辑Flink SQL边缘作业，处理终端设备数据。当前支持edgehub类型 、encode为json或csv的输入输出，具体SQL语法可参考[Flink SQL语法参考](https://support.huaweicloud.com/sqlref-flink-dli/dli_08_0075.html)。

   参考示例：功能为输出学生成绩大于或者等于80分的姓名和成绩。

   ```
   create source stream student_scores(name string, score int) with (
   type = "edgehub",
   topic = "abc",
   encode = "json",
   json_config = "score = student.score; name=student.name"
   );
   create sink stream excellent_students(name string, score int) with (
   type = "edgehub",
   topic = "abcd",
   encode = "csv",
   field_delimiter = ","
   );
   insert into excellent_students select name, score from student_scores where score >= 80;
   ```

8. 在界面右侧选择作业所需并行数和作业所属边缘节点，支持选择多个边缘节点部署作业。

9. 单击界面右上角“启动”，进入作业费用清单界面，单击“立即”。可在作业管理界面查看作业运行状态，单击具体作业可查看作业详情、作业监控、执行计划等信息。

#### 验证作业运行效果

1. 登录任一节点（该节点需与边缘节点网络互通），安装mosquitto软件。

   mosquitto软件的下载请参见https://mosquitto.org/download/。

2. 执行如下命令订阅数据。

   **mosquitto_sub -h 127.0.0.1** **-t** *abcd*

   abcd为作业中定义的topic名称。

3. 打开一个新的窗口，执行如下命令发布数据。

   **mosquitto_pub -h 127.0.0.1** **-t** *abcd* **-m '{"student":{"score":90,"name":"1bc2"}}'**

   abcd为作业中定义的topic名称。

   数据发布后，在[2](https://support.huaweicloud.com/bestpractice-ief/ief_04_0005.html#ief_04_0005__li23981124175411)中订阅就能收到对应的数据。