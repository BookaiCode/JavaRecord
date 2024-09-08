在数据库处理中，Join操作是最基本且最重要的操作之一，它能将不同的表连接起来，实现对数据集的更深层次分析。

MySQL作为一款流行的关系型数据库管理系统，其在执行Join操作时使用了多种高效的算法，包括Index Nested-Loop Join（NLJ）和Block Nested-Loop Join（BNL）。这些算法各有优缺点，本文将探讨这两种算法的工作原理，以及如何在MySQL中使用它们。

## 什么是Join

在MySQL中，Join是一种用于组合两个或多个表中数据的查询操作。Join操作通常基于两个表中的某些共同的列进行，这些列在两个表中都存在。MySQL支持多种类型的Join操作，如**Inner Join**、**Left Join**、**Right Join**等。

Inner Join是最常见的Join类型之一。在Inner Join操作中，只有在两个表中都存在的行才会被返回。

例如，如果我们有一个“customers”表和一个“orders”表，我们可以通过在这两个表中共享“customer_id”列来组合它们的数据。

```sql
SELECT *
FROM customers
INNER JOIN orders
ON customers.customer_id = orders.customer_id;
```

上面的查询将返回所有存在于“customers”和“orders”表中的“customer_id”列相同的行。

## Index Nested-Loop Join

Index Nested-Loop Join（NLJ）算法是Join算法中最基本的算法之一。

在NLJ算法中，MySQL首先会选择一个表（通常是小型表）作为驱动表，并迭代该表中的每一行。然后，MySQL在第二个表中搜索匹配条件的行，这个搜索过程通常使用索引来完成。一旦找到匹配的行，MySQL将这些行组合在一起，并将它们作为结果集返回。

工作流程如图：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIl94cTiasZKjpEmlWicZ0zBBeNq1eNPiahuOxSGQXX6aYSQJSDn6I3ymnSQ/640)

例如，执行下面这个语句：

```sql
select * from t1 straight_join t2 on (t1.a=t2.a);
```

> 注：当使用 `straight_join` 时，MySQL会强制按照在查询中指定的从左到右的顺序执行连接。

在这个语句里，假设 t1 是驱动表，t2 是被驱动表。我们来看一下这条语句的explain结果。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIlEhryCJl5jj5PV23LPSQkQaoW3IRbgfKO3v0WpUUr1g1SvzWpNRP6iaw/640)

可以看到，在这条语句里，被驱动表t2的字段a上有索引，join过程用上了这个索引，因此这个语句的执行流程是这样的：

1. 从表t1中读入一行数据 R；
2. 从数据行R中，取出a字段到表t2里去查找；
3. 取出表t2中满足条件的行，跟R组成一行，作为结果集的一部分；
4. 重复执行步骤1到3，直到表t1的末尾循环结束。

这个过程就跟我们写程序时的嵌套查询类似，并且可以用上被驱动表的索引，所以我们称之为「**Index Nested-Loop Join**」，简称**NLJ**。

NLJ是使用上了索引的情况，那如果查询条件没有使用到索引呢？

MySQL会选择使用另一个叫作「**Block Nested-Loop Join**」的算法，简称**BNL**。

## Block Nested-Loop Join

Block Nested Loop Join（BNL）算法与NLJ算法不同的是，BNL算法使用一个类似于缓存的机制，将表数据分成多个块，然后逐个处理这些块，以减少内存和CPU的消耗。

例如，执行下面这个语句：

```sql
select * from t1 straight_join t2 on (t1.a=t2.b);
```

如果 t2 表的字段b上是没有建立索引的。这时候，被驱动表上没有可用的索引，算法的流程是这样的：

1. 把表t1的数据读入线程内存`join_buffer`中，由于我们这个语句中写的是select *，因此是把整个表t1放入了内存；
2. 扫描表t2，把表t2中的每一行取出来，跟`join_buffer`中的数据做对比，满足join条件的，作为结果集的一部分返回。

这条SQL语句的explain结果如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIlibay271rhTD6FiaNEEGAAEQVC5jTic0fIVMh2vznm2ibjZlRGLAKZNMajw/640)

可以看到，在这个过程中，MySQL对表 t1 和 t2 都做了一次全表扫描，因此总的扫描行数是1100。

由于`join_buffer`是以无序数组的方式组织的，因此对表t2中的每一行，都要做100次判断，总共需要在内存中做的判断次数是：**100*1000=10万次**。

虽然Block Nested-Loop Join算法是全表扫描。但是是在内存中进行的判断操作，速度上会快很多。但是性能仍然不如NLJ。

`join_buffer`的大小是由参数**join_buffer_size**设定的，默认值是256k。

**那如果join_buffer_size的大小不足以放下表t1的所有数据呢？**

办法很简单，就是分段放，执行流程如下：

1. 顺序读取数据行放入`join_buffer`中，直到`join_buffer`满了。
2. 扫描被驱动表跟`join_buffer`中的数据做对比，满足join条件的，作为结果集的一部分返回。
3. 清空`join_buffer`，重复上述步骤。

虽然分成多次放入`join_buffer`，但是判断等值条件的次数还是不变的，依然是10万次。

## MRR & BKA

上篇文章里我们有提到MRR（Multi-Range Read）。MySQL在5.6版本后引入了**Batched Key Acess(BKA)**算法，这个BKA算法，其实就是对NLJ算法的优化，而BKA算法正是基于MRR。

NLJ算法执行的逻辑是：**从驱动表t1，一行行地取出a的值，再到被驱动表t2去做join。也就是说，对于表t2来说，每次都是匹配一个值。这时，MRR的优势就用不上了**。

其实我们可以从表t1里一次性地多拿些行出来，先放到一个临时内存，一起传给表t2。这个临时内存不是别人，就是`join_buffer`。

通过上一篇文章，我们知道`join_buffer` 在BNL算法里的作用，是暂存驱动表的数据。但是在NLJ算法里并没有用。那么，我们刚好就可以复用`join_buffer`到BKA算法中。

NLJ算法优化后的BKA算法的流程，如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScM0uen1PDv6hsZTdwWPECIlW4MXORyOusGOGvbCkwgts385bNicgS9IZWOJnPic9SeGCxPF1lfqnw7A/640)

图中，在`join_buffer`中放入的数据是R1~R100，表示的是只会取查询需要的字段。当然，如果`join buffer`放不下R1~R100的所有数据，就会把这100行数据分成多段执行上图的流程。

如果要使用BKA优化算法的话，你需要在执行SQL语句之前，先设置

```sql
set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
```

其中，前两个参数的作用是要启用MRR。这么做的原因是，BKA算法的优化要依赖于MRR。

对于BNL，我们可以通过建立索引转为BKA。但是，有时候你确实会碰到一些不适合在被驱动表上建索引的情况。比如下面这个语句：

```sql
select * from t1 join t2 on (t1.b=t2.b) where t2.b>=1 and t2.b<=2000;
```

假设t1表1000行，t2表100万行，t2.b<=2000过滤后，t2表需要参与join的只有2000行数据。

如果这条语句是一个低频的SQL语句，那么在表t2的字段b上创建索引就很浪费了。

这时候，我们可以考虑使用临时表。使用临时表的大致思路是：

1. 把表t2中满足条件的数据放在临时表tmp_t中；
2. 为了让join使用BKA算法，给临时表tmp_t的字段b加上索引；
3. 让表t1和tmp_t做join操作。

此时，对应的SQL语句的写法如下：

```sql
create temporary table temp_t(id int primary key, a int, b int, index(b))engine=innodb;
insert into temp_t select * from t2 where b>=1 and b<=2000;
select * from t1 join temp_t on (t1.b=temp_t.b);
```

总体来看，不论是在原表上加索引，还是用有索引的临时表，我们的思路都是让join语句能够用上被驱动表上的索引，来触发BKA算法，提升查询性能。

## 总结

在MySQL中，不管Join使用的是NLJ还是BNL总是应该使用小表做驱动表。更准确地说，**在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与join的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表**。

另外应当尽量避免使用BNL算法，如果确认优化器会使用BNL算法，就需要做优化。优化的常见做法是，给被驱动表的join字段加上索引，把BNL算法转成BKA算法。对于不好在索引的情况，可以基于临时表的改进方案，提前过滤出小数据添加索引。
