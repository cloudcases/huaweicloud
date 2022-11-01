# Flink介绍和基本概念

[stephen_k](https://www.jianshu.com/u/23e2e71af03f)

2017.05.30 14:28:42字数 4,048阅读 40,196

## 介绍

#### 概述

Apache Flink是一个面向**数据流处理和批量数据处理**的可分布式的开源计算框架，它基于同一个Flink流式执行模型（streaming execution model），能够支持**流处理和批处理**两种应用类型。由于流处理和批处理所提供的SLA(服务等级协议)是完全不相同， **流处理一般需要支持低延迟、Exactly-once保证**，而**批处理需要支持高吞吐、高效处理**，所以在实现的时候通常是分别给出两套实现方法，或者通过一个独立的开源框架来实现其中每一种处理方案。比较典型的有：实现批处理的开源方案有MapReduce、Spark；实现流处理的开源方案有Storm；Spark的Streaming 其实本质上也是微批处理。
　　Flink在实现流处理和批处理时，与传统的一些方案完全不同，它从另一个视角看待流处理和批处理，将二者统一起来：Flink是完全支持流处理，也就是说作为流处理看待时输入数据流是无界的；批处理被作为一种特殊的流处理，只是它的输入数据流被定义为有界的。

#### 特性

- 有状态计算的Exactly-once语义。状态是指flink能够维护数据在时序上的聚类和聚合，同时它的checkpoint机制
- 支持带有事件时间（event time）语义的流处理和窗口处理。事件时间的语义使流计算的结果更加精确，尤其在事件到达无序或者延迟的情况下。
- 支持高度灵活的窗口（window）操作。支持基于time、count、session，以及data-driven的窗口操作，能很好的对现实环境中的创建的数据进行建模。
- 轻量的容错处理（ fault tolerance）。 它使得系统既能保持高的吞吐率又能保证exactly-once的一致性。通过轻量的state snapshots实现
- 支持高吞吐、低延迟、高性能的流处理
- 支持savepoints 机制（一般手动触发）。即可以将应用的运行状态保存下来；在升级应用或者处理历史数据是能够做到无状态丢失和最小停机时间。
- 支持大规模的集群模式，支持yarn、Mesos。可运行在成千上万的节点上
- 支持具有Backpressure功能的持续流模型
- Flink在JVM内部实现了自己的内存管理
- 支持迭代计算
- 支持程序自动优化：避免特定情况下Shuffle、排序等昂贵操作，中间结果进行缓存

#### API支持

- DataStream API
- DataSet API
- Table API
- Streaming SQL

#### Libs支持

- 支持复杂事件处理（CEP）
- 支持机器学习（FlinkML）
- 支持图分析处理（Gelly）
- 支持关系数据处理（Table）

#### 整体组件栈

![img](https://upload-images.jianshu.io/upload_images/4092059-b6ff29d09ba6da97.png?imageMogr2/auto-orient/strip|imageView2/2/w/781/format/webp)

组件栈

- Deployment层： 该层主要涉及了Flink的部署模式，Flink支持多种部署模式：本地、集群（Standalone/YARN），（GCE/EC2）。
- Runtime层：Runtime层提供了支持Flink计算的全部核心实现，比如：支持分布式Stream处理、JobGraph到ExecutionGraph的映射、调度等等，为上层API层提供基础服务。
- API层： 主要实现了面向无界Stream的流处理和面向Batch的批处理API，其中面向流处理对应DataStream API，面向批处理对应DataSet API。
- Libraries层：该层也可以称为Flink应用框架层，根据API层的划分，在API层之上构建的满足特定应用的实现计算框架，也分别对应于面向流处理和面向批处理两类。面向流处理支持：CEP（复杂事件处理）、基于SQL-like的操作（基于Table的关系操作）；面向批处理支持：FlinkML（机器学习库）、Gelly（图处理）

## 编程模型

#### 抽象的层级

![img](https://upload-images.jianshu.io/upload_images/4092059-e1cb446b2c5f8459.png?imageMogr2/auto-orient/strip|imageView2/2/w/974/format/webp)

抽象的层级

- 有状态的数据流处理层。最底层的抽象仅仅提供有状态的数据流，它通过处理函数（Process Function）嵌入到数据流api(DataStream API). 用户可以通过它自由的处理单流或者多流，并保持一致性和容错。同时用户可以注册事件时间和处理时间的回调处理，以实现复杂的计算逻辑。
- 核心API层。 它提供了数据处理的基础模块，像各种transformation, join,aggregations,windows,stat 以及数据类型等等
- Table API层。 定了围绕关系表的DSL(领域描述语言)。Table API遵循了关系模型的标准：Table类型关系型数据库中的表，API也提供了相应的操作，像select,project,join,group-by,aggregate等。Table API声明式的定义了逻辑上的操作（logical operation）不是code for the operation；Flink会对Table API逻辑在执行前进行优化。同时代码上，Flink允许混合使用Table API和DataStram/DataSet API
- SQL层。 它很类似Table API的语法和表达，也是定义与Table API层次之上的，但是提供的是纯SQL的查询表达式。

#### 程序和数据流

用户实现的Flink程序是由Stream和Transformation这两个基本构建块组成，其中**Stream是一个中间结果数据，而Transformation是一个操作，它对一个或多个输入Stream进行计算处理，输出一个或多个结果Stream**。

当一个Flink程序被执行的时候，它会被映射为Streaming Dataflow。一个Streaming Dataflow是由一组Stream和Transformation Operator组成，它类似于一个DAG图，在启动的时候从一个或多个Source Operator开始，结束于一个或多个Sink Operator。

下面是一个由Flink程序映射为Streaming Dataflow的示意图，如下所示：

![img](https://upload-images.jianshu.io/upload_images/4092059-50877b8a53d2c224.png?imageMogr2/auto-orient/strip|imageView2/2/w/633/format/webp)

程序和数据流

上图中，FlinkKafkaConsumer是一个Source Operator，map、keyBy、timeWindow、apply是Transformation Operator，RollingSink是一个Sink Operator。

#### 并行的数据流

在Flink中，程序天生是并行和分布式的：一个Stream可以被分成多个Stream分区（Stream Partitions），一个Operator可以被分成多个Operator Subtask，每一个Operator Subtask是在不同的线程中独立执行的。一个Operator的并行度，等于Operator Subtask的个数，一个Stream的并行度总是等于生成它的Operator的并行度。有关Parallel Dataflow的实例，如下图所示：

![img](https://upload-images.jianshu.io/upload_images/4092059-05d48f178331ee71.png?imageMogr2/auto-orient/strip|imageView2/2/w/658/format/webp)

并行的数据流

上图Streaming Dataflow的并行视图中，展现了在两个Operator之间的Stream的两种模式：

- One-to-one模式：比如从Source[1]到map()[1]，它保持了Source的分区特性（Partitioning）和分区内元素处理的有序性，也就是说map()[1]的Subtask看到数据流中记录的顺序，与Source[1]中看到的记录顺序是一致的。
- Redistribution模式：这种模式改变了输入数据流的分区，比如从map()[1]、map()[2]到keyBy()/window()/apply()[1]、keyBy()/window()/apply()[2]，上游的Subtask向下游的多个不同的Subtask发送数据，改变了数据流的分区，这与实际应用所选择的Operator有关系。
  另外，Source Operator对应2个Subtask，所以并行度为2，而Sink Operator的Subtask只有1个，故而并行度为1。

#### 窗口（Windows）

流处理中的聚合操作（counts,sums等等）不同于批处理，因为数据流是无限，无法在其上应用聚合，所以通过限定窗口(window)的范围，来进行流的聚合操作。例如：5分钟的数据计数，或者计算100个元素的总和等等。
窗口可以由时间驱动 (every 30 seconds) 或者数据驱动(every 100 elements)。如：滚动窗口tumbling windows（无叠加），滑动窗口sliding windows（有叠加），以及会话窗口session windows(被无事件活动的间隔隔开)



![img](https://upload-images.jianshu.io/upload_images/4092059-2ced312405b2b07f.png?imageMogr2/auto-orient/strip|imageView2/2/w/676/format/webp)

窗口

#### 时间（Time）

三种不同的时间概念：

- 事件时间 Event Time：事件的创建时间，通常通过时间中的一个时间戳来描述
- 摄入时间 Ingestion time： 事件进入Flink 数据流的source的时间
- 处理时间 Processing Time:Processing Time表示某个Operator对事件进行处理时的本地系统时间（是在TaskManager节点上）

![img](https://upload-images.jianshu.io/upload_images/4092059-bdb6d19e2506408c.png?imageMogr2/auto-orient/strip|imageView2/2/w/444/format/webp)

时间

#### 有状态的数据操作（Stateful Operations）

在流处理中，有些操作仅仅在某一时间针对单一事件（如事件转换map），有些操作需要记住多个事件的信息并进行处理（window operators）,后者的这些操作称为有状态的操作。
有状态的操作一般被维护在内置的key/value存储中。这些状态信息会跟数据流一起分区并且分布存储，并且可以通过有状态的数据操作来访问。因此这些key/value的状态信息仅在带key的数据流（通过keyBy() 函数处理过）中才能访问到。数据流按照key排列能保证所有的状态更新都是本地操作，保证一致性且无事务问题。同时这种排列方式使Flink能够透明的再分发状态信息和调整数据流分区。

![img](https://upload-images.jianshu.io/upload_images/4092059-29aa454e5ab250b5.png?imageMogr2/auto-orient/strip|imageView2/2/w/382/format/webp)

有状态的数据操作

#### 容错的Checkpoint

Flink 通过流回放和设置检查点的方式实现容错。一个checkpoint关联了输入流中的某个记录和相应状态和操作。数据流可以从checkpoint中进行恢复，并保证一致性（exactly-once 的处理语义）。 Checkpoint的间隔关系到执行是的容错性和恢复时间。

#### 流上的批处理

Flink把批处理作为特殊的流处理程序来执行，许多概念也都可以应用的批处理中，除了一些小的不同：

- 批处理的API(DataSet API )不使用checkpoints，恢复通过完整的流回放来实现
- DataSet API的有状态操作使用简单的内存和堆外内存 的数据结构，而不是key/value的索引
- DataSet API 引入一种同步的迭代操作，这个仅应用于有界数据流。

## 分布式执行环境

#### 任务和运算（算子）链（Tasks and Operator Chains）

在Flink分布式执行环境中，会将多个运算子任务Operator Subtask串起来组成一个Operator Chain，实际上就是一个运算链。每个运算会在TaskManager上一个独立的线程中执行。将算子串连到任务中是一种很好的优化：它能减少线程间的数据交接和缓存，并且提高整体的吞吐，降低处理的时延。这种串联的操作，可以通过API来进行配置。如下图的数据流就有5个子任务，通过5个并行的线程来执行，所示：

![img](https://upload-images.jianshu.io/upload_images/4092059-33c69eb95706a48f.png?imageMogr2/auto-orient/strip|imageView2/2/w/576/format/webp)

任务和运算（算子）链

#### Job Managers，Task Managers，Clients

Flink的运行时，由两种类型的进程组成：

- JobManagers： 也就是masters ，协调分布式任务的执行 。用来调度任务，协调checkpoints，协调错误恢复等等。至少需要一个JobManager，高可用的系统会有多个，一个leader，其他是standby
- TaskManagers： 也就是workers，用来执行数据流任务或者子任务，缓存和交互数据流。 至少需要一个TaskManager
- Client: Client不是运行是和程序执行的一部分，它是用来准备和提交数据流到JobManagers。之后，可以断开连接或者保持连接以获取任务的状态信息。

![img](https://upload-images.jianshu.io/upload_images/4092059-2ca24a524fd21b08.png?imageMogr2/auto-orient/strip|imageView2/2/w/851/format/webp)

基本架构

从上图可以分析出Flink运行时的整体状态。 Flink的Driver程序会将代码逻辑构建成一个Program Dataflow(区分source,operator,sink等等)，在通过Graph Builder构建DAG的Dataflow graph, 构建job,划分出task 和subtask等等。 Client 将job 提交到JobManager. Client 通过Actor System和JobManager 进行消息通讯，接收JobManager返回的状态更新和任务执行统计结果。 JobMangaer 按照Dataflow的Task 和Subtask的划分，将任务调度分配到各个TaskManager中进行执行。TaskManager会将内存抽象成多个TaskSlot，用于执行Task任务。JobManagers与TaskManagers之间的任务管理，Checkpoints的触发，任务状态，心跳等等消息处理都是通过ActorSystem。

#### Task Slots 和资源

每个Worker(Task Manager)是一个JVM进程，通常会在单独的线程里执行一个或者多个子任务。为了控制一个Worker能够接受多少个任务，会在Worker上抽象多个Task Slot (至少一个)。
每个Task Slot代表固定的资源子集。比如一个TaskManager有3个Slots，每个Slot能管理对这个Worker分配的资源的3分之1的内存。 对资源分槽，意味着Subtask不会同其他Subtasks竞争内存，同时可以预留一定的可用内存。目前Task Slot没有对CPU进行隔离，仅是针对内存。通过动态的调整task slots的个数，用户可以定义哪些子任务可以相互隔离。只有一个slot的TaskManager意味着每个任务组运行在一个单独JVM中。 在拥有多个slot的TaskManager上，subtask共用JVM，可以共用TCP连接和心跳消息，同时可以共用一些数据集和数据结构，从而减小任务的开销。

![img](https://upload-images.jianshu.io/upload_images/4092059-82f7648439cf6531.png?imageMogr2/auto-orient/strip|imageView2/2/w/923/format/webp)

Task Slots（1）

默认情况下，Flink允许子任务共享slots,即便它们是不同任务的子任务，只要属于同一个job。这样的结果就是一个slot会负责一个job的整个pipeline。共用slot有两个好处:

- Flink 集群的task slot的个数就是job的最高并行度。
- 更实现更好的资源利用。没有共享的slots，非密集的source/map() subtask 会占用和 window 这类密集型的subtask 同样多的资源。 使用共享的slot的将充分的利用分槽的资源，使代价较大的subtask能够均匀的分布在TaskManager上。如，下图中的共享slot的执行模式中可以并行运行6个pipeline而上图的只可以运行2个pipeline.
  同时APIs也提供了资源组的机制，可以实现不想进行资源隔离的情况。
  实践中，比较好的每个TaskManager的task slot的默认数量最好是CPU的核数。

![img](https://upload-images.jianshu.io/upload_images/4092059-103b590f4c5cd7fd.png?imageMogr2/auto-orient/strip|imageView2/2/w/923/format/webp)

Task Slots（2）

#### 状态后端

数据的KV索引信息存储在设定的状态后端的存储中。一种是内存中的Hash map，另一种是存在Rocksdb（KV存储）中。另外，状态后端还是实现了在时间点上对KV状态的快照，并作为Checkpoint的一部分存储起来。



![img](https://upload-images.jianshu.io/upload_images/4092059-72a9932a66386477.png?imageMogr2/auto-orient/strip|imageView2/2/w/482/format/webp)

State Backends

#### 保存点（Savepoints）

通过Data Stream AP编写的程序可以从一个保存点重新开始执行。即便你更新了你的程序和Flink集群都不会有状态数据丢失。
保存点是手动触发的，触发时会将它写入状态后端。Savepoints的实现也是依赖Checkpoint的机制。Flink 程序在执行中会周期性的在worker 节点上进行快照并生成Checkpoint。因为任务恢复的时候只需要最后一个完成的Checkpoint的，所以旧有的Checkpoint会在新的Checkpoint完成时被丢弃。
Savepoints和周期性的Checkpoint非常的类似，只是有两个重要的不同。一个是由用户触发，而且不会随着新的Checkpoint生成而被丢弃。

参考： [https://ci.apache.org/projects/flink/flink-docs-release-1.3/concepts/programming-model.html](https://link.jianshu.com/?t=https://ci.apache.org/projects/flink/flink-docs-release-1.3/concepts/programming-model.html)



20人点赞

[日记本](https://www.jianshu.com/nb/8481634)



[Flink介绍和基本概念 - 简书 (jianshu.com)](https://www.jianshu.com/p/2ee7134d7373)