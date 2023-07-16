[TOC]

因为公司用到大数据技术栈的缘故，之前也写过HBase，Spark等文章，公司离线用的是Spark，实时用的是Flink，所以这篇文章是关于Flink的，这篇文章对Flink的相关概念介绍的比较全面，希望对大家学习Flink能有所帮助。

Flink的一些概念和Spark非常像，看这篇文章之前，强烈建议翻看之前的Spark文章，这样学习Flink的时候能够举一反三，有助于理解。

## 流处理 & 批处理

事实上 Flink 本身是流批统一的处理架构，批量的数据集本质上也是流。在 Flink 的视角里，一切数据都可以认为是流，**流数据是无界流，而批数据则是有界流，流数据每输入一条数据，就有一次对应的输出**。

批处理，也叫作离线处理。针对的是有界数据集，非常适合需要访问海量的全部数据才能完成的计算工作，一般用于离线统计。

流处理主要针对的是数据流，特点是无界、实时，对系统传输的每个数据依次执行操作，一般用于实时统计。

### 无界流Unbounded streams

无界流有定义流的开始，但没有定义流的结束。它们会无休止地产生数据。无界流的数据必须持续处理，即数据被摄取后需要立刻处理。我们不能等到所有数据都到达再处理，因为输入是无限的，在任何时候输入都不会完成。处理无界数据通常要求以特定顺序摄取事件，例如事件发生的顺序，以便能够推断结果的完整性。

### 有界流Bounded streams

有界流有定义流的开始，也有定义流的结束。有界流可以在摄取所有数据后再进行计算。有界流所有数据可以被排序，所以并不需要有序摄取。有界流处理通常被称为批处理。所以在Flink里批计算其实指的就是有界流。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnjPQcmW39R8wibMMpqpQGLoiaMFiakpuFUajrXE8dL0yHwib8icmg6Y1fib5Tv06EnMMtH7jtXJJQVAMQ/640?wx_fmt=png)

## Flink的特点和优势

- 同时支持高吞吐、低延迟、高性能。
- 支持事件时间（Event Time）概念，结合Watermark处理乱序数据
- 支持有状态计算，并且支持多种状态内存、 文件、RocksDB。
- 支持高度灵活的窗口(Window) 操作time、 count、 session。
- 基于轻量级分布式快照(CheckPoint) 实现的容错保证Exactly- Once语义。
- 基于JVM实现独立的内存管理。
- Save Points (保存点)。

## Flink VS Spark

Spark 和 Flink 在不同的应用领域上表现会有差别。一般来说，Spark 基于微批处理的方式做同步总有一个“攒批”的过程，所以会有额外开销，因此无法在流处理的低延迟上做到极致。在低延迟流处理场景，Flink 已经有明显的优势。而在海量数据的批处理领域，Spark 能够处理的吞吐量更大。

**Spark Streaming的流计算其实是微批计算，实时性不如Flink，还有一点很重要的是Spark Streaming不适合有状态的计算，得借助一些存储如：Redis，才能实现。而Flink天然支持有状态的计算**。

## Flink API

Flink 本身提供了多层 API：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLp9EtLdWwjR2aiaWXxBr5q1JOc9Q1k8P7OibcCvlYxib3HzE9VQKzx3ktAg/640?wx_fmt=png)

- **Stateful Stream Processing** 最低级的抽象接口是状态化的数据流接口（stateful streaming）。这个接口是通过 ProcessFunction 集成到 DataStream API 中的。该接口允许用户自由的处理来自一个或多个流中的事件，并使用一致的容错状态。另外，用户也可以通过注册 event time 和 processing time 处理回调函数的方法来实现复杂的计算。
- **DataStream/DataSet API** DataStream / DataSet API 是 Flink 提供的核心 API ，DataSet 处理有界的数据集，DataStream 处理有界或者无界的数据流。用户可以通过各种方法（map / flatmap / window / keyby / sum / max / min / avg / join 等）将数据进行转换 / 计算。
- **Table API**  Table API 提供了例如 select、project、join、group-by、aggregate 等操作，使用起来却更加简洁，可以在表与 DataStream/DataSet 之间无缝切换，也允许程序将 Table API 与 DataStream 以及 DataSet 混合使用。
- **SQL** Flink 提供的最高层级的抽象是 SQL，这一层抽象在语法与表达能力上与 Table API 类似，SQL 抽象与 Table API 交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

## Dataflows数据流图

所有的 Flink 程序都可以归纳为由三部分构成：Source、Transformation 和 Sink。

- Source 表示“源算子”，负责读取数据源。

- Transformation 表示“转换算子”，利用各种算子进行处理加工。

- Sink 表示“下沉算子”，负责数据的输出。

source数据源会源源不断的产生数据，transformation将产生的数据进行各种业务逻辑的数据处理，最终由sink输出到外部（console、kafka、redis、DB......）。

基于Flink开发的程序都能够映射成一个Dataflows。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLptqzt0pibVTiazpib2DbAt6DJKrqnSX5iaJdgt2ShOAF8ibGz7MBVScwiaxCQ/640?wx_fmt=png)

当source数据源的数量比较大或计算逻辑相对比较复杂的情况下，需要提高并行度来处理数据，采用并行数据流。

通过设置不同算子的并行度， source并行度设置为2 ， map也是2。代表会启动2个并行的线程来处理数据：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLpCaXHUMG7RLn3A4FX2Pq4SqcvwdAFhN74T5lcquFZEA37untZibCObsw/640?wx_fmt=png)

## Flink基本架构

Flink系统架构中包含了两个角色，分别是JobManager和TaskManager，是一个典型的Master-Slave架构。JobManager相当于是Master，TaskManager相当于是Slave。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLpvf34vfkbzvdq19zWJoAMNNMVJkWI5tgc1r2Xwcc0w07sswUricnibNQA/640?wx_fmt=png)

### Job Manager & Task Manager

在Flink中，JobManager负责整个Flink集群任务的调度以及资源的管理。它从客户端中获取提交的应用，然后根据集群中TaskManager上TaskSlot的使用情况，为提交的应用分配相应的TaskSlot资源并命令TaskManager启动从客户端中获取的应用。

TaskManager负责执行作业流的Task，并且缓存和交换数据流。在TaskManager中资源调度的最小单位是Task slot。TaskManager中Task slot的数量表示并发处理Task的数量。**一台机器节点可以运行多个TaskManager** 。

**TaskManager会向JobManager发送心跳保持连接**。

## 集群 & 部署

### 部署模式

Flink支持多种部署模式，包括本地模式、Standalone模式、YARN模式、Mesos模式和Kubernetes模式。

- 本地模式：本地模式是在单个JVM中启动Flink，主要用于开发和测试。它不需要任何集群管理器，但也不能跨多台机器运行。本地模式的优点是部署简单，缺点是不能利用分布式计算的优势。
- Standalone模式：Standalone模式是在一个独立的集群中运行Flink。它需要手动启动Flink集群，并且需要手动管理资源。Standalone模式的优点是部署简单，可以跨多台机器运行，缺点是需要手动管理资源。
- YARN模式：YARN模式是在Hadoop YARN集群中运行Flink。它可以利用YARN进行资源管理和调度。YARN模式的优点是可以利用现有的Hadoop集群，缺点是需要安装和配置Hadoop YARN，**这是在企业中使用最多的方式**。
- Mesos模式：Mesos模式是在Apache Mesos集群中运行Flink。它可以利用Mesos进行资源管理和调度。Mesos模式的优点是可以利用现有的Mesos集群，缺点是需要安装和配置Mesos。
- Kubernetes模式：Kubernetes模式是在Kubernetes集群中运行Flink。它可以利用Kubernetes进行资源管理和调度。Kubernetes模式的优点是可以利用现有的Kubernetes集群，缺点是需要安装和配置Kubernetes。

每种部署模式都有其优缺点，选择哪种部署模式取决于具体的应用场景和需求。

**Session、Per-Job和Application**是Flink在YARN和Kubernetes上运行时的三种不同模式，它们不是独立的部署模式，而是在YARN和Kubernetes部署模式下的子模式。

- Session模式：在Session模式下，Flink集群会一直运行，用户可以在同一个Flink集群中提交多个作业。Session模式的优点是作业提交快，缺点是作业之间可能会相互影响。
- Per-Job模式：在Per-Job模式下，每个作业都会启动一个独立的Flink集群。Per-Job模式的优点是作业之间相互隔离，缺点是作业提交慢。
- Application模式：Application模式是在Flink 1.11版本中引入的一种新模式，它结合了Session模式和Per-Job模式的优点。在Application模式下，每个作业都会启动一个独立的Flink集群，但是作业提交快。

这三种模式都可以在YARN和Kubernetes部署模式下使用。

### 提交作业流程

1. Session 模式：
   - 在 Session 模式下，Flink 运行在交互式会话中，允许用户在一个 Flink 集群上连续地提交和管理多个作业。
   - 用户可以通过 Flink 命令行界面（CLI）或 Web UI 进行交互。
   - 提交流程如下：
     - 用户启动 Flink 会话，并连接到 Flink 集群。
     - 用户使用 CLI 或 Web UI 提交作业，提交的作业被发送到 Flink 集群的 JobManager。
     - JobManager 接收作业后，会对作业进行解析和编译，生成作业图（JobGraph）。
     - 生成的作业图被发送到 JobManager 的调度器进行调度。
     - 调度器将作业图划分为任务并将其分配给 TaskManager 执行。
     - TaskManager 在其本地执行环境中运行任务。
2. Per-Job 模式：
   - 在 Per-Job 模式下，每个作业都会启动一个独立的 Flink 集群，用于执行该作业。
   - 这种模式适用于独立的批处理或流处理作业，不需要与其他作业共享资源。
   - 提交流程如下：
     - 用户准备好作业程序和所需的配置文件。
     - 用户使用 Flink 提供的命令行工具或编程 API 将作业程序和配置文件打包成一个作业 JAR 文件。
     - 用户将作业 JAR 文件上传到 Flink 集群所在的环境（例如 Hadoop 分布式文件系统）。
     - 用户使用 Flink 提供的命令行工具或编程 API 在指定的 Flink 集群上提交作业。
     - JobManager 接收作业 JAR 文件并进行解析、编译和调度。
     - 调度器将作业图划分为任务并将其分配给可用的 TaskManager 执行。
     - TaskManager 在其本地执行环境中运行任务。
3. Application 模式：
   - Application 模式是 Flink 1.11 版本引入的一种模式，用于在常驻的 Flink 集群上执行多个应用程序。
   - 在 Application 模式下，用户可以在运行中的 Flink 集群上动态提交、更新和停止应用程序。
   - 提交流程如下：
     - 用户准备好应用程序程序和所需的配置文件。
     - 用户使用 Flink 提供的命令行工具或编程 API 将应用程序程序和配置文件打包成一个应用程序 JAR 文件。
     - 用户将应用程序 JAR 文件上传到 Flink 集群所在的环境（例如 Hadoop 分布式文件系统）。
     - 用户使用 Flink 提供的命令行工具或编程 API 在指定的 Flink 集群上提交应用程序。
     - JobManager 接收应用程序 JAR 文件并进行解析、编译和调度。
     - 调度器将应用程序图划分为任务并将其分配给可用的 TaskManager 执行。
     - TaskManager 在其本地执行环境中运行任务。

## 配置开发环境

每个 Flink 应用都需要依赖一组 Flink 类库。Flink 应用至少需要依赖 Flink APIs。许多应用还会额外依赖连接器类库(比如 Kafka、Cassandra 等)。 当用户运行 Flink 应用时(无论是在 IDEA 环境下进行测试，还是部署在分布式环境下)，运行时类库都必须可用

开发工具：IntelliJ IDEA

配置开发Maven依赖：

```xml
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-scala_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

注意点：

- **如果要将程序打包提交到集群运行，打包的时候不需要包含这些依赖，因为集群环境已经包含了这些依赖，此时依赖的作用域应该设置为provided**。
- Flink 应用在 IntelliJ IDEA 中运行，这些 Flink 核心依赖的作用域需要设置为 compile 而不是 provided 。 否则 IntelliJ 不会添加这些依赖到 classpath，会导致应用运行时抛出 `NoClassDefFountError` 异常。

添加打包插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <artifactSet>
                            <excludes>
                                <exclude>com.google.code.findbugs:jsr305</exclude>
                                <exclude>org.slf4j:*</exclude>
                                <exclude>log4j:*</exclude>
                            </excludes>
                        </artifactSet>
                        <filters>
                            <filter>
                                <!--不要拷贝 META-INF 目录下的签名，
                                否则会引起 SecurityExceptions 。 -->
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>my.programs.main.clazz</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### WordCount流批计算程序

配置好开发环境之后写一个简单的Flink程序。

实现：统计HDFS文件单词出现的次数

读取HDFS数据需要添加Hadoop依赖

```xml
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>2.6.5</version>
</dependency>
```

批计算：

```scala
val env = ExecutionEnvironment.getExecutionEnvironment
val initDS: DataSet[String] = env.readTextFile("hdfs://node01:9000/flink/data/wc")
val restDS: AggregateDataSet[(String, Int)] = initDS.flatMap(_.split(" ")).map((_,1)).groupBy(0).sum(1)
restDS.print()
```

------

流计算：

```scala
	/** 准备环境
      * createLocalEnvironment 创建一个本地执行的环境，local
      * createLocalEnvironmentWithWebUI 创建一个本地执行的环境，同时还开启Web UI的查看端口，8081
      * getExecutionEnvironment 根据你执行的环境创建上下文，比如local  cluster
      */
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    /**
      * DataStream：一组相同类型的元素 组成的数据流
      */
    val initStream:DataStream[String] = env.socketTextStream("node01",8888)
    val wordStream = initStream.flatMap(_.split(" "))
    val pairStream = wordStream.map((_,1))
    val keyByStream = pairStream.keyBy(0)
    val restStream = keyByStream.sum(1)
    restStream.print()
    //启动Flink 任务
    env.execute("first flink job")
```

## 并行度

**特定算子的子任务（subtask）的个数称之为并行度（parallel），并行度是几，这个task内部就有几个subtask。**

怎样实现算子并行呢？其实也很简单，我们把一个算子操作，“复制”多份到多个节点，数据来了之后就可以到其中任意一个执行。这样一来，一个算子任务就被拆分成了多个并行的“子任务”（subtasks），再将它们分发到不同节点，就真正实现了并行计算。

**整个流处理程序的并行度，理论上是所有算子并行度中最大的那个，这代表了运行程序需要的 slot 数量**。

### 并行度设置

在 Flink 中，可以用不同的方法来设置并行度，它们的有效范围和优先级别也是不同的。

**代码中设置** 

- 我们在代码中，可以很简单地在算子后跟着调用 `setParallelism()`方法，来设置当前算子的并行度： `stream.map(word -> Tuple2.of(word, 1L)).setParallelism(2);`这种方式设置的并行度，只针对当前算子有效。
- 我们也可以直接调用执行环境的 `setParallelism()`方法，全局设定并行度：`env.setParallelism(2);`这样代码中所有算子，默认的并行度就都为 2 了。

**提交应用时设置**

在使用 flink run 命令提交应用时，可以增加 `-p` 参数来指定当前应用程序执行的并行度，它的作用类似于执行环境的全局设置。如果我们直接在 Web UI 上提交作业，也可以在对应输入框中直接添加并行度。

**配置文件中设置**

我们还可以直接在集群的配置文件 flink-conf.yaml 中直接更改默认并行度：parallelism.default: 2（初始值为 1）

这个设置对于整个集群上提交的所有作业有效。

**在开发环境中，没有配置文件，默认并行度就是当前机器的 CPU 核心数**。

### 并行度生效优先级

1. 对于一个算子，首先看在代码中是否单独指定了它的并行度，这个特定的设置优先级最高，会覆盖后面所有的设置。
2. 如果没有单独设置，那么采用当前代码中执行环境全局设置的并行度。 
3. 如果代码中完全没有设置，那么采用提交时-p 参数指定的并行度。
4. 如果提交时也未指定-p 参数，那么采用集群配置文件中的默认并行度。 

**这里需要说明的是，算子的并行度有时会受到自身具体实现的影响。比如读取 socket 文本流的算子 socketTextStream，它本身就是非并行的 Source 算子，所以无论怎么设置，它在运行时的并行度都是 1**。

## Task

在 Flink 中，Task 是一个阶段多个功能相同 subTask 的集合，Flink 会尽可能地将 operator 的 subtask 链接（chain）在一起形成 task。每个 task 在一个线程中执行。将 operators 链接成 task 是非常有效的优化：它能减少线程之间的切换，减少消息的序列化/反序列化，减少数据在缓冲区的交换，减少了延迟的同时提高整体的吞吐量。

**要是之前学过Spark，这里可以用Spark的思想来看，Flink的Task就好比Spark中的Stage，而我们知道Spark的Stage是根据宽依赖来拆分的。所以我们也可以认为Flink的Task也是根据宽依赖拆分的（尽管Flink中并没有宽依赖的概念），这样会更好理解，如下图：**

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOiaicvO7VztvmFhl4p3h3icdPLDnH9XhJ7ojiaxpbII45iabaDJico8S9WsHo0qnCMRMJYQjfic2M54CWkw/640?wx_fmt=png)

## Operator Chain（算子链)

在Flink中，为了分布式执行，Flink会将算子子任务链接在一起形成任务。每个任务由一个线程执行。将算子链接在一起形成任务是一种有用的优化：**它减少了线程间切换和缓冲的开销，并增加了整体吞吐量，同时降低了延迟**。

举个例子，假设我们有一个简单的Flink流处理程序，它从一个源读取数据，然后应用`map`和`filter`操作，最后将结果写入到一个接收器。这个程序可能看起来像这样：

```Java
DataStream<String> data = env.addSource(new CustomSource());
data.map(new MapFunction<String, String>() {
    @Override
    public String map(String value) throws Exception {
        return value.toUpperCase();
    }
})
.filter(new FilterFunction<String>() {
    @Override
    public boolean filter(String value) throws Exception {
        return value.startsWith("A");
    }
})
.addSink(new CustomSink());
```

**在这个例子中，`map`和`filter`操作可以被链接在一起形成一个任务，被优化为算子链，这意味着它们将在同一个线程中执行，而不是在不同的线程中执行并通过网络进行数据传输**。

## Task Slots

Task Slots即是任务槽，slot 在 Flink 里面可以认为是资源组，Flink 将每个任务分成子任务并且将这些子任务分配到 slot 来并行执行程序，我们可以通过集群的配置文件来设定 TaskManager 的 slot 数量：**taskmanager.numberOfTaskSlots**: 8。

例如，如果 Task Manager 有2个 slot，那么它将为每个 slot 分配 50％ 的内存。 可以在一个 slot 中运行一个或多个线程。 同一 slot 中的线程共享相同的 JVM。

**需要注意的是，slot 目前仅仅用来隔离内存，不会涉及 CPU 的隔离。在具体应用时，可以将 slot 数量配置为机器的 CPU 核心数，尽量避免不同任务之间对 CPU 的竞争。这也是开发环境默认并行度设为机器 CPU 数量的原因**。

### 分发规则

- **不同的Task下的subtask要分发到同一个TaskSlot中，降低数据传输、提高执行效率**。 
- **相同的Task下的subtask要分发到不同的TaskSlot**。

### Slot共享组

如果希望某个算子对应的任务完全独占一个 slot，或者只有某一部分算子共享 slot，在Flink中，可以通过在代码中使用`slotSharingGroup`方法来设置slot共享组。Flink会将具有相同slot共享组的操作放入同一个slot中，同时保持不具有slot共享组的操作在其他slot中。这可以用来隔离slot。

例如，你可以这样设置：

```Java
dataStream.map(...).slotSharingGroup("group1");
```

默认情况下，所有操作都被分配相同的SlotSharingGroup。

这样，只有属于同一个 slot 共享组的子任务，才会开启 slot 共享；不同组之间的任务是完全隔离的，必须分配到不同的 slot 上。

### 并行度和Slots的例子

听了上面并行度和Slots的理论，可能有点疑惑，通过一个例子简单说明下：

假设一共有3个TaskManager，每一个TaskManager中的slot数量设置为3个，那么一共有9个task slot，表示最多能并行执行9个任务。

假设我们写了一个WordCount程序，有四个转换算子：**source —> flatMap —> reduce —> sink**。

当所有算子并行度相同时，容易看出source和flatMap可以优化合并算子链，于是最终有三个任务节点：source & flatMap，reduce 和sink。
如果我们没有任何并行度设置，而配置文件中默认parallelism.default=1，那么程序运行的默认并行度为1，总共有3个任务。**由于不同算子的任务可以共享任务槽，所以最终占用的slot只有1个。9个slot只用了1个，有8个空闲**。如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnjPQcmW39R8wibMMpqpQGLbsA32ngsNbhXic9yRxkicMDt9joDuuGeeT3dBtyp0jibQ6Ewib5wFx5Yeg/640?wx_fmt=png)

我们可以直接把并行度设置为 9，这样所有 3*9=27 个任务就会完全占用 9 个 slot。这是当前集群资源下能执行的最大并行度，计算资源得到了充分的利用。

另外再考虑对于某个算子单独设置并行度的场景。例如，如果我们考虑到输出可能是写入文件，那会希望不要并行写入多个文件，就需要设置 sink 算子的并行度为 1。这时其他的算子并行度依然为 9，所以总共会有 19 个子任务。根据 slot 共享的原则，它们最终还是会占用全部的 9 个 slot，而 sink 任务只在其中一个 slot 上执行，通过这个例子也可以明确地看到，**整个流处理程序的并行度，就应该是所有算子并行度中最大的那个，这代表了运行程序需要的 slot 数量**。

## DataSource数据源

Flink内嵌支持的数据源非常多，比如HDFS、Socket、Kafka、Collections。Flink也提供了addSource方式，可以自定义数据源，下面介绍一些常用的数据源。

### File Source

- 通过读取本地、HDFS文件创建一个数据源。

如果读取的是HDFS上的文件，那么需要导入Hadoop依赖

```xml
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>2.6.5</version>
</dependency>
```

代码示例：每隔10s去读取HDFS指定目录下的新增文件内容，并且进行WordCount。

```scala
import org.apache.flink.api.java.io.TextInputFormat
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.source.FileProcessingMode
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
//在算子转换的时候，会将数据转换成Flink内置的数据类型，所以需要将隐式转换导入进来，才能自动进行类型转换
import org.apache.flink.streaming.api.scala._

object FileSource {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    //读取hdfs文件
    val filePath = "hdfs://node01:9000/flink/data/"
    val textInputFormat = new TextInputFormat(new Path(filePath))
    //每隔10s中读取 hdfs上新增文件内容
    val textStream = env.readFile(textInputFormat,filePath,FileProcessingMode.PROCESS_CONTINUOUSLY,10)
    textStream.flatMap(_.split(" ")).map((_,1)).keyBy(0).sum(1).print()
    env.execute()
  }
}
```

**readTextFile底层调用的就是readFile方法，readFile是一个更加底层的方式，使用起来会更加的灵活**

------

### Collection Source

基于本地集合的数据源，一般用于测试场景，没有太大意义。

```scala
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._

object CollectionSource {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.fromCollection(List("hello flink msb","hello msb msb"))
    stream.flatMap(_.split(" ")).map((_,1)).keyBy(0).sum(1).print()
    env.execute()
  }
}
```

------

### Socket Source

接受Socket Server中的数据。

```scala
val initStream:DataStream[String] = env.socketTextStream("node01",8888)
```

------

### Kafka Source

Flink接受Kafka中的数据，首先要配置flink与kafka的连接器依赖。

Maven依赖：

```xml
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-kafka_2.11</artifactId>
  <version>1.9.2</version>
</dependency>
```

代码：

```scala
	val env = StreamExecutionEnvironment.getExecutionEnvironment
    val prop = new Properties()
    prop.setProperty("bootstrap.servers","node01:9092,node02:9092,node03:9092")
    prop.setProperty("group.id","flink-kafka-id001")
    prop.setProperty("key.deserializer",classOf[StringDeserializer].getName)
    prop.setProperty("value.deserializer",classOf[StringDeserializer].getName)
    /**
      * earliest:从头开始消费，旧数据会频繁消费
      * latest:从最近的数据开始消费，不再消费旧数据
      */
    prop.setProperty("auto.offset.reset","latest")
	val kafkaStream = env.addSource(new FlinkKafkaConsumer[(String, String)]("flink-kafka", new KafkaDeserializationSchema[(String, String)] {
      override def isEndOfStream(t: (String, String)): Boolean = false

      override def deserialize(consumerRecord: ConsumerRecord[Array[Byte], Array[Byte]]): (String, String) = 	  {
        val key = new String(consumerRecord.key(), "UTF-8")
        val value = new String(consumerRecord.value(), "UTF-8")
        (key, value)
      }
      //指定返回数据类型
      override def getProducedType: TypeInformation[(String, String)] =
        createTuple2TypeInformation(createTypeInformation[String], createTypeInformation[String])
    }, prop))
    kafkaStream.print()
    env.execute()
```

## Transformations

Transformations算子可以将一个或者多个算子转换成一个新的数据流，使用Transformations算子组合可以进行复杂的业务处理。

### Map

DataStream → DataStream

遍历数据流中的每一个元素，产生一个新的元素。

### FlatMap

DataStream → DataStream

遍历数据流中的每一个元素，产生N个元素 N=0，1，2,......。

### Filter

DataStream → DataStream

过滤算子，根据数据流的元素计算出一个boolean类型的值，true代表保留，false代表过滤掉。

### KeyBy

DataStream → KeyedStream

根据数据流中指定的字段来分区，相同指定字段值的数据一定是在同一个分区中，内部分区使用的是HashPartitioner。

指定分区字段的方式有三种：

1、根据索引号指定
2、通过匿名函数来指定
3、通过实现KeySelector接口  指定分区字段

```scala
	val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.generateSequence(1, 100)
    stream
      .map(x => (x % 3, 1))
      //根据索引号来指定分区字段
      //      .keyBy(0)
      //通过传入匿名函数 指定分区字段
      //      .keyBy(x=>x._1)
      //通过实现KeySelector接口  指定分区字段
      .keyBy(new KeySelector[(Long, Int), Long] {
      override def getKey(value: (Long, Int)): Long = value._1
    })
      .sum(1)
      .print()
    env.execute()
```

### Reduce

KeyedStream：根据key分组 → DataStream

**注意，reduce是基于分区后的流对象进行聚合，也就是说，DataStream类型的对象无法调用reduce方法**。

```scala
.reduce((v1,v2) => (v1._1,v1._2 + v2._2))
```

代码例子：读取kafka数据，实时统计各个卡口下的车流量。

- 实现kafka生产者，读取卡口数据并且往kafka中生产数据：

```scala
 	val prop = new Properties()
    prop.setProperty("bootstrap.servers", "node01:9092,node02:9092,node03:9092")
    prop.setProperty("key.serializer", classOf[StringSerializer].getName)
    prop.setProperty("value.serializer", classOf[StringSerializer].getName)

    val producer = new KafkaProducer[String, String](prop)

    val iterator = Source.fromFile("data/carFlow_all_column_test.txt", "UTF-8").getLines()
    for (i <- 1 to 100) {
      for (line <- iterator) {
        //将需要的字段值 生产到kafka集群  car_id monitor_id event-time speed
        //车牌号 卡口号 车辆通过时间 通过速度
        val splits = line.split(",")
        val monitorID = splits(0).replace("'","")
        val car_id = splits(2).replace("'","")
        val eventTime = splits(4).replace("'","")
        val speed = splits(6).replace("'","")
        if (!"00000000".equals(car_id)) {
          val event = new StringBuilder
          event.append(monitorID + "\t").append(car_id+"\t").append(eventTime + "\t").append(speed)
          producer.send(new ProducerRecord[String, String]("flink-kafka", event.toString()))
        }

        Thread.sleep(500)
      }
    }
```

- 实现kafka消费者：

```scala
	val env = StreamExecutionEnvironment.getExecutionEnvironment
    val props = new Properties()
    props.setProperty("bootstrap.servers","node01:9092,node02:9092,node03:9092")
    props.setProperty("key.deserializer",classOf[StringDeserializer].getName)
    props.setProperty("value.deserializer",classOf[StringDeserializer].getName)
    props.setProperty("group.id","flink001")
    props.getProperty("auto.offset.reset","latest")

    val stream = env.addSource(new FlinkKafkaConsumer[String]("flink-kafka", new 		SimpleStringSchema(),props))
    stream.map(data => {
      val splits = data.split("\t")
      val carFlow = CarFlow(splits(0),splits(1),splits(2),splits(3).toDouble)
      (carFlow,1)
    }).keyBy(_._1.monitorId)
        .sum(1)
        .print()
    env.execute()
```

### Aggregations

KeyedStream → DataStream

Aggregations代表的是一类聚合算子，具体算子如下：

```scala
keyedStream.sum(0)
keyedStream.sum("key")
keyedStream.min(0)
keyedStream.min("key")
keyedStream.max(0)
keyedStream.max("key")
keyedStream.minBy(0)
keyedStream.minBy("key")
keyedStream.maxBy(0)
keyedStream.maxBy("key")
```

代码例子：实时统计各个卡口最先通过的汽车的信息

```scala
val stream = env.addSource(new FlinkKafkaConsumer[String]("flink-kafka", new SimpleStringSchema(),props))
    stream.map(data => {
      val splits = data.split("\t")
      val carFlow = CarFlow(splits(0),splits(1),splits(2),splits(3).toDouble)
      val eventTime = carFlow.eventTime
      val format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
      val date = format.parse(eventTime)
      (carFlow,date.getTime)
    }).keyBy(_._1.monitorId)
        .min(1)
        .map(_._1)
        .print()
    env.execute()
```

### Union 真合并

DataStream → DataStream

Union of two or more data streams creating a new stream containing all the elements from all the streams

**合并两个或者更多的数据流产生一个新的数据流，这个新的数据流中包含了所合并的数据流的元素**。

注意：需要保证数据流中元素类型一致

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
    val ds1 = env.fromCollection(List(("a",1),("b",2),("c",3)))
    val ds2 = env.fromCollection(List(("d",4),("e",5),("f",6)))
    val ds3 = env.fromCollection(List(("g",7),("h",8)))
    val unionStream = ds1.union(ds2,ds3)
    unionStream.print()
    env.execute()

输出：
("a", 1)
("b", 2)
("c", 3)
("d", 4)
("e", 5)
("f", 6)
("g", 7)
("h", 8)
```

### Connect 假合并

DataStream,DataStream → ConnectedStreams

合并两个数据流并且保留两个数据流的数据类型，能够共享两个流的状态

```scala
val ds1 = env.socketTextStream("node01", 8888)
val ds2 = env.socketTextStream("node01", 9999)
val wcStream1 = ds1.flatMap(_.split(" ")).map((_,1)).keyBy(0).sum(1)
val wcStream2 = ds2.flatMap(_.split(" ")).map((_,1)).keyBy(0).sum(1)
val restStream: ConnectedStreams[(String, Int), (String, Int)] = wcStream2.connect(wcStream1)
```

### CoMap, CoFlatMap

ConnectedStreams → DataStream

CoMap, CoFlatMap并不是具体算子名字，而是一类操作名称

凡是基于ConnectedStreams数据流做map遍历，这类操作叫做CoMap

凡是基于ConnectedStreams数据流做flatMap遍历，这类操作叫做CoFlatMap

**CoMap第一种实现方式：**

```scala
restStream.map(new CoMapFunction[(String,Int),(String,Int),(String,Int)] {
      //对第一个数据流做计算
      override def map1(value: (String, Int)): (String, Int) = {
        (value._1+":first",value._2+100)
      }
      //对第二个数据流做计算
      override def map2(value: (String, Int)): (String, Int) = {
        (value._1+":second",value._2*100)
      }
    }).print()
```

**CoMap第二种实现方式：**

```scala
restStream.map(
      //对第一个数据流做计算
      x=>{(x._1+":first",x._2+100)}
      //对第二个数据流做计算
      ,y=>{(y._1+":second",y._2*100)}
    ).print()
```

代码例子：现有一个配置文件存储车牌号与车主的真实姓名，通过数据流中的车牌号实时匹配出对应的车主姓名（注意：配置文件可能实时改变）

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setParallelism(1)
val filePath = "data/carId2Name"
val carId2NameStream = env.readFile(new TextInputFormat(new Path(filePath)),filePath,FileProcessingMode.PROCESS_CONTINUOUSLY,10)
val dataStream = env.socketTextStream("node01",8888)
dataStream.connect(carId2NameStream).map(new CoMapFunction[String,String,String] {
    private val hashMap = new mutable.HashMap[String,String]()
    override def map1(value: String): String = {
        hashMap.getOrElse(value,"not found name")
    }

    override def map2(value: String): String = {
        val splits = value.split(" ")
        hashMap.put(splits(0),splits(1))
        value + "加载完毕..."
    }
}).print()
env.execute()
```

**CoFlatMap第一种实现方式：**

```scala
ds1.connect(ds2).flatMap((x,c:Collector[String])=>{
      //对第一个数据流做计算
      x.split(" ").foreach(w=>{
        c.collect(w)
      })

    }
      //对第二个数据流做计算
      ,(y,c:Collector[String])=>{
      y.split(" ").foreach(d=>{
        c.collect(d)
      })
    }).print
```

**CoFlatMap第二种实现方式：**

```scala
 ds1.connect(ds2).flatMap(
      //对第一个数据流做计算
      x=>{
      x.split(" ")
    }
      //对第二个数据流做计算
      ,y=>{
        y.split(" ")
      }).print()
```

**CoFlatMap第三种实现方式：**

```scala
ds1.connect(ds2).flatMap(new CoFlatMapFunction[String,String,(String,Int)] {
    //对第一个数据流做计算 
    override def flatMap1(value: String, out: Collector[(String, Int)]): Unit = {
        val words = value.split(" ")
        words.foreach(x=>{
          out.collect((x,1))
        })
      }

    //对第二个数据流做计算
    override def flatMap2(value: String, out: Collector[(String, Int)]): Unit = {
        val words = value.split(" ")
        words.foreach(x=>{
          out.collect((x,1))
        })
      }
    }).print()
```

### Split

DataStream → SplitStream

根据条件将一个流分成两个或者更多的流

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,100)
val splitStream = stream.split(
    d => {
        d % 2 match {
            case 0 => List("even")
            case 1 => List("odd")
        }
    }
)
splitStream.select("even").print()
env.execute()
```

### Select

SplitStream → DataStream

从SplitStream中选择一个或者多个数据流

```scala
splitStream.select("even").print()
```

### Iterate

DataStream → IterativeStream → DataStream

Iterate算子提供了对数据流迭代的支持

迭代由两部分组成：迭代体、终止迭代条件，不满足终止迭代条件的数据流会返回到stream流中，进行下一次迭代，满足终止迭代条件的数据流继续往下游发送：

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val initStream = env.socketTextStream("node01",8888)
val stream = initStream.map(_.toLong)
stream.iterate {
    iteration => {
        //定义迭代逻辑
        val iterationBody = iteration.map ( x => {
            println(x)
            if(x > 0) x - 1
            else x
        } )
        //> 0  大于0的值继续返回到stream流中,当 <= 0 继续往下游发送
        (iterationBody.filter(_ > 0), iterationBody.filter(_ <= 0))
    }
}.print()
env.execute()
```

### 函数类和富函数类

在使用Flink算子的时候，可以通过传入匿名函数和函数类对象。

函数类分为：普通函数类、富函数类。

富函数类相比于普通的函数，可以获取运行环境的上下文（Context），拥有一些生命周期方法，管理状态，可以实现更加复杂的功能

| 普通函数类      | 富函数类            |
| :-------------- | ------------------- |
| MapFunction     | RichMapFunction     |
| FlatMapFunction | RichFlatMapFunction |
| FilterFunction  | RichFilterFunction  |
| ......          | ......              |

- 使用普通函数类过滤掉车速高于100的车辆信息

```scala
	val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.readTextFile("./data/carFlow_all_column_test.txt")
    stream.filter(new FilterFunction[String] {
      override def filter(value: String): Boolean = {
        if (value != null && !"".equals(value)) {
          val speed = value.split(",")(6).replace("'", "").toLong
          if (speed > 100)
            false
          else
            true
        }else
          false
      }
    }).print()
    env.execute()

```

- 使用富函数类，将车牌号转化成车主真实姓名，映射表存储在Redis中

添加redis依赖，数据写入到redis。

```xml
<dependency>
		<groupId>redis.clients</groupId>
		<artifactId>jedis</artifactId>
		<version>${redis.version}</version>
</dependency>
```

```scala
	val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.socketTextStream("node01", 8888)
    stream.map(new RichMapFunction[String, String] {

      private var jedis: Jedis = _

      //初始化函数  在每一个thread启动的时候（处理元素的时候，会调用一次）
      //在open中可以创建连接redis的连接
      override def open(parameters: Configuration): Unit = {
        //getRuntimeContext可以获取flink运行的上下文环境  AbstractRichFunction抽象类提供的
        val taskName = getRuntimeContext.getTaskName
        val subtasks = getRuntimeContext.getTaskNameWithSubtasks
        println("=========open======"+"taskName:" + taskName + "\tsubtasks:"+subtasks)
        jedis = new Jedis("node01", 6379)
        jedis.select(3)
      }

      //每处理一个元素，就会调用一次
      override def map(value: String): String = {
        val name = jedis.get(value)
        if(name == null){
          "not found name"
        }else
          name
      }

      //元素处理完毕后，会调用close方法
      //关闭redis连接
      override def close(): Unit = {
        jedis.close()
      }
    }).setParallelism(2).print()

    env.execute()
```

### ProcessFunction（处理函数）

ProcessFunction属于低层次的API，我们前面讲的map、filter、flatMap等算子都是基于这层高层封装出来的。

越低层次的API，功能越强大，用户能够获取的信息越多，比如可以拿到元素状态信息、事件时间、设置定时器等

- 代码例子：监控每辆汽车，车速超过100迈，2s钟后发出超速的警告通知：

  ```scala
  object MonitorOverSpeed02 {
    case class CarInfo(carId:String,speed:Long)
    def main(args: Array[String]): Unit = {
      val env = StreamExecutionEnvironment.getExecutionEnvironment
      val stream = env.socketTextStream("node01",8888)
      stream.map(data => {
        val splits = data.split(" ")
        val carId = splits(0)
        val speed = splits(1).toLong
        CarInfo(carId,speed)
      }).keyBy(_.carId)
        //KeyedStream调用process需要传入KeyedProcessFunction
        //DataStream调用process需要传入ProcessFunction
        .process(new KeyedProcessFunction[String,CarInfo,String] {
  
        override def processElement(value: CarInfo, ctx: KeyedProcessFunction[String, CarInfo, String]#Context, out: Collector[String]): Unit = {
          val currentTime = ctx.timerService().currentProcessingTime()
          if(value.speed > 100 ){
            val timerTime = currentTime + 2 * 1000
            ctx.timerService().registerProcessingTimeTimer(timerTime)
          }
        }
  
        override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[String, CarInfo, String]#OnTimerContext, out: Collector[String]): Unit = {
          var warnMsg = "warn... time:" + timestamp + "  carID:" + ctx.getCurrentKey
          out.collect(warnMsg)
        }
      }).print()
  
      env.execute()
    }
  }
  ```

### 总结

使用Map Filter....算子的适合，可以直接传入一个匿名函数、普通函数类对象(MapFuncation FilterFunction)，富函数类对象（RichMapFunction、RichFilterFunction），传入的富函数类对象：可以拿到任务执行的上下文，生命周期方法、管理状态.....。

如果业务比较复杂，通过Flink提供这些算子无法满足我们的需求，通过process算子直接使用比较底层API（获取上下文、生命周期方法、测输出流、时间服务等）。

KeyedDataStream调用process，KeyedProcessFunction 。

DataStream调用process，ProcessFunction 。

## Sink

Flink内置了大量sink，可以将Flink处理后的数据输出到HDFS、kafka、Redis、ES、MySQL等。

工程场景中，会经常消费kafka中数据，处理结果存储到Redis或者MySQL中

### Redis Sink

Flink处理的数据可以存储到Redis中，以便实时查询

Flink内嵌连接Redis的连接器，只需要导入连接Redis的依赖就可以

```xml
<dependency>
    <groupId>org.apache.bahir</groupId>
    <artifactId>flink-connector-redis_2.11</artifactId>
</dependency>
```

WordCount写入到Redis中，选择的是HSET数据类型，代码如下：

```scala
	val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.socketTextStream("node01",8888)
    val result = stream.flatMap(_.split(" "))
      .map((_, 1))
      .keyBy(0)
      .sum(1)

    //若redis是单机
    val config = new FlinkJedisPoolConfig.Builder().setDatabase(3).setHost("node01").setPort(6379).build()
    //如果是 redis集群
    /*val addresses = new util.HashSet[InetSocketAddress]()
    addresses.add(new InetSocketAddress("node01",6379))
    addresses.add(new InetSocketAddress("node01",6379))
   val clusterConfig = new FlinkJedisClusterConfig.Builder().setNodes(addresses).build()*/

    result.addSink(new RedisSink[(String,Int)](config,new RedisMapper[(String,Int)] {

      override def getCommandDescription: RedisCommandDescription = {
        new RedisCommandDescription(RedisCommand.HSET,"wc")
      }

      override def getKeyFromData(t: (String, Int))  = {
        t._1
      }

      override def getValueFromData(t: (String, Int))  = {
        t._2 + ""
      }
    }))
    env.execute()
```

### Kafka Sink

处理结果写入到kafka topic中，Flink也是默认支持，需要添加连接器依赖，跟读取kafka数据用的连接器依赖相同，之前添加过就不需要再次添加了

```xml
		<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_2.11</artifactId>
            <version>${flink-version}</version>
        </dependency>
```

```scala
import java.lang
import java.util.Properties

import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaProducer, KafkaSerializationSchema}
import org.apache.kafka.clients.producer.ProducerRecord
import org.apache.kafka.common.serialization.StringSerializer

object KafkaSink {
  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.socketTextStream("node01",8888)
    val result = stream.flatMap(_.split(" "))
      .map((_, 1))
      .keyBy(0)
      .sum(1)

    val props = new Properties()
    props.setProperty("bootstrap.servers","node01:9092,node02:9092,node03:9092")
//    props.setProperty("key.serializer",classOf[StringSerializer].getName)
//    props.setProperty("value.serializer",classOf[StringSerializer].getName)


    /**
    public FlinkKafkaProducer(
     FlinkKafkaProducer(defaultTopic: String, serializationSchema: KafkaSerializationSchema[IN], producerConfig: Properties, semantic: FlinkKafkaProducer.Semantic)
      */
    result.addSink(new FlinkKafkaProducer[(String,Int)]("wc",new KafkaSerializationSchema[(String, Int)] {
      override def serialize(element: (String, Int), timestamp: lang.Long): ProducerRecord[Array[Byte], Array[Byte]] = {
        new ProducerRecord("wc",element._1.getBytes(),(element._2+"").getBytes())
      }
    },props,FlinkKafkaProducer.Semantic.EXACTLY_ONCE))

    env.execute()
  }
}
```

### MySQL Sink

Flink处理结果写入到MySQL中，这并不是Flink默认支持的，需要添加MySQL的驱动依赖

```xml
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <version>5.1.44</version>
</dependency>
```

因为不是内嵌支持的，所以需要基于RichSinkFunction自定义sink。

代码例子：消费kafka中数据，统计各个卡口的流量，并且存入到MySQL中

注意点：需要去重，操作MySQL需要幂等性

```scala
import java.sql.{Connection, DriverManager, PreparedStatement}
import java.util.Properties

import org.apache.flink.api.common.functions.ReduceFunction
import org.apache.flink.api.common.typeinfo.TypeInformation
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.functions.sink.{RichSinkFunction, SinkFunction}
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaConsumer, KafkaDeserializationSchema}
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringSerializer

object MySQLSink {

  case class CarInfo(monitorId: String, carId: String, eventTime: String, Speed: Long)

  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //设置连接kafka的配置信息
    val props = new Properties()
    //注意   sparkstreaming + kafka（0.10之前版本） receiver模式  zookeeper url（元数据）
    props.setProperty("bootstrap.servers", "node01:9092,node02:9092,node03:9092")
    props.setProperty("group.id", "flink-kafka-001")
    props.setProperty("key.deserializer", classOf[StringSerializer].getName)
    props.setProperty("value.deserializer", classOf[StringSerializer].getName)

    //第一个参数 ： 消费的topic名
    val stream = env.addSource(new FlinkKafkaConsumer[(String, String)]("flink-kafka", new KafkaDeserializationSchema[(String, String)] {
      //什么时候停止，停止条件是什么
      override def isEndOfStream(t: (String, String)): Boolean = false

      //要进行序列化的字节流
      override def deserialize(consumerRecord: ConsumerRecord[Array[Byte], Array[Byte]]): (String, String) = {
        val key = new String(consumerRecord.key(), "UTF-8")
        val value = new String(consumerRecord.value(), "UTF-8")
        (key, value)
      }

      //指定一下返回的数据类型  Flink提供的类型
      override def getProducedType: TypeInformation[(String, String)] = {
        createTuple2TypeInformation(createTypeInformation[String], createTypeInformation[String])
      }
    }, props))

    stream.map(data => {
      val value = data._2
      val splits = value.split("\t")
      val monitorId = splits(0)
      (monitorId, 1)
    }).keyBy(_._1)
      .reduce(new ReduceFunction[(String, Int)] {
        //t1:上次聚合完的结果  t2:当前的数据
        override def reduce(t1: (String, Int), t2: (String, Int)): (String, Int) = {
          (t1._1, t1._2 + t2._2)
        }
      }).addSink(new MySQLCustomSink)

    env.execute()
  }

  //幂等性写入外部数据库MySQL
  class MySQLCustomSink extends RichSinkFunction[(String, Int)] {
    var conn: Connection = _
    var insertPst: PreparedStatement = _
    var updatePst: PreparedStatement = _

    //每来一个元素都会调用一次
    override def invoke(value: (String, Int), context: SinkFunction.Context[_]): Unit = {
      println(value)
      updatePst.setInt(1, value._2)
      updatePst.setString(2, value._1)
      updatePst.execute()
      println(updatePst.getUpdateCount)
      if(updatePst.getUpdateCount == 0){
        println("insert")
        insertPst.setString(1, value._1)
        insertPst.setInt(2, value._2)
        insertPst.execute()
      }
    }

    //thread初始化的时候执行一次
    override def open(parameters: Configuration): Unit = {
      conn = DriverManager.getConnection("jdbc:mysql://node01:3306/test", "root", "123123")
      insertPst = conn.prepareStatement("INSERT INTO car_flow(monitorId,count) VALUES(?,?)")
      updatePst = conn.prepareStatement("UPDATE car_flow SET count = ? WHERE monitorId = ?")
    }

    //thread关闭的时候 执行一次
    override def close(): Unit = {
      insertPst.close()
      updatePst.close()
      conn.close()
    }
  }

}
```

### Socket Sink

Flink处理结果发送到套接字（Socket），基于RichSinkFunction自定义sink：

```scala
import java.io.PrintStream
import java.net.{InetAddress, Socket}
import java.util.Properties

import org.apache.flink.api.common.functions.ReduceFunction
import org.apache.flink.api.common.typeinfo.TypeInformation
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.functions.sink.{RichSinkFunction, SinkFunction}
import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, createTuple2TypeInformation, createTypeInformation}
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaConsumer, KafkaDeserializationSchema}
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringSerializer

//sink 到 套接字 socket
object SocketSink {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //设置连接kafka的配置信息
    val props = new Properties()
    //注意   sparkstreaming + kafka（0.10之前版本） receiver模式  zookeeper url（元数据）
    props.setProperty("bootstrap.servers", "node01:9092,node02:9092,node03:9092")
    props.setProperty("group.id", "flink-kafka-001")
    props.setProperty("key.deserializer", classOf[StringSerializer].getName)
    props.setProperty("value.deserializer", classOf[StringSerializer].getName)

    //第一个参数 ： 消费的topic名
    val stream = env.addSource(new FlinkKafkaConsumer[(String, String)]("flink-kafka", new KafkaDeserializationSchema[(String, String)] {
      //什么时候停止，停止条件是什么
      override def isEndOfStream(t: (String, String)): Boolean = false

      //要进行序列化的字节流
      override def deserialize(consumerRecord: ConsumerRecord[Array[Byte], Array[Byte]]): (String, String) = {
        val key = new String(consumerRecord.key(), "UTF-8")
        val value = new String(consumerRecord.value(), "UTF-8")
        (key, value)
      }

      //指定一下返回的数据类型  Flink提供的类型
      override def getProducedType: TypeInformation[(String, String)] = {
        createTuple2TypeInformation(createTypeInformation[String], createTypeInformation[String])
      }
    }, props))

    stream.map(data => {
      val value = data._2
      val splits = value.split("\t")
      val monitorId = splits(0)
      (monitorId, 1)
    }).keyBy(_._1)
      .reduce(new ReduceFunction[(String, Int)] {
        //t1:上次聚合完的结果  t2:当前的数据
        override def reduce(t1: (String, Int), t2: (String, Int)): (String, Int) = {
          (t1._1, t1._2 + t2._2)
        }
      }).addSink(new SocketCustomSink("node01",8888))

    env.execute()
  }

  class SocketCustomSink(host:String,port:Int) extends RichSinkFunction[(String,Int)]{
    var socket: Socket  = _
    var writer:PrintStream = _

    override def open(parameters: Configuration): Unit = {
      socket = new Socket(InetAddress.getByName(host), port)
      writer = new PrintStream(socket.getOutputStream)
    }

    override def invoke(value: (String, Int), context: SinkFunction.Context[_]): Unit = {
      writer.println(value._1 + "\t" +value._2)
      writer.flush()
    }

    override def close(): Unit = {
      writer.close()
      socket.close()
    }
  }
}
```

### File Sink

Flink处理的结果保存到文件，这种使用方式不是很常见

支持分桶写入，每一个桶就是一个目录，默认每隔一个小时会产生一个分桶，每个桶下面会存储每一个Thread的处理结果，可以设置一些文件滚动的策略（文件打开、文件大小等），防止出现大量的小文件。

Flink默认支持，导入连接文件的连接器依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-filesystem_2.11</artifactId>
     <version>1.9.2</version>
 </dependency>
```

```scala
import org.apache.flink.api.common.functions.ReduceFunction
import org.apache.flink.api.common.serialization.SimpleStringEncoder
import org.apache.flink.api.common.typeinfo.TypeInformation
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy
import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, createTuple2TypeInformation, createTypeInformation}
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaConsumer, KafkaDeserializationSchema}
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringSerializer

object FileSink {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //设置连接kafka的配置信息
    val props = new Properties()
    //注意   sparkstreaming + kafka（0.10之前版本） receiver模式  zookeeper url（元数据）
    props.setProperty("bootstrap.servers", "node01:9092,node02:9092,node03:9092")
    props.setProperty("group.id", "flink-kafka-001")
    props.setProperty("key.deserializer", classOf[StringSerializer].getName)
    props.setProperty("value.deserializer", classOf[StringSerializer].getName)

    //第一个参数 ： 消费的topic名
    val stream = env.addSource(new FlinkKafkaConsumer[(String, String)]("flink-kafka", new KafkaDeserializationSchema[(String, String)] {
      //什么时候停止，停止条件是什么
      override def isEndOfStream(t: (String, String)): Boolean = false

      //要进行序列化的字节流
      override def deserialize(consumerRecord: ConsumerRecord[Array[Byte], Array[Byte]]): (String, String) = {
        val key = new String(consumerRecord.key(), "UTF-8")
        val value = new String(consumerRecord.value(), "UTF-8")
        (key, value)
      }

      //指定一下返回的数据类型  Flink提供的类型
      override def getProducedType: TypeInformation[(String, String)] = {
        createTuple2TypeInformation(createTypeInformation[String], createTypeInformation[String])
      }
    }, props))

    val restStream = stream.map(data => {
      val value = data._2
      val splits = value.split("\t")
      val monitorId = splits(0)
      (monitorId, 1)
    }).keyBy(_._1)
      .reduce(new ReduceFunction[(String, Int)] {
        //t1:上次聚合完的结果  t2:当前的数据
        override def reduce(t1: (String, Int), t2: (String, Int)): (String, Int) = {
          (t1._1, t1._2 + t2._2)
        }
      }).map(x=>x._1 + "\t" + x._2)

      //设置文件滚动策略
    val rolling:DefaultRollingPolicy[String,String] = DefaultRollingPolicy.create()
      //当文件超过2s没有写入新数据，则滚动产生一个小文件
      .withInactivityInterval(2000)
      //文件打开时间超过2s 则滚动产生一个小文件  每隔2s产生一个小文件
      .withRolloverInterval(2000)
      //当文件大小超过256 则滚动产生一个小文件
      .withMaxPartSize(256*1024*1024)
      .build()

    /**
      * 默认：
      * 每一个小时对应一个桶（文件夹），每一个thread处理的结果对应桶下面的一个小文件
      * 当小文件大小超过128M或者小文件打开时间超过60s,滚动产生第二个小文件
      */
     val sink: StreamingFileSink[String] = StreamingFileSink.forRowFormat(
      new Path("d:/data/rests"),
      new SimpleStringEncoder[String]("UTF-8"))
         .withBucketCheckInterval(1000)
         .withRollingPolicy(rolling)
         .build()

//    val sink = StreamingFileSink.forBulkFormat(
//      new Path("./data/rest"),
//      ParquetAvroWriters.forSpecificRecord(classOf[String])
//    ).build()

    restStream.addSink(sink)
    env.execute()
  }
}
```

### HBase Sink

计算结果写入sink 两种实现方式：

1. map算子写入，频繁创建hbase连接。
2. process写入，适合批量写入hbase。

导入HBase依赖包

```xml
		<dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>${hbase.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-common</artifactId>
            <version>${hbase.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>${hbase.version}</version>
        </dependency>
```

读取kafka数据，统计卡口流量保存至HBase数据库中

1. HBase中创建对应的表

```
create 'car_flow',{NAME => 'count', VERSIONS => 1}
```

2. 实现代码

```scala
import java.util.{Date, Properties}

import com.msb.stream.util.{DateUtils, HBaseUtil}
import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.functions.ProcessFunction
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer
import org.apache.flink.util.Collector
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.client.{HTable, Put}
import org.apache.hadoop.hbase.util.Bytes
import org.apache.kafka.common.serialization.StringSerializer


object HBaseSinkTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //设置连接kafka的配置信息
    val props = new Properties()
    //注意   sparkstreaming + kafka（0.10之前版本） receiver模式  zookeeper url（元数据）
    props.setProperty("bootstrap.servers", "node01:9092,node02:9092,node03:9092")
    props.setProperty("group.id", "flink-kafka-001")
    props.setProperty("key.deserializer", classOf[StringSerializer].getName)
    props.setProperty("value.deserializer", classOf[StringSerializer].getName)

    val stream = env.addSource(new FlinkKafkaConsumer[String]("flink-kafka", new SimpleStringSchema(), props))


    stream.map(row => {
      val arr = row.split("\t")
      (arr(0), 1)
    }).keyBy(_._1)
      .reduce((v1: (String, Int), v2: (String, Int)) => {
        (v1._1, v1._2 + v2._2)
      }).process(new ProcessFunction[(String, Int), (String, Int)] {

      var htab: HTable = _

      override def open(parameters: Configuration): Unit = {
        val conf = HBaseConfiguration.create()
        conf.set("hbase.zookeeper.quorum", "node01:2181,node02:2181,node03:2181")
        val hbaseName = "car_flow"
        htab = new HTable(conf, hbaseName)
      }

      override def close(): Unit = {
        htab.close()
      }

      override def processElement(value: (String, Int), ctx: ProcessFunction[(String, Int), (String, Int)]#Context, out: Collector[(String, Int)]): Unit = {
        // rowkey:monitorid   时间戳（分钟） value：车流量
        val min = DateUtils.getMin(new Date())
        val put = new Put(Bytes.toBytes(value._1))
        put.addColumn(Bytes.toBytes("count"), Bytes.toBytes(min), Bytes.toBytes(value._2))
        htab.put(put)
      }
    })
    env.execute()
  }
}
```

## 分区策略
在 Apache Flink 中，分区（Partitioning）是将数据流按照一定的规则划分成多个子数据流或分片，以便在不同的并行任务或算子中并行处理数据。分区是实现并行计算和数据流处理的基础机制。Flink 的分区决定了数据在作业中的流动方式，以及在并行任务之间如何分配和处理数据。

在 Flink 中，数据流可以看作是一个有向图，图中的节点代表算子（Operators），边代表数据流（Data Streams）。数据从源算子流向下游算子，这些算子可能并行地处理输入数据，而分区就是决定数据如何从一个算子传递到另一个算子的机制。

### shuffle  

场景：增大分区、提高并行度，解决数据倾斜

DataStream → DataStream

**分区元素随机均匀分发到下游分区，网络开销比较大**

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,10).setParallelism(1)
println(stream.getParallelism)
stream.shuffle.print()
env.execute()
```

console result：上游数据比较随意的分发到下游

```scala
2> 1
1> 4
7> 10
4> 6
6> 3
5> 7
8> 2
1> 5
1> 8
1> 9
```

### rebalance 

场景：增大分区、提高并行度，解决数据倾斜

DataStream → DataStream

轮询分区元素，均匀的将元素分发到下游分区，下游每个分区的数据比较均匀，在发生数据倾斜时非常有用，网络开销比较大

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setParallelism(3)
val stream = env.generateSequence(1,100)
val shuffleStream = stream.rebalance
shuffleStream.print()
env.execute()
```

console result：上游数据比较均匀的分发到下游

```scala
8> 6
3> 1
5> 3
7> 5
1> 7
2> 8
6> 4
4> 2
3> 9
4> 10
```

### rescale

场景：减少分区  防止发生大量的网络传输   不会发生全量的重分区

DataStream → DataStream

通过轮询分区元素，将一个元素集合从上游分区发送给下游分区，发送单位是集合，而不是一个个元素

注意：rescale发生的是本地数据传输，而不需要通过网络传输数据，比如taskmanager的槽数。简单来说，上游的数据只会发送给本TaskManager中的下游。

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,10).setParallelism(2)
stream.writeAsText("./data/stream1").setParallelism(2)
stream.rescale.writeAsText("./data/stream2").setParallelism(4)
env.execute()
```

console result：stream1:1内容分发给stream2:1和stream2:2

stream1:1

```scala
1
3
5
7
9
```

stream1:2

```scala
2
4
6
8
10
```

stream2:1

```scala
1
5
9
```

stream2:2

```scala
3
7
```

stream2:3

```scala
2
6
10
```

stream2:4

```scala
4
8
```

### broadcast

场景：需要使用映射表、并且映射表会经常发生变动的场景

DataStream → DataStream

上游中每一个元素内容广播到下游每一个分区中

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,10).setParallelism(2)
stream.writeAsText("./data/stream1").setParallelism(2)
stream.broadcast.writeAsText("./data/stream2").setParallelism(4)
env.execute()
```

console result：stream1:1、2内容广播到了下游每个分区中

stream1:1

```scala
1
3
5
7
9
```

stream1:2

```scala
2
4
6
8
10
```

stream2:1

```scala
1
3
5
7
9
2
4
6
8
10
```

### global

场景：并行度降为1

DataStream → DataStream

上游分区的数据只分发给下游的第一个分区

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,10).setParallelism(2)
stream.writeAsText("./data/stream1").setParallelism(2)
stream.global.writeAsText("./data/stream2").setParallelism(4)
env.execute()
```

console result：stream1:1、2内容只分发给了stream2:1

stream1:1

```scala
1
3
5
7
9
```

stream1:2

```scala
2
4
6
8
10
```

stream2:1

```scala
1
3
5
7
9
2
4
6
8
10
```

### forward

场景：一对一的数据分发，map、flatMap、filter 等都是这种分区策略

DataStream → DataStream

上游分区数据分发到下游对应分区中

partition1->partition1

partition2->partition2

注意：必须保证上下游分区数（并行度）一致，不然会有如下异常:

```scala
Forward partitioning does not allow change of parallelism
* Upstream operation: Source: Sequence Source-1 parallelism: 2,
* downstream operation: Sink: Unnamed-4 parallelism: 4
* stream.forward.writeAsText("./data/stream2").setParallelism(4)
```

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,10).setParallelism(2)
stream.writeAsText("./data/stream1").setParallelism(2)
stream.forward.writeAsText("./data/stream2").setParallelism(2)
env.execute()
```

console result：stream1:1->stream2:1、stream1:2->stream2:2

stream1:1

```scala
1
3
5
7
9
```

stream1:2

```scala
2
4
6
8
10
```

stream2:1

```scala
1
3
5
7
9
```

stream2:2

```scala
2
4
6
8
10
```

### keyBy

场景：与业务场景匹配

DataStream → DataStream

根据上游分区元素的Hash值与下游分区数取模计算出，将当前元素分发到下游哪一个分区

```scala
MathUtils.murmurHash(keyHash)（每个元素的Hash值） % maxParallelism（下游分区数）
```

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val stream = env.generateSequence(1,10).setParallelism(2)
stream.writeAsText("./data/stream1").setParallelism(2)
stream.keyBy(0).writeAsText("./data/stream2").setParallelism(2)
env.execute()
```

console result：根据元素Hash值分发到下游分区中

### PartitionCustom

DataStream → DataStream

通过自定义的分区器，来决定元素是如何从上游分区分发到下游分区

```scala
object ShuffleOperator {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(2)
    val stream = env.generateSequence(1,10).map((_,1))
    stream.writeAsText("./data/stream1")
    stream.partitionCustom(new customPartitioner(),0)
      .writeAsText("./data/stream2").setParallelism(4)
    env.execute()
  }
  class customPartitioner extends Partitioner[Long]{
    override def partition(key: Long, numPartitions: Int): Int = {
      key.toInt % numPartitions
    }
  }
}
```

## Flink State状态

Flink是一个有状态的流式计算引擎，所以会将中间计算结果(状态)进行保存，默认保存到TaskManager的堆内存中，但是当task挂掉，那么这个task所对应的状态都会被清空，造成了数据丢失，无法保证结果的正确性，哪怕想要得到正确结果，所有数据都要重新计算一遍，效率很低。想要保证 At -least-once 和 Exactly-once，需要把数据状态持久化到更安全的存储介质中，Flink提供了堆内内存、堆外内存、HDFS、RocksDB等存储介质。

先来看下Flink提供的状态有哪些，Flink中状态分为两种类型：

- Keyed State

  基于KeyedStream上的状态，这个状态是跟特定的Key绑定，KeyedStream流上的每一个Key都对应一个State，每一个Operator可以启动多个Thread处理，但是相同Key的数据只能由同一个Thread处理，因此一个Keyed状态只能存在于某一个Thread中，一个Thread会有多个Keyed state。

- Non-Keyed State（Operator State）

  Operator State与Key无关，而是与Operator绑定，整个Operator只对应一个State。比如：Flink中的Kafka Connector就使用了Operator State，它会在每个Connector实例中，保存该实例消费Topic的所有(partition, offset)映射。

Flink针对Keyed State提供了以下可以保存State的数据结构

- ValueState<T>:类型为T的单值状态，这个状态与对应的Key绑定，最简单的状态，通过update更新值，通过value获取状态值。
- ListState<T>：Key上的状态值为一个列表，这个列表可以通过add方法往列表中添加值，也可以通过get()方法返回一个Iterable<T>来遍历状态值。
- ReducingState<T>：每次调用add()方法添加值的时候，会调用用户传入的reduceFunction，最后合并到一个单一的状态值。
- MapState<UK, UV>:状态值为一个Map，用户通过put或putAll方法添加元素，get(key)通过指定的key获取value，使用entries()、keys()、values()检索。
- AggregatingState`<IN, OUT>`:保留一个单值，表示添加到状态的所有值的聚合。和 `ReducingState` 相反的是, 聚合类型可能与添加到状态的元素的类型不同。使用 `add(IN)` 添加的元素会调用用户指定的 `AggregateFunction` 进行聚合。
- FoldingState<T, ACC>:已过时建议使用AggregatingState   保留一个单值，表示添加到状态的所有值的聚合。 与 `ReducingState` 相反，聚合类型可能与添加到状态的元素类型不同。 使用`add（T）`添加的元素会调用用户指定的 `FoldFunction` 折叠成聚合值。

案例1：使用ValueState keyed state检查车辆是否发生了急加速

```scala
object ValueStateTest {

  case class CarInfo(carId: String, speed: Long)

  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.socketTextStream("node01", 8888)
    stream.map(data => {
      val arr = data.split(" ")
      CarInfo(arr(0), arr(1).toLong)
    }).keyBy(_.carId)
      .map(new RichMapFunction[CarInfo, String]() {

        //保存上一次车速
        private var lastTempState: ValueState[Long] = _

        override def open(parameters: Configuration): Unit = {
          val lastTempStateDesc = new ValueStateDescriptor[Long]("lastTempState", createTypeInformation[Long])
          lastTempState = getRuntimeContext.getState(lastTempStateDesc)
        }

        override def map(value: CarInfo): String = {
          val lastSpeed = lastTempState.value()
          this.lastTempState.update(value.speed)
          if ((value.speed - lastSpeed).abs > 30 && lastSpeed != 0)
            "over speed" + value.toString
          else
            value.carId
        }
      }).print()
    env.execute()
  }
}
```

案例2：使用 MapState 统计单词出现次数

```scala
import org.apache.flink.api.common.functions.RichMapFunction
import org.apache.flink.api.common.state.{MapState, MapStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._

//MapState 实现 WordCount
object KeyedStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.fromCollection(List("I love you","hello spark","hello flink","hello hadoop"))
    val pairStream = stream.flatMap(_.split(" ")).map((_,1)).keyBy(_._1)
    pairStream.map(new RichMapFunction[(String,Int),(String,Int)] {

      private var map:MapState[String,Int] = _
      override def open(parameters: Configuration): Unit = {
        //定义map state存储的数据类型
        val desc = new MapStateDescriptor[String,Int]("sum",createTypeInformation[String],createTypeInformation[Int])
        //注册map state
        map = getRuntimeContext.getMapState(desc)
      }

      override def map(value: (String, Int)): (String, Int) = {
        val key = value._1
        val v = value._2
        if(map.contains(key)){
          map.put(key,map.get(key) + 1)
        }else{
          map.put(key,1)
        }
        val iterator = map.keys().iterator()
        while (iterator.hasNext){
          val key = iterator.next()
          println("word:" + key + "\t count:" + map.get(key))
        }
        value
      }
    }).setParallelism(3)
    env.execute()
  }
}

```

案例3：使用ReducingState统计每辆车的速度总和

```scala
import com.msb.state.ValueStateTest.CarInfo
import org.apache.flink.api.common.functions.{ReduceFunction, RichMapFunction}
import org.apache.flink.api.common.state.{ReducingState, ReducingStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._

//统计每辆车的速度总和
object ReduceStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.socketTextStream("node01", 8888)
    stream.map(data => {
      val arr = data.split(" ")
      CarInfo(arr(0), arr(1).toLong)
    }).keyBy(_.carId)
      .map(new RichMapFunction[CarInfo, CarInfo] {
        private var reduceState: ReducingState[Long] = _

        override def map(elem: CarInfo): CarInfo = {
          reduceState.add(elem.speed)
          println("carId:" + elem.carId + " speed count:" + reduceState.get())
          elem
        }

        override def open(parameters: Configuration): Unit = {
          val reduceDesc = new ReducingStateDescriptor[Long]("reduceSpeed", new ReduceFunction[Long] {
            override def reduce(value1: Long, value2: Long): Long = value1 + value2
          }, createTypeInformation[Long])
          reduceState = getRuntimeContext.getReducingState(reduceDesc)
        }
      })
    env.execute()
  }
}
```

案例4：使用AggregatingState统计每辆车的速度总和

```scala
import com.msb.state.ValueStateTest.CarInfo
import org.apache.flink.api.common.functions.{AggregateFunction, ReduceFunction, RichMapFunction}
import org.apache.flink.api.common.state.{AggregatingState, AggregatingStateDescriptor, ReducingState, ReducingStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.api.scala._

//统计每辆车的速度总和
object ReduceStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.socketTextStream("node01", 8888)
    stream.map(data => {
      val arr = data.split(" ")
      CarInfo(arr(0), arr(1).toLong)
    }).keyBy(_.carId)
      .map(new RichMapFunction[CarInfo, CarInfo] {
        private var aggState: AggregatingState[Long,Long] = _

        override def map(elem: CarInfo): CarInfo = {
          aggState.add(elem.speed)
          println("carId:" + elem.carId + " speed count:" + aggState.get())
          elem
        }

        override def open(parameters: Configuration): Unit = {
          val aggDesc = new AggregatingStateDescriptor[Long,Long,Long]("agg",new AggregateFunction[Long,Long,Long] {
            //初始化累加器值
            override def createAccumulator(): Long = 0

            //往累加器中累加值
            override def add(value: Long, acc: Long): Long = acc + value

            //返回最终结果
            override def getResult(accumulator: Long): Long = accumulator

            //合并两个累加器值
            override def merge(a: Long, b: Long): Long = a+b
          },createTypeInformation[Long])

          aggState = getRuntimeContext.getAggregatingState(aggDesc)
        }
      })
    env.execute()
  }
}
```

### CheckPoint & SavePoint

有状态流应用中的检查点（checkpoint），其实就是所有任务的状态在某个时间点的一个快照（一份拷贝）。简单来讲，就是一次“存盘”，让我们之前处理数据的进度不要丢掉。在一个流应用程序运行时，Flink 会定期保存检查点，在检查点中会记录每个算子的 id 和状态；如果发生故障，Flink 就会用最近一次成功保存的检查点来恢复应用的状态，重新启动处理流程，就如同“读档”一样。

默认情况下，检查点是被禁用的，需要在代码中手动开启。直接调用执行环境的enableCheckpointing()方法就可以开启检查点。

```Java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();
env.enableCheckpointing(1000);
```

这里传入的参数是检查点的间隔时间，单位为毫秒。

除了检查点之外，Flink 还提供了“保存点”（savepoint）的功能。保存点在原理和形式上跟检查点完全一样，也是状态持久化保存的一个快照；**保存点与检查点最大的区别，就是触发的时机。检查点是由 Flink 自动管理的，定期创建，发生故障之后自动读取进行恢复，这是一个“自动存盘”的功能；而保存点不会自动创建，必须由用户明确地手动触发保存操作，所以就是“手动存盘”。因此两者尽管原理一致，但用途就有所差别了：检查点主要用来做故障恢复，是容错机制的核心；保存点则更加灵活，可以用来做有计划的手动备份和恢复**。

检查点具体的持久化存储位置，取决于“检查点存储”（CheckpointStorage）的设置。默认情况下，检查点存储在 JobManager 的堆（heap）内存中。而对于大状态的持久化保存，Flink也提供了在其他存储位置进行保存的接口，这就是 CheckpointStorage。具体可以通过调用检查点配置的 setCheckpointStorage()来配置，需要传入一个CheckpointStorage 的实现类。Flink 主要提供了两种 CheckpointStorage：作业管理器的堆内存（JobManagerCheckpointStorage）和文件系统（FileSystemCheckpointStorage）。对于实际生产应用，我们一般会将 CheckpointStorage 配置为高可用的分布式文件系统（HDFS，S3 等）。

Flink中基于异步轻量级的分布式快照技术提供了Checkpoint容错机制，分布式快照可以将同一时间点Task/Operator的状态数据全局统一快照处理，包括上面提到的用户自定义使用的Keyed State和Operator State，当未来程序出现问题，可以基于保存的快照容错。

#### CheckPoint原理

Flink会在输入的数据集上间隔性地生成checkpoint barrier，通过栅栏（barrier）将间隔时间段内的数据划分到相应的checkpoint中。当程序出现异常时，Operator就能够从上一次快照中恢复所有算子之前的状态，从而保证数据的一致性。例如在KafkaConsumer算子中维护offset状态，当系统出现问题无法从Kafka中消费数据时，可以将offset记录在状态中，当任务重新恢复时就能够从指定的偏移量开始消费数据。

默认情况Flink不开启检查点，用户需要在程序中通过调用方法配置和开启检查点，另外还可以调整其他相关参数

- Checkpoint开启和时间间隔指定

  开启检查点并且指定检查点时间间隔为1000ms，根据实际情况自行选择，如果状态比较大，则建议适当增加该值

  ```scala
  env.enableCheckpointing(1000)
  ```

- exactly-ance和at-least-once语义选择

  选择exactly-once语义保证整个应用内端到端的数据一致性，这种情况比较适合于数据要求比较高，不允许出现丢数据或者数据重复，与此同时，Flink的性能也相对较弱，而at-least-once语义更适合于时廷和吞吐量要求非常高但对数据的一致性要求不高的场景。如下通过setCheckpointingMode()方法来设定语义模式，默认情况下使用的是exactly-once模式

  ```scala
  env.getCheckpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)
  ```

- Checkpoint超时时间

  超时时间指定了每次Checkpoint执行过程中的上限时间范围，一旦Checkpoint执行时间超过该阈值，Flink将会中断Checkpoint过程，并按照超时处理。该指标可以通过setCheckpointTimeout方法设定，默认为10分钟

  ```scala
  env.getCheckpointConfig.setCheckpointTimeout(5 * 60 * 1000)
  ```

- Checkpoint之间最小时间间隔

  该参数主要目的是设定两个Checkpoint之间的最小时间间隔，防止Flink应用密集地触发Checkpoint操作，会占用了大量计算资源而影响到整个应用的性能

  ```scala
  env.getCheckpointConfig.setMinPauseBetweenCheckpoints(600)
  ```

- 最大并行执行的Checkpoint数量

  在默认情况下只有一个检查点可以运行，根据用户指定的数量可以同时触发多个Checkpoint，进而提升Checkpoint整体的效率

  ```scala
  env.getCheckpointConfig.setMaxConcurrentCheckpoints(1)
  ```

- 任务取消后，是否删除Checkpoint中保存的数据

  设置为RETAIN_ON_CANCELLATION：表示一旦Flink处理程序被cancel后，会保留CheckPoint数据，以便根据实际需要恢复到指定的CheckPoint

  设置为DELETE_ON_CANCELLATION：表示一旦Flink处理程序被cancel后，会删除CheckPoint数据，只有Job执行失败的时候才会保存CheckPoint

  ```scala
  env.getCheckpointConfig.enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION)
  ```

- 容忍的检查的失败数

  设置可以容忍的检查的失败数，超过这个数量则系统自动关闭和停止任务

  ```scala
  env.getCheckpointConfig.setTolerableCheckpointFailureNumber(1)
  ```

#### SavePoint原理

Savepoints 是检查点的一种特殊实现，底层实现其实也是使用Checkpoints的机制。Savepoints是用户以手工命令的方式触发Checkpoint，并将结果持久化到指定的存储路径中，其主要目的是帮助用户在升级和维护集群过程中保存系统中的状态数据，避免因为停机运维或者升级应用等正常终止应用的操作而导致系统无法恢复到原有的计算状态的情况，从而无法实现从端到端的 Excatly-Once 语义保证。

要使用Savepoints，需要按照以下步骤进行：

1. 配置状态后端： 在Flink中，状态可以保存在不同的后端存储中，例如内存、文件系统或分布式存储系统（如HDFS）。要启用Savepoint，您需要在Flink配置文件中配置合适的状态后端。通常，使用分布式存储系统作为状态后端是比较常见的做法，因为它可以提供更好的可靠性和容错性。

2. 生成Savepoint： 在您的Flink应用程序运行时，可以通过以下方式手动触发生成Savepoint：

   ```bash
   bin/flink savepoint <jobID> [targetDirectory]
   ```

   其中，`<jobID>`是您要保存状态的Flink作业的Job ID，`[targetDirectory]`是可选的目标目录，用于保存Savepoint数据。如果没有提供`targetDirectory`，Savepoint将会保存到Flink配置中所配置的状态后端中。

3. 恢复Savepoint： 要恢复到Savepoint状态，可以通过以下方式提交作业：

   ```bash
   bin/flink run -s :savepointPath [:runArgs]
   ```

   其中，`savepointPath`是之前生成的Savepoint的路径，`runArgs`是您提交作业时的其他参数。

4. 确保应用程序状态的兼容性： 在使用Savepoints时，应用程序的状态结构和代码必须与生成Savepoint的版本保持兼容。这意味着在更新应用程序代码后，可能需要做一些额外的工作来保证状态的向后兼容性，以便能够成功恢复到旧的Savepoint。

### StateBackend状态后端

在Flink中提供了StateBackend来存储和管理状态数据

Flink一共实现了三种类型的状态管理器：MemoryStateBackend、FsStateBackend、RocksDBStateBackend

#### MemoryStateBackend

基于内存的状态管理器将状态数据全部存储在JVM堆内存中。基于内存的状态管理具有非常快速和高效的特点，但也具有非常多的限制，最主要的就是内存的容量限制，一旦存储的状态数据过多就会导致系统内存溢出等问题，从而影响整个应用的正常运行。同时如果机器出现问题，整个主机内存中的状态数据都会丢失，进而无法恢复任务中的状态数据。因此从数据安全的角度建议用户尽可能地避免在生产环境中使用MemoryStateBackend。

Flink将MemoryStateBackend作为默认状态后端管理器

```scala
env.setStateBackend(new MemoryStateBackend(100*1024*1024))
```

注意：聚合类算子的状态会同步到JobManager内存中，因此对于聚合类算子比较多的应用会对JobManager的内存造成一定的压力，进而影响集群。

#### FsStateBackend

和MemoryStateBackend有所不同，FsStateBackend是基于文件系统的一种状态管理器，这里的文件系统可以是本地文件系统，也可以是HDFS分布式文件系统

```
env.setStateBackend(new FsStateBackend("path",true))
```

如果path是本地文件路径，其格式：file:///

如果path是HDFS文件路径，格式为：hdfs://

第二个参数代表是否异步保存状态数据到HDFS，异步方式能够尽可能避免checkpoint的过程中影响流式计算任务。FsStateBackend更适合任务量比较大的应用，例如：包含了时间范围非常长的窗口计算，或者状态比较大的场景。

#### RocksDBStateBackend

RocksDBStateBackend是Flink中内置的第三方状态管理器，和前面的状态管理器不同，RocksDBStateBackend需要单独引入相关的依赖包到工程中。

```maven
 <dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
  <version>1.9.2</version>
</dependency>
```

```scala
env.setStateBackend(new RocksDBStateBackend("hdfs://"))
```

RocksDBStateBackend采用异步的方式进行状态数据的Snapshot，任务中的状态数据首先被写入本地RockDB中，这样在RockDB仅会存储正在进行计算的热数据，而需要进行CheckPoint的时候，会把本地的数据直接复制到远端的FileSystem中。

与FsStateBackend相比，RocksDBStateBackend在性能上要比FsStateBackend高一些，主要是因为借助于RocksDB在本地存储了最新热数据，然后通过异步的方式再同步到文件系统中，但RocksDBStateBackend和MemoryStateBackend相比性能就会较弱一些。RocksDB克服了State受内存限制的缺点，同时又能够持久化到远端文件系统中，推荐在生产中使用。

#### 集群级配置StateBackend

全局配置需要需改集群中的配置文件，修改flink-conf.yaml

- 配置FsStateBackend

```
state.backend: filesystem
state.checkpoints.dir: hdfs://namenode-host:port/flink-checkpoints
```

- 配置MemoryStateBackend

```
state.backend: jobmanager
```

- 配置RocksDBStateBackend

```
  state.backend.rocksdb.checkpoint.transfer.thread.num: 1 同时操作RocksDB的线程数
  state.backend.rocksdb.localdir: 本地path   RocksDB存储状态数据的本地文件路径
```

## Window

在流处理中，我们往往需要面对的是连续不断、无休无止的无界流，不可能等到所有数据都到齐了才开始处理。所以聚合计算其实在实际应用中，我们往往更关心一段时间内数据的统计结果，比如在过去的 1 分钟内有多少用户点击了网页。在这种情况下，我们就可以定义一个窗口，收集最近一分钟内的所有用户点击数据，然后进行聚合统计，最终输出一个结果就可以了。

**说白了窗口就是将无界流通过窗口切割成一个个的有界流，窗口是左开右闭的**。

**Flink中的窗口分为两类：基于时间的窗口（Time-based Window）和基于数量的窗口（Count-based Window）**。

- 时间窗口（Time Window）：按照时间段去截取数据，这在实际应用中最常见。
- 计数窗口（Count Window）：由数据驱动，也就是说按照固定的个数，来截取一段数据集。

时间窗口中又包含了：**滚动时间窗口（Tumbling Window）、滑动时间窗口（Sliding Window）、会话窗口（Session Window）**。

计数窗口包含了：**滚动计数窗口和滑动计数窗口**。

时间窗口、计数窗口只是对窗口的一个大致划分。在具体应用时，还需要定义更加精细的规则，来控制数据应该划分到哪个窗口中去。不同的分配数据的方式，就可以由不同的功能应用。

根据分配数据的规则，窗口的具体实现可以分为 4 类：滚动窗口（Tumbling Window）、滑动窗口（Sliding Window）、会话窗口（Session Window），以及全局窗口（Global Window）。

### 滚动窗口（Tumbling Windows）

滚动窗口每个窗口的大小固定，且相邻两个窗口之间没有重叠。滚动窗口可以基于时间定义，也可以基于数据个数定义；需要的参数只有窗口大小，我们可以定义一个长度为1小时的滚动时间窗口，那么每个小时就会进行一次统计；或者定义一个长度为10的滚动计数窗口，就会每10个数进行一次统计。

基于时间的滚动窗口：

```java
DataStream<T> input = ...
// tumbling event-time windows
input
  .keyBy(...)
  .window(TumblingEventTimeWindows.of(Time.seconds(5)))
  .<window function> (...)

// tumbling processing-time windows
input
  .keyBy(...)
  .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
  .<window function> (...)
```

在上面的代码中，我们使用了`TumblingEventTimeWindows`和`TumblingProcessingTimeWindows`来创建基于Event Time或Processing Time的滚动时间窗口。窗口的长度可以用`org.apache.flink.streaming.api.windowing.time.Time`中的`seconds`、`minutes`、`hours`和`days`来设置。

基于计数的滚动窗口：

```java
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TumblingCountWindowExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Long> input = env.fromElements(1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L);

        input
            .keyBy(value -> 1)
            .countWindow(3)
            .reduce(new ReduceFunction<Long>() {
                @Override
                public Long reduce(Long value1, Long value2) throws Exception {
                    return value1 + value2;
                }
            })
            .print();

        env.execute();
    }
}
```

在上面的代码中，我们使用了`countWindow`方法来创建一个基于数量的滚动窗口，窗口大小为3个元素。当窗口中的元素数量达到3时，窗口就会触发计算。在这个例子中，我们使用了`reduce`函数来对窗口中的元素进行求和。

### 滑动窗口（Sliding Windows）

滑动窗口的大小固定，但窗口之间不是首尾相接，而有部分重合。同样，滑动窗口也可以基于时间和计算定义。

滑动窗口的参数有两个：**窗口大小和滑动步长。滑动步长是固定的**。

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnIpzib7JiaDHLjvtsZnfQWtEeuYhwFF04QvTRmK6FcXaBshE5c8QBYBg7SaMfzTPmqFMQY8lXWuNQ/640?wx_fmt=png)

基于时间的滑动窗口：

```java
DataStream<T> input = ...
// sliding event-time windows
input
  .keyBy(...)
  .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
  .<window function> (...)
```

基于计数的滑动窗口：

```java
DataStream<T> input = ...
input
  .keyBy(...)
  .countWindow(10, 5)
  .<window function> (...)
```

`countWindow`方法来创建一个基于计数的滑动窗口，窗口大小为10个元素，滑动步长为5个元素。当窗口中的元素数量达到10时，窗口就会触发计算。

### 会话窗口（Session Windows）

会话窗口是Flink中一种基于时间的窗口类型，每个窗口的大小不固定，且相邻两个窗口之间没有重叠。**“会话”终止的标志就是隔一段时间没有数据来**：

```java
import org.apache.flink.streaming.api.windowing.assigners.EventTimeSessionWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

DataStream<T> input = ...
input
  .keyBy(...)
  .window(EventTimeSessionWindows.withGap(Time.minutes(10)))
  .<window function> (...)
```

在上面的代码中，使用了`EventTimeSessionWindows`来创建基于Event Time的会话窗口。`withGap`方法用来设置会话窗口之间的间隔时间，当两个元素之间的时间差超过这个值时，它们就会被分配到不同的会话窗口中。

### 按键分区窗口和非按键分区窗口

在Flink中，数据流可以按键分区（keyed）或非按键分区（non-keyed）。按键分区是指将数据流根据特定的键值进行分区，使得相同键值的元素被分配到同一个分区中。这样可以保证相同键值的元素由同一个worker实例处理。只有按键分区的数据流才能使用键分区状态和计时器。

非按键分区是指数据流没有根据特定的键值进行分区。这种情况下，数据流中的元素可以被任意分配到不同的分区中。

在定义窗口操作之前，首先需要确定，到底是基于按键分区（Keyed）来开窗，还是直接在没有按键分区的DataStream上开窗。也就是在调用窗口算子之前是否有keyBy操作。

按键分区窗口：

```java
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

public class KeyedWindowExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Long> input = env.fromElements(1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L);

        input
            .keyBy(value -> 1)
            .window(TumblingEventTimeWindows.of(Time.seconds(5)))
            .reduce(new ReduceFunction<Long>() {
                @Override
                public Long reduce(Long value1, Long value2) throws Exception {
                    return value1 + value2;
                }
            })
            .print();

        env.execute();
    }
}
```

在上面的代码中，使用了`keyBy`方法来对数据流进行按键分区，然后使用`window`方法来创建一个基于Event Time的滚动时间窗口。在这个例子中，我们使用了`reduce`函数来对窗口中的元素进行求和。

非按键分区窗口：

```java
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.streaming.api.datastream.AllWindowedStream;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

public class NonKeyedWindowExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Long> input = env.fromElements(1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L);

        AllWindowedStream<Long, ?> windowedStream = input.windowAll(TumblingEventTimeWindows.of(Time.seconds(5)));

        windowedStream.reduce(new ReduceFunction<Long>() {
            @Override
            public Long reduce(Long value1, Long value2) throws Exception {
                return value1 + value2;
            }
        }).print();

        env.execute();
    }
}
```

在上面的代码中，使用了`windowAll`方法来对非按键分区的数据流进行窗口操作。`windowAll`方法接受一个`WindowAssigner`参数，用来指定窗口类型。然后使用了`reduce`函数来对窗口中的元素进行求和。

按键分区窗口（Keyed Windows）经过按键分区keyBy操作后，数据流会按照key被分为多条逻辑流（logical streams），这就是KeyedStream。基于KeyedStream进行窗口操作时，窗口计算会在多个并行子任务上同时执行。相同key的数据会被发送到同一个并行子任务，而窗口操作会基于每个key进行单独的处理。所以可以认为，每个key上都定义了一组窗口，各自独立地进行统计计算。

非按键分区（Non-Keyed Windows）如果没有进行keyBy，那么原始的DataStream就不会分成多条逻辑流。这时窗口逻辑只能在一个任务（task）上执行，就相当于并行度变成了1。所以在实际应用中一般不推荐使用这种方式

### 窗口函数（WindowFunction）

所谓的“窗口函数”（window functions），就是定义窗口如何进行计算的操作。

窗口函数根据处理的方式可以分为两类：增量聚合函数和全量聚合函数。

#### 增量聚合函数

增量聚合函数每来一条数据就立即进行计算，中间保持着聚合状态；但是不立即输出结果。等到窗口到了结束时间需要输出计算结果的时候，取出之前聚合的状态直接输出。

常见的增量聚合的函数有：reduce(reduceFunction)、aggregate(aggregateFunction)、sum()、min()、max()。

下面是一个使用增量聚合函数的Java代码示例：
```java
DataStream<Tuple2<String, Integer>> input = ...
input.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
        @Override
        public String getKey(Tuple2<String, Integer> value) throws Exception {
            return value.f0;
        }
    })
    .timeWindow(Time.seconds(5))
    .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
        @Override
        public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t0, Tuple2<String, Integer> t1) throws Exception {
            return new Tuple2<>(t0.f0, t0.f1 + t1.f1);
        }
    });
```

这段代码首先使用`keyBy`方法按照Tuple2中的第一个元素（f0）进行分组。然后，它定义了一个5秒的时间窗口，并使用`reduce`方法对每个窗口内的数据进行聚合操作。在这个例子中，聚合操作是将具有相同key（即f0相同）的元素的第二个元素（f1）相加。最终，这段代码将输出一个包含每个key在每个5秒窗口内f1值之和的数据流。

另外还有一个常用的函数是**聚合函数（AggregateFunction）**，ReduceFunction和AggregateFunction都是增量聚合函数，但它们之间有一些区别。AggregateFunction则更加灵活，ReduceFunction的输入类型、输出类型和中间状态类型必须相同，而AggregateFunction则允许这三种类型不同。

**例如，如果我们希望计算一组数据的平均值，应该怎样做聚合呢？这时我们需要计算两个状态量：数据的总和（sum），以及数据的个数（count），而最终输出结果是两者的商（sum/count）。如果用ReduceFunction，那么我们应该先把数据转换成二元组 (sum, count)的形式，然后进行归约聚合，最后再将元组的两个元素相除转换得到最后的平均值。本来应该只是一个任务，可我们却需要 map-reduce-map 三步操作，这显然不够高效。而使用AggregateFunction则可以更加简单地实现这个需求**。

下面是使用AggregateFunction计算平均值的代码示例：
```java
DataStream<Tuple2<String, Double>> input = ...
input
    .keyBy(new KeySelector<Tuple2<String, Double>, String>() {
        @Override
        public String getKey(Tuple2<String, Double> value) throws Exception {
            return value.f0;
        }
    })
    .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    .aggregate(new AggregateFunction<Tuple2<String, Double>, Tuple2<Double, Integer>, Double>() {
        @Override
        public Tuple2<Double, Integer> createAccumulator() {
            return new Tuple2<>(0.0, 0);
        }

        @Override
        public Tuple2<Double, Integer> add(Tuple2<String, Double> value, Tuple2<Double, Integer> accumulator) {
            return new Tuple2<>(accumulator.f0 + value.f1, accumulator.f1 + 1);
        }

        @Override
        public Double getResult(Tuple2<Double, Integer> accumulator) {
            return accumulator.f0 / accumulator.f1;
        }

        @Override
        public Tuple2<Double, Integer> merge(Tuple2<Double, Integer> a, Tuple2<Double, Integer> b) {
            return new Tuple2<>(a.f0 + b.f0, a.f1 + b.f1);
        }
    });
```

这段代码首先使用`keyBy`方法按照Tuple2中的第一个元素（f0）进行分组。然后，它定义了一个5秒的翻滚事件时间窗口，并使用`aggregate`方法对每个窗口内的数据进行聚合操作。在这个例子中，聚合操作是计算具有相同key（即f0相同）的元素的第二个元素（f1）的平均值。最终，这段代码将输出一个包含每个key在每个5秒窗口内f1值平均值的数据流。

#### 全量聚合函数

全量聚合函数（Full Window Functions）是指在整个窗口中的所有数据都准备好后才进行计算。Flink中的全窗口函数有两种：**WindowFunction和ProcessWindowFunction**。

与增量聚合函数不同，全窗口函数可以访问窗口中的所有数据，因此可以执行更复杂的计算。例如，可以计算窗口中数据的中位数，或者对窗口中的数据进行排序。

WindowFunction接收一个Iterable类型的输入，其中包含了窗口中所有的数据。ProcessWindowFunction则更加强大，它不仅可以访问窗口中的所有数据， 还可以获取到一个“上下文对象”（Context）。这个上下文对象非常强大，不仅能够获取窗口信息，还可以访问当前的时间和状态信息。这里的时间就包括了处理时间（processing time）和事件时间水位线（event time watermark）。这就使得 ProcessWindowFunction 更加灵活、功能更加丰富。WindowFunction作用可以被 ProcessWindowFunction 全覆盖。**一般在实际应用，用 ProcessWindowFunction比较多，直接使用 ProcessWindowFunction 就可以了**。

下面是使用WindowFunction计算窗口内数据总和的代码示例：

```java
public class SumWindowFunction extends WindowFunction<Tuple2<String, Integer>, Tuple2<String, Integer>, String, TimeWindow> {
    @Override
    public void apply(String key, TimeWindow window, Iterable<Tuple2<String, Integer>> input, Collector<Tuple2<String, Integer>> out) throws Exception {
        int sum = 0;
        for (Tuple2<String, Integer> value : input) {
            sum += value.f1;
        }
        out.collect(new Tuple2<>(key, sum));
    }
}

DataStream<Tuple2<String, Integer>> input = ...
input.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
        @Override
        public String getKey(Tuple2<String, Integer> value) throws Exception {
            return value.f0;
        }
    })
    .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    .apply(new SumWindowFunction());
```

下面是一个使用ProcessWindowFunction统计网站1天UV的代码示例。在这个例子中，我们使用了状态来存储每个窗口中访问过网站的用户ID，以便在窗口结束时计算UV。此外，我们还使用了定时器，在窗口结束时触发计算UV的操作。我们还使用了context对象来获取窗口的开始时间和结束时间，并将它们输出到结果中：

```java
public class UVProcessWindowFunction extends ProcessWindowFunction<Tuple2<String, String>, Tuple3<String, Long, Integer>, String, TimeWindow> {
    private ValueState<Set<String>> userIdState; // 状态，用来存储每个窗口中访问过网站的用户ID

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        // 初始化状态
        ValueStateDescriptor<Set<String>> stateDescriptor = new ValueStateDescriptor<>("userIdState", new SetTypeInfo<>(Types.STRING));
        userIdState = getRuntimeContext().getState(stateDescriptor);
    }

    @Override
    public void process(String key, Context context, Iterable<Tuple2<String, String>> input, Collector<Tuple3<String, Long, Integer>> out) throws Exception {
        Set<String> userIds = userIdState.value();
        if (userIds == null) {
            userIds = new HashSet<>();
        }
        for (Tuple2<String, String> value : input) {
            userIds.add(value.f0); // 将用户ID添加到状态中
        }
        userIdState.update(userIds);
        context.timerService().registerEventTimeTimer(context.window().getEnd()); // 注册定时器，在窗口结束时触发计算UV的操作
    }

    @Override
    public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple3<String, Long, Integer>> out) throws Exception {
        super.onTimer(timestamp, ctx, out);
        Set<String> userIds = userIdState.value();
        if (userIds != null) {
            long windowStart = ctx.window().getStart();
            out.collect(new Tuple3<>(ctx.getCurrentKey(), windowStart, userIds.size())); // 计算UV并输出结果，包括窗口的开始时间和结束时间
            userIdState.clear(); // 清空状态
        }
    }
}

DataStream<Tuple2<String, String>> input = ... // 输入数据流，其中第一个字段为用户ID，第二个字段为网站URL
input.keyBy(new KeySelector<Tuple2<String, String>, String>() {
        @Override
        public String getKey(Tuple2<String, String> value) throws Exception {
            return value.f1; // 按照网站URL分组
        }
    })
    .window(TumblingEventTimeWindows.of(Time.days(1))) // 设置窗口大小为1天
    .process(new UVProcessWindowFunction());
```

#### 增量聚合函数和全量聚合函数结合使用

全窗口函数需要先收集窗口中的数据，并在内部缓存起来，等到窗口要输出结果的时候再取出数据进行计算。所以运行效率较低，很少直接单独使用，往往会和增量聚合函数结合在一起，共同实现窗口的处理计算。

增量聚合的优点：高效，输出更加实时。增量聚合相当于把计算量“均摊”到了窗口收集数据的过程中，自然就会比全窗口聚合更加高效、输出更加实时。

全窗口的优点：提供更多的信息，可以认为是更加“通用”的窗口操作。
 它只负责收集数据、提供上下文相关信息，把所有的原材料都准备好，至于拿来做什么我们完全可以任意发挥。这就使得窗口计算更加灵活，功能更加强大。

在实际应用中，我们往往希望兼具这两者的优点，把它们结合在一起使用。Flink 的Window API 就给我们实现了这样的用法。

之前在调用 WindowedStream 的.reduce()和.aggregate()方法时，只是简单地直接传入了一个 ReduceFunction 或 AggregateFunction 进行增量聚合。除此之外，其实还可以传入第二个参数：一个全窗口函数，可以是 WindowFunction 或者ProcessWindowFunction。

```xml
// ReduceFunction 与 WindowFunction 结合
public <R> SingleOutputStreamOperator<R> reduce(ReduceFunction<T> reduceFunction, WindowFunction<T, R, K, W> function)

// ReduceFunction 与 ProcessWindowFunction 结合
public <R> SingleOutputStreamOperator<R> reduce(ReduceFunction<T> reduceFunction, ProcessWindowFunction<T, R, K, W> function)

// AggregateFunction 与 WindowFunction 结合
public <ACC, V, R> SingleOutputStreamOperator<R> aggregate(AggregateFunction<T, ACC, V> aggFunction, WindowFunction<V, R, K, W> windowFunction)

// AggregateFunction 与 ProcessWindowFunction 结合
public <ACC, V, R> SingleOutputStreamOperator<R> aggregate(AggregateFunction<T, ACC, V> aggFunction, ProcessWindowFunction<V, R, K, W> windowFunction)
```

这样调用的处理机制是：基于第一个参数（增量聚合函数）来处理窗口数据，每来一个数据就做一次聚合；等到窗口需要触发计算时，则调用第二个参数（全窗口函数）的处理逻辑输出结果。**需要注意的是，这里的全窗口函数就不再缓存所有数据了，而是直接将增量聚合函数的结果拿来当作了 Iterable 类型的输入。一般情况下，这时的可迭代集合中就只有一个元素了**。

下面我们举一个具体的实例来说明。在网站的各种统计指标中，一个很重要的统计指标就是热门的链接，想要得到热门的 url，前提是得到每个链接的“热门度”。一般情况下，可以用url 的浏览量（点击量）表示热门度。我们这里统计 10 秒钟的 url 浏览量，每 5 秒钟更新一次；另外为了更加清晰地展示，还应该把窗口的起始结束时间一起输出。我们可以定义滑动窗口，并结合增量聚合函数和全窗口函数来得到统计结果：

```java
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

public class UrlCountViewExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.getConfig().setAutoWatermarkInterval(100);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                //乱序流的watermark生成
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(0))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        }));
        stream.print("input");

        //统计每个url的访问量
        stream.keyBy(data -> data.url)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .aggregate(new UrlViewCountAgg(),new UrlViewCountResult())
                .print();


        env.execute();
    }

    //增量聚合，来一条数据 + 1
    public static class UrlViewCountAgg implements AggregateFunction<Event,Long,Long>{

        @Override
        public Long createAccumulator() {
            return 0L;
        }

        @Override
        public Long add(Event value, Long accumulator) {
            return accumulator + 1;
        }

        @Override
        public Long getResult(Long accumulator) {
            return accumulator;
        }

        @Override
        public Long merge(Long a, Long b) {
            return null;
        }
    }

    //包装窗口信息，输出UrlViewCount
    public static class UrlViewCountResult extends ProcessWindowFunction<Long,UrlViewCount,String, TimeWindow>{

        @Override
        public void process(String s, Context context, Iterable<Long> elements, Collector<UrlViewCount> out) throws Exception {
            Long start = context.window().getStart();
            Long end = context.window().getEnd();
            Long count = elements.iterator().next();
            out.collect(new UrlViewCount(s,count,start,end));
        }
    }
}
```

为了方便处理，单独定义了一个POJO类，来表示输出结果的数据类型

```kotlin
public class UrlViewCount {
    public String url;
    public Long count;
    public Long windowStart;
    public Long windowEnd;

    public UrlViewCount() {
    }

    public UrlViewCount(String url, Long count, Long windowStart, Long windowEnd) {
        this.url = url;
        this.count = count;
        this.windowStart = windowStart;
        this.windowEnd = windowEnd;
    }

    @Override
    public String toString() {
        return "UrlViewCount{" +
                "url='" + url + '\'' +
                ", count=" + count +
                ", windowStart=" + new Timestamp(windowStart) +
                ", windowEnd=" + new Timestamp(windowEnd) +
                '}';
    }
}
```

代码中用一个 AggregateFunction 来实现增量聚合，每来一个数据就计数加一，得到的结果交给 ProcessWindowFunction，结合窗口信息包装成我们想要的 UrlViewCount，最终输出统计结果。

窗口处理的主体还是增量聚合，而引入全窗口函数又可以获取到更多的信息包装输出，这样的结合兼具了两种窗口函数的优势，在保证处理性能和实时性的同时支持了更加丰富的应用场景。

### Window重叠优化

窗口重叠是指在使用滑动窗口时，多个窗口之间存在重叠部分。这意味着同一批数据可能会被多个窗口同时处理。

例如，假设我们有一个数据流，它包含了0到9的整数。我们定义了一个大小为5的滑动窗口，滑动距离为2。那么，我们将会得到以下三个窗口：

- 窗口1：包含0, 1, 2, 3, 4
- 窗口2：包含2, 3, 4, 5, 6
- 窗口3：包含4, 5, 6, 7, 8

在这个例子中，窗口1和窗口2之间存在重叠部分，即2, 3, 4。同样，窗口2和窗口3之间也存在重叠部分，即4, 5, 6。

`enableOptimizeWindowOverlap`方法是用来启用Flink的窗口重叠优化功能的。它可以减少计算重叠窗口时的计算量。

在我之前给出的代码示例中，我没有使用`enableOptimizeWindowOverlap`方法来启用窗口重叠优化功能。这意味着Flink不会尝试优化计算重叠窗口时的计算量。

如果你想使用窗口重叠优化功能，你可以在你的代码中添加以下行：

```Java
env.getConfig().enableOptimizeWindowOverlap();
```

这将启用窗口重叠优化功能，Flink将尝试优化计算重叠窗口时的计算量。

## 触发器（Trigger）

触发器主要是用来控制窗口什么时候触发计算。所谓的“触发计算”，本质上就是执行窗口函数，所以可以认为是计算得到结果并输出的过程。

基于 WindowedStream 调用.trigger()方法，就可以传入一个自定义的窗口触发器（Trigger）。

```css
stream.keyBy(...)
    .window(...)
    .trigger(new MyTrigger())
```

Trigger 是窗口算子的内部属性，每个窗口分配器（WindowAssigner）都会对应一个默认的触发器；对于 Flink 内置的窗口类型，它们的触发器都已经做了实现。例如，所有事件时间窗口，默认的触发器都是EventTimeTrigger；类似还有 ProcessingTimeTrigger 和 CountTrigger。所以一般情况下是不需要自定义触发器的，这块了解一下即可。

## 移除器（Evictor）

在 Apache Flink 中，移除器（Evictor）是用于在滚动窗口或会话窗口中控制数据保留和清理的组件。它可以根据特定的策略从窗口中删除一些数据，以确保窗口中保留的数据量不超过指定的限制。移除器通常与窗口分配器一起使用，窗口分配器负责确定数据属于哪个窗口，而移除器则负责清理窗口中的数据。

以下是一个使用 Flink 移除器的代码示例，演示如何在滚动窗口中使用基于计数的移除器。

```java
javaCopy codeimport org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.evictors.CountEvictor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.triggers.CountTrigger;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

public class FlinkEvictorExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 创建一个包含整数和时间戳的流
        DataStream<Tuple2<Integer, Long>> dataStream = env.fromElements(
                Tuple2.of(1, System.currentTimeMillis()),
                Tuple2.of(2, System.currentTimeMillis() + 1000),
                Tuple2.of(3, System.currentTimeMillis() + 2000),
                Tuple2.of(4, System.currentTimeMillis() + 3000),
                Tuple2.of(5, System.currentTimeMillis() + 4000),
                Tuple2.of(6, System.currentTimeMillis() + 5000)
        );

        // 在滚动窗口中使用基于计数的移除器，保留最近3个元素
        dataStream
                .keyBy(value -> value.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .trigger(CountTrigger.of(3))
                .evictor(CountEvictor.of(3))
                .aggregate(new MyAggregateFunction(), new MyProcessWindowFunction())
                .print();

        env.execute("Flink Evictor Example");
    }

    // 自定义聚合函数
    private static class MyAggregateFunction implements AggregateFunction<Tuple2<Integer, Long>, Integer, Integer> {

        @Override
        public Integer createAccumulator() {
            return 0;
        }

        @Override
        public Integer add(Tuple2<Integer, Long> value, Integer accumulator) {
            return accumulator + 1;
        }

        @Override
        public Integer getResult(Integer accumulator) {
            return accumulator;
        }

        @Override
        public Integer merge(Integer a, Integer b) {
            return a + b;
        }
    }

    // 自定义处理窗口函数
    private static class MyProcessWindowFunction extends ProcessWindowFunction<Integer, String, Integer, TimeWindow> {

        private transient ListState<Integer> countState;

        @Override
        public void open(Configuration parameters) throws Exception {
            super.open(parameters);
            ListStateDescriptor<Integer> descriptor = new ListStateDescriptor<>("countState", Integer.class);
            countState = getRuntimeContext().getListState(descriptor);
        }

        @Override
        public void process(Integer key, Context context, Iterable<Integer> elements, Collector<String> out) throws Exception {
            int count = elements.iterator().next();
            countState.add(count);

            long windowStart = context.window().getStart();
            long windowEnd = context.window().getEnd();
            String result = "Window: " + windowStart + " to " + windowEnd + ", Count: " + countState.get().iterator().next();
            out.collect(result);
        }
    }
}
```

在上述示例中，创建了一个包含整数和时间戳的数据流，并使用基于计数的移除器将滚动窗口的大小限制为最近的3个元素。在聚合函数中，我们简单地将元素的数量累加起来，并在处理窗口函数中收集结果。最后，我们打印窗口的开始时间、结束时间和元素数量。

## Flink Time时间语义

Flink定义了三类时间

- **事件时间（Event Time）**数据在数据源产生的时间，一般由事件中的时间戳描述，比如用户日志中的TimeStamp。
- **处理时间（Process Time）**数据进入Flink被处理的系统时间（Operator处理数据的系统时间）。
- **摄取时间（Ingestion Time）**数据进入Flink的时间，记录被Source节点观察到的系统时间。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLpBZRCkBIDh9K3y5YzxXux7GLYLpqhPy6DpDwb0geTjibkv5JPnDoEtXQ/640?wx_fmt=png)

Flink流式计算的时候需要显示定义时间语义，根据不同的时间语义来处理数据，比如指定的时间语义是事件时间，那么我们就要切换到事件时间的世界观中，窗口的起始与终止时间都是以事件时间为依据

在Flink中默认使用的是Process Time，如果要使用其他的时间语义，在执行环境中可以设置

```scala
//设置时间语义为Ingestion Time
env.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime)
//设置时间语义为Event Time 我们还需要指定一下数据中哪个字段是事件时间（下文会讲）
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

- 基于事件时间的Window操作

  ```scala
  import org.apache.flink.streaming.api.TimeCharacteristic
  import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, _}
  import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows
  import org.apache.flink.streaming.api.windowing.time.Time
  
  object EventTimeWindow {
    def main(args: Array[String]): Unit = {
      val env = StreamExecutionEnvironment.getExecutionEnvironment
      env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
      val stream = env.socketTextStream("node01", 8888).assignAscendingTimestamps(data => {
        val splits = data.split(" ")
        splits(0).toLong
      })
  
      stream
        .flatMap(x=>x.split(" ").tail)
        .map((_, 1))
        .keyBy(_._1)
  //      .timeWindow(Time.seconds(10))
        .window(TumblingEventTimeWindows.of(Time.seconds(10)))
        .reduce((v1: (String, Int), v2: (String, Int)) => {
          (v1._1, v1._2 + v2._2)
        })
        .print()
  
      env.execute()
    }
  }
  ```

------

### Watermark(水印)

**Watermark本质就是时间戳，说白了Watermark就是来处理迟到数据的**。

在使用Flink处理数据的时候，数据通常都是按照事件产生的时间（事件时间）的顺序进入到Flink，但是在遇到特殊情况下，比如遇到网络延迟或者使用Kafka（多分区） 很难保证数据都是按照事件时间的顺序进入Flink，很有可能是乱序进入。

如果使用的是事件时间这个语义，数据一旦是乱序进入，那么在使用Window处理数据的时候，就会出现延迟数据不会被计算的问题

- 举例： Window窗口长度10s，滚动窗口

  001 zs 2020-04-25 10:00:01

  001 zs 2020-04-25 10:00:02

  001 zs 2020-04-25 10:00:03

  001 zs 2020-04-25 10:00:11  窗口触发执行

  001 zs 2020-04-25 10:00:05  延迟数据，不会被上一个窗口所计算导致计算结果不正确

Watermark+Window可以很好的解决延迟数据的问题。

Flink窗口计算的过程中，如果数据全部到达就会到窗口中的数据做处理，如果过有延迟数据，那么窗口需要等待全部的数据到来之后，再触发窗口执行，需要等待多久？不可能无限期等待，我们用户可以自己来设置延迟时间，这样就可以尽可能保证延迟数据被处理。

根据用户指定的延迟时间生成水印（Watermak = 最大事件时间-指定延迟时间），当Watermak 大于等于窗口的停止时间，这个窗口就会被触发执行。

- 举例：Window窗口长度10s(01~10)，滚动窗口，指定延迟时间3s

  001 ls 2020-04-25 10:00:01 wm:2020-04-25 09:59:58

  001 ls 2020-04-25 10:00:02 wm:2020-04-25 09:59:59

  001 ls 2020-04-25 10:00:03 wm:2020-04-25 10:00:00

  001 ls 2020-04-25 10:00:09 wm:2020-04-25 10:00:06

  001 ls 2020-04-25 10:00:12 wm:2020-04-25 10:00:09

  001 ls 2020-04-25 10:00:08 wm:2020-04-25 10:00:05    延迟数据

  001 ls 2020-04-25 10:00:13 wm:2020-04-25 10:00:10    

**如果没有Watermark在倒数第三条数据来的时候，就会触发执行，那么倒数第二条的延迟数据就不会被计算，那么有了水印可以处理延迟3s内的数据**。

**注意：如果数据不会乱序进入Flink，没必要使用Watermark**

DataStream API提供了自定义水印生成器和内置水印生成器。

#### 生成水印策略

- 周期性水印（Periodic Watermark）根据事件或者处理时间周期性的触发水印生成器(Assigner)，默认100ms，每隔100毫秒自动向流里注入一个Watermark

  周期性水印API 1：

  ```scala
  	val env = StreamExecutionEnvironment.getExecutionEnvironment
      env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
      env.getConfig.setAutoWatermarkInterval(100)
      val stream = env.socketTextStream("node01", 8888).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[String](Time.seconds(3)) {
        override def extractTimestamp(element: String): Long = {
          element.split(" ")(0).toLong
        }
      })
  ```

  周期性水印API 2：

  ```scala
  import org.apache.flink.streaming.api.TimeCharacteristic
  import org.apache.flink.streaming.api.functions.AssignerWithPeriodicWatermarks
  import org.apache.flink.streaming.api.scala.function.ProcessWindowFunction
  import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, _}
  import org.apache.flink.streaming.api.watermark.Watermark
  import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows
  import org.apache.flink.streaming.api.windowing.time.Time
  import org.apache.flink.streaming.api.windowing.windows.TimeWindow
  import org.apache.flink.util.Collector
  
  object EventTimeDelayWindow {
  
    class MyTimestampAndWatermarks(delayTime:Long) extends AssignerWithPeriodicWatermarks[String] {
  
      var maxCurrentWatermark: Long = _
   
      //水印=最大事件时间-延迟时间   后被调用    水印是递增，小于上一个水印不会被发射出去
      override def getCurrentWatermark: Watermark = {
        //产生水印
        new Watermark(maxCurrentWatermark - delayTime)
      }
  
      //获取当前的时间戳  先被调用
      override def extractTimestamp(element: String, previousElementTimestamp: Long): Long = {
        val currentTimeStamp = element.split(" ")(0).toLong
        maxCurrentWatermark = math.max(currentTimeStamp,maxCurrentWatermark)
        currentTimeStamp
      }
    }
  
    def main(args: Array[String]): Unit = {
      val env = StreamExecutionEnvironment.getExecutionEnvironment
      env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
      env.getConfig.setAutoWatermarkInterval(100)
      val stream = env.socketTextStream("node01", 8888).assignTimestampsAndWatermarks(new MyTimestampAndWatermarks(3000L))
  
      stream
        .flatMap(x => x.split(" ").tail)
        .map((_, 1))
        .keyBy(_._1)
        //      .timeWindow(Time.seconds(10))
        .window(TumblingEventTimeWindows.of(Time.seconds(10)))
        .process(new ProcessWindowFunction[(String, Int), (String, Int), String, TimeWindow] {
          override def process(key: String, context: Context, elements: Iterable[(String, Int)], out: Collector[(String, Int)]): Unit = {
            val start = context.window.getStart
            val end = context.window.getEnd
            var count = 0
            for (elem <- elements) {
              count += elem._2
            }
            println("start:" + start + " end:" + end + " word:" + key + " count:" + count)
          }
        })
        .print()
  
      env.execute()
    }
  }
  ```

- 间歇性水印生成器

  间歇性水印（Punctuated Watermark）在观察到事件后，会依据用户指定的条件来决定是否发射水印。

  比如，在车流量的数据中，001卡口通信经常异常，传回到服务器的数据会有延迟问题，其他的卡口都是正常的，那么这个卡口的数据需要打上水印。

  ```scala
  import org.apache.flink.streaming.api.TimeCharacteristic
  import org.apache.flink.streaming.api.functions.AssignerWithPunctuatedWatermarks
  import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, _}
  import org.apache.flink.streaming.api.watermark.Watermark
  import org.apache.flink.streaming.api.windowing.time.Time
  
  object PunctuatedWatermarkTest {
    def main(args: Array[String]): Unit = {
      val env = StreamExecutionEnvironment.getExecutionEnvironment
      env.setParallelism(1)
      env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
      //卡口号、时间戳
      env.socketTextStream("node01", 8888)
        .map(data => {
          val splits = data.split(" ")
          (splits(0), splits(1).toLong)
        })
        .assignTimestampsAndWatermarks(new myWatermark(3000))
        .keyBy(_._1)
        .timeWindow(Time.seconds(5))
        .reduce((v1: (String, Long), v2: (String, Long)) => {
          (v1._1 + "," + v2._1, v1._2 + v2._2)
        }).print()
  
      env.execute()
    }
  
    class myWatermark(delay: Long) extends AssignerWithPunctuatedWatermarks[(String, Long)] {
      var maxTimeStamp:Long = _
  
      override def checkAndGetNextWatermark(elem: (String, Long), extractedTimestamp: Long): Watermark = {
        maxTimeStamp = extractedTimestamp.max(maxTimeStamp)
        if ("001".equals(elem._1)) {
          new Watermark(maxTimeStamp - delay)
        } else {
          new Watermark(maxTimeStamp)
        }
      }
  
      override def extractTimestamp(element: (String, Long), previousElementTimestamp: Long): Long = {
        element._2
      }
    }
  }
  ```

### 允许延迟（Allowed Lateness）

#### 将迟到的数据放入侧输出流

Flink 还提供了另外一种方式处理迟到数据。我们可以将未收入窗口的迟到数据，放入“侧输出流”（side output）进行另外的处理。所谓的侧输出流，相当于是数据流的一个“分支”，这个流中单独放置那些错过了、本该被丢弃的数据。

基于 WindowedStream 调用.sideOutputLateData() 方法，就可以实现这个功能。方法需要传入一个“输出标签”（OutputTag），用来标记分支的迟到数据流。因为保存的就是流中的原始数据，所以 OutputTag 的类型与流中数据类型相同。

sideOutputLateData() 方法，传入一个输出标签，用来标记分治的迟到数据流

```dart
DataStream<Event> stream = env.addSource(...);
OutputTag<Event> outputTag = new OutputTag<Event>("late") {};
stream.keyBy(...)
    .window(TumblingEventTimeWindows.of(Time.hours(1)))
    .sideOutputLateData(outputTag)
```

将迟到数据放入侧输出流之后，还应该可以将它提取出来。基于窗口处理完成之后的DataStream，调用.getSideOutput()方法，传入对应的输出标签，就可以获取到迟到数据所在的流了。

```dart
SingleOutputStreamOperator<AggResult> winAggStream = stream.keyBy(...)
    .window(TumblingEventTimeWindows.of(Time.hours(1)))
    .sideOutputLateData(outputTag)
    .aggregate(new MyAggregateFunction())
DataStream<Event> lateStream = winAggStream.getSideOutput(outputTag);
```

这里注意，getSideOutput()是 SingleOutputStreamOperator 的方法，获取到的侧输出流数据类型应该和 OutputTag 指定的类型一致，与窗口聚合之后流中的数据类型可以不同。

## Flink关联维表实战

在Flink实际开发过程中，可能会遇到source 进来的数据，需要连接数据库里面的字段，再做后面的处理，比如，想要通过id获取对应的地区名字，这时候需要通过id查询地区维度表，获取具体的地区名。

对于不同的应用场景，关联维度表的方式不同

- 场景1：维度表信息基本不发生改变，或者发生改变的频率很低。

  实现方案：采用Flink提供的CachedFile。

  Flink提供了一个分布式缓存（CachedFile），类似于hadoop，可以使用户在并行函数中很方便的读取本地文件，并把它放在TaskManager节点中，防止task重复拉取。 此缓存的工作机制如下：程序注册一个文件或者目录(本地或者远程文件系统，例如hdfs或者s3)，通过ExecutionEnvironment注册缓存文件并为它起一个名称。 当程序执行，Flink自动将文件或者目录复制到所有TaskManager节点的本地文件系统，**仅会执行一次**。用户可以通过这个指定的名称查找文件或者目录，然后从TaskManager节点的本地文件系统访问它。

  ```scala
  val env = StreamExecutionEnvironment.getExecutionEnvironment
  env.registerCachedFile("/root/id2city","id2city")
  
  val socketStream = env.socketTextStream("node01",8888)
  val stream = socketStream.map(_.toInt)
  stream.map(new RichMapFunction[Int,String] {
  
      private val id2CityMap = new mutable.HashMap[Int,String]()
      override def open(parameters: Configuration): Unit = {
          val file = getRuntimeContext().getDistributedCache().getFile("id2city")
          val str = FileUtils.readFileUtf8(file)
          val strings = str.split("\r\n")
          for(str <- strings){
              val splits = str.split(" ")
              val id = splits(0).toInt
              val city = splits(1)
              id2CityMap.put(id,city)
          }
      }
      override def map(value: Int): String = {
          id2CityMap.getOrElse(value,"not found city")
      }
  }).print()
  env.execute()
  ```

  在集群中查看对应TaskManager的log日志，发现注册的file会被拉取到各个TaskManager的工作目录区。

- 场景2：对于维度表更新频率比较高并且对于查询维度表的实时性要求比较高

  实现方案：使用定时器，定时加载外部配置文件或者数据库

  ```scala
      val env = StreamExecutionEnvironment.getExecutionEnvironment
      env.setParallelism(1)
      val stream = env.socketTextStream("node01",8888)
  
      stream.map(new RichMapFunction[String,String] {
  
        private val map = new mutable.HashMap[String,String]()
  
        override def open(parameters: Configuration): Unit = {
          println("init data ...")
          query()
          val timer = new Timer(true)
          timer.schedule(new TimerTask {
            override def run(): Unit = {
              query()
            }
            //1s后，每隔2s执行一次
          },1000,2000)
        }
  
        def query()={
          val source = Source.fromFile("D:\\code\\StudyFlink\\data\\id2city","UTF-8")
          val iterator = source.getLines()
          for (elem <- iterator) {
            val vs = elem.split(" ")
            map.put(vs(0),vs(1))
          }
        }
  
        override def map(key: String): String = {
          map.getOrElse(key,"not found city")
        }
      }).print()
  
      env.execute()
  
  ```

- 场景3：对于维度表更新频率高并且对于查询维度表的实时性要求高

  实现方案：将更改的信息同步值Kafka配置Topic中，然后将kafka的配置流信息变成广播流，广播到业务流的各个线程中。


```scala
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //设置连接kafka的配置信息
    val props = new Properties()
    //注意   sparkstreaming + kafka（0.10之前版本） receiver模式  zookeeper url（元数据）
    props.setProperty("bootstrap.servers","node01:9092,node02:9092,node03:9092")
    props.setProperty("group.id","flink-kafka-001")
    props.setProperty("key.deserializer",classOf[StringSerializer].getName)
    props.setProperty("value.deserializer",classOf[StringSerializer].getName)
    val consumer = new FlinkKafkaConsumer[String]("configure",new SimpleStringSchema(),props)
    //从topic最开始的数据读取
//    consumer.setStartFromEarliest()
    //从最新的数据开始读取
    consumer.setStartFromLatest()

    //动态配置信息流
    val configureStream = env.addSource(consumer)
    //业务流
    val busStream = env.socketTextStream("node01",8888)

    val descriptor = new MapStateDescriptor[String,  String]("dynamicConfig",
      BasicTypeInfo.STRING_TYPE_INFO,
      BasicTypeInfo.STRING_TYPE_INFO)
    //设置广播流的数据描述信息
    val broadcastStream = configureStream.broadcast(descriptor)

    //connect关联业务流与配置信息流，broadcastStream流中的数据会广播到下游的各个线程中
    busStream.connect(broadcastStream)
        .process(new BroadcastProcessFunction[String,String,String] {
          override def processElement(line: String, ctx: BroadcastProcessFunction[String, String, String]#ReadOnlyContext, out: Collector[String]): Unit = {
            val broadcast = ctx.getBroadcastState(descriptor)
            val city = broadcast.get(line)
            if(city == null){
              out.collect("not found city")
            }else{
              out.collect(city)
            }
          }

          //kafka中配置流信息，写入到广播流中
          override def processBroadcastElement(line: String, ctx: BroadcastProcessFunction[String, String, String]#Context, out: Collector[String]): Unit = {
            val broadcast = ctx.getBroadcastState(descriptor)
            //kafka中的数据
            val elems = line.split(" ")
            broadcast.put(elems(0),elems(1))
          }
        }).print()
    env.execute()
```

## Table API & Flink SQL

在Spark中有DataFrame这样的关系型编程接口，因其强大且灵活的表达能力，能够让用户通过非常丰富的接口对数据进行处理，有效降低了用户的使用成本。Flink也提供了关系型编程接口Table API以及基于Table API的SQL API，让用户能够通过使用结构化编程接口高效地构建Flink应用。同时Table API以及SQL能够统一处理批量和实时计算业务，无须切换修改任何应用代码就能够基于同一套API编写流式应用和批量应用，从而达到真正意义的批流统一

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLpJCicXKXH7TiaetzpicQGVxxmjkCalOFRRDkabzHia16WjeDZOicMYrlnmcA/640?wx_fmt=png)

在 Flink 1.8 架构里，如果用户需要同时流计算、批处理的场景下，用户需要维护两套业务代码，开发人员也要维护两套技术栈，非常不方便。 Flink 社区很早就设想过将批数据看作一个有界流数据，将批处理看作流计算的一个特例，从而实现流批统一，阿里巴巴的 Blink 团队在这方面做了大量的工作，已经实现了 Table API & SQL 层的流批统一。阿里巴巴已经将 Blink 开源回馈给 Flink 社区。

### 开发环境构建

在 Flink 1.9 中，Table 模块迎来了核心架构的升级，引入了阿里巴巴Blink团队贡献的诸多功能，取名叫： Blink Planner。在使用Table API和SQL开发Flink应用之前，通过添加Maven的依赖配置到项目中，在本地工程中引入相应的依赖库，库中包含了Table API和SQL接口。

```xml
	<dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-planner_2.11</artifactId>
        <version>1.9.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-api-scala-bridge_2.11</artifactId>
        <version>1.9.1</version>
    </dependency>
```

### Table Environment

和DataStream API一样，Table API和SQL中具有相同的基本编程模型。首先需要构建对应的TableEnviroment创建关系型编程环境，才能够在程序中使用Table API和SQL来编写应用程序，另外Table API和SQL接口可以在应用中同时使用，Flink SQL基于Apache Calcite框架实现了SQL标准协议，是构建在Table API之上的更高级接口。

首先需要在环境中创建TableEnvironment对象，TableEnvironment中提供了注册内部表、执行Flink SQL语句、注册自定义函数等功能。根据应用类型的不同，TableEnvironment创建方式也有所不同，但是都是通过调用create()方法创建

流计算环境下创建TableEnviroment：

```scala
//创建流式计算的上下文环境
val env = StreamExecutionEnvironment.getExecutionEnvironment
//创建Table API的上下文环境
val tableEvn =StreamTableEnvironment.create(env)
```

### Table API

Table API 顾名思义，就是基于“表”（Table）的一套 API，专门为处理表而设计的，它提供了关系型编程模型，可以用来处理结构化数据，支持表和视图的概念。在此基础上，Flink 还基于 Apache Calcite 实现了对 SQL 的支持。这样一来，我们就可以在 Flink 程序中直接写 SQL 来实现处理需求了，非常实用。

下面是一个简单的例子，它使用Java编写了一个Flink程序，该程序使用Table API从CSV文件中读取数据，然后执行简单的查询并将结果写入到另一个CSV文件中。

首先我们需要导入maven依赖：

```xml
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
</dependency>
```

代码示例如下：

```Java
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.BatchTableEnvironment;

public class TableAPIExample {
    public static void main(String[] args) throws Exception {
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment tableEnv = BatchTableEnvironment.create(env);

        DataSet<Tuple2<String, Integer>> data = env.readCsvFile("input.csv")
                .includeFields("11")
                .types(String.class, Integer.class);

        Table table = tableEnv.fromDataSet(data, "name, age");
        tableEnv.createTemporaryView("people", table);

        Table result = tableEnv.sqlQuery("SELECT name, age FROM people WHERE age > 30");

        DataSet<Tuple2<String, Integer>> output = tableEnv.toDataSet(result, Tuple2.class);
        output.writeAsCsv("output.csv");

        env.execute();
    }
}
```

在这个例子中，使用`readCsvFile`方法从CSV文件中读取数据，并使用`includeFields`和`types`方法指定要包含的字段和字段类型。接下来，使用`fromDataSet`方法将数据集转换为表，并使用`createTemporaryView`方法创建一个临时视图。然后，使用`sqlQuery`方法执行SQL查询，并使用`toDataSet`方法将结果转换为数据集。最后，使用`writeAsCsv`方法将结果写入到CSV文件中，并使用`execute`方法启动执行。

除了上面这种写法外，我们还有下面2种写法：

```Java
//这里每个方法的参数都是一个“表达式”（Expression），用方法调用的形式直观地说明
//“$”符号用来指定表中的一个字段。代码和直接执行SQL是等效的。
Table maryClickTable = eventTable.where($("user").isEqual("Alice")).select($("url"),$("user"))
  
//这其实是一种简略的写法，我们将 Table 对象名 eventTable 直接以字符串拼接的形式添加到 SQL 语句中，在解析时会自动注册一个同名的虚拟表到环境中，这样就省略了创建虚拟视图的步骤。
Table clickTable = tableEnvironment.sqlQuery("select url, user from " +eventTable);
```

#### Virtual Tables（虚拟表）

在环境中注册之后，我们就可以在 SQL 中直接使用这张表进行查询转换了。

```Java
Table newTable = tableEnv.sqlQuery("SELECT ... FROM MyTable... ");
```

得到的 newTable 是一个中间转换结果，如果之后又希望直接使用这个表执行 SQL，又该怎么做呢？由于 newTable 是一个 Table 对象，并没有在表环境中注册；所以我们还需要将这个中间结果表注册到环境中，才能在 SQL 中使用：

```Java
tableEnv.createTemporaryView("NewTable", newTable);
```

这里的注册其实是创建了一个“虚拟表”（Virtual Table）。这个概念与 SQL 语法中的视图（View）非常类似，所以调用的方法也叫作创建“虚拟视图” （createTemporaryView）。

#### 表流互转

```Java
// 将表转换成数据流，并打印
tableEnv.toDataStream(aliceVisitTable).print();
// 将数据流转换成表。
// 另外，我们还可以在 fromDataStream()方法中增加参数，用来指定提取哪些属性作为表中的字段名，并可以任意指定位置：
Table eventTable2 = tableEnv.fromDataStream(eventStream, $("timestamp").as("ts"),$("url"));
```

#### 动态表和持续查询

在Flink中，动态表（Dynamic Tables）是一种特殊的表，它可以随时间变化。它们通常用于表示无限流数据，例如事件流或服务器日志。与静态表不同，动态表可以在运行时插入、更新和删除行。

动态表可以像静态的批处理表一样进行查询操作。由于数据在不断变化，因此基于它定义的 SQL 查询也不可能执行一次就得到最终结果。这样一来，我们对动态表的查询也就永远不会停止，一直在随着新数据的到来而继续执行。这样的查询就被称作持续查询（Continuous Query）。

下面是一个简单的例子，它使用Java编写了一个Flink程序，该程序使用Table API从Kafka主题中读取数据，然后执行持续查询并将结果写入到另一个Kafka主题中。

```Java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

public class DynamicTableExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        EnvironmentSettings settings = 		   EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build();
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env, settings);

        tableEnv.executeSql("CREATE TABLE input (" +
                "  name STRING," +
                "  age INT" +
                ") WITH (" +
                "  'connector' = 'kafka'," +
                "  'topic' = 'input-topic'," +
                "  'properties.bootstrap.servers' = 'localhost:9092'," +
                "  'format' = 'json'" +
                ")");

        tableEnv.executeSql("CREATE TABLE output (" +
                "  name STRING," +
                "  age INT" +
                ") WITH (" +
                "  'connector' = 'kafka'," +
                "  'topic' = 'output-topic'," +
                "  'properties.bootstrap.servers' = 'localhost:9092'," +
                "  'format' = 'json'" +
                ")");

        Table result = tableEnv.sqlQuery("SELECT name, age FROM input WHERE age > 30");
        tableEnv.toAppendStream(result, Row.class).print();

        result.executeInsert("output");

        env.execute();
    }
}
```

在这个例子中，首先创建了一个`StreamExecutionEnvironment`来设置执行环境，并使用`StreamTableEnvironment.create`方法创建了一个`StreamTableEnvironment`。然后，使用`executeSql`方法创建了两个Kafka表：一个用于读取输入数据，另一个用于写入输出数据。接下来，使用`sqlQuery`方法执行持续查询，并使用`toAppendStream`方法将结果转换为数据流。最后，使用`executeInsert`方法将结果写入到输出表中，并使用`execute`方法启动执行。

#### 连接到外部系统

在 Table API编写的 Flink 程序中，可以在创建表的时候用 WITH 子句指定连接器（connector），这样就可以连接到外部系统进行数据交互了。

其中最简单的当然就是连接到控制台打印输出：

```Java
CREATE TABLE ResultTable (
  user STRING,
  cnt BIGINT
WITH (
  'connector' = 'print'
);
```

##### Kafka

需要导入maven依赖：

```XML
<dependency>
   <groupId>org.apache.flink</groupId>
   <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
   <version>${flink.version}</version>
</dependency>
```

创建一个连接到 Kafka 表，需要在 CREATE TABLE 的 DDL 中在 WITH 子句里指定连接器为 Kafka，并定义必要的配置参数。

```sql
CREATE TABLE KafkaTable (
 `user` STRING,
 `url` STRING,
 `ts` TIMESTAMP(3) METADATA FROM 'timestamp'
) WITH (
 'connector' = 'kafka',
 'topic' = 'events',
 'properties.bootstrap.servers' = 'localhost:9092',
 'properties.group.id' = 'testGroup',
 'scan.startup.mode' = 'earliest-offset',
 'format' = 'csv'
)
```

##### MySQL

```XML
<dependency>
   <groupId>org.apache.flink</groupId>
   <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
   <version>${flink.version}</version>
</dependency>
```

创建 JDBC 表的方法与前面Kafka 大同小异：

```sql
-- 创建一张连接到 MySQL 的 表
CREATE TABLE MyTable (
 id BIGINT,
 name STRING,
 age INT,
 status BOOLEAN,
 PRIMARY KEY (id) NOT ENFORCED
) WITH (
 'connector' = 'jdbc',
 'url' = 'jdbc:mysql://localhost:3306/mydatabase',
 'table-name' = 'users'
);
-- 将另一张表 T 的数据写入到 MyTable 表中
INSERT INTO MyTable
SELECT id, name, age, status FROM T;
```

### Table API实战

在Flink中创建一张表有两种方法：

- 从一个文件中导入表结构（Structure）（常用于批计算）（静态）
- 从DataStream或者DataSet转换成Table  （动态）

#### 1.创建Table

Table API中已经提供了TableSource从外部系统获取数据，例如常见的数据库、文件系统和Kafka消息队列等外部系统。

1. 从文件中创建Table（静态表）

   Flink允许用户从本地或者分布式文件系统中读取和写入数据，在Table API中可以通过CsvTableSource类来创建，只需指定相应的参数即可。但是文件格式必须是CSV格式的。其他文件格式也支持（在Flink还有Connector的来支持其他格式或者自定义TableSource）

   ```scala
       //创建流式计算的上下文环境
       val env = StreamExecutionEnvironment.getExecutionEnvironment
       //创建Table API的上下文环境
       val tableEvn = StreamTableEnvironment.create(env)
   
   
       val source = new CsvTableSource("D:\\code\\StudyFlink\\data\\tableexamples"
         , Array[String]("id", "name", "score")
         , Array(Types.INT, Types.STRING, Types.DOUBLE)
       )
       //将source注册成一张表  别名：exampleTab
       tableEvn.registerTableSource("exampleTab",source)
       tableEvn.scan("exampleTab").printSchema()
   ```

   代码最后不需要env.execute()，这并不是一个流式计算任务

2. 从DataStream中创建Table（动态表）

   前面已经知道Table API是构建在DataStream API和DataSet API之上的一层更高级的抽象，因此用户可以灵活地使用Table API将Table转换成DataStream或DataSet数据集，也可以将DataSteam或DataSet数据集转换成Table，这和Spark中的DataFrame和RDD的关系类似

#### 2.修改Table中字段名

​	Flink支持把自定义POJOs类的所有case类的属性名字变成字段名，也可以通过基于字段偏移位置和字段名称两种方式重新修改：

```scala
    //导入table库中的隐式转换
    import org.apache.flink.table.api.scala._ 
    // 基于位置重新指定字段名称为"field1", "field2", "field3"
    val table = tStreamEnv.fromDataStream(stream, 'field1, 'field2, 'field3)
    // 将DataStream转换成Table,并且将字段名称重新成别名
    val table: Table = tStreamEnv.fromDataStream(stream, 'rowtime as 'newTime, 'id as 'newId,'variable as 'newVariable)
```

**注意：要导入隐式转换。如果使用as 修改字段，必须修改表中所有的字段。**

#### 3.查询和过滤

在Table对象上使用select操作符查询需要获取的指定字段，也可以使用filter或where方法过滤字段和检索条件，将需要的数据检索出来。	

```scala
object TableAPITest {

  def main(args: Array[String]): Unit = {
    val streamEnv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    streamEnv.setParallelism(1)
    //初始化Table API的上下文环境
    val tableEvn =StreamTableEnvironment.create(streamEnv)
    //导入隐式转换，建议写在这里，可以防止IDEA代码提示出错的问题
    import org.apache.flink.streaming.api.scala._
    import org.apache.flink.table.api.scala._
    val data = streamEnv.socketTextStream("hadoop101",8888)
          .map(line=>{
            var arr =line.split(",")
            new StationLog(arr(0).trim,arr(1).trim,arr(2).trim,arr(3).trim,arr(4).trim.toLong,arr(5).trim.toLong)
          })

    val table: Table = tableEvn.fromDataStream(data)
    //查询
    tableEvn.toAppendStream[Row](
      table.select('sid,'callType as 'type,'callTime,'callOut))
      .print()
    //过滤查询
    tableEvn.toAppendStream[Row](
      table.filter('callType==="success") //filter
        .where('callType==="success"))    //where
      .print()
    tableEvn.execute("sql")
  }
```

其中toAppendStream函数是吧Table对象转换成DataStream对象。

#### 4.分组聚合

​	举例：我们统计每个基站的日志数量。

```scala
val table: Table = tableEvn.fromDataStream(data)
    tableEvn.toRetractStream[Row](
      table.groupBy('sid).select('sid, 'sid.count as 'logCount))
      .filter(_._1==true) //返回的如果是true才是Insert的数据
      .print()
```

在代码中可以看出，使用toAppendStream和toRetractStream方法将Table转换为DataStream[T]数据集，T可以是Flink自定义的数据格式类型Row，也可以是用户指定的数据格式类型。在使用toRetractStream方法时，返回的数据类型结果为DataStream[(Boolean,T)]，Boolean类型代表数据更新类型，True对应INSERT操作更新的数据，False对应DELETE操作更新的数据。

#### 5.UDF自定义的函数

用户可以在Table API中自定义函数类，常见的抽象类和接口是：

- ScalarFunction 
- TableFunction
- AggregateFunction
- TableAggregateFunction

案例：使用Table完成基于流的WordCount

```scala
object TableAPITest2 {

  def main(args: Array[String]): Unit = {
    val streamEnv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
        streamEnv.setParallelism(1)
    //初始化Table API的上下文环境
    val tableEvn =StreamTableEnvironment.create(streamEnv)
    //导入隐式转换，建议写在这里，可以防止IDEA代码提示出错的问题
    import org.apache.flink.streaming.api.scala._
    import org.apache.flink.table.api.scala._

    val stream: DataStream[String] = streamEnv.socketTextStream("hadoop101",8888)
    val table: Table = tableEvn.fromDataStream(stream,'words)
    var my_func =new MyFlatMapFunction()//自定义UDF
    val result: Table = table.flatMap(my_func('words)).as('word, 'count)
      .groupBy('word) //分组
      .select('word, 'count.sum as 'c) //聚合
    tableEvn.toRetractStream[Row](result)
      .filter(_._1==true)
      .print()

    tableEvn.execute("table_api")

  }
  //自定义UDF
  class MyFlatMapFunction extends TableFunction[Row]{
    //定义类型
    override def getResultType: TypeInformation[Row] = {
      Types.ROW(Types.STRING, Types.INT)
    }
    //函数主体
    def eval(str:String):Unit ={
      str.trim.split(" ")
        .foreach({word=>{
          var row =new Row(2)
          row.setField(0,word)
          row.setField(1,1)
          collect(row)
        }})
    }
  }
}
```

#### 6.Window

​	Flink支持ProcessTime、EventTime和IngestionTime三种时间概念，针对每种时间概念，Flink Table API中使用Schema中单独的字段来表示时间属性，当时间字段被指定后，就可以在基于时间的操作算子中使用相应的时间属性。

在Table API中通过使用.rowtime来定义EventTime字段，在ProcessTime时间字段名后使用.proctime后缀来指定ProcessTime时间属性.

案例：统计最近5秒钟，每个基站的呼叫数量

```scala
object TableAPITest {

  def main(args: Array[String]): Unit = {
    val streamEnv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    //指定EventTime为时间语义
    streamEnv.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
    streamEnv.setParallelism(1)
    //初始化Table API的上下文环境
    val tableEvn =StreamTableEnvironment.create(streamEnv)
    //导入隐式转换，建议写在这里，可以防止IDEA代码提示出错的问题
    import org.apache.flink.streaming.api.scala._
    import org.apache.flink.table.api.scala._

    val data = streamEnv.socketTextStream("hadoop101",8888)
          .map(line=>{
            var arr =line.split(",")
            new StationLog(arr(0).trim,arr(1).trim,arr(2).trim,arr(3).trim,arr(4).trim.toLong,arr(5).trim.toLong)
          })
      .assignTimestampsAndWatermarks( //引入Watermark
        new BoundedOutOfOrdernessTimestampExtractor[StationLog](Time.seconds(2)){//延迟2秒
          override def extractTimestamp(element: StationLog) = {
            element.callTime
          }
        })

    //设置时间属性
    val table: Table = tableEvn.fromDataStream(data,'sid,'callOut,'callIn,'callType,'callTime.rowtime)
    //滚动Window ,第一种写法
    val result: Table = table.window(Tumble over 5.second on 'callTime as 'window)
    //第二种写法
    val result: Table = table.window(Tumble.over("5.second").on("callTime").as("window"))
      .groupBy('window, 'sid)
      .select('sid, 'window.start, 'window.end, 'window.rowtime, 'sid.count)
    //打印结果
    tableEvn.toRetractStream[Row](result)
      .filter(_._1==true)
      .print()
  

    tableEvn.execute("sql")
  }
}
```

上面的案例是滚动窗口，如果是滑动窗口也是一样，代码如下：

```scala
//滑动窗口，窗口大小为：10秒，滑动步长为5秒 :第一种写法
table.window(Slide over 10.second every 5.second on 'callTime as 'window)
//滑动窗口第二种写法 table.window(Slide.over("10.second").every("5.second").on("callTime").as("window"))
```

### Flink SQL

**企业中Flink SQL比Table API用的多**。


Flink SQL 是 Apache Flink 提供的一种使用 SQL 查询和处理数据的方式。它允许用户通过 SQL 语句对数据流或批处理数据进行查询、转换和分析，无需编写复杂的代码。Flink SQL 提供了一种更直观、易于理解和使用的方式来处理数据，同时也可以与 Flink 的其他功能无缝集成。

Flink SQL 支持 ANSI SQL 标准，并提供了许多扩展和优化来适应流式处理和批处理场景。它能够处理无界数据流，具备事件时间和处理时间的语义，支持窗口、聚合、连接等常见的数据操作，还提供了丰富的内置函数和扩展插件机制。

下面是一个简单的 Flink SQL 代码示例，展示了如何使用 Flink SQL 对流式数据进行查询和转换。

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction.SourceContext;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;

public class FlinkSqlExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);  // 设置并行度为1，方便观察输出结果

        // 创建 Kafka 数据源
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "flink-consumer");

        FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>("input-topic", new SimpleStringSchema(), properties);
        DataStream<String> sourceStream = env.addSource(kafkaConsumer);

        // 注册数据源表
        env.createTemporaryView("source_table", sourceStream, "message");

        // 执行 SQL 查询和转换
        String query = "SELECT message, COUNT(*) AS count FROM source_table GROUP BY message";
        DataStream<Result> resultStream = env.sqlQuery(query).map(value -> new Result(value.getField(0), value.getField(1)));

        // 打印结果
        resultStream.print();

        env.execute("Flink SQL Example");
    }

    // 自定义结果类
    public static class Result {
        public String message;
        public Long count;

        public Result() {}

        public Result(String message, Long count) {
            this.message = message;
            this.count = count;
        }

        @Override
        public String toString() {
            return "Result{" +
                    "message='" + message + '\'' +
                    ", count=" + count +
                    '}';
        }
    }
}
```

在上述示例中，我们使用 Apache Kafka 作为数据源，并创建了一个消费者从名为 "input-topic" 的 Kafka 主题中读取数据。然后，我们将数据流注册为名为 "source_table" 的临时表。

接下来，我们使用 Flink SQL 执行 SQL 查询和转换。在这个例子中，我们查询 "source_table" 表，对 "message" 字段进行分组并计算每个消息出现的次数。查询结果会映射到自定义的 `Result` 类，并最终通过 `print()` 方法打印到标准输出。

最后，我们通过调用 `env.execute()` 方法来启动 Flink 作业的执行。

## Flink的复杂事件处理CEP

复杂事件处理（CEP）是一种基于流处理的技术，将系统数据看作不同类型的事件，通过分析事件之间的关系，建立不同的事件关系序列库，并利用过滤、关联、聚合等技术，最终由简单事件产生高级事件，并通过模式规则的方式对重要信息进行跟踪和分析，从实时数据中发掘有价值的信息。复杂事件处理主要应用于防范网络欺诈、设备故障检测、风险规避和智能营销等领域。Flink基于DataStrem API提供了FlinkCEP组件栈，专门用于对复杂事件的处理，帮助用户从流式数据中发掘有价值的信息。

CEP(Complex Event Processing)就是在无界事件流中检测事件模式，让我们掌握数据中重要的部分。flink CEP是在flink中实现的复杂事件处理库。

###  CEP相关概念

#### 配置依赖

在使用FlinkCEP组件之前，需要将FlinkCEP的依赖库引入项目工程中。

``` xml
 <dependency>  
 	<groupId>org.apache.flink</groupId>  
 	<artifactId>flink-cep-scala_2.11</artifactId>  
 	<version>1.9.1</version>
 </dependency>

 <dependency> 
     <groupId>org.apache.flink</groupId>  
     <artifactId>flink-cep-scala_2.11</artifactId>  
 	<version>1.9.1</version>
</dependency>
```

#### 事件定义

- 简单事件：简单事件存在于现实场景中，主要的特点为处理单一事件，事件的定义可以直接观察出来，处理过程中无须关注多个事件之间的关系，能够通过简单的数据处理手段将结果计算出来。
- 复杂事件：相对于简单事件，复杂事件处理的不仅是单一的事件，也处理由多个事件组成的复合事件。复杂事件处理监测分析事件流(Event Streaming)，当特定事件发生时来触发某些动作。

​	复杂事件中事件与事件之间包含多种类型关系，常见的有时序关系、聚合关系、层次关系、依赖关系及因果关系等。

### Pattern API

Flink CEP中提供了Pattern API用于对输入流数据的复杂事件规则定义，并从事件流中抽取事件结果。包含四个步骤：

1. 输入事件流的创建
2. Pattern的定义
3. Pattern应用在事件流上检测
4. 选取结果

#### 模式定义

定义Pattern可以是单次执行模式，也可以是循环执行模式。单词执行模式一次只接受一个事件，循环执行模式可以接收一个或者多个事件。通常情况下，可以通过指定循环次数将单次执行模式变为循环执行模式。每种模式能够将多个条件组合应用到同一事件之上，条件组合可以通过where方法进行叠加。每个Pattern都是通过begin方法定义的

```scala
val start = Pattern.begin[Event]("start_pattern")
```

下一步通过Pattern.where()方法在Pattern上指定Condition，只有当Condition满足之后，当前的Pattern才会接受事件。

```scala
start.where(_.getCallType == "success")
```

#### 设置循环次数

对于已经创建好的Pattern，可以指定循环次数，形成循环执行的Pattern。

- times：可以通过times指定固定的循环执行次数。


```scala
//指定循环触发4次
start.times(4);
//可以执行触发次数范围,让循环执行次数在该范围之内
start.times(2, 4);
```


- optional：也可以通过optional关键字指定要么不触发要么触发指定的次数。

```scala
  start.times(4).optional();
  start.times(2, 4).optional();
```

- greedy：可以通过greedy将Pattern标记为贪婪模式，在Pattern匹配成功的前提下，会尽可能多地触发。

```scala
  //触发2、3、4次,尽可能重复执行
  start.times(2, 4).greedy();
  //触发0、2、3、4次,尽可能重复执行
  start.times(2, 4).optional().greedy();
```


- oneOrMore：可以通过oneOrMore方法指定触发一次或多次。

```
  // 触发一次或者多次
  start.oneOrMore();
  //触发一次或者多次,尽可能重复执行
  start.oneOrMore().greedy();
  // 触发0次或者多次
  start.oneOrMore().optional();
  // 触发0次或者多次,尽可能重复执行
  start.oneOrMore().optional().greedy();
```


- timesOrMore：通过timesOrMore方法可以指定触发固定次数以上，例如执行两次以上。

```
// 触发两次或者多次
  start.timesOrMore(2);
  // 触发两次或者多次,尽可能重复执行
  start.timesOrMore(2).greedy();
  // 不触发或者触发两次以上,尽可能重复执行
  start.timesOrMore(2).optional().greedy();
```

#### 定义条件

每个模式都需要指定触发条件，作为事件进入到该模式是否接受的判断依据，当事件中的数值满足了条件时，便进行下一步操作。在FlinkCFP中通过pattern.where()、pattern.or()及pattern.until()方法来为Pattern指定条件，且Pattern条件有Simple Conditions及Combining Conditions等类型。

- 简单条件：Simple Condition继承于Iterative Condition类，其主要根据事件中的字段信息进行判断，决定是否接受该事件。

```
  // 把通话成功的事件挑选出来
  start.where(_.getCallType == "success")
```


- 组合条件：组合条件是将简单条件进行合并，通常情况下也可以使用where方法进行条件的组合，默认每个条件通过AND逻辑相连。如果需要使用OR逻辑，直接使用or方法连接条件即可。

```scala
  // 把通话成功，或者通话时长大于10秒的事件挑选出来
  val start = Pattern.begin[StationLog]("start_pattern")
  .where(_.callType=="success")
  .or(_.duration>10)
```


- 终止条件：如果程序中使用了oneOrMore或者oneOrMore().optional()方法，则必须指定终止条件，否则模式中的规则会一直循环下去，如下终止条件通过until()方法指定。

```
  pattern.oneOrMore.until(_.callOut.startsWith("186"))
```

#### 模式序列

将相互独立的模式进行组合然后形成模式序列。模式序列基本的编写方式和独立模式一致，各个模式之间通过邻近条件进行连接即可，其中有严格邻近、宽松邻近、非确定宽松邻近三种邻近连接条件。

- 严格邻近：严格邻近条件中，需要所有的事件都按照顺序满足模式条件，不允许忽略任意不满足的模式。

`val strict: Pattern[Event] = start.next("middle").where(...)`

- 宽松邻近：在宽松邻近条件下，会忽略没有成功匹配模式条件，并不会像严格邻近要求得那么高，可以简单理解为OR的逻辑关系。

`val relaxed: Pattern[Event, _] = start.followedBy("middle").where(...)`

- 非确定宽松邻近：和宽松邻近条件相比，非确定宽松邻近条件指在模式匹配过程中可以忽略已经匹配的条件。 

`val nonDetermin: Pattern[Event, _] = start.followedByAny("middle").where(...)`

- 除以上模式序列外，还可以定义“不希望出现某种近邻关系”：


​		.notNext()  —— 不想让某个事件严格紧邻前一个事件发生。

​		.notFollowedBy() —— 不想让某个事件在两个事件之间发生。

**注意**：

1. 所有模式序列必须以 .begin() 开始

2. 模式序列不能以 .notFollowedBy() 结束

3. “not” 类型的模式不能被 optional 所修饰

4. 此外，还可以为模式指定时间约束，用来要求在多长时间内匹配有效

   ```java
   //指定模式在10秒内有效
   pattern.within(Time.seconds(10));
   ```

#### 模式检测

调用 CEP.pattern()，给定输入流和模式，就能得到一个 PatternStream

```scala
//cep 做模式检测
val patternStream = CEP.pattern[EventLog](dataStream.keyBy(_.id),pattern)
```

#### 选择结果

得到PatternStream类型的数据集后，接下来数据获取都基于PatternStream进行。该数据集中包含了所有的匹配事件。目前在FlinkCEP中提供select和flatSelect两种方法从PatternStream提取事件结果事件。

**通过Select Funciton抽取正常事件**

可以通过在PatternStream的Select方法中传入自定义Select Funciton完成对匹配事件的转换与输出。其中Select Funciton的输入参数为Map[String, Iterable[IN]]，Map中的key为模式序列中的Pattern名称，Value为对应Pattern所接受的事件集合，格式为输入事件的数据类型。

```scala
def selectFunction(pattern : Map[String, Iterable[IN]]): OUT = {  
//获取pattern中的startEvent  
val startEvent = pattern.get("start_pattern").get.next    
//获取Pattern中middleEvent    
val middleEvent = pattern.get("middle").get.next    
//返回结果    
OUT(startEvent, middleEvent)}
```

**通过Flat Select Funciton抽取正常事件**

​	Flat Select Funciton和Select Function相似，不过Flat Select Funciton在每次调用可以返回任意数量的结果。因为Flat Select Funciton使用Collector作为返回结果的容器，可以将需要输出的事件都放置在Collector中返回。

```scala
def flatSelectFn(pattern : Map[String, Iterable[IN]], collector : Collector[OUT]) = {    //获取pattern中startEvent  
val startEvent = pattern.get("start_pattern").get.next    
//获取Pattern中middleEvent  
val middleEvent = pattern.get("middle").get.next    
//并根据startEvent的Value数量进行返回  
for (i <- 0 to startEvent.getValue) {    
	collector.collect(OUT(startEvent, middleEvent))  
}}
```

**通过Select Funciton抽取超时事件**

如果模式中有within(time)，那么就很有可能有超时的数据存在，通过PatternStream. Select方法分别获取超时事件和正常事件。首先需要创建OutputTag来标记超时事件，然后在PatternStream.select方法中使用OutputTag，就可以将超时事件从PatternStream中抽取出来。

```scala
// 通过CEP.pattern方法创建
PatternStream  val patternStream: PatternStream[Event] = CEP.pattern(input, pattern)  //创建OutputTag,并命名为timeout-output  
val timeoutTag = OutputTag[String]("timeout-output")  
//调用PatternStream select()并指定timeoutTag  val result: SingleOutputStreamOperator[NormalEvent] =   patternStream.select(timeoutTag){  
//超时事件获取    
(pattern: Map[String, Iterable[Event]], timestamp: Long) =>       
TimeoutEvent()//返回异常事件  
} { 
//正常事件获取
pattern: Map[String, Iterable[Event]] =>      
NormalEvent()
//返回正常事件  
}
//调用getSideOutput方法,并指定timeoutTag将超时事件输出val timeoutResult: DataStream[TimeoutEvent] = result.getSideOutput(timeoutTag)
```

## Flink内存优化

在大数据领域，大多数开源框架（Hadoop、Spark、Flink）都是基于JVM运行，但是JVM的内存管理机制往往存在着诸多类似OutOfMemoryError的问题，主要是因为创建过多的对象实例而超过JVM的最大堆内存限制，却没有被有效回收掉，这在很大程度上影响了系统的稳定性，尤其对于大数据应用，面对大量的数据对象产生，仅仅靠JVM所提供的各种垃圾回收机制很难解决内存溢出的问题。在开源框架中有很多框架都实现了自己的内存管理，例如Apache Spark的Tungsten项目，在一定程度上减轻了框架对JVM垃圾回收机制的依赖，从而更好地使用JVM来处理大规模数据集。

**Flink也基于JVM实现了自己的内存管理，将JVM根据内存区分为Unmanned Heap、Flink Managed Heap、Network Buffers三个区域**。在Flink内部对Flink Managed Heap进行管理，在启动集群的过程中直接将堆内存初始化成Memory Pages Pool，也就是将内存全部以二进制数组的方式占用，形成虚拟内存使用空间。新创建的对象都是以序列化成二进制数据的方式存储在内存页面池中，当完成计算后数据对象Flink就会将Page置空，而不是通过JVM进行垃圾回收，保证数据对象的创建永远不会超过JVM堆内存大小，也有效地避免了因为频繁GC导致的系统稳定性问题。

### JobManager配置

JobManager在Flink系统中主要承担管理集群资源、接收任务、调度Task、收集任务状态以及管理TaskManager的功能，JobManager本身并不直接参与数据的计算过程中，因此JobManager的内存配置项不是特别多，只要指定JobManager堆内存大小即可。

`jobmanager.heap.size：设定JobManager堆内存大小，默认为1024MB。`

### TaskManager配置

TaskManager作为Flink集群中的工作节点，所有任务的计算逻辑均执行在TaskManager之上，因此对TaskManager内存配置显得尤为重要，可以通过以下参数配置对TaskManager进行优化和调整。

- **taskmanager.heap.size**：设定TaskManager堆内存大小，默认值为1024M，如果在Yarn的集群中，TaskManager取决于Yarn分配给TaskManager Container的内存大小，且Yarn环境下一般会减掉一部分内存用于Container的容错。

- **taskmanager.jvm-exit-on-oom**：设定TaskManager是否会因为JVM发生内存溢出而停止，默认为false，当TaskManager发生内存溢出时，也不会导致TaskManager停止。

- **taskmanager.memory.size**：设定TaskManager内存大小，默认为0，如果不设定该值将会使用taskmanager.memory.fraction作为内存分配依据。

- **taskmanager.memory.fraction**：设定TaskManager堆中去除Network Buffers内存后的内存分配比例。该内存主要用于TaskManager任务排序、缓存中间结果等操作。例如，如果设定为0.8，则代表TaskManager保留80%内存用于中间结果数据的缓存，剩下20%的内存用于创建用户定义函数中的数据对象存储。注意，该参数只有在taskmanager.memory.size不设定的情况下才生效。

- **taskmanager.memory.off-heap**：设置是否开启堆外内存供Managed Memory或者Network Buffers使用。

- **taskmanager.memory.preallocate**：设置是否在启动TaskManager过程中直接分配TaskManager管理内存。

- **taskmanager.numberOfTaskSlots**：每个TaskManager分配的slot数量。

### Flink的网络缓存优化

Flink将JVM堆内存切分为三个部分，其中一部分为Network Buffers内存。Network Buffers内存是Flink数据交互层的关键内存资源，主要目的是缓存分布式数据处理过程中的输入数据。。通常情况下，比较大的Network Buffers意味着更高的吞吐量。如果系统出现“Insufficient number of network buffers”的错误，一般是因为Network Buffers配置过低导致，因此，在这种情况下需要适当调整TaskManager上Network Buffers的内存大小，以使得系统能够达到相对较高的吞吐量。

目前Flink能够调整Network Buffer内存大小的方式有两种：一种是通过直接指定Network Buffers内存数量的方式，另外一种是通过配置内存比例的方式。

#### 设定Network Buffer内存数量（过时了）

直接设定Nework Buffer数量需要通过如下公式计算得出：

`NetworkBuffersNum = total-degree-of-parallelism \* intra-node-parallelism * n`

其中total-degree-of-parallelism表示每个TaskManager的总并发数量，intra-node-parallelism表示每个TaskManager输入数据源的并发数量，n表示在预估计算过程中Repar-titioning或Broadcasting操作并行的数量。intra-node-parallelism通常情况下与Task-Manager的所占有的CPU数一致，且Repartitioning和Broadcating一般下不会超过4个并发。可以将计算公式转化如下：

`NetworkBuffersNum  = <slots-per-TM>^2 \* < TMs>* 4`

其中slots-per-TM是每个TaskManager上分配的slots数量，TMs是TaskManager的总数量。对于一个含有20个TaskManager，每个TaskManager含有8个Slot的集群来说，总共需要的Network Buffer数量为8^2**20*4=5120个，因此集群中配置Network Buffer内存的大小约为160M较为合适。

计算完Network Buffer数量后，可以通过添加如下两个参数对Network Buffer内存进行配置。其中segment-size为每个Network Buffer的内存大小，默认为32KB，一般不需要修改，通过设定numberOfBuffers参数以达到计算出的内存大小要求。

- **taskmanager.network.numberOfBuffers**：指定Network堆栈Buffer内存块的数量。

- **taskmanager.memory.segment-size**：内存管理器和Network栈使用的内存Buffer大小，默认为32KB。

#### 设定Network内存比例（推荐）

从1.3版本开始，Flink就提供了通过指定内存比例的方式设置Network Buffer内存大小。

- **taskmanager.network.memory.fraction**：JVM中用于Network Buffers的内存比例。

- **taskmanager.network.memory.min**：最小的Network Buffers内存大小，默认为64MB。

- **taskmanager.network.memory.max**：最大的Network Buffers内存大小，默认1GB。

- **taskmanager.memory.segment-size**：内存管理器和Network栈使用的Buffer大小，默认为32KB。
