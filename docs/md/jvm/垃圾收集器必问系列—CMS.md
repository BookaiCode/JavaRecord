> 应该相信，自己是生活的战胜者。——雨果

[TOC]

纵观全书《深入理解JVM虚拟机》第三版，在垃圾回收器这一篇章，对于CMS的笔墨是非常多的。CMS收集器是HotSpot虚拟机追求低停顿的第一次成功尝试，CMS 可以说是垃圾回收器的一个里程碑，其开启了 GC 回收器关注 GC 停顿时间的历史。

![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScNSWyYdcwhyuGyFAHA8HjTSicRd14q6xUND92YbCXkQEPMItxxxAjnstkFN4aWZ9mloQMYickSv4IQQ/0?wx_fmt=jpeg)

## CMS简介

**CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器，**启用参数：`-XX:+UseConMarkSweepGC`，使用的是**标记-清除算法**。在这之前的垃圾回收器，要么就是串行垃圾回收方式，要么就是关注系统吞吐量。而 CMS 垃圾回收器的出现，则打破了这个尴尬的局面。**JDK9之后使用CMS垃圾收集器后，默认年轻代就为ParNew收集器，并且不可更改，同时JDK9之后被标记为不推荐使用，JDK14就被删除了。**

CMS 垃圾回收器之所以能够实现对 GC 停顿时间的控制，其关键是**三色标记算法**（不了解的同学去翻我之前写的文章），通过三色标记算法，实现了垃圾回收线程与用户线程并发执行，从而极大地降低了系统响应时间。

## 运作过程

CMS整个过程分为四个步骤，包括：

1. 初始标记（CMS initial mark）
2. 并发标记（CMS concurrent mark）
3. 重新标记（CMS remark）
4. 并发清除（CMS concurrent sweep）

**其中初始标记、重新标记这两个步骤仍然需要“Stop The World”**。

- 初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快；
- 并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程，**这个过程是四个阶段中耗时最长的，但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行；**
- 而重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，**这里使用的是增量更新**，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短；
- 最后是并发清除阶段，清理删除掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的。

**由于在整个过程中耗时最长的并发标记和并发清除阶段中**，垃圾收集器线程都可以与用户线程一起工作，所以从总体上来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNSWyYdcwhyuGyFAHA8HjTSDCu3CePySY7t7ic41QzWtUDrXZEYfyRxKzL6K9uiaibLgPxwuOCgIscWw/0?wx_fmt=png)

## CMS的缺陷

**CMS有三个最大的缺点**

![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScNSWyYdcwhyuGyFAHA8HjTSK2A60QHmdkDO6mCvabgnxVYbtp6CwAJ4iceicv1YelBhEoD8lcYWbicfQ/0?wx_fmt=jpeg)

### 处理器资源敏感

CMS收集器是比较消耗CPU资源的，对处理器资源是比较敏感的。**在并发阶段，它不会导致用户线程停顿，但会占用了一部分线程（或者说处理器的计算能力）而导致应用程序变慢，降低总吞吐量。**

低延迟和高吞吐，往往无法同时达成，低延迟有时是牺牲高吞吐换得的，有得必有失。

**CMS默认启动的回收线程数是（处理器核心数量+3）/4**，也就是说，如果处理器核心数在四个或以上，并发回收时垃圾收集线程只占用不超过25%的处理器运算资源。但是当处理器核心数量不足四个时，CMS对用户程序的影响就可能变得很大。如果应用本来的处理器负载就很高，还要分出一半的运算能力去执行收集器线程，就可能导致用户程序的执行速度忽然大幅降低。

为了缓解这种情况，虚拟机提供了一种称为“增量式并发收集器”（Incremental Concurrent Mark Sweep/i-CMS）的CMS收集器变种，**在并发标记、清理的时候让收集器线程、用户线程交替运行，尽量减少垃圾收集线程的独占资源的时间，这样整个垃圾收集的过程会更长，但对用户程序的影响就会显得较少一些**，直观感受是速度变慢的时间更多了，但速度下降幅度就没有那么明显。实践证明增量式的CMS收集器效果很一般，从JDK 7开始，i-CMS模式已经被声明为“deprecated”，即已过时不再提倡用户使用，到 JDK 9发布后i-CMS模式被完全废弃。

### 无法处理“浮动垃圾”

由于CMS收集器无法处理“**浮动垃圾**”（Floating Garbage），有可能出现“Con-current Mode Failure”失败进而导致另一次完全“Stop The World”的Full GC的产生。

**在CMS的并发标记和并发清理阶段，用户线程是还在继续运行的，程序在运行自然就还会伴随有新的垃圾对象不断产生，但这一部分垃圾对象是出现在标记过程结束以后，CMS无法在当次收集中处理掉它们，只好留待下一次垃圾收集时再清理掉。这一部分垃圾就称为“浮动垃圾”。**

![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScNSWyYdcwhyuGyFAHA8HjTSXoBeQ2CXibziak9ArfibSKXCRXM14nL24yczSeiaia2hgZTmVAzPal2t7hA/0?wx_fmt=jpeg)

由于在垃圾收集阶段用户线程还需要持续运行，那就还需要预留足够内存空间提供给用户线程使用，因此CMS收集器不能像其他收集器那样等待到老年代几乎完全被填满了再进行收集，**在JDK5的默认设置下，CMS收集器当老年代使用了68%的空间后就会被激活。到了JDK 6时，CMS收集器的启动阈值就已经默认提升至92%**，我们可以通过 `-XX:CMSInitiatingOccupancyFraction` 参数自行调节。

但这又会更容易面临另一种风险：要是CMS运行期间预留的内存无法满足程序分配新对象的需要，就会出现一次“并发失败”（Concurrent Mode Failure），这时候虚拟机将不得不启动后备预案：**冻结用户线程的执行，临时启用Serial Old收集器来重新进行老年代的垃圾收集**，但这样停顿时间就很长了。

### 内存碎片

CMS是一款基于“标记-清除”算法实现的收集器，在垃圾收集算法的时候我们说过，标记-清除会产生`内存碎片`。空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很多剩余空间，但就是无法找到足够大的连续空间来分配当前对象，而不得不提前触发一次Full GC的情况。

为了解决这个问题，CMS收集器提供了一个`-XX：+UseCMS-CompactAtFullCollection`开关参数（默认是开启的，此参数从JDK 9开始废弃），**用于在CMS收集器不得不进行Full GC时开启内存碎片的合并整理过程**，由于这个内存整理必须移动存活对象。这样空间碎片问题是解决了，但停顿时间又会变长，因此虚拟机设计者们还提供了另外一个参数`-XX：CMSFullGCsBefore-Compaction`（此参数从JDK 9开始废弃），这个参数的作用是要求CMS收集器在执行过若干次（数量由参数值决定）不整理空间的Full GC之后，下一次进入Full GC前会先进行碎片整理（默认值为0，表示每次进入Full GC时都进行碎片整理）。

------

如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。

![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScPibyOvOuNiasKa7qicaZgo5DIcDAickDKoU6KZUmLyibpnRc6ibzTxT9WAnkfPhFcq6iamGRo2ITZlPPczA/0?wx_fmt=jpeg)
