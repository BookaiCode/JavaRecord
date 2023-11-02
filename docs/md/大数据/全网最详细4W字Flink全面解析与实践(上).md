**注：原文字数过多，单篇阅读时间过长，故将文章拆分为上下两篇**

在大数据技术栈的探索中，我们曾讨论了离线计算的Spark，而当谈到实时计算，就不得不提Flink。本文将集中讨论Flink，旨在详尽展示其核心概念，从而助力你在大数据旅程中向前迈进。

值得注意的是，Flink和Spark有许多相似的概念。因此，在深入学习Flink之前，建议先浏览我之前关于Spark的文章，这将为你提供扎实的基础，并帮助在学习Flink时能更好地举一反三，加深对其理解。

话不多说，开启我们的Flink学习之旅。


## 流处理 & 批处理

在我们深入探讨Flink之前，首先要掌握一些流计算的基础概念。

- **流处理**：流处理主要针对的是数据流，特点是无界、实时，对系统传输的每个数据依次执行操作，一般用于实时统计。在流处理中，数据被视为无限连续的流，并且会尽快地进行处理。Flink在此模型下可以提供秒级甚至毫秒级的延迟，使其成为需要快速反应和决策的场景（例如实时推荐、欺诈检测等）的理想选择。
- **批处理**：批处理，也叫作离线处理，一般用于离线统计。这是一种处理存储在系统中的静态数据集的模型。在批处理中，所有数据都被看作是一个有限集合，处理过程通常在非交互式模式下进行，即作业开始时所有数据都已经可用，作业结束时给出所有计算结果。由于批处理允许对整个数据集进行全面分析，因此它适合于需要长期深度分析的场景（如历史数据分析、大规模ETL任务等）。

事实上 Flink 本身是流批统一的处理架构，批量的数据集本质上也是流。

**在 Flink 的视角里，一切数据都可以认为是流，流数据是无界流，而批数据则是有界流**

### 无界流Unbounded Streams

无界流有定义流的开始，但没有定义流的结束。它们会无休止地产生数据。无界流的数据必须持续处理，即数据被摄取后需要立刻处理。

我们不能等到所有数据都到达再处理，因为输入是无限的，在任何时候输入都不会完成。

处理无界数据通常要求以特定顺序摄取事件，例如事件发生的顺序，以便能够推断结果的完整性。

### 有界流Bounded Streams

有界流有定义流的开始，也有定义流的结束。有界流可以在摄取所有数据后再进行计算，有界流所有数据可以被排序，所以并不需要有序摄取。

有界流处理通常被称为批处理。所以在Flink里批计算其实指的就是有界流。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnjPQcmW39R8wibMMpqpQGLoiaMFiakpuFUajrXE8dL0yHwib8icmg6Y1fib5Tv06EnMMtH7jtXJJQVAMQ/640)

## Flink的特点和优势

Flink具有如下特点和优势：

- 同时支持高吞吐、低延迟、高性能。
- 支持事件时间（Event Time）概念，结合Watermark处理乱序数据。
- 支持有状态计算，并且支持多种状态内存、 文件、RocksDB。
- 支持高度灵活的窗口（Window） 操作 time、 count、 session等。
- 基于轻量级分布式快照（CheckPoint） 实现的容错保证Exactly- Once语义。
- 基于JVM实现独立的内存管理。

## Flink VS Spark

Spark 和 Flink 在不同的应用领域上表现会有差别。

一般来说，Spark 基于微批处理的方式做同步总有一个“攒批”的过程，所以会有额外开销，因此无法在流处理的低延迟上做到极致。

**在低延迟流处理场景，Flink 已经有明显的优势。而在海量数据的批处理领域，Spark 能够处理的吞吐量更大**

另外，Spark Streaming中的流计算其实是微批计算，实时性不如Flink，还有一点很重要的是Spark Streaming不适合有状态的计算，得借助一些存储如：Redis，才能实现。而Flink天然支持有状态的计算。

## Flink API

Flink 本身提供了多层 API：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLp9EtLdWwjR2aiaWXxBr5q1JOc9Q1k8P7OibcCvlYxib3HzE9VQKzx3ktAg/640)

- **Stateful Stream Processing** ：最低级的抽象接口是状态化的数据流接口（stateful streaming）。这个接口是通过 ProcessFunction 集成到 DataStream API 中的。该接口允许用户自由的处理来自一个或多个流中的事件，并使用一致的容错状态。另外，用户也可以通过注册 event time 和 processing time 处理回调函数的方法来实现复杂的计算。
- **DataStream/DataSet API**： DataStream / DataSet API 是 Flink 提供的核心 API ，DataSet 处理有界的数据集，DataStream 处理有界或者无界的数据流。用户可以通过各种方法（map / flatmap / window / keyby / sum / max / min / avg / join 等）将数据进行转换 / 计算。
- **Table API**：  Table API 提供了例如 select、project、join、group-by、aggregate 等操作，使用起来却更加简洁，可以在表与 DataStream/DataSet 之间无缝切换，也允许程序将 Table API 与 DataStream 以及 DataSet 混合使用。
- **SQL**： Flink 提供的最高层级的抽象是 Flink SQL，这一层抽象在语法与表达能力上与 Table API 类似，SQL 抽象与 Table API 交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

## Dataflows数据流图

所有的 Flink 程序都可以归纳为由三部分构成：`Source`、`Transformation` 和 `Sink`。

- Source 表示“源算子”，负责读取数据源。

- Transformation 表示“转换算子”，利用各种算子进行处理加工。

- Sink 表示“下沉算子”，负责数据的输出。

Source数据源会源源不断的产生数据，Transformation将产生的数据进行各种业务逻辑的数据处理，最终由Sink输出到外部（console、kafka、redis、DB......）。

所有基于Flink开发的程序都能够映射成一个Dataflows（数据流图）：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLptqzt0pibVTiazpib2DbAt6DJKrqnSX5iaJdgt2ShOAF8ibGz7MBVScwiaxCQ/640)

当Source数据源的数量比较大或计算逻辑相对比较复杂的情况下，需要提高并行度来处理数据，采用并行数据流。

通过设置不同算子的并行度，比如 Source并行度设置为2 ，map也是2。代表会启动2个并行的线程来处理数据：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLpCaXHUMG7RLn3A4FX2Pq4SqcvwdAFhN74T5lcquFZEA37untZibCObsw/640)

## Job Manager & Task Manager

Flink是一个典型的Master-Slave架构，架构中包含了两个重要角色，分别是「**JobManager**」和「**TaskManager**」。

JobManager相当于是Master，TaskManager相当于是Slave。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMFdkqmnnJ526HR8ktibGOLpvf34vfkbzvdq19zWJoAMNNMVJkWI5tgc1r2Xwcc0w07sswUricnibNQA/640)



在Flink中，JobManager负责整个Flink集群任务的调度以及资源的管理。它从客户端中获取提交的应用，然后根据集群中TaskManager上TaskSlot的使用情况，为提交的应用分配相应的TaskSlot资源并命令TaskManager启动从客户端中获取的应用。

TaskManager则负责执行作业流的Task，并且缓存和交换数据流。

在TaskManager中资源调度的最小单位是Task slot。TaskManager中Task slot的数量表示并发处理Task的数量。

**一台机器节点可以运行多个TaskManager，TaskManager工作期间会向JobManager发送心跳保持连接**

## 部署 & 运行

### 部署模式

Flink支持多种部署模式，包括本地模式、Standalone模式、YARN模式、Mesos模式和Kubernetes模式。

- **本地模式**：本地模式是在单个JVM中启动Flink，主要用于开发和测试。它不需要任何集群管理器，但也不能跨多台机器运行。本地模式的优点是部署简单，缺点是不能利用分布式计算的优势。
- **Standalone模式**：Standalone模式是在一个独立的集群中运行Flink。它需要手动启动Flink集群，并且需要手动管理资源。Standalone模式的优点是部署简单，可以跨多台机器运行，缺点是需要手动管理资源。
- **YARN模式**：在这个模式下，Flink作为YARN的一个应用程序运行在YARN集群中。Flink会从YARN获取所需的资源来运行JobManager和TaskManager。如果你已经有了一个运行Hadoop/YARN的大数据平台，选择这个模式可以方便地利用已有的资源，这是企业中用的比较多的方式。
- **Mesos模式**：Mesos是一个更通用的集群管理框架，可以运行非Java应用程序，并具有良好的容错性和伸缩性。Flink在Mesos上的运行方式与在YARN上类似，也是从Mesos请求资源来运行JobManager和TaskManager。
- **Kubernetes模式**：Kubernetes是一个开源的容器编排平台。Flink可以作为一组分布式的Docker容器在Kubernetes集群上运行。Kubernetes提供了自动化、扩展和管理应用程序容器的功能，对于云原生应用程序部署非常合适。

每种部署模式都有其优缺点，选择哪种部署模式取决于具体的应用场景和需求。

### 运行模式

Flink 有三种不同的运行模式：Session、Per-Job 和 Application。

- **Session**：在这种模式下，一个 Flink 集群会被启动且会一直运行，直到明确地被终止。用户可以在这个集群中提交多个作业。这个模式适合多个短作业的场景。
- **Per-Job**：在这种模式下，对于每个提交的作业，都会启动一个新的 Flink 集群，然后再执行该作业。作业完成后，相应的 Flink 集群也会被终止。这种模式适合长时间运行的作业。
- **Application**：这种模式是一种特殊的 Per-Job 模式，它允许用户以反应式的方式与作业进行交互（比如，使用 DataStream API）。这是 Flink 1.11 版本引入的新模式，它结合了Session模式和Per-Job模式的优点。在Application模式下，每个作业都会启动一个独立的Flink集群，但是作业提交快。

以上所述的部署环境可以与任何一种运行模式结合使用。例如，你可以在本地模式、Standalone 模式或 YARN 模式下运行 Session、Per-Job 或 Application 模式的 Flink 作业。

### 提交和执行作业流程

Flink在不同运行模型下的作业提交和执行流程大致如下：

- **Session 模式**：

   - 启动Flink集群：在Session模式下，首先需要启动一个运行中的Flink集群。这个集群可以是Standalone Session Cluster，也可以是在Yarn或Kubernetes等资源管理器上的Session Cluster。
   - 作业提交：然后，用户通过Flink客户端（例如CLI、REST API或Web UI）将作业提交给Flink Dispatcher服务。Dispatcher服务是Flink集群的主要入口点，负责接收和协调作业请求。
   - 作业解析与优化：一旦Flink Dispatcher接收到作业，它会对作业执行图（JobGraph）进行解析，并使用Flink的优化器对执行图进行优化。
   - 创建作业执行环境：Dispatcher会为新的作业创建一个JobManager，这个JobManager就是一次作业的执行环境。并且，每个Job都有属于自己的JobManager。
   - 作业执行：JobManager将优化后的执行图发送到TaskManager节点来执行具体的任务。TaskManager节点包含若干个slot，每个slot可以运行作业图中的一个并行操作。
   - 结果和状态管理：作业执行过程中，输出结果被发送回JobManager，并提供给用户。同时，作业的状态也由JobManager管理，以支持故障恢复。

   当你的作业完成运行后，该作业的JobManager会被停止，但是Flink集群（包括Dispatcher和其余的TaskManager）仍然处于运行状态，等待新的作业提交。这就是所谓的Session模式，它允许在同一个Flink集群上连续运行多个作业。
- **Per-Job 模式**：

   - 用户通过命令行或者UI将程序包含所有依赖提交到Flink集群。
   - Flink Master节点接收到用户提交的作业后，会启动一个新的JobManager来负责这个作业的资源管理与任务调度。
   - JobManager通过ResourceManager向Flink集群请求所需的TaskManager资源。
   - ResourceManager分配TaskManager给JobManager，并启动TaskManager进程。
   - TaskManager向JobManager注册并提供自己的状态及可用的slot信息。
   - JobManager根据程序的DAG图计算出ExecutionGraph，然后按照stages将相应的tasks分配到TaskManager的Slots中去执行。
   - 如果作业执行完毕或执行失败，JobManager会释放所有资源，并将结果返回给用户。

   在Per-Job模式下，每个作业都有自己的资源隔离，互不干扰，资源利用率较高，但是如果作业数量大，则可能会因为每个作业都需要单独申请、释放资源导致效率较低。
- **Application 模式**：

   - 构建Flink Job：客户端或者用户在本地环境上构建Flink作业。
   - 提交Flink Job：通过Flink命令行工具或者Web UI，将序列化后的JobGraph提交到Flink集群。也可以通过REST API直接提交作业。
   - JobManager接收Job：JobManager是Flink中负责任务调度和协调的组件。它会接收到提交的JobGraph，并将其封装成ExecutionGraph。
   - 任务调度：JobManager会根据ExecutionGraph对任务进行调度，决定何时启动任务，以及哪个TaskManager上启动任务。
   - 任务执行：TaskManager接收到JobManager分配的任务后开始执行。每个TaskManager包含一到多个Slot，这些Slot用于运行任务。
   - 状态反馈：TaskManager在执行任务过程中会将状态信息（如进度、日志等）反馈给JobManager。
   - 结果返回：当所有任务执行完成后，JobManager会将执行结果返回给客户端。

## 配置开发环境

每个 Flink 应用都需要依赖一组 Flink 类库。Flink 应用至少需要依赖 Flink APIs。许多应用还会额外依赖连接器类库(比如 Kafka、Cassandra 等)。 当用户运行 Flink 应用时(无论是在 IDEA 环境下进行测试，还是部署在分布式环境下)，运行时类库都必须可用。

开发工具：IntelliJ IDEA

以Java语言为例，配置开发Maven依赖：

```xml
    <properties>
        <flink.version>1.13.6</flink.version>
        <scala.binary.version>2.12</scala.binary.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
    </dependencies>
```

**注意点**：

- 如果要将程序打包提交到集群运行，打包的时候不需要包含这些依赖，因为集群环境已经包含了这些依赖，此时依赖的作用域应该设置为`provided`。
- Flink 应用在 IntelliJ IDEA 中运行，这些 Flink 核心依赖的作用域需要设置为 `compile` 而不是 `provided` 。 否则 IntelliJ 不会添加这些依赖到 classpath，会导致应用运行时抛出 `NoClassDefFountError` 异常。

添加打包插件：

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <artifactSet>
                                <excludes>
                                    <exclude>org.apache.flink:force-shading</exclude>
                                    <exclude>com.google.code.findbugs:jsr305</exclude>
                                    <exclude>org.slf4j:*</exclude>
                                    <exclude>log4j:*</exclude>
                                </excludes>
                            </artifactSet>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### WordCount程序

配置好开发环境之后我们来写一个简单的Flink程序。

需求：统计单词出现的次数

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 设置并行度为 1
        env.setParallelism(1);
        // 构建输入数据流
        DataStream<String> text = env.fromElements(
                "Hello World",
                "Hello Flink",
                "Hello Java");
        // 对输入数据进行操作，包括分割、映射和聚合
        DataStream<Tuple2<String, Integer>> counts = text
                .flatMap(new Tokenizer())
                //keyBy(0) 操作按照元组的第一个字段（索引为0）进行分组。在这个例子中，这就表示根据每个单词（字符串）进行分组。         
                .keyBy(0)
                //sum(1) 是一个聚合操作，它对每个分组内的元素进行求和。在这个例子中，对元组的第二个字段（索引为1）进行求和，表示每个单词的出现次数。
                .sum(1);
        // 输出结果
        counts.print();
        // 执行任务
        env.execute("Flink Streaming Java WordCount");
    }

    public static final class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            // 规范化并分割行
            String[] words = value.toLowerCase().split("\\W+");
            // 为每个单词发出 Tuple2<String, Integer>(word, 1)
            for (String word : words) {
                if (word.length() > 0) {
                    out.collect(new Tuple2<>(word, 1));
                }
            }
        }
    }
```

输出如下：

```
(hello,1)
(world,1)
(hello,2)
(flink,1)
(hello,3)
(java,1)
```

对代码简要解析一下：

这是一个基本的单词计数程序，它使用Apache Flink的流处理环境。这个程序读入一系列的字符串，然后把每个字符串分割成单词，对每个单词进行计数，并且输出计数结果。

在提供的例子中，有三个输入字符串："Hello World", "Hello Flink", "Hello Java"，'Hello'这个单词出现了三次，其余单词 ('World', 'Flink', 'Java') 各出现了一次。

由于Flink是一个流处理框架，它实时地处理数据，所以它会在每一次遇到一个新的单词时就更新和输出计数。因此，每当 'Hello' 出现时，都会更新和输出其计数。

对于这个例子:

- 首先遇到 'Hello' 和 'World'，所以输出 (hello,1) 和 (world,1)。
- 然后再次遇到 'Hello' 和第一次遇到 'Flink'，所以输出 (hello,2) 和 (flink,1)。
- 最后再次遇到 'Hello' 和第一次遇到 'Java'，所以输出 (hello,3) 和 (java,1)。



这段代码已在本地运行和测试过，且相关部分已添加注释，大家可以实际运行感受一下。

## 并行度

**特定算子的子任务（subtask）的个数称之为并行度（parallel），并行度是几，这个task内部就有几个subtask**

怎样实现算子并行呢？其实也很简单，我们把一个算子操作，“复制”多份到多个节点，数据来了之后就可以到其中任意一个执行。这样一来，一个算子任务就被拆分成了多个并行的“子任务”（subtasks），再将它们分发到不同节点，就真正实现了并行计算。

**整个流处理程序的并行度，理论上是所有算子并行度中最大的那个，这代表了运行程序需要的 slot 数量**

如果我们将上面WordCount程序的并行度设置为3

```
env.setParallelism(3);
```

就会看到如下输出：

```
2> (world,1)
3> (flink,1)
1> (hello,1)
1> (hello,2)
1> (java,1)
1> (hello,3)
```

前面的数字代表线程，Flink会将相同的 key 分配到不同的 slot 进行处理。

### 并行度设置

在 Flink 中，可以用不同的方法来设置并行度，它们的有效范围和优先级别也是不同的。

**代码中设置** 

- 我们在代码中，可以很简单地在算子后跟着调用 `setParallelism()`方法，来设置当前算子的并行度： `stream.map(word -> Tuple2.of(word, 1L)).setParallelism(2);`这种方式设置的并行度，只针对当前算子有效。
- 我们也可以直接调用执行环境的 `setParallelism()`方法，全局设定并行度：`env.setParallelism(2);`这样代码中所有算子，默认的并行度就都为2了。

**提交应用时设置**

在使用 flink run 命令提交应用时，可以增加 `-p` 参数来指定当前应用程序执行的并行度，它的作用类似于执行环境的全局设置。如果我们直接在 Web UI 上提交作业，也可以在对应输入框中直接添加并行度。

**配置文件中设置**

我们还可以直接在集群的配置文件 flink-conf.yaml 中直接更改默认并行度：`parallelism.default: 2`（初始值为 1）

这个设置对于整个集群上提交的所有作业有效。

### 并行度生效优先级

1. 对于一个算子，首先看在代码中是否单独指定了它的并行度，这个特定的设置优先级最高，会覆盖后面所有的设置。
2. 如果没有单独设置，那么采用当前代码中执行环境全局设置的并行度。 
3. 如果代码中完全没有设置，那么采用提交时`-p` 参数指定的并行度。
4. 如果提交时也未指定`-p` 参数，那么采用集群配置文件中的默认并行度。 

**这里需要说明的是，算子的并行度有时会受到自身具体实现的影响。比如读取 socket 文本流的算子 socketTextStream，它本身就是非并行的 Source 算子，所以无论怎么设置，它在运行时的并行度都是 1**

## Task

在 Flink 中，Task 是一个阶段多个功能相同 subTask 的集合，Flink 会尽可能地将 operator 的 subtask 链接（chain）在一起形成 task。每个 task 在一个线程中执行。将 operators 链接成 task 是非常有效的优化：它能减少线程之间的切换，减少消息的序列化/反序列化，减少数据在缓冲区的交换，减少了延迟的同时提高整体的吞吐量。

**要是之前学过Spark，这里可以用Spark的思想来看，Flink的Task就好比Spark中的Stage，而我们知道Spark的Stage是根据宽依赖来拆分的。所以我们也可以认为Flink的Task也是根据宽依赖拆分的（尽管Flink中并没有宽依赖的概念），这样会更好理解**

如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOiaicvO7VztvmFhl4p3h3icdPLDnH9XhJ7ojiaxpbII45iabaDJico8S9WsHo0qnCMRMJYQjfic2M54CWkw/640)

## Operator Chain（算子链)

在Flink中，为了分布式执行，Flink会将算子子任务链接在一起形成任务。每个任务由一个线程执行。将算子链接在一起形成任务是一种有用的优化：**它减少了线程间切换和缓冲的开销，并增加了整体吞吐量，同时降低了延迟**

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

**在这个例子中，`map`和`filter`操作可以被链接在一起形成一个任务，被优化为算子链，这意味着它们将在同一个线程中执行，而不是在不同的线程中执行并通过网络进行数据传输**

## Task Slots

Task Slots即是任务槽，slot 在 Flink 里面可以认为是资源组，Flink 将每个任务分成子任务并且将这些子任务分配到 slot 来并行执行程序，我们可以通过集群的配置文件来设定 TaskManager 的 slot 数量： `taskmanager.numberOfTaskSlots : 8`。

例如，如果 Task Manager 有2个 slot，那么它将为每个 slot 分配 50％ 的内存。 可以在一个 slot 中运行一个或多个线程。 同一 slot 中的线程共享相同的 JVM。

**需要注意的是，slot 目前仅仅用来隔离内存，不会涉及 CPU 的隔离。在具体应用时，可以将 slot 数量配置为机器的 CPU 核心数，尽量避免不同任务之间对 CPU 的竞争。这也是开发环境默认并行度设为机器 CPU 数量的原因**

### 分发规则

- **不同的Task下的subtask要分发到同一个TaskSlot中，降低数据传输、提高执行效率**
- **相同的Task下的subtask要分发到不同的TaskSlot**

### Slot共享组

如果希望某个算子对应的任务完全独占一个 slot，或者只有某一部分算子共享 slot，在Flink中，可以通过在代码中使用`slotSharingGroup`方法来设置slot共享组。

Flink会将具有相同slot共享组的操作放入同一个slot中，同时保持不具有slot共享组的操作在其他slot中。这可以用来隔离slot。

例如，你可以这样设置：

```Java
dataStream.map(...).slotSharingGroup("group1");
```

默认情况下，所有操作都被分配相同的SlotSharingGroup。

这样，只有属于同一个 slot 共享组的子任务，才会开启 slot 共享，不同组之间的任务是完全隔离的，必须分配到不同的 slot 上。

### 并行度和Slots解释

听了上面并行度和Slots的理论，可能还是有点疑惑，下面通过例子解释说明下：

假设一共有3个TaskManager，每一个TaskManager中的slot数量设置为3个，那么一共有9个task slot，表示最多能并行执行9个任务。

假设我们写了一个WordCount程序，有四个转换算子：**source —> flatMap —> reduce —> sink**

当所有算子并行度相同时，很容易看出source和flatMap可以优化合并算子链，于是最终有三个任务节点：source & flatMap，reduce 和sink。
如果我们没有任何并行度设置，而配置文件中默认`parallelism.default：1`，那么默认并行度为1，总共有3个任务。**由于不同算子的任务可以共享任务槽，所以最终占用的slot只有1个。9个slot只用了1个，有8个空闲**

如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnjPQcmW39R8wibMMpqpQGLbsA32ngsNbhXic9yRxkicMDt9joDuuGeeT3dBtyp0jibQ6Ewib5wFx5Yeg/640)

我们可以直接把并行度设置为 9，这样所有 3*9=27 个任务就会完全占用 9 个 slot。这是当前集群资源下能执行的最大并行度，计算资源得到了充分的利用。

另外再考虑对于某个算子单独设置并行度的场景。例如，如果我们考虑到输出可能是写入文件，那会希望不要并行写入多个文件，就需要设置 sink 算子的并行度为 1。这时其他的算子并行度依然为 9，所以总共会有 19 个子任务。

根据 slot 共享的原则，它们最终还是会占用全部的 9 个 slot，而 sink 任务只在其中一个 slot 上执行，通过这个例子也可以明确地看到，**整个流处理程序的并行度，就应该是所有算子并行度中最大的那个，这代表了运行程序需要的 slot 数量**

## DataSource数据源

Flink内嵌支持的数据源非常多，比如HDFS、Socket、Kafka、Collections。Flink也提供了addSource方式，可以自定义数据源，下面介绍一些常用的数据源。

### File Source

- 通过读取本地、HDFS文件创建一个数据源。

如果读取的是HDFS上的文件，那么需要导入Hadoop依赖

```xml
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.3.1</version>
        </dependency>
```

代码示例：每隔10s去读取HDFS指定目录下的新增文件内容，并且进行WordCount。

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        String filePath = "hdfs://node01:9000/flink/data/";
        FileInputFormat<String> textInputFormat = new TextInputFormat(new Path(filePath));
        //PROCESS_CONTINUOUSLY模式时，Flink会持续监视给定的路径，并在发现新数据时将其引入流中进行处理。
        DataStream<String> textStream = env.readFile(textInputFormat, filePath, FileProcessingMode.PROCESS_CONTINUOUSLY, 10000);
        DataStream<Tuple2<String, Integer>> result = textStream
                .flatMap(new WordSplitter())
                .map(new WordMapper())
                .keyBy(0)
                .sum(1);
        result.print();
        env.execute();
    }

    public static class WordSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            String[] words = value.split(" ");
            for (String word : words) {
                out.collect(new Tuple2<>(word, 1));
            }
        }
    }
    public static class WordMapper implements MapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>> {
        @Override
        public Tuple2<String, Integer> map(Tuple2<String, Integer> wordCountTuple) {
            //f0, f1 等是用来访问元组中的元素的字段。Tuple2<String, Integer> 表示这是一个大小为 2 的元组，其中 f0 是 String 类型，f1 是 Integer 类型。
            // 在代码中，wordCountTuple.f0 表示的就是单词（即String类型的值），wordCountTuple.f1 则表示的是这个单词的计数（即 Integer 类型的值）。
            return new Tuple2<>(wordCountTuple.f0, wordCountTuple.f1);
        }
    }
```

### Collection Source

基于本地集合的数据源，一般用于测试场景，对于线上环境没有太大意义。

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        List<String> data = Arrays.asList("hello word flink","hello java flink");
        DataStream<String> text = env.fromCollection(data);
        DataStream<Tuple2<String, Integer>> counts = text
                .flatMap(new Tokenizer())
                .keyBy(0)
                .sum(1);
        counts.print();
        env.execute("WordCount Example");
    }
```

### Socket Source

接受Socket Server中的数据：

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 连接 socket 获取输入数据
        DataStream<String> text = env.socketTextStream("localhost", 9999);
        // 解析数据、对数据进行分组、窗口处理和聚合计算
        DataStream<Tuple2<String, Integer>> wordCount = text.flatMap(new Tokenizer())
                .keyBy(0)
                .sum(1);
        wordCount.print();
        env.execute("WordCount from Socket TextStream Example");
    }
```

### Kafka Source

Flink想要接受Kafka中的数据，首先要配置flink与kafka的连接器依赖。

Maven依赖：

```xml
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.12</artifactId>
    <version>1.13.6</version>
</dependency>
```

示例代码：

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 设置Kafka的相关参数并从Kafka中读取数据
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "test");
        FlinkKafkaConsumer<String> flinkKafkaConsumer = new FlinkKafkaConsumer<>("topic_name", new SimpleStringSchema(), properties);
        DataStream<String> stream = env.addSource(flinkKafkaConsumer);
        // 对接收的每一行数据进行处理，分割出每个单词并初始化其数量为1
        DataStream<Tuple2<String, Integer>> words = stream.flatMap(new Tokenizer());
        DataStream<Tuple2<String, Integer>> wordCounts = words.keyBy(0).sum(1);
        wordCounts.print().setParallelism(1);
        env.execute("WordCountFromKafka");
    }
```

## Transformations

Transformations算子可以将一个或者多个算子转换成一个新的数据流，使用Transformations算子组合可以处理复杂的业务处理。

### Map

DataStream → DataStream

遍历数据流中的每一个元素，产生一个新的元素。

### FlatMap

DataStream → DataStream

遍历数据流中的每一个元素，产生N个元素 N=0，1，2......。

### Filter

DataStream → DataStream

过滤算子，根据数据流的元素计算出一个boolean类型的值，true代表保留，false代表过滤掉。

### KeyBy

DataStream → KeyedStream

根据数据流中指定的字段来分区，相同指定字段值的数据一定是在同一个分区中，内部分区使用的是HashPartitioner。

指定分区字段的方式有三种：

- 根据索引号指定。

- 通过匿名函数来指定。
- 通过实现KeySelector接口  指定分区字段。

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<Long> stream = env.fromSequence(1, 100);
        stream.map((MapFunction<Long, Tuple2<Long, Integer>>) (Long x) -> new Tuple2<>(x % 3, 1), TypeInformation.of(new TypeHint<Tuple2<Long, Integer>>() {}))
                //根据索引号来指定分区字段：.keyBy(0)
                //通过传入匿名函数 指定分区字段：.keyBy(x=>x._1)
                //通过实现KeySelector接口  指定分区字段                
                .keyBy((KeySelector<Tuple2<Long, Integer>, Long>) (Tuple2<Long, Integer> value) -> value.f0, BasicTypeInfo.LONG_TYPE_INFO)
                .sum(1).print();
        env.execute("Flink Job");
    }
```

### Reduce

适用于KeyedStream

KeyedStream：根据key分组 → DataStream

**注意，reduce是基于分区后的流对象进行聚合，也就是说，DataStream类型的对象无法调用reduce方法**

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Tuple2<String, Integer>> dataStream = env.fromElements(
                new Tuple2<>("apple", 3),
                new Tuple2<>("banana", 1),
                new Tuple2<>("apple", 5),
                new Tuple2<>("banana", 2),
                new Tuple2<>("apple", 4)
        );
        // 使用reduce操作，将input中的所有元素合并到一起
        DataStream<Tuple2<String, Integer>> result = dataStream
                .keyBy(0)
                .reduce((ReduceFunction<Tuple2<String, Integer>>) (value1, value2) -> new Tuple2<>(value1.f0, value1.f1 + value2.f1));
        result.print();
        env.execute();
    }
```

### Aggregations

KeyedStream → DataStream

Aggregations代表的是一类聚合算子，上面说的reduce就属于Aggregations，以下是一些常用的：

- `sum()`: 计算数字类型字段的总和。
- `min()`: 计算最小值。
- `max()`: 计算最大值。
- `count()`: 计数元素个数。
- `avg()`: 计算平均值。

另外，Flink 还支持自定义聚合函数，即使用 `AggregateFunction` 接口实现更复杂的聚合逻辑。

### Union 真合并

DataStream → DataStream

> Union of two or more data streams creating a new stream containing all the elements from all the streams
>

**合并两个或者更多的数据流产生一个新的数据流，这个新的数据流中包含了所合并的数据流的元素**

注意：需要保证数据流中元素类型一致

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Tuple2<String, Integer>> ds1 = env.fromCollection(Arrays.asList(Tuple2.of("a",1),Tuple2.of("b",2),Tuple2.of("c",3)));
        DataStream<Tuple2<String, Integer>> ds2 = env.fromCollection(Arrays.asList(Tuple2.of("d",4),Tuple2.of("e",5),Tuple2.of("f",6)));
        DataStream<Tuple2<String, Integer>> ds3 = env.fromCollection(Arrays.asList(Tuple2.of("g",7),Tuple2.of("h",8)));
        DataStream<Tuple2<String, Integer>> unionStream = ds1.union(ds2,ds3);
        unionStream.print();
        env.execute();
    }
```

在 Flink 中，Union 操作被称为 "真合并" 是因为它将两个或多个数据流完全融合在一起，没有特定的顺序，并且不会去除重复项。这种操作方式类似于在数学概念中的集合联合（Union）操作，所以被称为 "真合并"。

请注意，与其他一些数据处理框架中的 Union 操作相比，例如 Spark 中的 Union 会根据某些条件去除重复的元素，Flink 的 Union 行为更接近于数学上的集合联合理论。

### Connect 假合并

DataStream,DataStream → ConnectedStreams

合并两个数据流并且保留两个数据流的数据类型，能够共享两个流的状态

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<String> ds1 = env.socketTextStream("localhost", 8888);
        DataStream<String> ds2 = env.socketTextStream("localhost", 9999);

        DataStream<Tuple2<String, Integer>> wcStream1 = ds1
                .flatMap(new Tokenizer())
                .keyBy(value -> value.f0)
                .sum(1);

        DataStream<Tuple2<String, Integer>> wcStream2 = ds2
                .flatMap(new Tokenizer())
                .keyBy(value -> value.f0)
                .sum(1);

        ConnectedStreams<Tuple2<String, Integer>, Tuple2<String, Integer>> connectedStreams = wcStream1.connect(wcStream2);
    }
```

与`union`不同，`connect`只能连接两个流，并且这两个流的类型可以不同。`connect`后的两个流会被看作是两个不同的流，可以使用`CoMap`或者`CoFlatMap`函数分别处理这两个流。

### CoMap, CoFlatMap

ConnectedStreams → DataStream

**CoMap, CoFlatMap并不是具体算子名字，而是一类操作的名称**

- 凡是基于ConnectedStreams数据流做map遍历，这类操作叫做CoMap。
- 凡是基于ConnectedStreams数据流做flatMap遍历，这类操作叫做CoFlatMap。

CoMap实现：

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 创建两个不同的数据流
        DataStream<Integer> nums = env.fromElements(1, 2, 3, 4, 5);
        DataStream<String> text = env.fromElements("a", "b", "c");
        // 连接两个数据流
        ConnectedStreams<Integer, String> connected = nums.connect(text);
        // 使用 CoMap 处理连接的流
        DataStream<String> result = connected.map(new CoMapFunction<Integer, String, String>() {
            @Override
            public String map1(Integer value) {
                return String.valueOf(value*2);
            }
            @Override
            public String map2(String value) {
                return "hello " + value;
            }
        });
        result.print();
        env.execute("CoMap example");
    }
```

CoFlatMap实现方式：

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Integer> nums = env.fromElements(1, 2, 3, 4, 5);
        DataStream<String> text = env.fromElements("a", "b", "c");
        ConnectedStreams<Integer, String> connected = nums.connect(text);
        DataStream<String> result = connected.flatMap(new CoFlatMapFunction<Integer, String, String>() {
            @Override
            public void flatMap1(Integer value, Collector<String> out) {
                out.collect(String.valueOf(value*2));
                out.collect(String.valueOf(value*3));
            }
            @Override
            public void flatMap2(String value, Collector<String> out) {
                out.collect("hello " + value);
                out.collect("hi " + value);
            }
        });
        result.print();
        env.execute("CoFlatMap example");
    }
```

### Split/Select

DataStream → SplitStream

根据条件将一个流分成多个流，示例代码如下：

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<Long> data = env.generateSequence(0, 10);
        SplitStream<Long> split = data.split((OutputSelector<Long>) value -> {
            List<String> output = new ArrayList<>();
            if (value % 2 == 0) {
                output.add("even");
            } else {
                output.add("odd");
            }
            return output;
        });
        split.select("odd").print();
        env.execute("Flink SplitStream Example");
    }
```

`select()`用于从SplitStream中选择一个或者多个数据流。

```scala
split.select("odd").print();
```

### SideOutput

**注意：在Flink 1.12 及之后的版本中，SplitStream 已经被弃用并移除，一般推荐使用 Side Outputs（侧输出流）来替代 Split和Select**

示例代码如下：

```java
private static final OutputTag<String> evenOutput = new OutputTag<String>("even"){};
private static final OutputTag<String> oddOutput = new OutputTag<String>("odd"){};

public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> input = env.fromElements("1", "2", "3", "4", "5");
        SingleOutputStreamOperator<String> processed = input.process(new ProcessFunction<String, String>() {
            @Override
            public void processElement(String value, Context ctx, Collector<String> out){
                int i = Integer.parseInt(value);
                if (i % 2 == 0) {
                    ctx.output(evenOutput, value);
                } else {
                    ctx.output(oddOutput, value);
                }
            }
        });
        DataStream<String> evenStream = processed.getSideOutput(evenOutput);
        DataStream<String> oddStream = processed.getSideOutput(oddOutput);
        evenStream.print("even");
        oddStream.print("odd");
        env.execute("Side Output Example");
    }
```

### Iterate

DataStream → IterativeStream → DataStream

Iterate算子提供了对数据流迭代的支持

一个数据集通过迭代运算符被划分为两部分：“反馈”部分（feedback）和“输出”部分（output）。反馈部分被反馈到迭代头（iteration head），从而形成下一次迭代。输出部分则构成该迭代的结果：

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Long> input = env.fromElements(10L);
        // 定义迭代流，最大迭代10次
        IterativeStream<Long> iteration = input.iterate(10000L);
        // 定义迭代逻辑
        DataStream<Long> minusOne = iteration.map((MapFunction<Long, Long>) value -> value - 1);
        // 定义反馈流（满足条件继续迭代）和输出流（不满足条件的结果）
        DataStream<Long> stillGreaterThanZero = minusOne.filter(value -> value > 0).setParallelism(1);;
        DataStream<Long> lessThanZero = minusOne.filter(value -> value <= 0);
        // 关闭迭代，定义反馈流
        iteration.closeWith(stillGreaterThanZero);
        // 打印结果
        lessThanZero.print();
        env.execute("Iterative Stream Example");
    }
```

### 普通函数 & 富函数

Apache Flink 中有两种类型的函数： 「**普通函数（Regular Functions）**」和 「**富函数（Rich Functions）**」。主要区别在于富函数相比普通函数提供了更多生命周期方法和上下文信息。

- **普通函数**：这些函数只需要覆盖一个或几个特定方法，如 `MapFunction` 需要实现 `map()` 方法。它们没有生命周期方法，也不能访问执行环境的上下文。
- **富函数**：除了覆盖特定函数外，富函数还提供了对 Flink API 更多的控制和操作，包括：
  - 生命周期管理：可以覆盖 `open()` 和 `close()` 方法以便在函数启动前和关闭后做一些设置或清理工作。
  - 获取运行时上下文信息：例如，通过 `getRuntimeContext()` 方法获取并行任务的信息，如当前子任务的索引等。
  - 状态管理和容错：可以定义和使用托管状态（Managed State），这在构建容错系统时非常重要。

简而言之，如果你需要在函数中使用 Flink 的高级功能，如状态管理或访问运行时上下文，则需要使用富函数。如果不需要这些功能，使用普通函数即可。

| 普通函数类      | 富函数类            |
| :-------------- | ------------------- |
| MapFunction     | RichMapFunction     |
| FlatMapFunction | RichFlatMapFunction |
| FilterFunction  | RichFilterFunction  |
| ......          | ......              |

普通函数：

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        List<String> words = Arrays.asList("hello", "world", "flink", "hello", "world");
        env.fromCollection(words)
                .map(new MapFunction<String, Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> map(String value) {
                        return new Tuple2<>(value, 1);
                    }
                })
                .keyBy(0)
                .sum(1)
                .print();

        env.execute("Word Count Example");
    }
```

富函数：

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        List<String> words = Arrays.asList("hello", "world", "flink", "hello", "world");
        env.fromCollection(words)
                .map(new RichMapFunction<String, Tuple2<String, Integer>>() {
                    @Override
                    public void open(Configuration parameters) throws Exception {
                        super.open(parameters);
                        // 可以在这里设置相关的配置或者资源，如数据库连接等
                    }
                    @Override
                    public Tuple2<String, Integer> map(String value) throws Exception {
                        return new Tuple2<>(value, 1);
                    }
                    @Override
                    public void close() throws Exception {
                        super.close();
                        // 可以在这里完成资源的清理工作
                    }
                })
                .keyBy(0)
                .sum(1)
                .print();
        env.execute("Word Count Example");
    }
```

### ProcessFunction（处理函数）

ProcessFunction属于低层次的API，在类继承关系上属于富函数。

我们前面讲的`map`、`filter`、`flatMap`等算子都是基于这层封装出来的。

越低层次的API，功能越强大，用户能够获取的信息越多，比如可以拿到元素状态信息、事件时间、设置定时器等

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> dataStream = env.socketTextStream("localhost", 9999)
                .map(new MapFunction<String, Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> map(String value) {
                        return new Tuple2<>(value, 1);
                    }
                })
                .keyBy(0)
                .process(new AlertFunction());
        dataStream.print();
        env.execute("Process Function Example");
    }

    public static class AlertFunction extends KeyedProcessFunction<Tuple, Tuple2<String, Integer>, String> {
        private transient ValueState<Integer> countState;
        @Override
        public void open(Configuration config) {
            ValueStateDescriptor<Integer> descriptor =
                    new ValueStateDescriptor<>(
                            "countState", // state name
                            TypeInformation.of(new TypeHint<Integer>() {}), // type information
                            0); // default value
            countState = getRuntimeContext().getState(descriptor);
        }
        @Override
        public void processElement(Tuple2<String, Integer> value, Context ctx, Collector<String> out) throws Exception {
            Integer currentCount = countState.value();
            currentCount += 1;
            countState.update(currentCount);
            if (currentCount >= 3) {
                out.collect("Warning! The key '" + value.f0 + "' has been seen " + currentCount + " times.");
            }
        }
    }
```

这里，我们创建一个名为`AlertFunction`的处理函数类，并继承`KeyedProcessFunction`。其中，`ValueState`用于保存状态信息，每个键会有其自己的状态实例。当计数达到或超过三次时，该系统将发出警告。这个例子主要展示了处理函数与其他运算符相比的两个优点：访问键控状态和生命周期管理方法（例如`open()`）。

注意：上述示例假设你已经在本地的9999端口上设置了一个socket服务器，用于流式传输文本数据。如果没有，你需要替换这部分以适应你的输入源。

## Sink

在Flink中，"Sink"是数据流计算的最后一步。它代表了一个输出端点，在那里计算结果被发送或存储。换句话说，Sink是数据流处理过程中的结束节点，负责将处理后的数据输出到外部系统，如数据库、文件、消息队列等。

Flink内置了大量Sink，可以将Flink处理后的数据输出到HDFS、kafka、Redis、ES、MySQL等。

### Redis Sink

Flink处理的数据可以存储到Redis中，以便实时查询。

首先，需要导入Flink和Redis的连接器依赖：

```xml
<!-- Flink Redis connector -->
        <dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_${scala.binary.version}</artifactId>
            <version>1.1.0</version>
        </dependency>
```

下面的代码展示了"Word Count"(词频统计)操作，并将结果存储到Redis数据库中：

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> text = env.fromElements(
                "Hello World",
                "Hello Flink",
                "Hello Java");
        DataStream<Tuple2<String, Integer>> counts =
                text.flatMap(new Tokenizer())
                        .keyBy(value -> value.f0)
                        .sum(1);
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("localhost").build();
        counts.addSink(new RedisSink<>(conf, new RedisExampleMapper()));
        env.execute("Word Count Example");
    }

    public static final class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            String[] words = value.toLowerCase().split("\\W+");

            for (String word : words) {
                if (word.length() > 0) {
                    out.collect(new Tuple2<>(word, 1));
                }
            }
        }
    }

    public static final class RedisExampleMapper implements RedisMapper<Tuple2<String, Integer>> {
        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET);
        }
        @Override
        public String getKeyFromData(Tuple2<String, Integer> data) {
            return data.f0;
        }
        @Override
        public String getValueFromData(Tuple2<String, Integer> data) {
            return data.f1.toString();
        }
    }
```

### Kafka Sink

处理结果写入到kafka topic中，Flink也是支持的，需要添加连接器依赖，跟读取kafka数据用的连接器依赖相同，之前添加过就不需要再添加了。

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.12</artifactId>
    <version>1.13.6</version>
</dependency>
```

还是用上面词频统计的例子：
```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> text = env.fromElements(
                "Hello World",
                "Hello Flink",
                "Hello Java");
        DataStream<Tuple2<String, Integer>> counts =
                text.flatMap(new Tokenizer())
                        .keyBy(value -> value.f0)
                        .sum(1);
        // Define Kafka properties
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        // Write the data stream to Kafka
        counts.map(new MapFunction<Tuple2<String,Integer>, String>() {
                    @Override
                    public String map(Tuple2<String,Integer> value) throws Exception {
                        return value.f0 + "," + value.f1.toString();
                    }
                })
                .addSink(new FlinkKafkaProducer<>("my-topic", new SimpleStringSchema(), properties));
        env.execute("Word Count Example");
    }
```

### MySQL Sink

Flink处理结果写入到MySQL中，这并不是Flink默认支持的，需要添加MySQL的驱动依赖：

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>
```

因为不是内嵌支持的，所以需要基于SinkFunction自定义Sink。

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> text = env.fromElements(
                "Hello World",
                "Hello Flink",
                "Hello Java");
        DataStream<Tuple2<String, Integer>> counts =
                text.flatMap(new Tokenizer())
                        .keyBy(value -> value.f0)
                        .sum(1);
        // Transform the Tuple2<String, Integer> to a format acceptable by MySQL
        DataStream<String> mysqlData = counts.map(new MapFunction<Tuple2<String, Integer>, String>() {
            @Override
            public String map(Tuple2<String, Integer> value) throws Exception {
                return "'" + value.f0 + "'," + value.f1.toString();
            }
        });
        // Write the data stream to MySQL
        mysqlData.addSink(new MySqlSink());
        env.execute("Word Count Example");
    }

    public static class MySqlSink implements SinkFunction<String> {
        private Connection connection;
        private PreparedStatement preparedStatement;

        @Override
        public void invoke(String value, Context context) throws Exception {
            if(connection == null) {
                connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "username", "password");
                preparedStatement = connection.prepareStatement("INSERT INTO my_table(word, count) VALUES("+ value +")");
            }
            preparedStatement.executeUpdate();
        }
    }
}
```

### HBase Sink

需要导入HBase的依赖：

```xml
        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.5.2</version>
        </dependency>
```

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> text = env.fromElements(
                "Hello World",
                "Hello Flink",
                "Hello Java");
        DataStream<Tuple2<String, Integer>> counts =
                text.flatMap(new Tokenizer())
                        .keyBy(value -> value.f0)
                        .sum(1);
        counts.addSink(new HBaseSink());
        env.execute("Word Count Example");
    }

    public static final class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            String[] words = value.toLowerCase().split("\\W+");
            for (String word : words) {
                if (word.length() > 0) {
                    out.collect(new Tuple2<>(word, 1));
                }
            }
        }
    }

    public static class HBaseSink extends RichSinkFunction<Tuple2<String, Integer>> {
        private org.apache.hadoop.conf.Configuration config;
        private org.apache.hadoop.hbase.client.Connection connection;
        private Table table;
        @Override
        public void invoke(Tuple2<String, Integer> value, Context context) throws IOException {
            Put put = new Put(Bytes.toBytes(value.f0));
            put.addColumn(Bytes.toBytes("cf"), Bytes.toBytes("count"), Bytes.toBytes(value.f1));
            table.put(put);
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            config = HBaseConfiguration.create();
            config.set("hbase.zookeeper.quorum", "localhost");
            config.set("hbase.zookeeper.property.clientPort", "2181");
            connection = ConnectionFactory.createConnection(config);
            table = connection.getTable(TableName.valueOf("my-table"));
        }

        @Override
        public void close() throws Exception {
            table.close();
            connection.close();
        }
    }
```

`HBaseSink`类是`RichSinkFunction`的实现，用于将结果写入HBase数据库。在`invoke`方法中，它将接收到的每个二元组（单词和计数）写入HBase。在`open`方法中，它创建了与HBase的连接，并指定了要写入的表。在`close`方法中，它关闭了与HBase的连接和表。

## 分区策略

在 Apache Flink 中，分区（Partitioning）是将数据流按照一定的规则划分成多个子数据流或分片，以便在不同的并行任务或算子中并行处理数据。分区是实现并行计算和数据流处理的基础机制。Flink 的分区决定了数据在作业中的流动方式，以及在并行任务之间如何分配和处理数据。

在 Flink 中，数据流可以看作是一个有向图，图中的节点代表算子（Operators），边代表数据流（Data Streams）。数据从源算子流向下游算子，这些算子可能并行地处理输入数据，而分区就是决定数据如何从一个算子传递到另一个算子的机制。

下面介绍Flink中常用的几种分区策略。

### shuffle  

场景：增大分区、提高并行度，解决数据倾斜。

DataStream → DataStream

**分区元素随机均匀分发到下游分区，网络开销比较大**

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Long> stream = env.fromSequence(1, 10).setParallelism(1);
        System.out.println(stream.getParallelism());
        stream.shuffle().print();
        env.execute();
    }
```

输出结果：上游数据比较随意地分发到下游

```scala
1> 7
7> 1
2> 8
4> 5
8> 3
1> 9
8> 4
8> 10
6> 2
6> 6
```

### rebalance 

场景：增大分区、提高并行度，解决数据倾斜

DataStream → DataStream

**轮询分区元素，均匀的将元素分发到下游分区，下游每个分区的数据比较均匀，在发生数据倾斜时非常有用，网络开销比较大**

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Long> stream = env.fromSequence(1, 10).setParallelism(1);
        System.out.println(stream.getParallelism());
        stream.rebalance().print();
        env.execute();
    }
```

输出：上游数据比较均匀的分发到下游

```scala
2> 2
1> 1
8> 8
5> 5
7> 7
4> 4
3> 3
6> 6
1> 9
2> 10
```

### rescale

场景：减少分区，防止发生大量的网络传输，不会发生全量的重分区

DataStream → DataStream

通过轮询分区元素，将一个元素集合从上游分区发送给下游分区，发送单位是集合，而不是一个个元素

**和其他重分区策略（如 rebalance、forward、broadcast 等）不同的是，rescale 在运行时不会改变并行度，而且它只在本地（同一个 TaskManager 内）进行数据交换，所以它比其他重分区策略更加高效**

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<String> dataStream = env.fromElements("1", "2", "3", "4", "5");

        // 使用MapFunction将元素转换为整数类型
        DataStream<Integer> intStream = dataStream.map(new MapFunction<String, Integer>() {
            @Override
            public Integer map(String value) {
                return Integer.parseInt(value);
            }
        });
        // 使用rescale()进行重分区
        DataStream<Integer> rescaledStream = intStream.rescale();
        rescaledStream.print();
        env.execute("Rescale Example");
    }
```

在这个例子中，我们创建了一个字符串类型的DataStream然后通过`map()`将每一个元素转换为整数。然后，我们对结果DataStream应用`rescale()`操作来重分区数据。

值得注意的是，`rescale()`的实际影响取决于你的并行度和集群环境，如果不同的并行实例都在同一台机器上，或者并行度只有1，那么可能不会看到`rescale()`的效果。而在大规模并行处理的情况下，使用`rescale()`操作可以提高数据处理的效率。

此外，我们不能直接在打印结果中看到`rescale`的影响，因为它改变的是内部数据分布和处理方式，而不是输出的结果。如果想观察`rescale`的作用，需要通过Flink的Web UI或者日志来查看任务执行情况，如数据流的分布、各个子任务的运行状态等信息。

### broadcast

场景：需要使用映射表、并且映射表会经常发生变动的场景

DataStream → DataStream

上游中每一个元素内容广播到下游每一个分区中

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Integer> dataStream = env.fromElements(1, 2, 3, 4, 5);
        DataStream<String> broadcastStream = env.fromElements("2", "4");
        MapStateDescriptor<String, String> descriptor = new MapStateDescriptor<>(
                "RulesBroadcastState",
                BasicTypeInfo.STRING_TYPE_INFO,
                BasicTypeInfo.STRING_TYPE_INFO);
        BroadcastStream<String> broadcastData = broadcastStream.broadcast(descriptor);
        dataStream.connect(broadcastData)
                .process(new BroadcastProcessFunction<Integer, String, String>() {
                    @Override
                    public void processElement(Integer value, ReadOnlyContext ctx, Collector<String> out) throws Exception {
                        if (ctx.getBroadcastState(descriptor).contains(String.valueOf(value))) {
                            out.collect("Value " + value + " matches with a broadcasted rule");
                        }
                    }
                    @Override
                    public void processBroadcastElement(String rule, Context ctx, Collector<String> out) throws Exception {
                        ctx.getBroadcastState(descriptor).put(rule, rule);
                    }
                }).print();
        env.execute("Broadcast State Example");
    }
```

上述代码首先定义了一个主流和一个要广播的流。然后，我们创建了一个`MapStateDescriptor`，用于存储广播数据。接着，我们将广播流转换为`BroadcastStream`。

最后，我们使用`connect()`方法连接主流和广播流，并执行`process()`方法。在这个`process()`方法中，我们定义了两个处理函数：`processElement()`和`processBroadcastElement()`。`processElement()`用于处理主流中的每个元素，并检查该元素是否存在于广播状态中。如果是，则输出一个字符串，表明匹配成功。而`processBroadcastElement()`则用于处理广播流中的每个元素，并将其添加到广播状态中。

注意：在分布式计算环境中，每个并行实例都会接收广播流中的所有元素。因此，广播状态对于所有的并行实例都是一样的。不过，在Flink 1.13版本中，广播状态尚未在故障恢复中提供完全的保障。所以在事件出现故障时，广播状态可能会丢失数据。

### global

场景：并行度降为1

DataStream → DataStream

在 Apache Flink 中，Global 分区策略意味着所有数据都被发送到下游算子的同一个分区中。这种情况下，下游算子只有一个任务处理全部数据。这是一种特殊的分区策略，只有在下游算子能够很快地处理所有数据，或者需要全局排序或全局聚合时才会使用。

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 创建一个从1到100的数字流
        DataStream<Long> numberStream = env.fromSequence(1, 100);
        // 对流应用 map function
        DataStream<Long> result = numberStream.global()
                .map(new MapFunction<Long, Long>() {
                    @Override
                    public Long map(Long value) {
                        System.out.println("Processing " + value);
                        return value * 2;
                    }
                });
        result.print();
        env.execute("Global Partition Example");
    }
```

以上代码创建了一个顺序生成 1-100 的数字流，并应用了 Global Partition，然后对每个数字进行乘2的操作。实际运行此代码时，你会观察到所有的数字都由同一任务处理，打印出来的处理顺序是连续的。这就是 Global Partition 的作用：所有数据都被发送到下游算子的同一实例进行处理。

需要注意的是，此示例只是为了演示 Global Partition 的工作原理，实际上并不推荐在负载均衡很重要的应用场景中使用这种分区策略，因为它可能导致严重的性能问题。

### forward

场景：一对一的数据分发,默认的分区策略，数据在各个算子之间不会重新分配。map、flatMap、filter 等都是这种分区策略

DataStream → DataStream

上游分区数据分发到下游对应分区中

partition1->partition1；partition2->partition2

注意：必须保证上下游分区数（并行度）一致，不然会有如下异常:

```scala
Forward partitioning does not allow change of parallelism. Upstream operation: Source: Socket Stream-1 parallelism: 1, downstream operation: Map-3 parallelism: 8 You must use another partitioning strategy, such as broadcast, rebalance, shuffle or global.
```

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Integer> dataStream = env.fromElements(1, 2, 3, 4, 5, 6, 7, 8, 9, 10).setParallelism(1);
        DataStream<Integer> forwardStream = dataStream.forward().map(new MapFunction<Integer, Integer>() {
            @Override
            public Integer map(Integer value) throws Exception {
                return value * value;
            }
        }).setParallelism(1);
        forwardStream.print();
        env.execute("Flink Forward Example");
    }
```

此代码首先创建一个从1到10的数据流。然后，它使用 Forward 策略将这个数据流送入一个 MapFunction 中，该函数将每个数字平方。然后，它打印出结果。注意：以上代码中的forward调用实际上并没有改变任何分区策略，因为forward是默认分区策略。这里添加forward调用主要是为了说明其存在和使用方法。

### keyBy

场景：与业务场景匹配

DataStream → DataStream

根据上游分区元素的Hash值与下游分区数取模计算出，将当前元素分发到下游哪一个分区

```scala
MathUtils.murmurHash(keyHash)（每个元素的Hash值） % maxParallelism（下游分区数）
```

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Tuple2<Integer, Integer>> dataStream = env.fromElements(
                new Tuple2<>(1, 3),
                new Tuple2<>(1, 5),
                new Tuple2<>(2, 4),
                new Tuple2<>(2, 6),
                new Tuple2<>(3, 7)
        );
        // 使用 keyBy 对流进行分区操作
        DataStream<Tuple2<Integer, Integer>> keyedStream = dataStream
                .keyBy(0) // 根据元组的第一个字段进行分区
                .sum(1);  // 对每个键对应的第二个字段求和
        keyedStream.print();
        env.execute("KeyBy example");
    }
```

以上程序首先创建了一个包含五个元组的流，然后使用 `keyBy` 方法根据元组的第一个字段进行分区，并对每个键对应的第二个字段求和。执行结果中，每个键的值集合都被映射成了一个新的元组，其第一个字段是键，第二个字段是相应的和。

注意：在以上代码中，`keyBy(0)` 表示根据元组的第一个字段（索引从0开始）进行分区操作。另外，无论什么情况，都需要确保你的 Flink 集群是正常运行的，否则程序可能无法执行成功。

### PartitionCustom

DataStream → DataStream

通过自定义的分区器，来决定元素是如何从上游分区分发到下游分区

```java
public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Integer> data = env.fromElements(1,2,3,4,5,6,7,8,9,10);
        // 使用自定义分区器进行分区
        data.partitionCustom(new MyPartitioner(), i -> i).print();
        env.execute("Custom partition example");
    }

    public static class MyPartitioner implements Partitioner<Integer> {
        @Override
        public int partition(Integer key, int numPartitions) {
            return key % numPartitions;
        }
    }
```

这个程序将创建一个数据流，其中包含从1到10的整数。然后，它使用了一个自定义的分区器`MyPartitioner`来对这个数据流进行分区。这个分区器根据元素的值对`numPartitions`取模来决定数据去到哪个分区。

由于篇幅限制，我们将在此结束本篇内容。稍微整理一下，下篇马上发。

希望这篇文章能够给你带来收获和思考，如果有收获，希望能不吝点个赞或者再看，谢谢。
