[TOC]

## 内存碎片

### 内存碎片如何产生的？
Redis内部有自己的内存分配器，**默认是jemalloc**，为了提高内存使用的效率，来对内存的申请和释放进行管理。
而内存分配器按照固定大小分配内存，并不是完全按照程序申请的内存大小来进行分配。
比如程序申请一个20字节的内存，内存分配器会分配一个32字节的内存空间，这么做是为了减少分配次数。redis会申请不同大小的内存空间来存储不同业务不同类型的数据，由于内存按照固定大小分配且会比实际申请的内存要大一些，这个过程中会产生内存碎片。
**举个例子**：
我们用高铁车厢说明，假设一个车厢的座位总共有60个，现在已经卖 了57张票，需要三张连在一起的票，但发现买不到了，只好换一趟车。我们可以把这些分散的空座位叫作**车厢座位碎片**。

内存碎片类似上面高铁座位的例子。**虽然操作系统的剩余空间总量足够，但申请一块连续地址空间N字节时，剩余内存空间中没有大小为N字节的连续空间，那么这些剩余空间就是内存碎片。**

**Redis的这种机制，提高了内存的使用率，但是会使Redis中有部分自己没在用，却不释放的内存，导致了内存碎片的发生。**
### 内存分配器
在编译时指定的Redis使用的内存分配器，可以是libc、jemalloc、tcmalloc，**默认是jemalloc**。

jemalloc在64位系统中，将内存空间划分为小、大、巨大三个范围；每个范围内又划分了许多小的内存块单位；存储数据的时候，会选择大小最合适的内存块进行存储。

jemalloc划分的内存单元如下图所示：
![img](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScOvgfyIKZkWibs3GVCFiauNTxddQ77Lpiaiccbyh3JkCr0uIYeicX7Nq1YCACcGvKoibgUL38LUd4T2o8Yg/0?wx_fmt=jpeg)

也就是说Redis是以指定大小的块为单位进行连续内存分配的，而不是按需分配的。Redis 会根据申请的内存最接近的固定值分配相应大小的空间。

这就像你有不同的箱子，为了装东西，你需要找一个体积最接近的箱子来装。但是装进去后，你发现还有空间可以放一些小东西，就无需再找箱子了。但是，这种分配空间的方式会带来一定程度的内存碎片。我们可以把固定大小的划分空间看成不同体积的箱子，每种箱子里的空间不同程度上都会有剩余。这些剩余的空间就是内存碎片。

### 怎么看是否有内存碎片？
我们登陆到Redis服务器上，执行以下命令：

```shell
redis> info memory
```
我们会看到这些信息：
![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOvgfyIKZkWibs3GVCFiauNTx7GEY7UibZ5q9kKoXnF6sDJFEq0InV8LDBn27eYiadukicuKp1TCbTfEWQ/0?wx_fmt=png)
指标**mem_fragmentation_ratio:1.86** 表示当前的内存碎片率。

```shell
mem_fragmentation_ratio = used_memory_rss / used_memory
```
used_memory_rss：是Redis向操作系统申请的内存。
used_memory：是Redis中的数据占用的内存。

所以，mem_fragmentation_ratio=1应该是最理想的情况
### 碎片率的意义？
mem_fragmentation_ratio的不同值，说明不同的情况。

 - 大于1：说明内存有碎片，一般在1到1.5之间是正常的。 
 - 大于1.5：说明内存碎片率比较大，需要考虑是否要进行内存碎片清理，要引起重视。
 - 小于1：说明已经开始使用交换内存，也就是使用硬盘了，正常的内存不够用了，需要考虑是否要进行内存的扩容，使用swap是相当影响性能的。

### 清理内存碎片
### 低于4.0-RC3版本的Redis
如果你的Redis版本是4.0-RC3以下的，Redis服务器重启后，Redis会将没用的内存归还给操作系统，碎片率会降下来。

### 高于4.0-RC3版本的Redis
Redis4.0-RC3版本开始，可以在不重启的情况下，线上整理内存碎片。
自动碎片清理，只要设置了如下的配置，内存就会自动清理了。
```shell
redis> config set activedefrag yes 
```
<font color=OrangeRed>自动清理内存碎片的功能需要该Redis的内存分配器是jemalloc时才能启用。</font>

启用后需要同时满足下面2个参数的设置条件时才会触发自动清理

```shell
active-defrag-ignore-bytes 100mb    # 默认100MB,表示内存碎片空间达到100MB时
active-defrag-threshold-lower 10    # 默认10，表示内存碎片空间占OS分配给redis的物理内存空间的比例达到10%时
```
**redis是单进程模型，内存碎片自动清理是通过主线程操作的，也会消耗一定的CPU资源**。为了避免自动清理降低Redis的处理性能，如下两个参数可以控制清理动作消耗的CPU时间比例的上下限。

```shell
active-defrag-cycle-min 5 : 默认5，表示自动清理过程所用 CPU 时间的比例不低于5%，保证清理能正常开展；
active-defrag-cycle-max 75: 默认75，表示自动清理过程所用 CPU 时间的比例不高于 75%，一旦超过，就停止清理，从而避免在清理时，大量的内存拷贝阻塞 Redis，导致响应延迟升高。
```

如果你对自动清理的效果不满意，可以使用如下命令，直接试下手动碎片清理：
```shell
redis > memory purge
```
需要注意的是，该清理命令也只当Redis的内存分配器是jemalloc时才能生效
```shell
#碎片整理总开关
activedefrag yes
 
#当碎片达到 100mb 时，开启内存碎片整理
active-defrag-ignore-bytes 100mb
 
#当碎片超过 10% 时，开启内存碎片整理
active-defrag-threshold-lower 10
 
#内存碎片超过 100%，则尽最大努力整理
active-defrag-threshold-upper 100
 
#内存自动整理占用资源最小百分比
active-defrag-cycle-min 5
 
#内存自动整理占用资源最大百分比
active-defrag-cycle-max 50
```
## Pipeline管道
### 为什么需要Pipeline
Redis客户端执行一条命令分4个过程：

```java
发送命令－〉命令排队－〉命令执行－〉返回结果
```
这个过程称为**Round Trip Time(简称RTT, 往返时间)**，mget mset有效节约了RTT，但大部分命令（如hgetall，并没有mhgetall）不支持批量操作，需要消耗N次RTT ，这个时候需要Pipeline来解决这个问题

Pipeline 模式则是将执行的命令写入到缓冲中，最后由exec命令一次性发送给Redis执行返回。

**1、未使用Pipeline执行N条命令**
![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOvgfyIKZkWibs3GVCFiauNTxKtyichpCF40qbpx79VUPFkibpbqX9SNVia44dKGj8w7e9TYMGpPqcnIWQ/0?wx_fmt=png)
**2、使用了Pipeline执行N条命令**
![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOvgfyIKZkWibs3GVCFiauNTxmjxtqlJ8vDXWNllebC7eI99hmPibmdLdnricLkb3jXibeIM2ibbib0qs5Hw/0?wx_fmt=png)

### 原生批命令(mset, mget)与Pipeline对比

 - 原生批命令是原子性，Pipeline是非原子性
 - 原生批命令一命令多个key, 但Pipeline支持多命令，Pipeline 不支持事务，因为命令是一条一条执行的。
 - 原生批命令是服务端实现，而Pipeline需要服务端与客户端共同完成

### Pipeline的优缺点

 - pipeline 每批打包的命令不能过多，因为 Pipeline 方式打包命令再发送，那么 Redis 必须在处理完所有命令前先缓存起所有命令的处理结果。这样就有一个内存的消耗。
 - Pipeline 操作是非原子性的，如果要求原子性的，不推荐使用 Pipeline
 - 使用Pipeline组装的命令个数不能太多，不然数据量过大，增加客户端的等待时间，还可能造成网络阻塞，可以将大量命令的拆分多个小的pipeline命令完成。

### 一些疑问
 **Pipeline 执行多少命令合适？**

> 根据官方的解释，推荐是以 10k 每批 (注意：这个是一个参考值，请根据自身实际业务情况调整)。

**Pipeline 批量执行的时候，是否对Redis进行了锁定，导致其他应用无法再进行读写？**

> Redis 采用多路I/O复用模型，非阻塞IO，所以Pipeline批量写入的时候，一定范围内不影响其他的读操作。

在编码时请注意，Pipeline 期间将“独占”链接，此期间将不能进行非“管道”类型的其他操作，直到 Pipeline 关闭；如果你的 Pipeline 的指令集很庞大，为了不干扰链接中的其他操作，你可以为 Pipeline 操作新建 Client 链接，让 Pipeline 和其他正常操作分离在2个 client 中。

### 相关代码

```java
    @Test
    void pipeline() {
        List<Object> result = redisTemplate.executePipelined((RedisCallback<String>) connection -> {
            for (int i = 0; i < 100; i++) {
                redisTemplate.opsForValue().set("pipel:" + i, i);
            }
            return null;
        });
    }
```
![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOvgfyIKZkWibs3GVCFiauNTx3IXRXAYpZ22Z9m4JsBzdEE7JRteZQiat0yv7wFPdibzxaMbdJAgHtRrA/0?wx_fmt=png)



------

本篇文章就到这里，感谢阅读，如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。
![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)