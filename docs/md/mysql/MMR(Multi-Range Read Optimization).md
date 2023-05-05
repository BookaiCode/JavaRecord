[TOC]

MRR 的全称是 Multi-Range Read Optimization，是优化器将随机 IO 转化为顺序 IO 以降低查询过程中 IO 开销的一种手段。

## 什么是MRR？

先来了解下回表，回表是指，InnoDB在普通索引a上查到主键id的值后，再根据一个个主键id的值到主键索引上去查整行数据的过程。
我们知道二级索引是有回表的过程的，由于二级索引上引用的主键值不一定是有序的，因此就有可能造成大量的随机 IO，如果回表前把主键值给它排一下序，那么在回表的时候就可以用顺序 IO 取代原本的随机 IO。

**简单说：MRR 通过把「随机磁盘读」，转化为「顺序磁盘读」，从而提高了索引查询的性能**。

顺序读带来了几个好处：

1. 磁盘和磁头不再需要来回做机械运动；

2. 可以充分利用磁盘预读；

比如在客户端请求一页的数据时，可以把后面几页的数据也一起返回，放到数据缓冲池中，这样如果下次刚好需要下一页的数据，就不再需要到磁盘读取。这样做的理论依据是计算机科学中著名的局部性原理：

> 当一个数据被用到时，其附近的数据也通常会马上被使用。

**MRR 在本质上是一种用空间换时间的算法**。MySQL 不可能给你无限的内存来进行排序，这块内存的大小就由参数 **read_rnd_buffer_size** 来控制,如果 read_rnd_buffer 满了，就会先把满了的 rowid 排好序去磁盘读取，接着清空，然后再往里面继续放 rowid，直到 read_rnd_buffer 又达到 read_rnd_buffe 配置的上限，如此循环。

假设，我执行这个语句：

```sql
select * from t1 where a>=1 and a<=100;
```

主键索引是一棵B+树，在这棵树上，每次只能根据一个主键id查到一行数据。因此，回表肯定是一行行搜索主键索引的，基本流程如图1所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIl4jmfxUkEB7Heh3o71Lk52uSztvD1FzkZYvemgibv7ltxGcBhrJFQLyg/640?wx_fmt=png)

如果随着a的值递增顺序查询的话，id的值就变成随机的，那么就会出现随机访问，性能相对较差。虽然“按行查”这个机制不能改，但是调整查询的顺序，还是能够加速的。

**因为大多数的数据都是按照主键递增顺序插入得到的，所以我们可以认为，如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，能够提升读性能**。

这，就是MRR优化的设计思路。此时，语句的执行流程变成了这样：

1. 根据索引a，定位到满足条件的记录，将id值放入read_rnd_buffer中;

2. 将read_rnd_buffer中的id进行递增排序；

3. 排序后的id数组，依次到主键id索引中查记录，并作为结果返回。

这里，read_rnd_buffer的大小是由read_rnd_buffer_size参数控制的。如果步骤1中，read_rnd_buffer放满了，就会先执行完步骤2和3，然后清空read_rnd_buffer。之后继续找索引a的下个记录，并继续循环。

下面两幅图就是使用了MRR优化后的执行流程和explain结果。

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIlnohcwLLXB2ACwzcdjs8NCEjSpD34Iwia8IJ0beVJeYb6RXjZxvVbxYg/640?wx_fmt=png)

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIlAlvKCjvgVba7iand56gCticKcJp8BNp0IZOocIHvSyYZXp2JnfGmeKicQ/640?wx_fmt=png)

从explain结果中，我们可以看到Extra字段多了**Using MRR**，表示的是用上了MRR优化。而且，由于我们在read_rnd_buffer中按照id做了排序，所以最后得到的结果集也是按照主键id递增顺序的，也就是与图1结果集中行的顺序相反。

**MRR能够提升性能的核心在于，这条查询语句在索引a上做的是一个范围查询（也就是说，这是一个多值查询），可以得到足够多的主键id。这样通过排序以后，再去主键索引查数据，才能体现出“顺序性”的优势**。

## MRR如何使用？

```
//如果你不打开，是一定不会用到 MRR 的。
set optimizer_switch='mrr=on';
set optimizer_switch ='mrr_cost_based=off';
set read_rnd_buffer_size = 32 * 1024 * 1024;
```

mrr_cost_based: on/off，则是用来告诉优化器，要不要基于使用 MRR 的成本，考虑使用 MRR 是否值得（cost-based choice），来决定具体的 sql 语句里要不要使用 MRR。

很明显，对于只返回一行数据的查询，是没有必要 MRR 的，而如果你把 mrr_cost_based 设为 off，那优化器就会通通使用 MRR，这在有些情况下是很 stupid 的，所以建议这个配置还是设为 on，毕竟优化器在绝大多数情况下都是正确的。

------

本篇文章就到这里，感谢阅读，如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。 
![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)