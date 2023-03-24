[TOC]

## 摘要

Redis是一款性能强劲的内存数据库，但是在使用过程中，我们可能会遇到**Big Key**问题，这个问题就是Redis中某个key的value过大，所以**Big Key问题本质是Big Value问题**，导致Redis的性能下降或者崩溃。本文将向大家介绍如何排查和解决这个问题。

## Big Key问题介绍

在Redis中，每个key都有一个对应的value，如果某个key的value过大，就会导致Redis的性能下降或者崩溃，比玄学更玄学，**因为Redis需要将大key全部加载到内存中，这会占用大量的内存空间，会降低Redis的响应速度，这个问题被称为Big Key问题**。不要小看这个问题，它可是能让你的Redis瞬间变成“乌龟”，由于Redis单线程的特性，操作Big Key的通常比较耗时，也就意味着阻塞Redis可能性越大，这样会造成客户端阻塞或者引起故障切换，有可能导致“慢查询”。

一般而言，下面这两种情况被称为大 key：

- String 类型的 key 对应的value超过 10 MB。
- list、set、hash、zset等集合类型，集合元素个数超过 5000个。

> 以上对 Big Key 的判断标准并不是唯一，只是一个大体的标准。在实际业务开发中，对 Big Key的判断是需要根据具体的使用场景做不同的判断。比如操作某个 key 导致请求响应时间变慢，那么这个 key 就可以判定成 Big Key。

**在Redis中，大key通常是由以下几种原因导致的**：

- 对象序列化后的大小过大
- 存储大量数据的容器，如set、list等
- 大型数据结构，如bitmap、hyperloglog等

如果不及时处理这些大key，它们会逐渐消耗Redis服务器的内存资源，最终导致Redis崩溃。

## Big Key问题排查

当出现Redis性能急剧下降的情况时，很可能是由于存在大key导致的。在排除大key问题时，可以考虑采取以下几种方法：

### 使用BIGKEYS命令

Redis自带的 BIGKEYS 命令可以查询当前Redis中所有key的信息，对整个数据库中的键值对大小情况进行统计分析，比如说，统计每种数据类型的键值对个数以及平均大小。此外，这个命令执行后，会输出每种数据类型中最大的 bigkey 的信息，对于 String 类型来说，会输出最大 bigkey 的字节长度，对于集合类型来说，会输出最大 bigkey 的元素个数

`BIGKEYS`命令会扫描整个数据库，这个命令本身会阻塞Redis，找出所有的大键，并将其以一个列表的形式返回给客户端。

命令格式如下：

```shell
$ redis-cli --bigkeys
```

返回示例如下：

```
# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far 'a' with 3 bytes
[05.14%] Biggest list   found so far 'b' with 100004 items
[35.77%] Biggest string found so far 'c' with 6 bytes
[73.91%] Biggest hash   found so far 'd' with 3 fields

-------- summary -------

Sampled 506 keys in the keyspace!
Total key length in bytes is 3452 (avg len 6.82)

Biggest string found 'c' has 6 bytes
Biggest   list found 'b' has 100004 items
Biggest   hash found 'd' has 3 fields

504 strings with 1403 bytes (99.60% of keys, avg size 2.78)
1 lists with 100004 items (00.20% of keys, avg size 100004.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
1 hashs with 3 fields (00.20% of keys, avg size 3.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00)
```

需要注意的是，由于`BIGKEYS`命令需要扫描整个数据库，所以它可能会对Redis实例造成一定的负担。**在执行这个命令之前，请确保您的Redis实例有足够的资源来处理它，建议在从节点执行**。

#### Debug Object

如果我们找到了Big Key，就需要对其进行进一步的分析。我们可以使用命令`debug object key`查看某个key的详细信息，包括该key的value大小等。这时候你就可以“窥探”Redis的内部，看看到底是哪个key太大了。

Debug Object 命令是一个调试命令，当 key 存在时，返回有关信息。 当 key 不存在时，返回一个错误。

```
redis 127.0.0.1:6379> DEBUG OBJECT key
Value at:0xb6838d20 refcount:1 encoding:raw serializedlength:9 lru:283790 lru_seconds_idle:150

redis 127.0.0.1:6379> DEBUG OBJECT key
(error) ERR no such key
```

serializedlength表示key对应的value序列化之后的字节数

#### memory usage

在Redis4.0之前，只能通过DEBUG OBJECT命令估算key的内存使用(字段serializedlength)，但DEBUG OBJECT命令是有误差的。

4.0版本及以上，我们可以使用memory usag命令。

memory usage命令使用非常简单，直接按memory usage key名字；如果当前key存在，则返回key的value实际使用内存估算值；如果key不存在，则返回nil。

```shell
127.0.0.1:6379> set k1 value1
OK
127.0.0.1:6379> memory usage k1    //这里k1 value占用57字节内存
(integer) 57
127.0.0.1:6379> memory usage aaa  // aaa键不存在，返回nil.
(nil)
```

对于除String类型之外的类型，memory usage命令采用抽样的方式，默认抽样5个元素，所以计算是近似值，我们也可以指定抽样的个数。

示例说明：生成一个100w个字段的hash键：hkey，每字段的value长度是从1~1024字节的随机值。

```
127.0.0.1:6379> hlen hkey    // hkey有100w个字段，每个字段的value长度介于1~1024个字节
(integer) 1000000
127.0.0.1:6379> MEMORY usage hkey   //默认SAMPLES为5，分析hkey键内存占用521588753字节
(integer) 521588753
127.0.0.1:6379> MEMORY usage hkey SAMPLES  1000 //指定SAMPLES为1000，分析hkey键内存占用617977753字节
(integer) 617977753
127.0.0.1:6379> MEMORY usage hkey SAMPLES  10000 //指定SAMPLES为10000，分析hkey键内存占用624950853字节
(integer) 624950853
```

要想获取key较精确的内存值，就指定更大抽样个数。但是抽样个数越大，占用cpu时间分片就越大。

### redis-rdb-tools

redis-rdb-tools 是一个 python 的解析 rdb 文件的工具，在分析内存的时候，我们主要用它生成内存快照。可以把 rdb 快照文件生成 CSV 或 JSON 文件，也可以导入到 MySQL 生成报表来分析。

使用 PYPI 安装

```
pip install rdbtools
```

生成内存快照

```
rdb -c memory dump.rdb > memory.csv
```

在生成的 CSV 文件中有以下几列：

- `database` key在Redis的db
- `type` key类型
- `key` key值
- `size_in_bytes` key的内存大小
- `encoding` value的存储编码形式
- `num_elements` key中的value的个数
- `len_largest_element` key中的value的长度

可以在MySQL中新建表然后导入进行分析，然后可以直接通过SQL语句进行查询分析。

```sql
CREATE TABLE `memory` (
     `database` int(128) DEFAULT NULL,
     `type` varchar(128) DEFAULT NULL,
     `KEY` varchar(128),
     `size_in_bytes` bigint(20) DEFAULT NULL,
     `encoding` varchar(128) DEFAULT NULL,
     `num_elements` bigint(20) DEFAULT NULL,
     `len_largest_element` varchar(128) DEFAULT NULL,
     PRIMARY KEY (`KEY`)
 );
```

例子：查询内存占用最高的3个 key

```sql
mysql> SELECT * FROM memory ORDER BY size_in_bytes DESC LIMIT 3;
+----------+------+-----+---------------+-----------+--------------+---------------------+
| database | type | key | size_in_bytes | encoding  | num_elements | len_largest_element |
+----------+------+-----+---------------+-----------+--------------+---------------------+
|        0 | set  | k1  |        624550 | hashtable |        50000 | 10                  |
|        0 | set  | k2  |        420191 | hashtable |        46000 | 10                  |
|        0 | set  | k3  |        325465 | hashtable |        38000 | 10                  |
+----------+------+-----+---------------+-----------+--------------+---------------------+
3 rows in set (0.12 sec)
```


## Big Key问题解决思路

当发现存在大key问题时，我们需要及时采取措施来解决这个问题。下面列出几种可行的解决思路：

### 分割大key

将Big Key拆分成多个小的key。这个方法比较简单，但是需要修改应用程序的代码。就像是把一个大蛋糕切成小蛋糕一样，有点费力，但是可以解决问题。

或者尝试将Big Key转换成Redis的数据结构。例如，将Big Key转换成Hash，List或者Set等数据结构。

###  对象压缩

如果大key的大小主要是由于对象序列化后的体积过大，我们可以考虑使用压缩算法来减小对象的大小。Redis自身支持多种压缩算法，例如LZF、Snappy等。

### 直接删除

如果你使用的是Redis 4.0+的版本，可以直接使用 unlink命令去异步删除。4.0以下的版本 可以考虑使用 scan ,分批次删除。



无论采用哪种方法，都需要注意以下几点：

1. 避免使用过大的value。如果需要存储大量的数据，可以将其拆分成多个小的value。就像是吃饭一样，一口一口的吃，不要贪多嚼不烂。
2. 避免使用不必要的数据结构。例如，如果只需要存储一个字符串，就不要使用Hash或者List等数据结构。
3. 定期清理过期的key。如果Redis中存在大量的过期key，就会导致Redis的性能下降。就像是家里的垃圾，需要定期清理。

2. 对象压缩

## 总结

Big Key问题是Redis中常见的问题之一，但是通过合理的排查和解决思路，我们可以有效地避免这个问题。在使用Redis时，需要注意避免使用过大的value和不必要的数据结构，以及定期清理过期的key。

------

本篇文章就到这里，感谢阅读，如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。
![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)
