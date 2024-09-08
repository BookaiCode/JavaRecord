## 摘要

在当今高度并发的数据库环境中，有效的并发控制是至关重要的。MVCC是MySQL中被广泛采用的并发控制机制，它通过版本管理来实现事务的隔离性，允许读写操作同时进行，提高数据库的并发性能和响应能力。

本文将深入解析MVCC机制的原理，帮助读者更好地理解和应用这一关键技术。

## MVCC 介绍

**MVCC，全称 Multi-Version Concurrency Control，即多版本并发控制**

MVCC的目的主要是为了提高数据库并发性能，用更好的方式去处理读-写冲突，做到即使有读写冲突时，也能做到不加锁。

这里的多版本指的是数据库中同时存在多个版本的数据，并不是整个数据库的多个版本，而是某一条记录的多个版本同时存在。

**并发控制的挑战**

在数据库系统中，同时执行的事务可能涉及相同的数据，因此需要一种机制来保证数据的一致性，传统的锁机制可以实现并发控制，但会导致阻塞和死锁等问题。

**MVCC的优点**

MVCC机制具有以下优点：

- 提高并发性能：读操作不会阻塞写操作，写操作也不会阻塞读操作，有效地提高数据库的并发性能。
- 降低死锁风险：由于无需使用显式锁来进行并发控制，MVCC可以降低死锁的风险。

## 当前读和快照读

在讲解MVCC原理之前，我们先来了解一下，当前读和快照读。

**当前读**

在MySQL中，当前读是一种读取数据的操作方式，它可以直接读取最新的数据版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。MySQL提供了两种实现当前读的机制：

- 一致性读（Consistent Read）：
  - 默认隔离级别下（可重复读），MySQL使用一致性读来实现当前读。
  - 在事务开始时，MySQL会创建一个一致性视图（Consistent View），该视图反映了事务开始时刻数据库的快照。
  - 在事务执行期间，无论其他事务对数据进行了何种修改，事务始终使用一致性视图来读取数据。
  - 这样可以保证在同一个事务内多次查询返回的结果是一致的，从而实现了当前读。
- 锁定读（Locking Read）：
  - 锁定读是一种特殊情况下的当前读方式，在某些场景下使用。
  - 当使用锁定读时，MySQL会在执行读取操作前获取共享锁或排他锁，以确保数据的一致性。
  - 共享锁（Shared Lock）允许多个事务同时读取同一数据，而排他锁（Exclusive Lock）则阻止其他事务读取或写入该数据。
  - 锁定读适用于需要严格控制并发访问的场景，但由于加锁带来的性能开销较大，建议仅在必要时使用。

下面列举的这些语法都是当前读：

| 语法                          |
| ----------------------------- |
| SELECT ... LOCK IN SHARE MODE |
| SELECT ... FOR UPDATE         |
| UPDATE                        |
| DELETE                        |
| INSERT                        |

当前读实际上是一种加锁的操作，是悲观锁的实现。

**快照读**

快照读是在读取数据时读取一个一致性视图中的数据，MySQL使用 MVCC 机制来支持快照读。

具体而言，每个事务在开始时会创建一个一致性视图（Consistent View），该视图反映了事务开始时刻数据库的快照。这个一致性视图会记录当前事务开始时已经提交的数据版本。 

当执行查询操作时，MySQL会根据事务的一致性视图来决定可见的数据版本。只有那些在事务开始之前已经提交的数据版本才是可见的，未提交的数据或在事务开始后修改的数据则对当前事务不可见。

像不加锁的 select 操作就是快照读，即不加锁的非阻塞读。

快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本。

**注意：快照读的前提是隔离级别不是串行级别，在串行级别下，事务之间完全串行执行，快照读会退化为当前读**

MVCC主要就是为了实现读-写冲突不加锁，而这个读指的就是快照读，是乐观锁的实现。

## MVCC 原理解析

### 隐式字段

MySQL中的行数据，除了我们肉眼能看到的字段之外，其实还包含了一些隐藏字段，它们在内部使用，默认情况下不会显示给用户。

| 字段        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| DB_ROW_ID   | 隐含的自增ID（隐藏主键），用于唯一标识表中的每一行数据，如果数据表没有主键，InnoDB会自动以DB_ROW_ID产生一个聚簇索引。 |
| DB_TRX_ID   | 该字段存储了当前行数据所属的事务ID。每个事务在数据库中都有一个唯一的事务ID。通过 DB_TRX_ID 字段，可以追踪行数据和事务的所属关系。 |
| DB_ROLL_PTR | 该字段存储了回滚指针（Roll Pointer），它指向用于回滚事务的Undo日志记录。 |

### Undo Log

上文提到了 Undo 日志，这个 Undo 日志是 MVCC 能够得以实现的核心所在。

Undo日志（Undo Log）是MySQL中的一种重要的事务日志，Undo日志的作用主要有两个方面：

- **事务回滚**：当事务需要回滚时，MySQL可以通过Undo日志中的旧值将数据还原到事务开始之前的状态，保证了事务回滚的一致性。
- **MVCC实现**：MVCC 是InnoDB存储引擎的核心特性之一。通过使用Undo日志，MySQL可以为每个事务提供独立的事务视图，使得事务读取数据时能看到一致且符合隔离级别要求的数据版本。

**在InnoDB存储引擎中，Undo日志分为两种：插入（insert）Undo日志 和 更新（update）Undo日志**

- insert undo log：插入Undo日志是指在插入操作中生成的Undo日志。由于插入操作的记录只对当前事务可见，对其他事务不可见，因此在事务提交后可以直接删除，无需进行purge操作。
- update undo log：更新Undo日志是指在更新或删除操作中生成的Undo日志。更新Undo日志可能需要提供MVCC机制，因此不能在事务提交时就立即删除。相反，它们会在提交时放入Undo日志链表中，并等待purge线程进行最终的删除。删除操作只是设置一下老记录的 DELETED_BIT，并不真正将过时的记录删除，为了节省磁盘空间，InnoDB有专门的purge线程来清理 DELETED_BIT 为true的记录。

注意：由于查询操作（SELECT）并不会修改任何记录，所以在查询操作执行时，并不需要记录相应的 undo log 。

**不同事务或者相同事务对同一记录行的修改，会使该记录行的 undo log 成为一条链表，链首就是最新的记录，链尾就是最早的旧记录**

举个例子，比如有个事务A插入了一条新记录：insert into user(id, name) values(1, "小明')

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJHziczHWpULcqicmNJpWfloHY6xnhfTLapwC6MK3FHSYkMZhvH1iadoywQ/640)

现在来了一个事务B对该记录的name做出了修改，改为 "小王"。

在事务B修改该行数据时，数据库会先对该行加排他锁，然后把该行数据拷贝到 undo log 中作为旧记录，即在 undo log 中有当前行的拷贝副本.

拷贝完毕后，修改该行name为 "小王，并且修改隐藏字段的事务ID为当前事务B的ID, 并将回滚指针指向拷贝到 undo log 的副本记录，即表示我的上一个版本就是它，事务提交后，释放锁。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJYIazBiabgj8QMspbg5QntMuiblGQGr1zq8A5Fj9ufvs85Yvht3CF85KQ/640)

此时又来了个事务C修改同一个记录，将name修改为 "小红"。

在事务C修改该行数据时，数据库也先为该行加锁，然后把该行数据拷贝到 undo log 中，作为旧记录，发现该行记录已经有 undo log 了，那么最新的旧数据作为链表的表头，插在该行记录的 undo log 最前面，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJ0QKAgRtyFdibGic6Db2B64xRqpUn0ORziamfYnIOMYOE8VqO0l8FGDrmg/640)



关于 DB_ROLL_PTR 与 Undo日志 的配合工作，具体流程如下：

1. 在更新或删除操作之前，MySQL会将旧值写入Undo日志中。
2. 当事务需要回滚时，MySQL会根据事务的Undo日志记录，通过 DB_ROLL_PTR 找到对应的Undo日志。
3. 根据Undo日志中记录的旧值，MySQL将旧值恢复到相应的数据行中，实现数据的回滚操作。

比方说现在想回滚到事务B，name值为 "小王" 的时候，只需通过 DB_ROLL_PTR 顺着列表找到对应的 Undo日志，将旧值恢复到数据行即可。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJBs1F1X44u8vub0jyFmk7x5MDIQKcOeTRV5rx7TSibiaBlwV9QbyiaibX8A/640)

通过 DB_ROLL_PTR 和 Undo日志 的配合工作，MySQL能够有效地管理事务的一致性和隔离性。Undo日志的使用也使得MySQL能够支持MVCC，从而提供了高并发环境下的读取一致性和事务隔离性。

### 版本链

在MVCC中，对于每次更新操作，旧值会被保存到一条undo日志中，即使它是该记录的旧版本。随着更新次数的增加，所有的版本都会通过roll_pointer属性连接成一个链表，称之为版本链。

版本链的头节点代表当前记录的最新值。此外，每个版本还包含生成该版本的事务ID。

### Read View

**一致性视图，全称 Read View ，是用来判断版本链中的哪个版本对当前事务是可见的**

Read View 说白了就是事务进行快照读操作时候生成的读视图（Read View），在该事务执行快照读的那一刻，会生成数据库系统当前的一个快照，记录并维护系统当前活跃事务的ID（每个事务开启时，都会被分配一个ID，这个ID是递增的）。

这里有一点要注意一下：**Read View只针对 RC 和 RR级别**

Read Uncommitted（RU）和 Serializable（串行化）是两个特殊的隔离级别，它们不需要使用 Read View 的主要原因是：

- Read Uncommitted（RU）隔离级别： 在 RU 隔离级别下，事务可以读取其他事务尚未提交的数据，即脏读。这意味着不需要通过 Read View 来限制访问范围，事务可以自由地读取其他事务的未提交数据。由于没有对可见性进行严格控制，因此不需要创建或使用 Read View。
- Serializable（串行化）隔离级别： 在 Serializable 隔离级别下，事务具有最高的隔离性，确保每次读取都能看到一致的快照。为了实现这种隔离级别，MySQL使用锁机制来保证事务之间的串行执行。由于事务按顺序执行，并且不允许并发操作，所以不需要使用 Read View 进行可见性判断。

Read Uncommitted 和 Serializable 隔离级别下的事务规则不涉及基于 Read View 的可见性判断。RU 允许脏读，而 Serializable 则通过锁机制保证串行执行。因此，在这两个隔离级别下，不需要创建或使用 Read View。

### Read View 可见性原则

Read View 遵循一个可见性原则，将要被修改的数据的 DB_TRX_ID 取出来，与系统当前其他活跃事务的ID去对比。

如果 DB_TRX_ID 跟 Read View 的属性做了某些比较，不符合可见性，那就通过 DB_ROLL_PTR 回滚指针去取出 Undo Log 中的 DB_TRX_ID 再比较。

即遍历链表的 DB_TRX_ID （从链首到链尾，即从最近的一次修改查起），直到找到满足特定条件的 DB_TRX_ID，那么这个 DB_TRX_ID 所在的记录就是当前事务能看见的最新老版本。

Read View 会维护以下几个字段：

| 字段             | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| m_ids            | `Read View` 创建时其他未提交的活跃事务 ID 列表。创建 `Read View`时，将当前未提交事务 ID 记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。`m_ids` 不包括当前事务自己和已提交的事务（正在内存中）。 |
| m_creator_trx_id | 创建该 `Read View` 的事务 ID。                               |
| m_low_limit_id   | 目前出现过的最大的事务 ID+1，即下一个将被分配的事务 ID。大于等于这个 ID 的数据版本均不可见。 |
| m_up_limit_id    | 活跃事务列表 `m_ids` 中最小的事务 ID，如果 `m_ids` 为空，则  `m_low_limit_id`为`m_up_limit_id` 。小于这个 ID 的数据版本均可见。 |

Read View 可见性具体判断如下：

1. 如果被访问版本的 `DB_TRX_ID ` 属性值与 Read View 中的 `m_creator_trx_id` 值相同，表示当前事务正在访问自己所修改的记录，因此该版本可以被当前事务访问。

2. 如果被访问版本的 `DB_TRX_ID ` 属性值小于 Read View 中的 `m_up_limit_id  `值，说明生成该版本的事务在当前事务生成 Read View 之前已经提交，因此该版本可以被当前事务访问。

3. 如果被访问版本的 `DB_TRX_ID ` 属性值大于或等于 Read View 中的  `m_low_limit_id ` 值，说明生成该版本的事务在当前事务生成 Read View 之后才提交，因此该版本不能被当前事务访问。

4. 如果被访问版本的 `DB_TRX_ID ` 属性值位于 Read View 的  `m_up_limit_id` 和  `m_low_limit_id ` 之间（包括边界），则需要进一步检查 `DB_TRX_ID ` 是否在`m_ids `列表中。如果在列表中，说明在创建ReadView时生成该版本的事务仍处于活跃状态，因此该版本不能被访问；如果不在列表中，说明在创建 Read View 时生成该版本的事务已经提交，因此该版本可以被访问。


事务可见性示意图：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJLogibJCiaCWD7IdRgZiaGiaKTPsFO4LM2z1oxGNdlRRibcYA81cwicf0CJXA/640)

## RC 和 RR 下的 Read View

RC 和 RR 下生成 `Read View` 的时机是有所差异的：

- **RC**：每次 SELECT 数据前都生成一个ReadView。
- **RR**：只在第一次读取数据时生成一个ReadView，后面会复用第一次生成的。

正因为RC 和 RR生成 Read View 的时机不同，导致两个级别下看到的数据会不一致。

举例说明，假设数据初始状态如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJuicCSEtYL2iczewvBkHjicGzRibF9mhJ5C0aaKUae4P16F5cIzia38Ka8rw/640)

有 A，B，C 三个事务，执行顺序如下：

|      | 事务A（事务ID: 100）                   | 事务B（事务ID: 200）                   | 事务C（事务ID: 300）            |
| ---- | -------------------------------------- | -------------------------------------- | ------------------------------- |
| T1   | begin                                  |                                        |                                 |
| T2   |                                        | begin                                  | begin                           |
| T3   | update user set name="小王" where id=1 |                                        |                                 |
| T4   | update user set name="小红" where id=1 |                                        | select * from user where id = 1 |
| T5   | commit                                 | update user set name="小黑" where id=1 |                                 |
| T6   |                                        | update user set name="小白" where id=1 | select * from user where id = 1 |
| T7   |                                        | commit                                 |                                 |
| T8   |                                        |                                        | select * from user where id = 1 |
| T9   |                                        |                                        | commit                          |
| T10  |                                        |                                        |                                 |

### RC 下的 Read View

**T4时刻**

我们来看 T4 时刻的情况，此时 事务A 和 事务B 都还没提交，所以活跃的事务ID，即 `m_ids` 为：[100，200]，四个字段的值分别如下：

| 字段             | 值         |
| ---------------- | ---------- |
| m_ids            | [100，200] |
| m_creator_trx_id | 300        |
| m_low_limit_id   | 400        |
| m_up_limit_id    | 100        |

T4时刻的版本链如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJqCxKKZIJ3wDRLdm47ibKuLMEBQKqtErTCXF5kvhj2CvvuXEPYjM5VCg/640)

依据我们之前说的可见性原则，事务C最终看到的应该是  `name = "小明"`  的数据，理由如下：

最新记录的 `DB_TRX_ID` 为 100，既不小于 `m_up_limit_id`，也不大于 `m_low_limit_id`，也不等于 `m_creator_trx_id`。

落在了黄区：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJLogibJCiaCWD7IdRgZiaGiaKTPsFO4LM2z1oxGNdlRRibcYA81cwicf0CJXA/640)

`DB_TRX_ID` 存在于 `m_ids` 列表中，故不可见，顺着版本链继续往下。

根据 `DB_ROLL_PTR` 找到 `undo log` 中的前一版本记录，前一条记录的 `DB_TRX_ID` 还是 100，还是不可见，继续往下。

继续找前一条 `DB_TRX_ID`为 1，满足 1 < `m_up_limit_id`，可见，所以事务C 查询到数据为  `name = "小明"`  。

**T6时刻**

T6时候的版本链如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJg6yFEgEYH29FsWDuia2ILVn3glQZrhAXbNC8laQnI1hNpTiaCPzQeh3w/640)



T6时刻，会再次生成新的 Read View，四个字段的值分别如下：

| 字段             | 值    |
| ---------------- | ----- |
| m_ids            | [200] |
| m_creator_trx_id | 300   |
| m_low_limit_id   | 400   |
| m_up_limit_id    | 200   |

根据可见性原则，最终T6时刻事务C 查询到数据为  `name = "小红"` 。

**T8时刻**

T8时刻的版本链和T6时刻是一致的，不同的是 Read View，因为T8时刻会再生成一个 Read View，四个字段的值分别如下：

| 字段             | 值   |
| ---------------- | ---- |
| m_ids            | []   |
| m_creator_trx_id | 300  |
| m_low_limit_id   | 400  |
| m_up_limit_id    | 400  |

根据可见性原则，最终T8时刻事务C 查询到数据为  `name = "小白"` 。

总结一下，事务C在 RC 级别下各个时刻看到的数据如下：

| 时刻 | name |
| ---- | ---- |
| T4   | 小明 |
| T6   | 小红 |
| T8   | 小白 |

下面我们来看看，RR 级别下的表现是如何的。

### RR 下的 Read View

（RR 的版本链和 RC 的版本链是一致的，区别在于 Read View）

**T4时刻**

T4 时刻的情况，和 R C的情况是一致的：

| 字段             | 值         |
| ---------------- | ---------- |
| m_ids            | [100，200] |
| m_creator_trx_id | 300        |
| m_low_limit_id   | 400        |
| m_up_limit_id    | 100        |

根据可见性原则，最终T4时刻事务C 查询到数据为  `name = "小明"` ，和 RC 的T4时刻是一致的。

**T6时刻**

RR 级别会复用 Read View，所以T6时刻也是：

| 字段             | 值         |
| ---------------- | ---------- |
| m_ids            | [100，200] |
| m_creator_trx_id | 300        |
| m_low_limit_id   | 400        |
| m_up_limit_id    | 100        |

根据可见性原则，T6时刻我们发现事务C查询到的数据还是  `name = "小明"` 。

继续看T8时刻。

**T8时刻**

T8时刻继续复用先前的 Read View。

根据可见性原则，T8时刻事务C查询到的数据依旧是   `name = "小明"` 。

### 小结

我们将事务C在 RC 和 RR 级别下看到的数据，放到一块来对比下：

| 时刻 | RC   | RR |
| ---- | ---- |   |
| T4   | 小明 | 小明 |
| T6   | 小红 | 小明 |
| T8   | 小白 | 小明 |

可以看出二者由于生成 Read View 的时机不同，导致在各个时刻看到的数据会存在差异。

回过头来看 RC 和 RR 隔离级别的定义，会有种恍然大悟的感觉：

- 读已提交（Read Committed）：事务只能读取到已经提交的数据。
- 可重复读（Repeatable Read）：事务在整个事务期间保持一致的快照视图，不受其他事务的影响。

**总之在 RC 隔离级别下，每个快照读都会生成并获取最新的 Read View；而在 RR 隔离级别下，则是只在第一个快照读创建Read View，之后的快照读获取的都是同一个Read View**

## RR 级别下能否防止幻读

**严谨的说，RR 级别下只能防止部分幻读**

首先，幻读通常指的是在同一个事务中，第二次查询发现了新增加的行，而第一次查询并没有返回这些新增加的行。

通过前面的例子，我们也看到了，在 RR 隔离级别下，由于一致性视图的存在，如果其他事务插入了新的行，在同一个事务中进行多次查询，这些新增的行将会被包含在事务的一致性视图中，确实可以避免部分幻读场景。

> 这里注意一下：MVCC解决的只是 RR 级别下快照读的幻读问题，而当前读的幻读问题则是通过临键锁来解决的。也就是说 RR 级别下是通过 MVCC+临键锁 来解决大部分幻读问题的。

为什么说是部分解决？看下面这个例子：

|      | 事务A                                        | 事务B                         |
| ---- | -------------------------------------------- | ----------------------------- |
| T1   | begin                                        |                               |
| T2   |                                              | begin                         |
| T3   |                                              | select * from user            |
| T4   | insert into user(id, name) values(2, "小张') |                               |
| T5   |                                              | select * from user for update |
| T6   | commit                                       |                               |
| T7   |                                              | commit                        |

假设数据初始状态如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP6Xr0ibupPNhyKvbXYAuBwJuicCSEtYL2iczewvBkHjicGzRibF9mhJ5C0aaKUae4P16F5cIzia38Ka8rw/640)

T3时刻看到的数据只有一条 `name = "小明"`，而T5时刻，由于 select * from user for update 使用的是当前读，读取的是最新的数据版本，T5时刻查询出来的数据是两条，name 分别为 "小明" 和 "小张"。

理解了上面的例子之后，再看下面这个例子：

|      | 事务A                                        | 事务B                                  |
| ---- | -------------------------------------------- | -------------------------------------- |
| T1   | begin                                        |                                        |
| T2   |                                              | begin                                  |
| T3   |                                              | select * from user                     |
| T4   | insert into user(id, name) values(2, "小张') |                                        |
| T5   |                                              | update user set name="小陈" where id=2 |
| T6   |                                              | select * from user                     |
| T7   | commit                                       |                                        |
| T8   |                                              | commit                                 |

UPDATE 语句也是当前读，也会发生幻读问题，最终看到的数据是name 分别为 "小明" 和 "小陈"。

这里发生幻读的原因，和上面的例子是一样的，本质都是在一个事务中，即使用了快照读又使用了当前读，RR 级别下无法预防此种情况，所以说 RR 级别下无法完全解决幻读问题。

## 总结

综上所述，MVCC 是一种强大的并发控制机制，在高并发环境中起着重要的作用。通过了解 MVCC 的原理和实现流程，我们可以更好地理解 MySQL 的并发控制机制，理解 MVCC 的原理对于接触 MySQL 的开发人员来说是必不可少的知识点。

希望本文对各位同学有所帮助，加深对 MVCC 及其在 MySQL 中的应用的理解。感谢阅读！

