在这个数据驱动的时代，信息的处理和分析变得越来越重要。而在众多的大数据处理框架中，「**Apache Spark**」以其独特的优势脱颖而出。

本篇文章，我们将一起走进Spark的世界，探索并理解其相关的基础概念和使用方法。本文主要目标是让初学者能够对Spark有一个全面的认识，并能实际应用到各类问题的解决之中。

## Spark是什么

学习一个东西之前先要知道这个东西是什么。

**Spark 是一个开源的大数据处理引擎，它提供了一整套开发 API，包括流计算和机器学习。它支持批处理和流处理。**

**Spark 的一个显著特点是它能够在内存中进行迭代计算，从而加快数据处理速度**。尽管 Spark 是用 Scala 开发的，但它也为 Java、Scala、Python 和 R 等高级编程语言提供了开发接口。

### Spark组件

Spark提供了6大核心组件：

- Spark Core
- Spark SQL
- Spark Streaming
- Spark MLlib
- Spark GraphX

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOyTOyXWbFqEUiakGXgsQon98rSf8TVazItMotAMjZdUzZKBSu6eubBVj1Duic5C5Ria0jBKMJQCjDPQ/640)

**Spark Core**

Spark Core 是 Spark 的基础，它提供了内存计算的能力，是分布式处理大数据集的基础。它将分布式数据抽象为弹性分布式数据集（RDD），并为运行在其上的上层组件提供 API。所有 Spark 的上层组件都建立在 Spark Core 的基础之上。

**Spark SQL**

Spark SQL 是一个用于处理结构化数据的 Spark 组件。它允许使用 SQL 语句查询数据。Spark 支持多种数据源，包括 Hive 表、Parquet 和 JSON 等。

**Spark Streaming**

Spark Streaming 是一个用于处理动态数据流的 Spark 组件。它能够开发出强大的交互和数据查询程序。**在处理动态数据流时，流数据会被分割成微小的批处理，这些微小批处理将会在 Spark Core 上按时间顺序快速执行**。

**Spark MLlib**

Spark MLlib 是 Spark 的机器学习库。它提供了常用的机器学习算法和实用程序，包括分类、回归、聚类、协同过滤、降维等。MLlib 还提供了一些底层优化原语和高层流水线 API，可以帮助开发人员更快地创建和调试机器学习流水线。

**Spark GraphX**

Spark GraphX 是 Spark 的图形计算库。它提供了一种分布式图形处理框架，可以帮助开发人员更快地构建和分析大型图形。

### Spark的优势

Spark 有许多优势，其中一些主要优势包括：

- **速度**：Spark 基于内存计算，能够比基于磁盘的计算快很多。对于迭代式算法和交互式数据挖掘任务，这种速度优势尤为明显。

- **易用性**：Spark 支持多种语言，包括 Java、Scala、Python 和 R。它提供了丰富的内置 API，可以帮助开发人员更快地构建和运行应用程序。

- **通用性**：Spark 提供了多种组件，可以支持不同类型的计算任务，包括批处理、交互式查询、流处理、机器学习和图形处理等。
- **兼容性**：Spark 可以与多种数据源集成，包括 Hadoop 分布式文件系统（HDFS）、Apache Cassandra、Apache HBase 和 Amazon S3 等。
- **容错性**：Spark 提供了弹性分布式数据集（RDD）抽象，可以帮助开发人员更快地构建容错应用程序。

### Word Count

上手写一个简单的代码例子，下面是一个Word Count的Spark程序：

```scala
import org.apache.spark.{SparkConf, SparkContext}

object SparkWordCount {
  def main (args:Array [String]): Unit = {
    //setMaster("local[9]") 表示在本地运行 Spark 程序，使用 9 个线程。local[*] 表示使用所有可用的处理器核心。
   //这种模式通常用于本地测试和开发。
    val conf = new SparkConf ().setAppName ("Word Count").setMaster("local[9]");
    val sc = new SparkContext (conf);
    sc.setLogLevel("ERROR")

    val data = List("Hello World", "Hello Spark")
    val textFile = sc.parallelize(data)
    val wordCounts = textFile.flatMap (line => line.split (" ")).map (
      word => (word, 1)).reduceByKey ( (a, b) => a + b)
    wordCounts.collect().foreach(println)
  }
}

输出：
(Hello,2)
(World,1)
(Spark,1)
```

程序首先创建了一个 SparkConf 对象，用来设置应用程序名称和运行模式。然后，它创建了一个 SparkContext 对象，用来连接到 Spark 集群。

接下来，程序创建了一个包含两个字符串的列表，并使用 `parallelize` 方法将其转换为一个 RDD。然后，它使用 `flatMap` 方法将每一行文本拆分成单词，并使用 `map` 方法将每个单词映射为一个键值对（key-value pair），其中键是单词，值是 1。

最后，程序使用 `reduceByKey` 方法将具有相同键的键值对进行合并，并对它们的值进行求和。最终结果是一个包含每个单词及其出现次数的 RDD。程序使用 `collect` 方法将结果收集到驱动程序，并使用 `foreach` 方法打印出来。

## Spark基本概念

Spark的理论较多，为了更有效地学习Spark，首先来理解下其基本概念。

### Application

**Application指的就是用户编写的Spark应用程序。**

如下，"Word Count"就是该应用程序的名字。

```scala
import org.apache.spark.sql.SparkSession

object WordCount {
  def main(args: Array[String]) {
    // 创建 SparkSession 对象，它是 Spark Application 的入口
    val spark = SparkSession.builder.appName("Word Count").getOrCreate()
    // 读取文本文件并创建 Dataset
    val textFile = spark.read.textFile("hdfs://...")
    // 使用 flatMap 转换将文本分割为单词，并使用 reduceByKey 转换计算每个单词的数量
    val counts = textFile.flatMap(line => line.split(" "))
                 .groupByKey(identity)
                 .count()
    // 将结果保存到文本文件中
    counts.write.text("hdfs://...")
    // 停止 SparkSession
    spark.stop()
  }
}
```

### Driver

Driver 是运行 Spark Application 的进程，它负责创建 SparkSession 和 SparkContext 对象，并将代码转换和操作。

它还负责创建逻辑和物理计划，并与集群管理器协调调度任务。

**简而言之，Spark Application 是使用 Spark API 编写的程序，而 Spark Driver 是负责运行该程序并与集群管理器协调的进程。**

**可以将Driver 理解为运行 Spark Application `main` 方法的进程。**

driver的内存大小可以进行设置，配置如下：

```scala
# 设置 driver内存大小
driver-memory 1024m
```

### Master & Worker

**在Spark中，Master是独立集群的控制者，而Worker是工作者。**

一个Spark独立集群需要启动一个Master和多个Worker。Worker就是物理节点，Worker上面可以启动Executor进程。

### Executor

在每个Worker上为某应用启动的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上。

每个任务都有各自独立的Executor。Executor是一个执行Task的容器。实际上它是一组计算资源(cpu核心、memory)的集合。

**一个Worker节点可以有多个Executor。一个Executor可以运行多个Task。**

Executor创建成功后，在日志文件会显示如下信息：

```
INFO Executor: Starting executor ID [executorId] on host [executorHostname]
```

### RDD

**RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象，它代表一个不可变、可分区、里面的元素可并行计算的集合。**

RDD的 Partition 是指数据集的分区。它是数据集中元素的集合，这些元素被分区到集群的节点上，可以并行操作。**对于RDD来说，每个分片都会被一个计算任务处理，并决定并行计算的粒度。用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU Core的数目**。

一个函数会被作用在每一个分区。Spark 中 RDD 的计算是以分片为单位的，`compute` 函数会被作用到每个分区上。

**RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算。**

### Job

一个Job包含多个RDD及作用于相应RDD上的各种操作，**每个Action的触发就会生成一个job**。用户提交的Job会提交给DAG Scheduler，Job会被分解成Stage，Stage会被细化成Task。

### Task

被发送到Executor上的工作单元。每个Task负责计算一个分区的数据。

### Stage

在 Spark 中，一个作业（Job）会被划分为多个阶段（Stage）。**同一个 Stage 可以有多个 Task 并行执行(Task 数=分区数）**。

阶段之间的划分是根据数据的依赖关系来确定的。当一个 RDD 的分区依赖于另一个 RDD 的分区时，这两个 RDD 就属于同一个阶段。当一个 RDD 的分区依赖于多个 RDD 的分区时，这些 RDD 就属于不同的阶段。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMQLUpLuRk6aZsckVcTHYoO1iaOmLKChIvXT2Rkib3v06icficTFyeJ3uAznFAGdHTGkHEdD2Rr8EVflA/640)

上图中，Stage表示一个可以顺滑完成的阶段。曲线表示 Shuffle 过程。

如果Stage能够复用前面的Stage的话，那么会显示灰色。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMQLUpLuRk6aZsckVcTHYoOSWPWp4BPQBaUVg6sA0Iutic2gmjPicJmn8iaRPCc8248VPKqAYo4gauwA/640)



#### Shuffle

在 Spark 中，**Shuffle 是指在不同阶段之间重新分配数据的过程。它通常发生在需要对数据进行聚合或分组操作的时候**，例如 `reduceByKey` 或 `groupByKey` 等操作。

在 Shuffle 过程中，Spark 会将数据按照键值进行分区，并将属于同一分区的数据发送到同一个计算节点上。这样，每个计算节点就可以独立地处理属于它自己分区的数据。

#### Stage的划分

**Stage的划分，简单来说是以宽依赖来划分的。**

对于窄依赖，Partition 的转换处理在 Stage 中完成计算，不划分（将窄依赖尽量放在在同一个 Stage 中，可以实现流水线计算）。

对于宽依赖，由于有 Shuffle 的存在，只能在父 RDD 处理完成后，才能开始接下来的计算，也就是说需要划分 Stage。

**Spark 会根据 Shuffle/宽依赖 使用回溯算法来对 DAG 进行 Stage 划分，从后往前，遇到宽依赖就断开，遇到窄依赖就把当前的 RDD 加入到当前的 Stage 阶段中**。

至于什么是窄依赖和宽依赖，下文马上就会提及。

#### 窄依赖 & 宽依赖

- 窄依赖

父 RDD 的一个分区只会被子 RDD 的一个分区依赖。比如：`map`，`filter`和`union`，这种依赖称之为「**窄依赖**」。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOyTOyXWbFqEUiakGXgsQon9tK9NoennqaqefnGbYx3rchUiaMsRGXOSG5f3nFl0Cibkt8sm3YjFmtFg/640)

**窄依赖的多个分区可以并行计算，并且窄依赖的一个分区的数据如果丢失只需要重新计算对应的分区的数据就可以了。**

- 宽依赖

指子RDD的分区依赖于父RDD的所有分区，称之为「**宽依赖**」。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOyTOyXWbFqEUiakGXgsQon97p8hpNl8qlGpxjmibgib0E8tJ6ktbfBX6U6MKDLDQDib2cL1tPJsJRHqA/640)

对于宽依赖，必须等到上一阶段计算完成才能计算下一阶段。

### DAG

有向无环图，其实说白了就是RDD之间的依赖关系图。

- **开始**：通过 SparkContext 创建的 RDD。
- **结束**：触发 Action，一旦触发 Action 就形成了一个完整的 DAG（**有几个 Action，就有几个 DAG**）。

## Spark执行流程

Spark的执行流程大致如下：

1. 构建Spark Application的运行环境（启动SparkContext），SparkContext向资源管理器（可以是Standalone、Mesos或YARN）注册并申请运行Executor资源。
2. 资源管理器为Executor分配资源并启动Executor进程，Executor运行情况将随着“心跳”发送到资源管理器上。
3. SparkContext构建DAG图，将DAG图分解成多个Stage，并把每个Stage的TaskSet（任务集）发送给Task Scheduler (任务调度器）。
4. Executor向SparkContext申请Task， Task Scheduler将Task发放给Executor，同时，SparkContext将应用程序代码发放给Executor。
5. Task在Executor上运行，把执行结果反馈给Task Scheduler，然后再反馈给DAG Scheduler。
6. 当一个阶段完成后，Spark 会根据数据依赖关系将结果传输给下一个阶段，并开始执行下一个阶段的任务。
7. 最后，当所有阶段都完成后，Spark 会将最终结果返回给驱动程序，并完成作业的执行。

## Spark运行模式

Spark 支持多种运行模式，包括本地模式、独立模式、Mesos 模式、YARN 模式和 Kubernetes 模式。

- **本地模式**：在本地模式下，Spark 应用程序会在单个机器上运行，不需要连接到集群。这种模式适用于开发和测试，但不适用于生产环境。
- **独立模式**：在独立模式下，Spark 应用程序会连接到一个独立的 Spark 集群，并在集群中运行。这种模式适用于小型集群，但不支持动态资源分配。
- **Mesos 模式**：在 Mesos 模式下，Spark 应用程序会连接到一个 Apache Mesos 集群，并在集群中运行。这种模式支持动态资源分配和细粒度资源共享，目前国内使用较少。
- **YARN 模式**：在 YARN 模式下，Spark 应用程序会连接到一个 Apache Hadoop YARN 集群，并在集群中运行。这种模式支持动态资源分配和与其他 Hadoop 生态系统组件的集成，Spark在Yarn模式下是不需要Master和Worker的。
- **Kubernetes 模式**：在 Kubernetes 模式下，Spark 应用程序会连接到一个 Kubernetes 集群，并在集群中运行。这种模式支持动态资源分配和容器化部署。

## RDD详解

RDD的概念在Spark中十分重要，上面只是简单的介绍了一下，下面详细的对RDD展开介绍。

RDD是“Resilient Distributed Dataset”的缩写，从全称就可以了解到RDD的一些典型特性：

- **Resilient（弹性）**：RDD之间会形成有向无环图（DAG），如果RDD丢失了或者失效了，可以从父RDD重新计算得到。即容错性。
- **Distributed（分布式）**：RDD的数据是以逻辑分区的形式分布在集群的不同节点的。
- **Dataset（数据集）**：即RDD存储的数据记录，可以从外部数据生成RDD，例如Json文件，CSV文件，文本文件，数据库等。

RDD里面的数据集会被逻辑分成若干个分区，这些分区是分布在集群的不同节点的，基于这样的特性，RDD才能在集群不同节点并行计算。

### RDD特性

- **内存计算**：Spark RDD运算数据是在内存中进行的，在内存足够的情况下，不会把中间结果存储在磁盘，所以计算速度非常高效。

- **惰性求值**：所有的转换操作都是惰性的，也就是说不会立即执行任务，只是把对数据的转换操作记录下来而已。只有碰到action操作才会被真正的执行。

- **容错性**：Spark RDD具备容错特性，在RDD失效或者数据丢失的时候，可以根据DAG从父RDD重新把数据集计算出来，以达到数据容错的效果。

- **不变性**：RDD是进程安全的，因为RDD是不可修改的。它可以在任何时间点被创建和查询，使得缓存，共享，备份都非常简单。在计算过程中，是RDD的不可修改特性保证了数据的一致性。

- **持久化**：可以调用cache或者persist函数，把RDD缓存在内存、磁盘，下次使用的时候不需要重新计算而是直接使用。

### RDD操作

RDD支持两种操作：

- 转换操作（Transformation）。
- 行动操作（Actions）。

#### 转换操作（Transformation）

转换操作以RDD做为输入参数，然后输出一个或者多个RDD。转换操作不会修改输入RDD。`Map()`、`Filter()`这些都属于转换操作。

转换操作是惰性求值操作，只有在碰到行动操作（Actions）的时候，转换操作才会真正实行。转换操作分两种：「**窄依赖**」和「**宽依赖**」。

下面是一些常见的转换操作：

| 转换操作    | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| map         | 将函数应用于 RDD 中的每个元素，并返回一个新的 RDD            |
| filter      | 返回一个新的 RDD，其中包含满足给定谓词的元素                 |
| flatMap     | 将函数应用于 RDD 中的每个元素，并将返回的迭代器展平为一个新的 RDD |
| union       | 返回一个新的 RDD，其中包含两个 RDD 的元素                    |
| distinct    | 返回一个新的 RDD，其中包含原始 RDD 中不同的元素              |
| groupByKey  | 将键值对 RDD 中具有相同键的元素分组到一起，并返回一个新的 RDD |
| reduceByKey | 将键值对 RDD 中具有相同键的元素聚合到一起，并返回一个新的 RDD |
| sortByKey   | 返回一个新的键值对 RDD，其中元素按照键排序                   |

#### 行动操作（Action）

Action是数据执行部分，其通过执行`count`，`reduce`，`collect`等方法真正执行数据的计算部分。

| Action 操作    | 描述                                                   |
| -------------- | ------------------------------------------------------ |
| reduce         | 通过函数聚合 RDD 中的所有元素                          |
| collect        | 将 RDD 中的所有元素返回到驱动程序                      |
| count          | 返回 RDD 中的元素个数                                  |
| first          | 返回 RDD 中的第一个元素                                |
| take           | 返回 RDD 中的前 n 个元素                               |
| takeOrdered    | 返回 RDD 中的前 n 个元素，按照自然顺序或指定的顺序排序 |
| saveAsTextFile | 将 RDD 中的元素保存到文本文件中                        |
| foreach        | 将函数应用于 RDD 中的每个元素                          |

### RDD 的创建方式

创建RDD有3种不同方式：

- 从外部存储系统。
- 从其他RDD。
- 由一个已经存在的 Scala 集合创建。

#### 从外部存储系统

**由外部存储系统的数据集创建，包括本地的文件系统，还有所有 `Hadoop` 支持的数据集，比如 `HDFS、Cassandra、HBase` 等：**

```scala
val rdd1 = sc.textFile("hdfs://node1:8020/wordcount/input/words.txt")
```

#### 从其他RDD

**通过已有的 RDD 经过算子转换生成新的 RDD：**

```scala
val rdd2=rdd1.flatMap(_.split(" "))
```

#### 由一个已经存在的 Scala 集合创建

```scala
val rdd3 = sc.parallelize(Array(1,2,3,4,5,6,7,8))
或者
val rdd4 = sc.makeRDD(List(1,2,3,4,5,6,7,8))
```

其实`makeRDD` 方法底层调用了` parallelize` 方法：

### RDD 缓存机制

RDD 缓存是在内存存储RDD计算结果的一种优化技术。把中间结果缓存起来以便在需要的时候重复使用，这样才能有效减轻计算压力，提升运算性能。

要持久化一个RDD，只要调用其`cache()`或者`persist()`方法即可。在该RDD第一次被计算出来时，就会直接缓存在每个节点中。而且Spark的持久化机制还是自动容错的，如果持久化的RDD的任何partition丢失了，那么Spark会自动通过其源RDD，使用transformation操作重新计算该partition。

```scala
val rdd1 = sc.textFile("hdfs://node01:8020/words.txt")
val rdd2 = rdd1.flatMap(x=>x.split(" ")).map((_,1)).reduceByKey(_+_)
rdd2.cache //缓存/持久化
rdd2.sortBy(_._2,false).collect//触发action,会去读取HDFS的文件,rdd2会真正执行持久化
rdd2.sortBy(_._2,false).collect//触发action,会去读缓存中的数据,执行速度会比之前快,因为rdd2已经持久化到内存中了
```

**需要注意的是，在触发action的时候，才会去执行持久化。**

`cache()`和`persist()`的区别在于，`cache()`是`persist()`的一种简化方式，`cache()`的底层就是调用的`persist()`的无参版本，就是调用`persist(MEMORY_ONLY)`，将数据持久化到内存中。

如果需要从内存中去除缓存，那么可以使用`unpersist()`方法。

```scala
rdd.persist(StorageLevel.MEMORY_ONLY)
rdd.unpersist()
```

#### 存储级别

RDD存储级别主要有以下几种。

| 级别                  | 使用空间 | CPU时间 | 是否在内存中 | 是否在磁盘上 | 备注                                                         |
| --------------------- | -------- | ------- | ------------ | ------------ | ------------------------------------------------------------ |
| MEMORY_ONLY           | 高       | 低      | 是           | 否           | 使用未序列化的Java对象格式，将数据保存在内存中。如果内存不够存放所有的数据，则数据可能就不会进行持久化。 |
| MEMORY_ONLY_2         | 高       | 低      | 是           | 否           | 数据存2份                                                    |
| MEMORY_ONLY_SER       | 低       | 高      | 是           | 否           | 基本含义同MEMORY_ONLY。唯一的区别是，会将RDD中的数据进行序列化。这种方式更加节省内存 |
| MEMORY_ONLY_SER_2     | 低       | 高      | 是           | 否           | 数据序列化，数据存2份                                        |
| MEMORY_AND_DISK       | 高       | 中等    | 部分         | 部分         | 如果数据在内存中放不下，则溢写到磁盘                         |
| MEMORY_AND_DISK_2     | 高       | 中等    | 部分         | 部分         | 数据存2份                                                    |
| MEMORY_AND_DISK_SER   | 低       | 高      | 部分         | 部分         | 基本含义同MEMORY_AND_DISK。唯一的区别是，会将RDD中的数据进行序列化 |
| MEMORY_AND_DISK_SER_2 | 低       | 高      | 部分         | 部分         | 数据存2份                                                    |
| DISK_ONLY             | 低       | 高      | 否           | 是           | 使用未序列化的Java对象格式，将数据全部写入磁盘文件中。       |
| DISK_ONLY_2           | 低       | 高      | 否           | 是           | 数据存2份                                                    |
| OFF_HEAP              |          |         |              |              | 这个目前是试验型选项，类似MEMORY_ONLY_SER，但是数据是存储在堆外内存的。 |

**对于上述任意一种持久化策略，如果加上后缀_2，代表的是将每个持久化的数据，都复制一份副本，并将副本保存到其他节点上。**

这种基于副本的持久化机制主要用于进行容错。假如某个节点挂掉了，节点的内存或磁盘中的持久化数据丢失了，那么后续对RDD计算时还可以使用该数据在其他节点上的副本。如果没有副本的话，就只能将这些数据从源头处重新计算一遍了。

### RDD的血缘关系

**血缘关系是指 RDD 之间的依赖关系。当你对一个 RDD 执行转换操作时，Spark 会生成一个新的 RDD，并记录这两个 RDD 之间的依赖关系。这种依赖关系就是血缘关系。**

血缘关系可以帮助 Spark 在发生故障时恢复数据。当一个分区丢失时，Spark 可以根据血缘关系重新计算丢失的分区，而不需要从头开始重新计算整个 RDD。

血缘关系还可以帮助 Spark 优化计算过程。Spark 可以根据血缘关系合并多个连续的窄依赖转换，减少数据传输和通信开销。

我们可以执行`toDebugString`打印RDD的依赖关系。

下面是一个简单的例子：

```scala
val conf = new SparkConf().setAppName("Lineage Example").setMaster("local")
val sc = new SparkContext(conf)

val data = sc.parallelize(List(1, 2, 3, 4, 5))
val mappedData = data.map(x => x + 1)
val filteredData = mappedData.filter(x => x % 2 == 0)

println(filteredData.toDebugString)
```

在这个例子中，我们首先创建了一个包含 5 个元素的 RDD，并对它执行了两个转换操作：`map` 和 `filter`。然后，我们使用 `toDebugString` 方法打印了最终 RDD 的血缘关系。

运行这段代码后，你会看到类似下面的输出：

```
(2) MapPartitionsRDD[2] at filter at <console>:26 []
 |  MapPartitionsRDD[1] at map at <console>:24 []
 |  ParallelCollectionRDD[0] at parallelize at <console>:22 []
```

这个输出表示最终的 RDD 是通过两个转换操作（`map` 和 `filter`）从原始的 `ParallelCollectionRDD` 转换而来的。

## CheckPoint

CheckPoint可以将RDD从其依赖关系中抽出来，保存到可靠的存储系统（例如HDFS，S3等)， 即它可以将数据和元数据保存到检查指向目录中。 因此，在程序发生崩溃的时候，Spark可以恢复此数据，并从停止的任何地方开始。

CheckPoint分为两类：

- **高可用CheckPoint**：容错性优先。这种类型的检查点可确保数据永久存储，如存储在HDFS或其他分布式文件系统上。 这也意味着数据通常会在网络中复制，这会降低检查点的运行速度。
- **本地CheckPoint**：性能优先。 RDD持久保存到执行程序中的本地文件系统。 因此，数据写得更快，但本地文件系统也不是完全可靠的，一旦数据丢失，工作将无法恢复。

开发人员可以使用`RDD.checkpoint()`方法来设置检查点。在使用检查点之前，必须使用`SparkContext.setCheckpointDir(directory: String)`方法设置检查点目录。

下面是一个简单的例子：

```SCALA
import org.apache.spark.{SparkConf, SparkContext}

object CheckpointExample {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Checkpoint Example").setMaster("local")
    val sc = new SparkContext(conf)

    // 设置 checkpoint 目录
    sc.setCheckpointDir("/tmp/checkpoint")

    val data = sc.parallelize(List(1, 2, 3, 4, 5))
    val mappedData = data.map(x => x + 1)
    val filteredData = mappedData.filter(x => x % 2 == 0)

    // 对 RDD 进行 checkpoint
    filteredData.checkpoint()
    // 触发 checkpoint
    filteredData.count()
  }
}

```

RDD的检查点机制就好比Hadoop将中间计算值存储到磁盘，即使计算中出现了故障，我们也可以轻松地从中恢复。通过对 RDD 启动检查点机制可以实现容错和高可用。

### Persist VS CheckPoint

- **位置**：Persist 和 Cache 只能保存在本地的磁盘和内存中(或者堆外内存–实验中)，而 Checkpoint 可以保存数据到 HDFS 这类可靠的存储上。
- **生命周期**：Cache 和 Persist 的 RDD 会在程序结束后会被清除或者手动调用 unpersist 方法，而 Checkpoint 的 RDD 在程序结束后依然存在，不会被删除**。CheckPoint将RDD持久化到HDFS或本地文件夹，如果不被手动remove掉，是一直存在的，也就是说可以被下一个driver使用，而Persist不能被其他dirver使用**。

## Spark-Submit 

### 详细参数说明

| 参数名                | 参数说明                                                     |
| --------------------- | ------------------------------------------------------------ |
| —master               | master 的地址，提交任务到哪里执行，例如 spark://host:port, yarn, local。具体指可参考下面关于Master_URL的列表 |
| —deploy-mode          | 在本地 (client) 启动 driver 或在 cluster 上启动，默认是 client |
| —class                | 应用程序的主类，仅针对 java 或 scala 应用                    |
| —name                 | 应用程序的名称                                               |
| —jars                 | 用逗号分隔的本地 jar 包，设置后，这些 jar 将包含在 driver 和 executor 的 classpath 下 |
| —packages             | 包含在driver 和executor 的 classpath 中的 jar 的 maven 坐标  |
| —exclude-packages     | 为了避免冲突 而指定不包含的 package                          |
| —repositories         | 远程 repository                                              |
| —conf PROP=VALUE      | 指定 spark 配置属性的值， 例如 -conf spark.executor.extraJavaOptions=”-XX:MaxPermSize=256m” |
| —properties-file      | 加载的配置文件，默认为 conf/spark-defaults.conf              |
| —driver-memory        | Driver内存，默认 1G                                          |
| —driver-java-options  | 传给 driver 的额外的 Java 选项                               |
| —driver-library-path  | 传给 driver 的额外的库路径                                   |
| —driver-class-path    | 传给 driver 的额外的类路径                                   |
| —driver-cores         | Driver 的核数，默认是1。在 yarn 或者 standalone 下使用       |
| —executor-memory      | 每个 executor 的内存，默认是1G                               |
| —total-executor-cores | 所有 executor 总共的核数。仅仅在 mesos 或者 standalone 下使用 |
| —num-executors        | 启动的 executor 数量。默认为2。在 yarn 下使用                |
| —executor-core        | 每个 executor 的核数。在yarn或者standalone下使用             |

### Master_URL的值

| Master URL        | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| local             | 使用1个worker线程在本地运行Spark应用程序                     |
| local[K]          | 使用K个worker线程在本地运行Spark应用程序                     |
| local[*]          | 使用所有剩余worker线程在本地运行Spark应用程序                |
| spark://HOST:PORT | 连接到Spark Standalone集群，以便在该集群上运行Spark应用程序  |
| mesos://HOST:PORT | 连接到Mesos集群，以便在该集群上运行Spark应用程序             |
| yarn-client       | 以client方式连接到YARN集群，集群的定位由环境变量HADOOP_CONF_DIR定义，该方式driver在client运行。 |
| yarn-cluster      | 以cluster方式连接到YARN集群，集群的定位由环境变量HADOOP_CONF_DIR定义，该方式driver也在集群中运行。 |

## Spark 共享变量

一般情况下，当一个传递给Spark操作（例如map和reduce）的函数在远程节点上面运行时，Spark操作实际上操作的是这个函数所用变量的一个独立副本。

**这些变量被复制到每台机器上，并且这些变量在远程机器上的所有更新都不会传递回驱动程序**。通常跨任务的读写变量是低效的，所以，Spark提供了两种共享变量：「**广播变量（broadcast variable）**」和「**累加器（accumulator）**」。

### 广播变量

**广播变量**允许程序员缓存一个只读的变量在每台机器上面，而不是每个任务保存一份拷贝。说白了其实就是共享变量。

**如果Executor端用到了Driver的变量，如果不使用广播变量在Executor有多少task就有多少Driver端的变量副本。如果使用广播变量在每个Executor中只有一份Driver端的变量副本。**

一个广播变量可以通过调用`SparkContext.broadcast(v)`方法从一个初始变量v中创建。广播变量是v的一个包装变量，它的值可以通过value方法访问，下面的代码说明了这个过程：

```scala
import org.apache.spark.{SparkConf, SparkContext}

object BroadcastExample {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Broadcast Example").setMaster("local")
    val sc = new SparkContext(conf)

    val data = sc.parallelize(List(1, 2, 3, 4, 5))

    // 创建一个广播变量
    val factor = sc.broadcast(2)

    // 使用广播变量
    val result = data.map(x => x * factor.value)

    result.collect().foreach(println)
  }
}

```

广播变量创建以后，我们就能够在集群的任何函数中使用它来代替变量v，这样我们就不需要再次传递变量v到每个节点上。另外，为了保证所有的节点得到广播变量具有相同的值，**对象v不能在广播之后被修改**。

### 累加器

累加器是一种只能通过关联操作进行“加”操作的变量，因此它能够高效的应用于并行操作中。它们能够用来实现counters和sums。

一个累加器可以通过调用`SparkContext.accumulator(v)`方法从一个初始变量v中创建。运行在集群上的任务可以通过add方法或者使用`+=`操作来给它加值。然而，它们无法读取这个值。只有驱动程序可以使用value方法来读取累加器的值。

示例代码如下：

```scala
import org.apache.spark.{SparkConf, SparkContext}

object AccumulatorExample {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("AccumulatorExample")
    val sc = new SparkContext(conf)

    val accum = sc.longAccumulator("My Accumulator")

    sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))

    println(accum.value) // 输出 10
  }
}
```

这个示例中，我们创建了一个名为 `My Accumulator` 的累加器，并使用 `sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))` 来对其进行累加。最后，我们使用 `println(accum.value)` 来输出累加器的值，结果为 `10`。

我们可以利用子类`AccumulatorParam`创建自己的累加器类型。AccumulatorParam接口有两个方法：zero方法为你的数据类型提供一个“0 值”（zero value），addInPlace方法计算两个值的和。例如，假设我们有一个Vector类代表数学上的向量，我们能够如下定义累加器：

```scala
object VectorAccumulatorParam extends AccumulatorParam[Vector] {
  def zero(initialValue: Vector): Vector = {
    Vector.zeros(initialValue.size)
  }
  def addInPlace(v1: Vector, v2: Vector): Vector = {
    v1 += v2
  }
}
// Then, create an Accumulator of this type:
val vecAccum = sc.accumulator(new Vector(...))(VectorAccumulatorParam)
```

## Spark SQL

Spark为结构化数据处理引入了一个称为Spark SQL的编程模块。它提供了一个称为DataFrame的编程抽象，并且可以充当分布式SQL查询引擎。

### Spark SQL的特性

- **集成**：无缝地将SQL查询与Spark程序混合。 Spark SQL允许将结构化数据作为Spark中的分布式数据集(RDD)进行查询，在Python，Scala和Java中集成了API。这种紧密的集成使得可以轻松地运行SQL查询以及复杂的分析算法。

- **Hive兼容性**：在现有仓库上运行未修改的Hive查询。 Spark SQL重用了Hive前端和MetaStore，提供与现有Hive数据，查询和UDF的完全兼容性。只需将其与Hive一起安装即可。

- **标准连接**：通过JDBC或ODBC连接。 Spark SQL包括具有行业标准JDBC和ODBC连接的服务器模式。

- **可扩展性**：对于交互式查询和长查询使用相同的引擎。 Spark SQL利用RDD模型来支持中查询容错，使其能够扩展到大型作业。不要担心为历史数据使用不同的引擎。

### Spark SQL 数据类型

Spark SQL 支持多种数据类型，包括数字类型、字符串类型、二进制类型、布尔类型、日期时间类型和区间类型等。

数字类型包括：

- `ByteType`：代表一个字节的整数，范围是 -128 到 127¹²。
- `ShortType`：代表两个字节的整数，范围是 -32768 到 32767¹²。
- `IntegerType`：代表四个字节的整数，范围是 -2147483648 到 2147483647¹²。
- `LongType`：代表八个字节的整数，范围是 -9223372036854775808 到 9223372036854775807¹²。
- `FloatType`：代表四字节的单精度浮点数¹²。
- `DoubleType`：代表八字节的双精度浮点数¹²。
- `DecimalType`：代表任意精度的十进制数据，通过内部的 java.math.BigDecimal 支持。BigDecimal 由一个任意精度的整型非标度值和一个 32 位整数组成¹²。

字符串类型包括：

- `StringType`：代表字符字符串值。

二进制类型包括：

- `BinaryType`：代表字节序列值。

布尔类型包括：

- `BooleanType`：代表布尔值。

日期时间类型包括：

- `TimestampType`：代表包含字段年、月、日、时、分、秒的值，与会话本地时区相关。时间戳值表示绝对时间点。
- `DateType`：代表包含字段年、月和日的值，不带时区。

区间类型包括：

- `YearMonthIntervalType (startField, endField)`：表示由以下字段组成的连续子集组成的年月间隔：MONTH（月份），YEAR（年份）。

- `DayTimeIntervalType (startField, endField)`：表示由以下字段组成的连续子集组成的日时间间隔：SECOND（秒），MINUTE（分钟），HOUR（小时），DAY（天）。

复合类型包括：

- `ArrayType (elementType, containsNull)`：代表由 elementType 类型元素组成的序列值。containsNull 用来指明 ArrayType 中的值是否有 null 值。
- `MapType (keyType, valueType, valueContainsNull)`：表示包括一组键值对的值。通过 keyType 表示 key 数据的类型，通过 valueType 表示 value 数据的类型。valueContainsNull 用来指明 MapType 中的值是否有 null 值。
- `StructType (fields)`：表示一个拥有 StructFields (fields) 序列结构的值。
- `StructField (name, dataType, nullable)`：代表 StructType 中的一个字段，字段的名字通过 name 指定，dataType 指定 field 的数据类型，nullable 表示字段的值是否有 null 值。

### DataFrame

DataFrame 是 Spark 中用于处理结构化数据的一种数据结构。它类似于关系数据库中的表，具有行和列。每一列都有一个名称和一个类型，每一行都是一条记录。

DataFrame 支持多种数据源，包括结构化数据文件、Hive 表、外部数据库和现有的 RDD。它提供了丰富的操作，包括筛选、聚合、分组、排序等。

DataFrame 的优点在于它提供了一种高级的抽象，使得用户可以使用类似于 SQL 的语言进行数据处理，而无需关心底层的实现细节。此外，Spark 会自动对 DataFrame 进行优化，以提高查询性能。

下面是一个使用DataFrame的代码例子：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("DataFrame Example").getOrCreate()
import spark.implicits._

val data = Seq(
  ("Alice", 25),
  ("Bob", 30),
  ("Charlie", 35)
)

val df = data.toDF("name", "age")

df.show()
```

在这个示例中，我们首先创建了一个 `SparkSession` 对象，然后使用 `toDF` 方法将一个序列转换为 DataFrame。最后，我们使用 `show` 方法来显示 DataFrame 的内容。

#### 创建 DataFrame

在 Scala 中，可以通过以下几种方式创建 DataFrame：

1. 从现有的 RDD 转换而来。例如：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Create DataFrame").getOrCreate()
import spark.implicits._

case class Person(name: String, age: Int)

val rdd = spark.sparkContext.parallelize(Seq(Person("Alice", 25), Person("Bob", 30)))
val df = rdd.toDF()
df.show()
```

2. 从外部数据源读取。例如，从 JSON 文件中读取数据并创建 DataFrame：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Create DataFrame").getOrCreate()

val df = spark.read.json("path/to/json/file")
df.show()
```

3. 通过编程方式创建。例如，使用 `createDataFrame` 方法：

```scala
import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}

val spark = SparkSession.builder.appName("Create DataFrame").getOrCreate()

val schema = StructType(
  List(
    StructField("name", StringType, nullable = true),
    StructField("age", IntegerType, nullable = true)
  )
)

val data = Seq(Row("Alice", 25), Row("Bob", 30))
val rdd = spark.sparkContext.parallelize(data)

val df = spark.createDataFrame(rdd, schema)
df.show()
```

#### DSL & SQL

在 Spark 中，可以使用两种方式对 DataFrame 进行查询：「**DSL（Domain-Specific Language）**」和「 **SQL**」。

DSL 是一种特定领域语言，它提供了一组用于操作 DataFrame 的方法。例如，下面是一个使用 DSL 进行查询的例子：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("DSL and SQL").getOrCreate()
import spark.implicits._

val df = Seq(
  ("Alice", 25),
  ("Bob", 30),
  ("Charlie", 35)
).toDF("name", "age")

df.select("name", "age")
  .filter($"age" > 25)
  .show()
```

SQL 是一种结构化查询语言，它用于管理关系数据库系统。在 Spark 中，可以使用 SQL 对 DataFrame 进行查询。例如，下面是一个使用 SQL 进行查询的例子：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("DSL and SQL").getOrCreate()
import spark.implicits._

val df = Seq(
  ("Alice", 25),
  ("Bob", 30),
  ("Charlie", 35)
).toDF("name", "age")

df.createOrReplaceTempView("people")

spark.sql("SELECT name, age FROM people WHERE age > 25").show()
```

DSL 和 SQL 的区别在于语法和风格。DSL 使用方法调用链来构建查询，而 SQL 使用声明式语言来描述查询。选择哪种方式取决于个人喜好和使用场景。

### Spark SQL 数据源

Spark SQL 支持多种数据源，包括 Parquet、JSON、CSV、JDBC、Hive 等。

下面是示例代码：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Data Sources Example").getOrCreate()
// Parquet
val df = spark.read.parquet("path/to/parquet/file")
// JSON 
val df = spark.read.json("path/to/json/file")
// CSV
val df = spark.read.option("header", "true").csv("path/to/csv/file")
// JDBC
val df = spark.read
  .format("jdbc")
  .option("url", "jdbc:mysql://host:port/database")
  .option("dbtable", "table")
  .option("user", "username")
  .option("password", "password")
  .load()

df.show()
```

### load & save

在 Spark 中，`load` 函数用于从外部数据源读取数据并创建 DataFrame，而 `save` 函数用于将 DataFrame 保存到外部数据源。

下面是从 Parquet 文件中读取数据并创建 DataFrame 的示例代码：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Load and Save Example").getOrCreate()

val df = spark.read.load("path/to/parquet/file")
df.show()
```

下面是将 DataFrame 保存到 Parquet 文件的示例代码：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Load and Save Example").getOrCreate()
import spark.implicits._

val df = Seq(
  ("Alice", 25),
  ("Bob", 30),
  ("Charlie", 35)
).toDF("name", "age")

df.write.save("path/to/parquet/file")
```

### 函数

Spark SQL 提供了丰富的内置函数，包括数学函数、字符串函数、日期时间函数、聚合函数等。你可以在 Spark SQL 的官方文档中查看所有可用的内置函数。

此外，Spark SQL 还支持「**自定义函数（User-Defined Function，UDF）**」，可以让用户编写自己的函数并在查询中使用。

下面是一个使用 SQL 语法编写自定义函数的示例代码：

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions.udf

val spark = SparkSession.builder.appName("UDF Example").getOrCreate()
import spark.implicits._

val df = Seq(
  ("Alice", 25),
  ("Bob", 30),
  ("Charlie", 35)
).toDF("name", "age")

df.createOrReplaceTempView("people")

val square = udf((x: Int) => x * x)
spark.udf.register("square", square)

spark.sql("SELECT name, square(age) FROM people").show()
```

在这个示例中，我们首先定义了一个名为 `square` 的自定义函数，它接受一个整数参数并返回它的平方。然后，我们使用 `createOrReplaceTempView` 方法创建一个临时视图，并使用 `udf.register` 方法注册自定义函数。

最后，我们使用 `spark.sql` 方法执行 SQL 查询，并在查询中调用自定义函数。

### DataSet

DataSet 是 Spark 1.6 版本中引入的一种新的数据结构，它提供了 RDD 的强类型和 DataFrame 的查询优化能力。

#### 创建DataSet

在 Scala 中，可以通过以下几种方式创建 DataSet：

1. 从现有的 RDD 转换而来。例如：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Create DataSet").getOrCreate()
import spark.implicits._

case class Person(name: String, age: Int)

val rdd = spark.sparkContext.parallelize(Seq(Person("Alice", 25), Person("Bob", 30)))
val ds = rdd.toDS()
ds.show()
```

2. 从外部数据源读取。例如，从 JSON 文件中读取数据并创建 DataSet：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Create DataSet").getOrCreate()
import spark.implicits._

case class Person(name: String, age: Long)

val ds = spark.read.json("path/to/json/file").as[Person]
ds.show()
```

3. 通过编程方式创建。例如，使用 `createDataset` 方法：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Create DataSet").getOrCreate()
import spark.implicits._

case class Person(name: String, age: Int)

val data = Seq(Person("Alice", 25), Person("Bob", 30))
val ds = spark.createDataset(data)
ds.show()
```

#### DataSet VS DataFrame

DataSet 和 DataFrame 都是 Spark 中用于处理结构化数据的数据结构。它们都提供了丰富的操作，包括筛选、聚合、分组、排序等。

**它们之间的主要区别在于类型安全性。DataFrame 是一种弱类型的数据结构，它的列只有在运行时才能确定类型。这意味着，在编译时无法检测到类型错误，只有在运行时才会抛出异常。**

**而 DataSet 是一种强类型的数据结构，它的类型在编译时就已经确定。这意味着，如果你试图对一个不存在的列进行操作，或者对一个列进行错误的类型转换，编译器就会报错。**

此外，DataSet 还提供了一些额外的操作，例如 `map`、`flatMap`、`reduce` 等。

### RDD & DataFrame & Dataset 转化

RDD、DataFrame、Dataset三者有许多共性，有各自适用的场景常常需要在三者之间转换。

- DataFrame/Dataset 转 RDD

```SCALA
val rdd1=testDF.rdd
val rdd2=testDS.rdd
```

- RDD 转 DataSet

```scala
import spark.implicits._
case class Coltest(col1:String,col2:Int)extends Serializable //定义字段名和类型
val testDS = rdd.map {line=>
      Coltest(line._1,line._2)
    }.toDS
```

可以注意到，定义每一行的类型（case class）时，已经给出了字段名和类型，后面只要往case class里面添加值即可。

- Dataset 转 DataFrame

```SCALA
import spark.implicits._
val testDF = testDS.toDF
```

- DataFrame 转 Dataset

```SCALA
import spark.implicits._
case class Coltest(col1:String,col2:Int)extends Serializable //定义字段名和类型
val testDS = testDF.as[Coltest]
```

这种方法就是在给出每一列的类型后，使用`as`方法，转成Dataset，这在数据类型在DataFrame需要针对各个字段处理时极为方便。

**注意**：在使用一些特殊的操作时，一定要加上 `import spark.implicits._` 不然`toDF`、`toDS`无法使用。

## Spark Streaming

Spark Streaming 的工作原理是将实时数据流拆分为小批量数据，并使用 Spark 引擎对这些小批量数据进行处理。这种微批处理（Micro-Batch Processing）的方式使得 Spark Streaming 能够以近乎实时的延迟处理大规模的数据流。

下面是一个简单的 Spark Streaming 示例代码：

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setAppName("Spark Streaming Example")
val ssc = new StreamingContext(conf, Seconds(1))

val lines = ssc.socketTextStream("localhost", 9999)
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)

wordCounts.print()

ssc.start()
ssc.awaitTermination()
```

我们首先创建了一个 `StreamingContext` 对象，并指定了批处理间隔为 1 秒。然后，我们使用 `socketTextStream` 方法从套接字源创建了一个 DStream。接下来，我们对 DStream 进行了一系列操作，包括 flatMap、map 和 reduceByKey。最后，我们使用 `print` 方法打印出单词计数的结果。

### Spark Streaming 优缺点

Spark Streaming 作为一种实时流处理框架，具有以下优点：

- **高性能**：Spark Streaming 基于 Spark 引擎，能够快速处理大规模的数据流。
- **易用性**：Spark Streaming 提供了丰富的 API，可以让开发人员快速构建实时流处理应用。
- **容错性**：Spark Streaming 具有良好的容错性，能够在节点故障时自动恢复。
- **集成性**：Spark Streaming 能够与 Spark 生态系统中的其他组件（如 Spark SQL、MLlib 等）无缝集成。

但是，Spark Streaming 也有一些缺点：

- **延迟**：由于 Spark Streaming 基于微批处理模型，因此它的延迟相对较高。对于需要极低延迟的应用场景，Spark Streaming 可能不是最佳选择。
- **复杂性**：Spark Streaming 的配置和调优相对复杂，需要一定的经验和技能。

### DStream

DStream（离散化流）是 Spark Streaming 中用于表示实时数据流的一种抽象。它由一系列连续的 RDD 组成，每个 RDD 包含一段时间内收集到的数据。

在 Spark Streaming 中，可以通过以下几种方式创建 DStream：

1. 从输入源创建。例如，从套接字源创建 DStream：

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setAppName("DStream Example")
val ssc = new StreamingContext(conf, Seconds(1))

val lines = ssc.socketTextStream("localhost", 9999)
lines.print()

ssc.start()
ssc.awaitTermination()
```

2. 通过转换操作创建。例如，对现有的 DStream 进行 map 操作：

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setAppName("DStream Example")
val ssc = new StreamingContext(conf, Seconds(1))

val lines = ssc.socketTextStream("localhost", 9999)
val words = lines.flatMap(_.split(" "))
words.print()

ssc.start()
ssc.awaitTermination()
```

3. 通过连接操作创建。例如，对两个 DStream 进行 union 操作：

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setAppName("DStream Example")
val ssc = new StreamingContext(conf, Seconds(1))

val lines1 = ssc.socketTextStream("localhost", 9999)
val lines2 = ssc.socketTextStream("localhost", 9998)
val lines = lines1.union(lines2)
lines.print()

ssc.start()
ssc.awaitTermination()
```

**总结：简单来说 DStream 就是对 RDD 的封装，你对 DStream 进行操作，就是对 RDD 进行操作。对于 DataFrame/DataSet/DStream 来说本质上都可以理解成 RDD。**

### 窗口函数

在 Spark Streaming 中，窗口函数用于对 DStream 中的数据进行窗口化处理。它允许你对一段时间内的数据进行聚合操作。

Spark Streaming 提供了多种窗口函数，包括：

- **window**：返回一个新的 DStream，它包含了原始 DStream 中指定窗口大小和滑动间隔的数据。
- **countByWindow**：返回一个新的单元素 DStream，它包含了原始 DStream 中指定窗口大小和滑动间隔的元素个数。
- **reduceByWindow**：返回一个新的 DStream，它包含了原始 DStream 中指定窗口大小和滑动间隔的元素经过 reduce 函数处理后的结果。
- **reduceByKeyAndWindow**：类似于 reduceByWindow，但是在进行 reduce 操作之前会先按照 key 进行分组。

下面是一个使用窗口函数的示例代码：

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setAppName("Window Example")
val ssc = new StreamingContext(conf, Seconds(1))

val lines = ssc.socketTextStream("localhost", 9999)
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKeyAndWindow((a: Int, b: Int) => a + b, Seconds(30), Seconds(10))

wordCounts.print()

ssc.start()
ssc.awaitTermination()
```

在这个示例中，我们首先创建了一个 DStream，并对其进行了一系列转换操作。然后，我们使用 `reduceByKeyAndWindow` 函数对 DStream 进行窗口化处理，指定了窗口大小为 30 秒，滑动间隔为 10 秒。最后，我们使用 `print` 方法打印出单词计数的结果。

### 输出操作

Spark Streaming允许DStream的数据输出到外部系统，如数据库或文件系统，输出的数据可以被外部系统所使用，该操作类似于RDD的输出操作。Spark Streaming支持以下输出操作：

- **print() **： 打印DStream中每个RDD的前10个元素到控制台。
-  **saveAsTextFiles(prefix, [suffix] **： 将此DStream中每个RDD的所有元素以文本文件的形式保存。每个批次的数据都会保存在一个单独的目录中，目录名为：`prefix-TIME_IN_MS[.suffix]`。
- **saveAsObjectFiles(prefix, [suffix])**： 将此DStream中每个RDD的所有元素以Java对象序列化的形式保存。每个批次的数据都会保存在一个单独的目录中，目录名为：`prefix-TIME_IN_MS[.suffix]`。
- **saveAsHadoopFiles(prefix, [suffix])**：将此DStream中每个RDD的所有元素以Hadoop文件（SequenceFile等）的形式保存。每个批次的数据都会保存在一个单独的目录中，目录名为：`prefix-TIME_IN_MS[.suffix]`。
- **foreachRDD(func)**：最通用的输出操作，将函数func应用于DStream中生成的每个RDD。通过此函数，可以将数据写入任何支持写入操作的数据源。

## Structured Streaming

Structured Streaming 是 Spark 2.0 版本中引入的一种新的流处理引擎。它基于 Spark SQL 引擎，提供了一种声明式的 API 来处理结构化数据流。

与 Spark Streaming 相比，Structured Streaming 具有以下优点：

- **易用性**：Structured Streaming 提供了与 Spark SQL 相同的 API，可以让开发人员快速构建流处理应用。
- **高性能**：Structured Streaming 基于 Spark SQL 引擎，能够快速处理大规模的数据流。
- **容错性**：Structured Streaming 具有良好的容错性，能够在节点故障时自动恢复。
- **端到端一致性**：Structured Streaming 提供了端到端一致性保证，能够确保数据不丢失、不重复。

下面是一个简单的 Structured Streaming 示例代码：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Structured Streaming Example").getOrCreate()

val lines = spark.readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

import spark.implicits._

val words = lines.as[String].flatMap(_.split(" "))
val wordCounts = words.groupBy("value").count()

val query = wordCounts.writeStream
  .outputMode("complete")
  .format("console")
  .start()

query.awaitTermination()
```

在这个示例中，我们首先创建了一个 `SparkSession` 对象。然后，我们使用 `readStream` 方法从套接字源创建了一个 DataFrame。接下来，我们对 DataFrame 进行了一系列操作，包括 flatMap、groupBy 和 count。最后，我们使用 `writeStream` 方法将结果输出到控制台。

**Structured Streaming 同样支持 DSL 和 SQL 语法**。

DSL 语法：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Structured Streaming Example").getOrCreate()

val lines = spark.readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

import spark.implicits._

val words = lines.as[String].flatMap(_.split(" "))
val wordCounts = words.groupBy("value").count()

val query = wordCounts.writeStream
  .outputMode("complete")
  .format("console")
  .start()

query.awaitTermination()
```

SQL 语法：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("Structured Streaming Example").getOrCreate()

val lines = spark.readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

lines.createOrReplaceTempView("lines")

val wordCounts = spark.sql(
  """
    |SELECT value, COUNT(*) as count
    |FROM (
    |    SELECT explode(split(value, ' ')) as value
    |    FROM lines
    |)
    |GROUP BY value
  """.stripMargin)

val query = wordCounts.writeStream
  .outputMode("complete")
  .format("console")
  .start()

query.awaitTermination()
```

### Source

Structured Streaming 支持多种输入源，包括文件源（如文本文件、Parquet 文件、JSON 文件等）、Kafka、Socket 等。下面是一个使用 Scala 语言从 Kafka 中读取数据的例子：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("StructuredStreaming").getOrCreate()

// 订阅一个主题
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1")
  .load()

df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]
```

### Output

Structured Streaming 支持多种输出方式，包括控制台输出、内存输出、文件输出、数据源输出等。下面是将数据写入到 Parquet 文件中的例子：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("StructuredStreaming").getOrCreate()

// 从 socket 中读取数据
val lines = spark
  .readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

// 将数据写入到 Parquet 文件中
lines.writeStream
  .format("parquet")
  .option("path", "path/to/output/dir")
  .option("checkpointLocation", "path/to/checkpoint/dir")
  .start()
```

#### Output Mode

每当结果表更新时，我们都希望将更改后的结果行写入外部接收器。

Output mode 指定了数据写入输出接收器的方式。Structured Streaming 支持以下三种 output mode：

| Output Mode | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| Append      | 只将流 DataFrame/Dataset 中的新行写入接收器。                |
| Complete    | 每当有更新时，将流 DataFrame/Dataset 中的所有行写入接收器。  |
| Update      | 每当有更新时，只将流 DataFrame/Dataset 中更新的行写入接收器。 |

#### Output Sink

Output sink 指定了数据写入的位置。Structured Streaming 支持多种输出接收器，包括文件接收器、Kafka 接收器、Foreach 接收器、控制台接收器和内存接收器等。下面是一些使用 Scala 语言将数据写入到不同输出接收器中的例子：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder.appName("StructuredStreaming").getOrCreate()

// 从 socket 中读取数据
val lines = spark
  .readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

// 将数据写入到 Parquet 文件中
lines.writeStream
  .format("parquet")
  .option("path", "path/to/output/dir")
  .option("checkpointLocation", "path/to/checkpoint/dir")
  .start()

// 将数据写入到 Kafka 中
//selectExpr 是一个 DataFrame 的转换操作，它允许你使用 SQL 表达式来选择 DataFrame 中的列。
//selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)") 表示选择 key 和 value 列，并将它们的类型转换为字符串类型。
//这是因为 Kafka 接收器要求数据必须是字符串类型或二进制类型。
lines.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .writeStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("topic", "topic1")
  .start()

// 将数据写入到控制台中
lines.writeStream
  .format("console")
  .start()

// 将数据写入到内存中
lines.writeStream
  .format("memory")
  .queryName("tableName")
  .start()
```

### PV，UV统计

下面是用Structured Streaming实现PV，UV统计的例子，我们来感受实战下：

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._

object PVUVExample {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder.appName("PVUVExample").getOrCreate()
    import spark.implicits._

    // 假设我们有一个包含用户ID和访问的URL的输入流
    val lines = spark.readStream.format("socket").option("host", "localhost").option("port", 9999).load()
    val data = lines.as[String].map(line => {
      val parts = line.split(",")
      (parts(0), parts(1))
    }).toDF("user", "url")

    // 计算PV
    val pv = data.groupBy("url").count().withColumnRenamed("count", "pv")
    val pvQuery = pv.writeStream.outputMode("complete").format("console").start()

    // 计算UV
    val uv = data.dropDuplicates().groupBy("url").count().withColumnRenamed("count", "uv")
    val uvQuery = uv.writeStream.outputMode("complete").format("console").start()

    pvQuery.awaitTermination()
    uvQuery.awaitTermination()
  }
}
```

这段代码演示了如何使用Structured Streaming对数据进行PV和UV统计。它首先从一个socket源读取数据，然后使用`groupBy`和`count`对数据进行PV统计，最后使用`dropDuplicates`、`groupBy`和`count`对数据进行UV统计。

假设我们在本地启动了一个socket服务器，并向其发送以下数据：

```
user1,http://example.com/page1
user2,http://example.com/page1
user1,http://example.com/page2
user3,http://example.com/page1
user2,http://example.com/page2
user3,http://example.com/page2
```

那么程序将输出以下结果：

```
-------------------------------------------
Batch: 0
-------------------------------------------
+--------------------+---+
|                 url| pv|
+--------------------+---+
|http://example.co...|  3|
|http://example.co...|  3|
+--------------------+---+

-------------------------------------------
Batch: 0
-------------------------------------------
+--------------------+---+
|                 url| uv|
+--------------------+---+
|http://example.co...|  2|
|http://example.co...|  3|
+--------------------+---+
```

## 总结

在此，我们对Spark的基本概念、使用方式以及部分原理进行了简单的介绍。Spark以其强大的处理能力和灵活性，已经成为大数据处理领域的一个重要工具。然而，这只是冰山一角。Spark的世界里还有许多深度和广度等待着我们去探索。

作为初学者，你可能会觉得这个领域庞大且复杂。但请记住，每个都是从初学者开始的。不断的学习和实践，你将能够更好的理解和掌握Spark，并将其应用于解决实际问题。这篇文章可能不能涵盖所有的知识点，但我希望它能带给你收获和思考。
