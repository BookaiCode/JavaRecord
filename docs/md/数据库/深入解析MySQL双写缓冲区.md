在数据库系统的世界中，保障数据的完整性和稳定性是至关重要的任务。为了实现这一目标，MySQL内部使用了许多精巧而高效的机制。

InnoDB是MySQL中一种常用的事务性存储引擎，它具有很多优秀的特性。其中，Doublewrite Buffer是InnoDB的一个重要特性之一，本文将介绍Doublewrite Buffer的原理和应用，帮助读者深入理解其如何提高MySQL的数据可靠性并防止可能的数据损坏。

## 为什么需要Doublewrite Buffer

我们常见的服务器一般都是Linux操作系统，Linux文件系统页（OS Page）的大小默认是4KB。而MySQL的页（Page）大小默认是16KB。

可以使用如下命令查看MySQL的Page大小：

```sql
SHOW VARIABLES LIKE 'innodb_page_size';
```

 一般情况下，其余程序因为需要跟操作系统交互，所以它们的页（Page）大小都为操作系统页大小的整数倍。比如，Oracle的Page大小为8KB。

MySQL程序是跑在Linux操作系统上的，理所当然要跟操作系统交互，所以MySQL中一页数据刷到磁盘，要写4个文件系统里的页。

如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOZqcx7RQ7OHo75SQpNXEukibOIibftFdJO3NibUiaosVOLSs0Nu1Grd14iaaNPmZoriaydwCFDnic3Z0dUg/640)

**需要注意的是，这个刷页的操作并非原子操作，比如我操作系统写到第二个页的时候，Linux机器断电了，这时候就会出现问题了。造成「页数据损坏」。并且这种页数据损坏靠 redo日志是无法修复的。**

redo重做日志中记录的是对页的物理操作，而不是页面的全量记录，当发生「**Partial Page Write（部分页写入）**」问题时，出现问题的是未修改过的数据，此时redo日志无能为力。

Doublewrite Buffer的出现就是为了解决上面的这种情况，给InnoDB存储引擎提供了数据页的可靠性，虽然名字带了Buffer，但实际上Doublewrite Buffer是「**内存+磁盘**」的结构。

**内存结构**：Doublewrite Buffer内存结构由128个页（Page）构成，大小是2MB。

**磁盘结构**：Doublewrite Buffer磁盘结构在系统表空间上是128个页（2个区，extend1和extend2），大小是2MB。

Doublewrite Buffer的原理是，再把数据页写到数据文件之前，InnoDB先把它们写到一个叫「**doublewrite buffer（双写缓冲区）**」的共享表空间内，在写doublewrite buffer完成后，InnoDB才会把页写到数据文件适当的位置。

如果在写页的过程中发生意外崩溃，InnoDB会在doublewrite buffer中找到完好的page副本用于恢复。

## Doublewrite Buffer原理

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOZqcx7RQ7OHo75SQpNXEuk0vQSvaAFscwJrFyzEyBzCO4mpr8icY58ne39Y3WTUYBW17QNrOULE6w/640)

如上图所示，当有数据页要刷盘时：

1. 页数据先通过`memcpy`函数拷贝至内存中的Doublewrite Buffer中。

2. Doublewrite Buffer的内存里的数据页，会`fsync`刷到Doublewrite Buffer的磁盘上，分两次写入磁盘共享表空间中(连续存储，顺序写，性能很高)，每次写1MB。

3. Doublewrite Buffer的内存里的数据页，再刷到数据磁盘存储.ibd文件上（离散写）。


**如果操作系统在将页写入磁盘的过程中发生了崩溃，在恢复过程中，InnoDB存储引擎可以从共享表空间中的Double write中找到该页的一个副本，将其复制到表空间文件，再应用redo日志**。

所以在正常的情况下，MySQL写数据页时，会写两遍到磁盘上，第一遍是写到doublewrite buffer，第二遍是写到真正的数据文件中，这便是「Doublewrite」的由来。

我们可以通过如下命令来监控Doublewrite Buffer工作负载，该命令用于显示有关双写缓冲区(doublewrite buffer)的统计信息。'%dblwr%' 是一个通配符，匹配所有包含 'dblwr' 的状态变量。

```sql
show global status like '%dblwr%';
```

这个命令可能会产生如下格式的输出：

```sql
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| Innodb_dblwr_writes    | 1000  |
| Innodb_dblwr_pages_written | 8000  |
+------------------------+-------+
```

## Doublewrite Buffer和redo log

在MySQL的InnoDB存储引擎中，Redo log和Doublewrite Buffer共同工作以确保数据的持久性和恢复能力。

1. 当有一个DML（如INSERT、UPDATE）操作发生时， InnoDB会首先将这个操作写入redo log（内存）。这些日志被称为未检查点（uncheckpointed）的redo日志。
2. 然后，在修改内存中相应的数据页之前，需要将这些更改记录在磁盘上。但是直接把这些修改的页写到其真正的位置可能会因发生故障导致页部分更新，从而导致数据不一致。因此，InnoDB的做法是先将这些修改的页按顺序写入doublewrite buffer。这就是为什么叫做 "doublewrite" —— 数据实际上被写了两次，先在doublewrite buffer，然后在它们真正的位置。
3. 一旦这些页被安全地写入doublewrite buffer，它们就可以按原始的顺序写回到文件系统中。即使这个过程在写回数据时发生故障，我们仍然可以从doublewrite buffer中恢复数据。
4. 最后，当事务提交时，相关联的redo log会被写入磁盘。这样即使系统崩溃，redo log也可以用来重播（replay）事务并恢复数据库。

在系统恢复期间，InnoDB会检查doublewrite buffer，并尝试从中恢复损坏的数据页。如果doublewrite buffer中的数据是完整的，那么InnoDB就会用doublewrite buffer中的数据来更新损坏的页。否则，如果doublewrite buffer中的数据不完整，InnoDB也有可能丢弃buffer内容，重新执行那条redo log以尝试恢复数据。

所以，Redo log和Doublewrite Buffer的协作可以确保数据的完整性和持久性。如果在写入过程中发生故障，我们可以从doublewrite buffer中恢复数据，并通过redo log来进行事务的重播。

## Doublewrite Buffer相关参数

以下是一些与Doublewrite Buffer相关的参数及其含义：

- `innodb_doublewrite`： 这个参数用于启用或禁用双写缓冲区。设置为1时启用，设置为0时禁用， 默认值为1。
- `innodb_doublewrite_files`： 这个参数定义了多少个双写文件被使用。默认值为2，有效范围从2到127。
- `innodb_doublewrite_dir`： 这个参数指定了存储双写缓冲文件的目录的路径。默认为空字符串，表示将文件存储在数据目录中。
- `innodb_doublewrite_batch_size`: 这个参数定义了每次批处理操作写入的字节数。默认值为0，表示InnoDB会选择最佳的批量大小。
- `innodb_doublewrite_pages`：这个参数定义了每个双写文件包含多少页面。默认值为128。

## 总结

Doublewrite Buffer是InnoDB的一个重要特性，用于保证MySQL数据的可靠性和一致性。

它的实现原理是通过将要写入磁盘的数据先写入到Doublewrite Buffer中的内存缓存区域，然后再写入到磁盘的两个不同位置，来避免由于磁盘损坏等因素导致数据丢失或不一致的问题。

总的来说，Doublewrite Buffer对于改善数据库性能和数据完整性起着至关重要的作用。尽管其引入了一些开销，但在大多数情况下，这些成本都被其提供的安全性和可靠性所抵消。

