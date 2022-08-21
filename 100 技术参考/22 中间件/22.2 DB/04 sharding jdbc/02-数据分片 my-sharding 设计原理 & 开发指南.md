# 数据分片 my-sharding 设计原理 & 开发指南

[![img](https://upload.jianshu.io/users/upload_avatars/26997955/e90c58a7-0a4f-48c2-a3e1-240c1b35cbc2?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/ef31e1e35362)

[leehomlee](https://www.jianshu.com/u/ef31e1e35362)关注

12022.01.25 15:10:49字数 2,887阅读 208

# 1    背景 

互联网时代数据日益增长，单表已满足不了数据存储，当前涌现很多 nosql，newsql 数据存储，但目前系统设计概念基于关系数据库，nosql，newsql 不能很好适应，基于关系数据库的数据分片显然是目前比较好的解决方案

组件基于 sharding-jdbc 3.1.0，针对 sharding-jdbc 使用上痛点和缺失在源码级别解决和增强，附有使用指南设计

# 2    核心概念

## 2.1   逻辑表

水平拆分的数据库/表的相同逻辑和数据结构表的总称。例：订单数据根据主键尾数拆分为 10 张表，分别是 t_order_0 到 t_order_9，他们的逻辑表名为 t_order。

## 2.2   真实表

在分片的数据库中真实存在的物理表。即上个示例中的 t_order_0 到 t_order_9。

## 2.3   绑定表

指分片规则**一致**的主表和子表。例如：t_order 表和 t_order_item 表，均按照 order_id 分片，则此两张表互为绑定表关系。绑定表之间的多表关联查询不会出现笛卡尔积关联，关联查询效率将大大提升。

## 2.4   逻辑库/真实库

与逻辑表/真实表类似，同时真实库配置连接参数，也是连接数据源

# 3    架构设计

![img](https://upload-images.jianshu.io/upload_images/26997955-86676931f631562c.png?imageMogr2/auto-orient/strip|imageView2/2/w/912/format/webp)

**sharding-spring** my-sharding 的 spring boot starter，接管 sharding-jdbc 启动，构建库表资源模型，初始化自有的分片策略框架，按 sharding-jdbc 要求初始化库表列表，即 data nodes；构建分片数据源

**jdbc 代理**sharding-jdbc 自身实现一套 jdbc 代理，嵌入分库分表的逻辑，无缝地接入持久层

**sql 解释**/**sql 路由**/**sql 改写/sql 执行**/**结果归并**sharding-jdbc 分片的核心功能模块

**apm 埋点 手动埋探针，采集 ql 解释/sql 执行执行性能**

**my-sharding  sharding-jdbc 扩展和改造** 

3.1   shardsphere

下图展现 shardingsphere 整体模块

![img](https://upload-images.jianshu.io/upload_images/26997955-c796856e37a94b15.png?imageMogr2/auto-orient/strip|imageView2/2/w/915/format/webp)

**sharding-jdbc/sharding-proxy**sharding-sphere 呈现的两个产品，sharding-jdbc 轻量级分片组件，内部实现一套 jdbc 标准组件，嵌入分库分表逻辑；sharding-proxy 数据库代理，实现数据库的交互协议

**sharding-core** sql 改写，解释，执行等数据分片的核心功能，sharding-jdbc/sharding-prox 代理壳不一样，共用分片的核心逻辑

**sharding-orchetration**分片治理，包括注册中心，配置中心，支持 zookeeper 分布式配置及配置变更

**sharding-transaction** 分片事务模块，支持 xa，v4.0 支持 base 事务，集成 seata

**sharding-opentracing** 原生支持 opentracing 0.30.0，my-sharding 重写代码，支持 0.31.0，至此新的链路跟踪平台，如 zipkin server 5.x

**sharding-spring** 包括分片的 starter，事务 starter，由于 my-sharding 自有的 spring boot starter，只使用事务 starter

***shardingsphere 规划还有一款产品，sharding-sidecar，称为数据库网格，规划中，还没实现**

3.2   my-sharding

![img](https://upload-images.jianshu.io/upload_images/26997955-ba6c5506d9e28a3d.png?imageMogr2/auto-orient/strip|imageView2/2/w/794/format/webp)

service 资源服务，库表资源管理 api，表的生成和扩充

resource 库表资源模型

strategy 分片策略/算法

function y=f(x), 投射函数，分片键值(x)投射到库/表(y)集合，包括预处理，如，取整，字符串截取，日期截取；投射，取模，=，表达式

springboot starter/springboot starter apm

3.2.1 问题                                           

Ø 原生提供的 inline 表达式的分片算法，不支持日期，字符等复杂分片算法, 需要自行扩展

Ø sql 不兼容，哪怕是肉眼看到是路由到一个库，主要是解释器问题

Ø opentracing 版本 0.30，不兼容主流版本的链路服务，如 zipkin 5.x

3.2.2   解决方案

\1.  库表资源建模， 从而掌握库表控制权，分库分表实质库表资源分配，是算法框架的核心，也是在 shardingsphere 解释不了 sql，绕过 sharding-jdbc 自行生成物理 sql 的关键

\2.  集合映射分片算法框架

Ø sharding-jdbc 自身只实现 inline 表达式的 hash 分片算法，不能满足实际需要，因此开发了基于集合分布分片算法框架，支持不同维度的分片算法

Ø**数据分片抽象成数学概念，就是某个数值或者范围值(即)分片主键值)与特定集合(真实表名称)的映射 f(x)->y, x 为分片键值，y 真实表集合**

这里要求真实表命名：逻辑表名称{后缀}，其中‘后缀‘是集合元素值

Ø 库表资源集合分布类型：

\>整数集合，例如，userId 取模，年龄范围

\>日期集合，年，月，周，日，时等

\>字符集合，如，“a~z”,“A~Z”

\>离散点集合，没有顺序数值集合

\>并集合，不连续同类型的集合的并

**示例**：2 个数据库，存放 6 个月数据，采用分片算法，按月分库，按日分表

考虑到数据分布不均匀，按经验，146 月分到一个库，235 月分到另一个库

库资源集合，采用离散点集合，[1,4,6]， [2,3,5]；

表资源集合，采用日集合并集，{[1.1~1.31]&[4.1~4.30]&[6.1~6.30]}，{[2.1~1.28]&[3.1~3.31]&[5.1~5.31]}

3.2.2.1     库表资源模型

库表资源模型

![img](https://upload-images.jianshu.io/upload_images/26997955-bc35880f73383f15.png?imageMogr2/auto-orient/strip|imageView2/2/w/913/format/webp)

3.2.2.1.1 库表集合模型

库表资源看成连续的集合，分库分表就是根据 sql 分片键值，映射到具体库和表元素，已实现整形，字符，日期等集合类型和映射

![img](https://upload-images.jianshu.io/upload_images/26997955-a859c68b718bc5a4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

3.2.2.2     定位真实表时序

分片路由计算变成，数值定位到哪个集合里(使用并集的情况下)的哪个元素的计算

![img](https://upload-images.jianshu.io/upload_images/26997955-dd335b3421649cbd.png?imageMogr2/auto-orient/strip|imageView2/2/w/912/format/webp)

1). 分片算法调用资源管理器，传入逻辑表名称查找逻辑表

2). 资源管理器找出逻辑表并返回

3). 分片算法调用逻辑表 locate 方法，传入分片键值

4). 逻辑表调用资源集合 locate 方法定位真实表 (集)

5). 数值预处理和值定位计算，预处理: 取模，日期截取；计算: =，in；range

3.2.2.3     性能监控 apm

apm 应用性能监控，发现性能瓶颈，持续改进

手动埋点，分别在 sql 解释，sql 执行埋点，监控其性能

![img](https://upload-images.jianshu.io/upload_images/26997955-95624cd4f582bd93.png?imageMogr2/auto-orient/strip|imageView2/2/w/780/format/webp)

apm 监控台示例

\1.   执行概览

开始(rootInvoke)->分析(parsesql)->执行(executesql)，下面示例不带分片键条件的 select sql，全域扫描，sql 路由到所有的库/表执行

![img](https://upload-images.jianshu.io/upload_images/26997955-2a1dcd58895d40c2.png?imageMogr2/auto-orient/strip|imageView2/2/w/753/format/webp)

\2.   分析阶段

执行时间，包括自定义标签，逻辑 sql

![img](https://upload-images.jianshu.io/upload_images/26997955-c47a7d20cef935dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/465/format/webp)

\3.   执行阶段

![img](https://upload-images.jianshu.io/upload_images/26997955-8648f62836f0e263.png?imageMogr2/auto-orient/strip|imageView2/2/w/461/format/webp)

实际执行 sql，实际路由到哪个库，哪个表

3.3   代码下载(收费)

mysharding 修改了 shardingsphere 分片策略源码，为了不混淆原生，修改 jar 包名字，包结构没变

[https://download.csdn.net/download/szlhj/77817675](https://links.jianshu.com/go?to=https%3A%2F%2Fdownload.csdn.net%2Fdownload%2Fszlhj%2F77817675)

测试包括，整形/日/月/字符的数据分片场景，例如，按店铺 id 取模分库+按月分表等，并接入 opentracing，apm 监

3.4   开发指南

3.4.1   分片设计

分库分表成败 90%+在于分片设计，分片设计 99%的关键在于表分析，分析表之间的关系，sql，选择分片键，

分片键的选择，总会有利于一些 sql，而另一些 sql 因分片不能在单库执行，需要拆分，或使用搜索引擎

3.4.2   分片配置

分片配置使用 xml，配置分绑定表，逻辑库，真实库，逻辑表，每部分可以拆开，避免文件过大，编辑阅读困难, 以后扩展到 yml，继而支持 nacos，zookeeper，apollo 等常用的配置中心

示例：

 参考 sharding-sphere/my-sharding/my-sharding-resouce-xml 工程

3.4.3   创建分片库

分片框架没有提供 api 直接创建分片库，需手动创建

3.4.4   创建分片表

 组件提供工具自动创建分片表，如下图：

![img](https://upload-images.jianshu.io/upload_images/26997955-1913b80a8add72e8.png?imageMogr2/auto-orient/strip|imageView2/2/w/251/format/webp)

框架提供工具类,  my-sharding-tools-gentable.jar, 生成分片表

命令行 java -jar my-sharding-tools-gentable.jar gentable [ddl]

其中，ddl 是创建表(逻辑表)ddl

3.4.5   系统改造

数据分片后，系统需相应的改造，以适应分片要求

3.4.5.1   分离分片/非分片表

3.4.5.1.1 多数据源配置

分片后，非分片使用原有数据源；分片表使用分片框架创建的代理数据源

3.4.5.1.2 服务和 dao

服务和 dao 分离到不同的文件夹，方便扫描，事务切入

3.4.5.2     sql 拆分

分片后，分片键选择，导致除了绑定表外，其他 join 需要跨库，需要拆分 sql 分段执行，合并结果，或者使用搜索引擎，宽表查询

3.4.5.3     手动计算和执行真实 sql

sharding-sphere 有限度支持 sql，例如，3.1.0 不支持 distinct

但有些情况，sql 有分片健作为条件，肉眼可见 sql 路由到单库，s 是可以执行的

例如，

**select distinct**

**t1.partno, t1.uoi**

**from #{shardingTable}  t1**

**where t1.parent_id = #{parentId}**

假设 t1 以 parentId 分片，而 parentId 是查询条件，sql 最终路由到单库

my-sharding 管理着库表资源，分片算法和数据源，可以轻松地手动计算出实际 sql，

\1)   ResourceCollection 的 locate 方法，传入分片键值，获得真实库真实表

\2)   替换逻辑 sql 的逻辑表，活动实际 sql

\3)   真实库获得真实数据源，执行实际 sql

3.4.6   分布式事务

xa/base

版本 3.x 只支持 xa，有 atomikos 实现

版本 4+，增加 base 支持，集成 seata 实现；xa 增加多种实现

分布式事务 3 个场景：

\1.   非分表/分表的事务 不支持

\2.   多分片表 支持

2.1  广播表

2.2  其他

对于 2.2 应尽量避免，相比分片前本地事务，xa/base 吞吐率/性能都会大幅下降；base 最终一直，业务需要识别是否合适

对于 2.1 广播比表，例如，数据字典，xa 是合适方案，时间容忍度高，需要严格保障一致性

3.4.7   搜索引擎

  数据分片后，一些数据跨库，需要拆分执行，也可以使用搜索引擎，完整的放到搜索引擎索引，数据查找在搜索引擎进行，同时数据变更使用数据同步，同步到搜索引擎

参考《搜索引擎 onesearch 设计与实现.docx》

规划

\1)   支持 yml 分片配置，进而支持 nacos，apollo 等配置中心

\2)   完善 function 模块

\3)   升级到 shardingsphere 5



[数据分片 my-sharding 设计原理 & 开发指南 - 简书 (jianshu.com)](https://www.jianshu.com/p/e35d5001ff0a)