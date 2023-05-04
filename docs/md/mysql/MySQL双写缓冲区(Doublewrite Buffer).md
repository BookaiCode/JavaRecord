[TOC]

## 摘要

InnoDB是MySQL中一种常用的事务性存储引擎，它具有很多优秀的特性。其中，Doublewrite Buffer是InnoDB的一个重要特性之一，本文将介绍Doublewrite Buffer的原理和应用。

## 为什么需要Doublewrite Buffer

我们常见的服务器一般都是Linux操作系统，Linux文件系统页（OS Page）的大小默认是4KB。而MySQL的页（Page）大小默认是16KB。

可以使用如下命令查看MySQL的Page大小：

```
SHOW VARIABLES LIKE 'innodb_page_size';
```

 一般情况下，其余程序因为需要跟操作系统交互，它们的页（Page）都会大于等于操作系统的页大小，为整数倍。比如，Oracle的Page大小为8KB。

MySQL程序是跑在Linux操作系统上的，需要跟操作系统交互，所以MySQL中一页数据刷到磁盘，要写4个文件系统里的页。

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOZqcx7RQ7OHo75SQpNXEukibOIibftFdJO3NibUiaosVOLSs0Nu1Grd14iaaNPmZoriaydwCFDnic3Z0dUg/640?wx_fmt=png)

**需要注意的是，这个操作并非原子操作，比如我操作系统写到第二个页的时候，Linux机器断电了，这时候就会出现问题了。造成”页数据损坏“。并且这种”页数据损坏“靠 redo日志是无法修复的**。

**重做日志中记录的是对页的物理操作，而不是页面的全量记录，而如果发生partial page write（部分页写入）问题时，出现问题的是未修改过的数据，此时重做日志(Redo Log)无能为力。写doublewrite buffer成功了，这个问题就不用担心了**。

Doublewrite Buffer的出现就是为了解决上面的这种情况，虽然名字带了Buffer，但实际上Doublewrite Buffer是**内存+磁盘**的结构。

Doublewrite Buffer是一种特殊文件flush技术，带给InnoDB存储引擎的是数据页的可靠性。它的作用是，在把页写到数据文件之前，InnoDB先把它们写到一个叫doublewrite buffer（双写缓冲区）的共享表空间内，在写doublewrite buffer完成后，InnoDB才会把页写到数据文件的适当的位置。如果在写页的过程中发生意外崩溃，InnoDB在稍后的恢复过程中在doublewrite buffer中找到完好的page副本用于恢复。

## Doublewrite Buffer原理

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOZqcx7RQ7OHo75SQpNXEuk0vQSvaAFscwJrFyzEyBzCO4mpr8icY58ne39Y3WTUYBW17QNrOULE6w/640?wx_fmt=png)

**如上图所示，当有页数据要刷盘时：**

1. 页数据先通过memcpy函数拷贝至内存中的Doublewrite Buffer中；

2. Doublewrite Buffer的内存里的数据页，会fsync刷到Doublewrite Buffer的磁盘上，分两次写入磁盘共享表空间中(连续存储，顺序写，性能很高)，每次写1MB；

3. Doublewrite Buffer的内存里的数据页，再刷到数据磁盘存储.ibd文件上（离散写）；

   

Doublewrite Buffer内存结构由128个页（Page）构成，大小是2MB。

Doublewrite Buffer磁盘结构在系统表空间上是128个页（2个区，extend1和extend2），大小是2MB。

**如果操作系统在将页写入磁盘的过程中发生了崩溃，在恢复过程中，InnoDB存储引擎可以从共享表空间中的Double write中找到该页的一个副本，将其复制到表空间文件，再应用重做日志**。

MySQL会检查double writer的数据的完整性，如果不完整直接丢弃double write buffer内容，重新执行那条redo log，如果double write buffer的数据是完整的，用double writer buffer的数据更新该数据页，跳过该redo log。

所以在正常的情况下，MySQL写数据页时，会写两遍到磁盘上，第一遍是写到doublewrite buffer，第二遍是写到真正的数据文件中，**这就是“Doublewrite”的由来。**

在数据库异常关闭的情况下启动时，都会做数据库恢复（redo）操作，恢复的过程中，数据库都会检查页面是不是合法（校验等等），如果发现一个页面校验结果不一致，则此时会用到双写这个功能。

我们可以通过如下命令来监控Doublewrite Buffer工作负载：

```
mysql> show global status like '%dblwr%';
```

## Doublewrite Buffer相关参数

- **innodb_doublewrite**：Doublewrite Buffer是否启用开关，默认是开启状态，InnoDB将所有数据存储两次，首先到双写缓冲区，然后到实际数据文件。
- **Innodb_dblwr_pages_written**：记录写入到DWB中的页数量。
- **Innodb_dblwr_writes**：记录DWB写操作的次数。

## 总结

InnoDB Doublewrite Buffer是InnoDB的一个重要特性，用于保证MySQL数据的可靠性和一致性。它的实现原理是通过将要写入磁盘的数据先写入到Doublewrite Buffer中的内存缓存区域，然后再写入到磁盘的两个不同位置，来避免由于磁盘损坏等因素导致数据丢失或不一致的问题。Doublewrite Buffer对于保证MySQL数据的安全性和一致性具有重要意义。

