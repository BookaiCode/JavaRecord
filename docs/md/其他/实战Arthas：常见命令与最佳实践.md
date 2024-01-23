本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

[toc]
当涉及到 Java 应用程序的诊断和调优时，Arthas 是一款备受推崇的开源工具，无论是线上问题的定位，还是实时性能监控和分析，Arthas 都能为您提供强大的支持。

本文将介绍 Arthas 的常用命令和使用技巧，帮助您更好地利用该工具进行故障排查和性能优化。

## 前言

在开始本文之前，先推荐两个东西：

一个是 Arthas 官网：https://arthas.aliyun.com/doc/，官方文档对 Arthas 的每个命令都做出了介绍和解释，并且还有在线教程，方便大家学习和熟悉命令。

![](https://img-blog.csdnimg.cn/img_convert/699b8877cd01afb0c3f5ca3c8a92900b.png)

另外还有一个向大家推荐的是一款名为 **Arthas Idea** 的 IDEA 插件。

这是一款能快速生成 Arthas命令的插件，可快速生成可用于该类或该方法的 Arthas 命令，大大提高排查问题的效率。

![](https://img-blog.csdnimg.cn/img_convert/8267b4aea9cb65985c5d0305bed6f777.png)

## 常用命令

尽管 Arthas 命令众多，但在实际使用中我们只需聚焦于那些常用命令。本文旨在重点介绍这些常用命令，并提供使用技巧和最佳实践，帮助您更好地运用 Arthas。

### 类命令

#### getstatic

查看类的静态属性。推荐直接使用 `ognl` 命令，更加灵活。

```Bash
# getstatic class_name field_name
getstatic demo.MathGame random

# 如果该静态属性是一个复杂对象，还可以支持在该属性上通过 ognl 表达式进行遍历，过滤，访问对象的内部属性等操作。
# 例如，假设 n 是一个 Map，Map 的 Key 是一个 Enum，我们想过滤出 Map 中 Key 为某个 Enum 的值，可以写如下命令
getstatic com.alibaba.arthas.Test n 'entrySet().iterator.{? #this.key.name()=="STOP"}'
```

#### jad

反编译指定已加载类的源码。`jad` 只能反编译单个类，如需批量下载指定包的目录的 class 字节码请使用 `dump` 命令。

比如我们想知道自己提交的代码是否生效了，这种场景`jad` 命令就特别有用。

```Bash
# 反编译 java.lang.String
jad java.lang.String
# 默认情况下，反编译结果里会带有 ClassLoader 信息，通过 --source-only 选项，可以只打印源代码。方便和 mc/retransform 命令结合使用。
jad --source-only java.lang.String
# 反编译指定的函数
jad java.lang.String substring
# 当有多个 ClassLoader 都加载了这个类时，jad 命令会输出对应 ClassLoader 实例的 hashcode
# 然后你只需要重新执行 jad 命令，并使用参数 -c <hashcode> 就可以反编译指定 ClassLoader 加载的那个类了
jad org.apache.log4j.Logger -c 69dcaba4
```

#### retransform

加载外部的 `.class` 文件，retransform jvm 已加载的类。

```Bash
# 结合 jad/mc 命令使用，jad 命令反编译，然后可以用其它编译器，比如 vim 来修改源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
# mc 命令来内存编译修改过的代码
mc /tmp/UserController.java -d /tmp
# 用 retransform 命令加载新的字节码
retransform /tmp/com/example/demo/arthas/user/UserController.class
```

加载指定的 .class 文件，然后解析出 class name，再 retransform jvm 中已加载的对应的类。每加载一个 .class 文件，则会记录一个 retransform entry。

如果多次执行 retransform 加载同一个 class 文件，则会有多条 retransform entry。

```bash
# 查看 retransform entry
retransform -l
# 删除指定 retransform entry，需要指定 id：
retransform -d 1
# 删除所有 retransform entry
retransform --deleteAll
# 显式触发 retransform
retransform --classPattern demo.MathGame
```

如果对某个类执行 retransform 之后，想消除 retransform 的影响，则需要：

- 删除这个类对应的 retransform entry。
- 重新显式触发 retransform。

retransform 的限制：

- 不允许新增加 field/method。
- 正在跑的函数，没有退出不能生效。

使用 `mc` 命令来编译 `jad` 的反编译的代码有可能失败。可以在本地修改代码，编译好后再上传到服务器上。有的服务器不允许直接上传文件，可以使用 `base64` 命令来绕过。

1. 在本地先转换 `.class` 文件为 base64，再保存为 result.txt。

```Bash
 base64  -i /tmp/test.class -o /tmp/result.txt
```

2. 到服务器上，新建并编辑 result.txt，复制本地的内容，粘贴再保存。

```Bash
vim  /tmp/result.txt
```

3. 把服务器上的 `result.txt `还原为`.class`。

```Bash
base64 -d /tmp/result.txt > /tmp/test.class
```

4. 用 md5 命令计算哈希值，校验是否一致。

```Bash
md5sum  /tmp/test.class
```

### 监测排查命令

监测排查命令是 Arthas 中最常用的命令。

> 请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 `stop` 或将增强过的类执行 `reset` 命令。

#### monitor

方法执行监控。可对方法的调用次数，成功次数，失败次数等维度进行统计。

```Bash
# -b：计算条件表达式过滤统计结果(方法执行完毕之前)，默认是方法执行之后过滤
# -c：统计周期，默认值为 120 秒
# params[0] <= 2：过滤条件，方法第一个参数小于等于2
monitor -b -c 5 com.test.testes.MathGame primeFactors "params[0] <= 2"
```

#### stack

输出当前方法被调用的调用路径。

很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 `stack` 命令。

```Bash
# -n：执行次数
stack demo.MathGame primeFactors  -n  2
```

#### thread

查看当前线程信息，查看线程的堆栈。

```Bash
# 没有参数时，默认按照 CPU 增量时间降序排列，只显示第一页数据
# -i 1000： 统计最近 1000ms 内的线程 CPU 时间
# -n 3： 展示当前最忙的前 N 个线程并打印堆栈
# --state WAITING：查看指定状态的线程
thread

# 显示指定线程的运行堆栈
thread id

# 找出当前阻塞其他线程的线程，注意，目前只支持找出 synchronized 关键字阻塞住的线程， 如果是 java.util.concurrent.Lock 目前还不支持。
thread -b
```

输出：

- Internal 表示为 JVM 内部线程，参考 `dashboard` 命令的介绍。 
- cpuUsage 为采样间隔时间内线程的 CPU 使用率，与 `dashboard` 命令的数据一致。 
- deltaTime 为采样间隔时间内线程的增量 CPU 时间，小于 1ms 时被取整显示为 0ms。
- time 为线程运行总 CPU 时间。

#### trace

方法内部调用路径，并输出方法路径上的每个节点上耗时。

`trace` 命令在定位性能问题的时候特别有用。

```Bash
# -n 1：限制匹配次数
# --skipJDKMethod false：默认情况下，trace 不会包含 jdk 里的函数调用，如果希望 trace jdk 里的函数，需要显式设置
# --exclude-class-pattern ：排除掉指定的类
trace javax.servlet.Filter * -n 1 --skipJDKMethod false --exclude-class-pattern com.demo.TestFilter
# 正则表达式匹配路径上的多个类和函数，达到多层 trace 的效果
trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
```

动态 tradce参考：[https://arthas.aliyun.com/doc/trace.html#动态-trace](https://arthas.aliyun.com/doc/trace.html#动态-trace)

#### tt

方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测。

说明：

- tt 命令的实现是：把函数的入参/返回值等，保存到一个`Map<Integer, TimeFragment>`里，默认的大小是 100。
- tt 相关功能在使用完之后，需要手动释放内存，否则长时间可能导致 OOM。退出 arthas 不会自动清除 tt 的缓存 map。
- 需要强调的是，tt 命令是将当前环境的对象引用保存起来，但仅仅也只能保存一个引用而已。如果方法内部对入参进行了变更，或者返回的对象经过了后续的处理，那么在  tt  查看的时候将无法看到当时最准确的值。这也是为什么 watch 命令存在的意义。

```Bash
# -l：显示tt记录
tt -l

# -s：检索tt记录，比如：-s 'method.name=="primeFactors"'
tt -s 'method.name=="primeFactors"'

# -t：这个参数的表明希望记录下类 *Test 的 print 方法的每次执行情况。
tt -t

# 查看具体调用信息
tt -i 1003

# -w：--watch-express 观察时空隧道使用 ognl 表达式
tt -w '@demo.MathGame@random.nextInt(100)'

# 重做一次调用，当我们对程序做出了修改之后，希望再次调用观测结果，此时你需要 -p 参数
# --replay-times：指定调用次数
# --replay-interval：指定多次调用间隔(单位 ms, 默认 1000ms)
tt -i 1004 -p

# 通过索引删除指定的 tt 记录
tt -d 1001

# 清除所有的 tt 记录
tt --delete-all
```

Spring MVC里获取对于的 bean：

```Bash
# 获取Spring Context里的bean
tt -n 1 -t org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter invokeHandlerMethod
tt -i 1000 -w 'target.getApplicationContext().getBean("helloWorldService").getHelloMessage()'
```

#### watch

函数执行数据观测，通过编写 OGNL 表达式进行对应变量的查看。

- watch 命令定义了 4 个观察事件点，即 `-b` 函数调用前，`-e` 函数异常后，`-s` 函数返回后，`-f` 函数结束后。
- 4 个观察事件点 `-b`、`-e`、`-s` 默认关闭，`-f` 默认打开，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出。
- 这里要注意`函数入参`和`函数出参`的区别，有可能在中间被修改导致前后不一致，除了 `-b` 事件点 `params` 代表函数入参外，其余事件都代表函数出参。
- 当使用 `-b` 时，由于观察事件点是在函数调用前，此时返回值或异常均不存在。
- 在 watch 命令的结果里，会打印出`location`信息。`location`有三种可能值：`AtEnter`，`AtExit`，`AtExceptionExit`。对应函数入口，函数正常 return，函数抛出异常。

```Bash
 # -x表示遍历深度，可以调整来打印具体的参数和结果内容，默认值是 1。
 # -x最大值是 4，防止展开结果占用太多内存。用户可以在ognl表达式里指定更具体的 field。
 watch demo.MathGame primeFactors -x 3
 
 # 可以使用ognl表达式进行条件过滤
 watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0" "#cost>200"
 
 # 可以使用 target.field_name 访问当前对象的某个属性
 watch demo.MathGame primeFactors 'target.illegalArgumentCount'
 
 #  watch 构造函数
 watch demo.MathGame <init> '{params,returnObj,throwExp}' -v
 
 # watch内部类
 watch OuterClass$InnerClass
```

### JVM命令

#### heapdump

生成堆转储文件。

```Bash
# dump 到指定文件
heapdump arthas-output/dump.hprof
# 只 dump live 对象
heapdump --live /tmp/dump.hprof
```

#### jfr

Java Flight Recorder (JFR) 是一种用于收集有关正在运行的 Java 应用程序的诊断和分析数据的工具。

它集成到 Java 虚拟机 (JVM) 中，几乎不会造成性能开销，因此即使在负载较重的生产环境中也可以使用。

```Bash
# 启动 JFR 记录
jfr start

# 启动 jfr 记录，指定记录名，记录持续时间，记录文件保存路径。
# --duration  JFR 记录持续时间，支持单位配置，60s, 2m, 5h, 3d，不带单位就是秒，默认一直记录。
jfr start -n myRecording --duration 60s -f /tmp/myRecording.jfr

# 查看所有 JFR 记录信息
jfr status

# 查看指定记录 id 的记录信息
jfr status -r 1

# 查看指定状态的记录信息
jfr status --state closed

# jfr dump 会输出从开始到运行该命令这段时间内的记录到 JFR 文件，且不会停止 jfr 的记录
# 生成的结果可以用支持 jfr 格式的工具来查看。比如：JDK Mission Control ： https://github.com/openjdk/jmc
jfr dump -r 1 -f /tmp/myRecording1.jfr

# 停止 jfr 记录
jfr stop -r 1
```

#### memory

查看 JVM 内存信息。

输出如下：

```bash
Memory                           used      total      max        usage
heap                             32M       256M       4096M      0.79%
g1_eden_space                    11M       68M        -1         16.18%
g1_old_gen                       17M       184M       4096M      0.43%
g1_survivor_space                4M        4M         -1         100.00%
nonheap                          35M       39M        -1         89.55%
codeheap_'non-nmethods'          1M        2M         5M         20.53%
metaspace                        26M       27M        -1         96.88%
codeheap_'profiled_nmethods'     4M        4M         117M       3.57%
compressed_class_space           2M        3M         1024M      0.29%
codeheap_'non-profiled_nmethods' 685K      2496K      120032K    0.57%
mapped                           0K        0K         -          0.00%
direct                           48M       48M        -          100.00%
```

#### dashboard

当前系统的实时数据面板，按 `ctrl+c` 退出。

```Bash
# i：刷新实时数据的时间间隔 (ms)，默认 5000m
# n：刷新实时数据的次数
dashboard -i 5000 -n 3
```

显示 ID 为 -1 的是 JVM的内部线程，JVM 内部线程包括下面几种：

- JIT 编译线程：如  `C1 CompilerThread0`, `C2 CompilerThread0`。
- GC 线程：如 `GC Thread0`, `G1 Young RemSet Sampling。`
- 其它内部线程：如 `VM Periodic Task Thread`, `VM Thread`, `Service Thread。`

当 JVM 堆(heap)/元数据(metaspace) 空间不足或 OOM 时， GC 线程的 CPU 占用率会明显高于其他的线程。

#### classloader

`classloader` 命令将 JVM 中所有的 classloader 的信息统计出来，并可以展示继承树，urls 等。

```Bash
# 按类加载类型查看统计信息
classloader

# 按类加载实例查看统计信息
classloader -l

# 查看 ClassLoader 的继承树
classloader -t

# 查看 URLClassLoader 实际的 urls，通过 classloader -l 可以获取到哈希值
classloader -c 3d4eac69
```

#### logger

查看 logger 信息，更新 logger level。

```Bash
# 查看所有 logger 信息
logger

# 查看指定名字的 logger 信息
logger -n org.springframework.web

# 更新 logger level
logger --name ROOT --level debug
```

#### sc

查看 JVM 已加载的类信息。

```bash
# 模糊搜索
sc demo.*

# 打印类的详细信息
sc -d demo.MathGame

# 打印出类的 Field 信息
sc -d -f demo.MathGame
```



#### mbean

查看 Mbean 的信息。

所谓 MBean 就是托管的Java对象，类似于 JavaBeans 组件，遵循 JMX(Java Management Extensions，即Java管理扩展) 规范中规定的设计模式。

MBean可以表示任何需要管理的资源。

```Bash
# 列出所有 Mbean 的名称
mbean

# 查看 Mbean 的元信息
mbean -m java.lang:type=Threading

# 查看 mbean 属性信息，mbean 的 name 支持通配符匹配 mbean java.lang:type=Th*
mbean java.lang:type=Threading

#通配符匹配特定的属性字段
mbean java.lang:type=Threading *Count

# 实时监控使用-i，使用-n命令执行命令的次数（默认为 100 次）
mbean -i 1000 -n 50 java.lang:type=Threading *Count
```

比如我们可以使用 `mbean` 命令来查看 Druid 连接池的属性：

```Bash
mbean com.alibaba.druid.pool:name=dataSource,type=DruidDataSource
```

#### profiler

生成应用热点的火焰图。本质上是通过不断的采样，然后把收集到的采样结果生成火焰图。

```Bash
# 启动 profiler
# 生成的是 cpu 的火焰图，即 event 为cpu。可以用--event参数来指定。
profiler start --event cpu

# 获取已采集的 sample 的数量
profiler getSamples

# 查看 profiler 状态
profiler status

# 停止 profiler，生成结果，结果文件是html格式，也可以用--format参数指定
profiler stop --format html

# 恢复采样，start和resume的区别是：start是新开始采样，resume会保留上次stop时的数据。
profiler resume

# 配置 include/exclude 来过滤数据
profiler start --include 'java/*' --include 'demo/*' --exclude '*Unsafe.park*'

# 生成 jfr 格式结果
profiler start --file /tmp/test.jfr
```

#### vmoption

查看，更新 VM 诊断相关的参数。

```Bash
# 查看所有的 option
vmoption

# 查看指定的 option
vmoption PrintGC

# 更新指定的 option
vmoption PrintGC true
```

#### vmtool

`vmtool`  利用 JVMTI 接口，实现查询内存对象，强制 GC 等功能。

```Bash
# --limit：可以限制返回值数量，避免获取超大数据时对 JVM 造成压力。默认值是 10
# --action：执行的动作
vmtool --action getInstances --className java.lang.String --limit 10

#强制 GC
vmtool --action forceGc

# interrupt 指定线程
vmtool --action interruptThread -t 1
```

### 特殊命令

可以使用 `-v`  查看观察匹配表达式的执行结果

#### ognl

执行 ognl 表达式，是Arthas中最为灵活的命令。

```bash
# -c：执行表达式的 ClassLoader 的 hashcode，默认值是 SystemClassLoader
# --classLoaderClass：指定执行表达式的 ClassLoader 的 class name
# -x：结果对象的展开层次，默认值 1
ognl --classLoaderClass org.springframework.boot.loader.LaunchedURLClassLoader @org.springframework.boot.SpringApplication@logger
```

有关 ognl 语法介绍，放在下文。

#### options

全局开关，慎用！

```Bash
# 查看所有的 options
options

# 设置指定的 option，默认情况下json-format为 false，如果希望watch/tt等命令结果以 json 格式输出，则可以设置json-format为 true。
options json-format true

# 默认情况下，watch/trace/tt/trace/monitor等命令不支持java.* package 下的类。可以设置unsafe为 true，则可以增强。
options unsafe true

# Arthas 默认启用strict模式，在ognl表达式里，禁止更新对象的 Property 或者调用setter函数
# 用户如果确定要在ognl表达式里更新对象，可以执行options strict false，关闭strict模式。
options strict false
```

### 帮助命令

#### help

查看命令帮助信息，可以查看当前 arthas 版本支持的指令，或者查看具体指令的使用说明。

```Bash
help dashboard 
或者
dashboard  -help
```

#### history

打印命令历史。

```Bash
#查看最近执行的3条指令
history 3

#清空指令
history -c
```

#### cls

清空当前屏幕区域。

#### quit

仅退出当前的连接，Attach 到目标进程上的 arthas 还会继续运行，端口会保持开放，下次连接时可以直接连接上。或者直接按 `Q` 也能退出。

#### stop

完全退出 arthas，stop 时会重置所有增强过的类。

#### reset

重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端 `stop` 时会重置所有增强过的类。

```Bash
# 还原指定类
reset Test

# 还原所有类
reset
```

## Advice

无论是匹配表达式也好、观察表达式也罢，他们核心判断变量都是围绕着一个 Arthas 中的通用通知对象 `Advice` 进行。

它的简略代码结构如下：

```java
public class Advice {
    private final ClassLoader loader;
    private final Class<?> clazz;
    private final ArthasMethod method;
    private final Object target;
    private final Object[] params;
    private final Object returnObj;
    private final Throwable throwExp;
    private final boolean isBefore;
    private final boolean isThrow;
    private final boolean isReturn;

    // getter/setter
}
```

这里列一个表格来说明不同变量的含义：

|    变量名 | 变量解释                                                     |
| --------: | :----------------------------------------------------------- |
|    loader | 本次调用类所在的 ClassLoader                                 |
|     clazz | 本次调用类的 Class 引用                                      |
|    method | 本次调用方法反射引用                                         |
|    target | 本次调用类的实例                                             |
|    params | 本次调用参数列表，这是一个数组，如果方法是无参方法则为空数组 |
| returnObj | 本次调用返回的对象。当且仅当 `isReturn==true` 成立时候有效，表明方法调用是以正常返回的方式结束。如果当前方法无返回值 `void`，则值为 null |
|  throwExp | 本次调用抛出的异常。当且仅当 `isThrow==true` 成立时有效，表明方法调用是以抛出异常的方式结束。 |
|  isBefore | 辅助判断标记，当前的通知节点有可能是在方法一开始就通知，此时 `isBefore==true` 成立，同时 `isThrow==false` 和 `isReturn==false`，因为在方法刚开始时，还无法确定方法调用将会如何结束。 |
|   isThrow | 辅助判断标记，当前的方法调用以抛异常的形式结束。             |
|  isReturn | 辅助判断标记，当前的方法调用以正常返回的形式结束。           |

所有变量都可以在表达式中直接使用，如果在表达式中编写了不符合 OGNL 脚本语法或者引入了不在表格中的变量，则退出命令的执行。

用户可以根据当前的异常信息修正 `条件表达式` 或 `观察表达式`。

## 快捷键

```Bash
# 自动补全，命令后敲 - 或 -- ，然后按 tab 键，可以展示出此命令具体的选项
Tab

# 退出当前连接
Q

# 后台异步命令相关快捷键
ctrl + c: 终止当前命令
ctrl + z: 挂起当前命令，后续可以 bg/fg 重新支持此命令，或 kill 掉
ctrl + a: 回到行首
ctrl + e: 回到行尾
```

## OGNL

OGNL(Object-Graph Navigation Language)是一种表达式语言(EL)，简单来说就是一种简化了的Java属性的取值语言，Arthas使用它做表达式过滤。

OGNL 表达式官网：[https://commons.apache.org/dormant/commons-ognl/language-guide.htm](https://commons.apache.org/dormant/commons-ognl/language-guide.html)

**变量引用** 

OGNL支持用变量来保存中间结果，并在后面的代码中再次引用它。

OGNL中的所有变量，对整个表达式都是全局可见的，引用变量的方法是在变量名之前加上 `#` 号，OGNL会将当前对象保存在 "this" 变量中，这个变量也可以像其他任何变量一样引用，用 `#this` 表示当前对象。

这里列举一些常用的语法：

```bash
# 调用静态属性
'@全路径类目@静态属性名'

# 调用静态方法
'@全路径类目@静态方法名("参数")'

# 过滤，判断，筛选
'params[0]'：查看第一个参数
'params[0].size()'：查看第一个参数的size
'params[0]=="xyz"'：判断字符串相等
'params[0]==123456789L'：判断long型
'params[0].{ #this.name }'：将结果按name属性映射
'params[0].{? #this.name == null }'：按条件过滤
'params[0].{? #this.age > 10 }.size()'：过滤后统计
'params[0].{^ #this.name != null}'：选择第一个满足条件
'params[0].{$ #this.name != null}'：选择最后一个满足条件
'params[0].{? #this.age > 10 }.size().(#this > 20 ? #this - 10 : #this + 10)'：子表达式求值
'name in { null,"Untitled" }':这条语句判断name是否等于null或者 Untitled


# 构造对象
'#{ "foo" : "foo value", "bar" : "bar value" }'：构造map参数
'#@java.util.LinkedHashMap@{ "foo" : "foo value", "bar" : "bar value" }'：构造特定类型map
'new com.Test("xiaoming",18)'：构造方法，new 全路径类名()
'new int[] { 1, 2, 3 }'：创建数组并初始化


# 访问对象
'@com.Test@getPerson("xiaoming",18).name':访问复杂对象属性，用 .属性名 访问属性
'@com.Test@getChilds({"xiaoming"})[0]':访问List或者数组类型，用 [索引] 访问
'@com.Test@getMap()["xiaoming"]': 访问Map对象，用 ["key"]，key要用双引号

# 临时变量
'#value1=@com.Test@getPerson("xiaoming",18), #value2=@com.Test@setPerson(#value1) ,{#value1,#value2}': 方法A的返回值当做方法B的入参
'#value1=@System@getProperty("java.home"), #value2=@System@getProperty("java.runtime.name"), {#value1, #value2}'：执行多行表达式，赋值给临时变量，返回一个List
'#obj=new com.User("xiaoming",18),@com.Test@inputObj(#obj)'：先用构造函数构造一个对象，然后把这个对象当做入参传入
```

## 实用功能

### 管道

Arthas 命令后可接 `grep`  进行进一步筛选或操作，比如：

```Bash
classloader -a | grep "String"
```

### 后台异步执行

当需要排查一个问题，但是这个问题的出现时间不能确定，那我们就可以把检测命令挂在后台运行，并将保存到输出日志。

```Bash
# 比如希望执行后台执行 trace 命令，那么调用下面命令
trace Test t &

# 如果希望查看当前有哪些 arthas 任务在执行，可以执行 jobs 命令
jobs

# 可通过 > 或者 >> 将任务输出结果输出到指定的文件中，可以和 & 一起使用，实现 arthas 命令的后台异步任务。比如：
trace Test t >> test.out &

#异步执行的命令，如果希望停止，可执行kill命令
kill <job-id>

# 当任务正在前台执行，可以执行 ‘ctrl + z’ 将任务暂停。通过jbos查看任务状态将会变为 Stopped，再通过bg <job-id>或者fg <job-id>可让任务重新开始执行
# 可以把对应的任务转到前台继续执行。在前台执行时，无法在 console 中执行其他命令
fg <job-id>
# 可以把对应的任务在后台继续执行
bg <job-id>
```

## 实用技巧

```bash
#  获取接口的响应时间
watch org.springframework.web.servlet.DispatcherServlet doService '{params[0].getRequestURI()+" "+ #cost}'  -n 5  -x 3 '#cost>100'  -f

# 获取指定header 头的信息，比如这里 获取 trace-id
 watch org.springframework.web.servlet.DispatcherServlet doService '{params[0].getRequestURI()+"  header="+params[1].getHeaders("trace-id")}'  -n 10  -x 3 -f
 
# 查看执行的SQL，下面两个都可以
watch java.sql.Connection prepareStatement '{params,throwExp}'  -n 5  -x 3 
watch org.apache.ibatis.mapping.BoundSql getSql '{params,returnObj,throwExp}'  -n 5  -x 3 


# 调用任意bean中的方法
# 1.先获取 classLoaderHash
sc -d com.alibaba.dubbo.config.spring.extension.SpringExtensionFactor

# 2.ognl 调用对应 bean 的方法，把 34f5090e 替换为对于的 classLoaderHash
ognl  -c 34f5090e '#context=@com.alibaba.dubbo.config.spring.extension.SpringExtensionFactory@contexts.iterator.next,#context.getBean("userServiceImpl").find("小明")'

# 当传参是复杂对象时
ognl -c 34f5090e '#context=@com.alibaba.dubbo.config.spring.extension.SpringExtensionFactory@contexts.iterator.next,#data=new Children(), #query=new User(),#query.setChildren(#data),#query.setRequestId("1"), #data.setName("小明"),#context.getBean("userServiceImpl").find(#query)'

# vmtool 命令提供了更简单的语法，也可以调用任意bean中的方法
vmtool --action getInstances  --className org.springframework.context.ApplicationContext --express 'instances[0].getBean("userServiceImpl").find("小明")'


# 动态修改 bean 属性值
# 本质原理就是先获取 bean 实例，通过反射去修改对应属性值
ognl -c 34f5090e org.ClassLoader
'#context=@com.alibaba.dubbo.config.spring.extension.SpringExtensionFactory@contexts.iterator.next, #instence=#context.getBean("userServiceImpl"),#fieldObj=@com.User@class.getDeclaredField("age"),#fieldObj.setAccessible(true), #fieldObj.set(#instence,18)'

# 除了 ognl 也可以通过 vmtool 去获取 bean
vmtool --action getInstances  --className org.springframework.context.ApplicationContext --express 'instances[0].getBean("userServiceImpl")'
```

Arthas 的强大之处确实令人惊叹！本文希望能够启发您去探索更多关于 Arthas 的用法和功能，相信它会为您的开发工作带来很大的帮助和便利。