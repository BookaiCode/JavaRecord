Redis是一款开源的高性能key-value数据库，广泛应用于各种场景。在Redis中，**数据类型（Type）和编码（Encoding）**是非常重要的概念。本篇博客将详细介绍Redis支持的数据类型以及相应的编码方式和底层实现原理。

要查看Redis某个key的内部编码，可以使用Redis自带的命令`OBJECT ENCODING key`。

其中，`key`是你想要查询的键名。例如，如果你想要查询名为`mykey`的键的内部编码，可以执行以下命令：

```bash
127.0.0.1:6379> object encoding mykey  // 查看某个Redis键值的编码
```

## redisObject

在Redis中，`redisObject` 是一个非常重要的数据结构，它用于保存字符串、列表、集合、哈希表和有序集合等类型的值。以下是关于 `redisObject` 结构体的定义：

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:24; /* lru time (relative to server.lruclock) */
    int refcount;
    void *ptr;
} robj;
```

各个属性解析如下：

- **type**：用于标识对象所属的类型，分别是 `REDIS_STRING`、`REDIS_LIST`、`REDIS_SET`、`REDIS_ZSET` 和 `REDIS_HASH` 等。
- **encoding**： 用于标识对象内部的编码方式， 如 `REDIS_ENCODING_INT`、`REDIS_ENCODING_HT`、`REDIS_ENCODING_ZIPMAP` 等。
- **lru**：这个字段记录了对象被命令调用的时间, 它是缓存淘汰策略（LRU）的一部分。
- **refcount**：引用计数，当 `refcount` 减少到0时，对象就可以被清理并回收内存。
- **ptr**：一个指针，根据对象的类型和编码方式的不同，这个指针可能会指向各种不同的类型，比如整数、动态字符串、链表、字典等。

其中，redisObject的encoding取值有如下几种：

```c
#define OBJ_ENCODING_RAW 0        //简单动态字符串，用于保存键值对的键和配置文件中的参数。
#define OBJ_ENCODING_INT 1        //整型值，用于优化小整数的内存使用。
#define OBJ_ENCODING_HT 2         //哈希表，用于存储普通哈希对象的字段和值。
#define OBJ_ENCODING_ZIPMAP 3     //缩字典，这是一种特殊类型的哈希表，用于优化小哈希对象的内存使用。
#define OBJ_ENCODING_LINKEDLIST 4 //双端链表，用于存储列表键。
#define OBJ_ENCODING_ZIPLIST 5    //压缩列表，用于优化小列表或者小哈希对象的内存使用。
#define OBJ_ENCODING_INTSET 6     //整数集合，用于优化只包含整数元素的集合的内存使用。
#define OBJ_ENCODING_SKIPLIST 7   //跳跃表和字典，用于存储有序集合键。
#define OBJ_ENCODING_EMBSTR 8     //对于长度小于44字节的字符串，Redis选择使用此特殊的编码方式。
#define OBJ_ENCODING_QUICKLIST 9  //对于列表对象（list object）的一种编码方式。quicklist是ziplist和双向链表的混合体。
```

## Type与Encoding介绍

Redis支持五种主要的数据类型：字符串(String)、列表(List)、集合(Set)、有序集合(Sorted Set)和哈希(Hash)。

每种数据类型都有对应的编码方式，数据类型与编码方式总览如下：

| 数据类型 | 编码方式                       |
| :------- | :----------------------------- |
| 字符串   | int、embstr、raw               |
| 哈希表   | ziplist、hashtable             |
| 列表     | ziplist、linkedlist、quicklist |
| 集合     | intset、hashtable              |
| 有序集合 | ziplist、skiplist              |

### 字符串

字符串是Redis中最基本的数据类型，通常用于存储文本或二进制数据。字符串在Redis中支持三种编码方式：

- **int**：当字符串可以表示为整数时，Redis会将其转换为整数，并采用int编码方式存储。int编码方式的优点是存储空间小，操作效率高。缺点是只能存储整数，不支持字符串操作。
- **embstr(embstr-encoded string)**：保存长度小于44字节的字符串，当一个字符串比较短，采用此编码方式存储，可以减少内存占用。
- **raw(raw-encoded string)**：保存长度大于44字节的字符串，当一个字符串比较长时，采用此编码方式存储。

### 列表

列表是一系列有序的字符串集合，可以添加、修改和删除元素。列表在Redis中支持三种编码方式：

- **ziplist**：在Redis3.2版本之前，当List列表中每个字符串的长度都「**小于64字节**」并且List列表中「**元素数量小于512个**」时，List对象使用ziplist编码，其他情况使用linkedlist编码。ziplist是一种紧凑的、压缩的列表结构，可以节省内存，适用于小型列表。
- **linkedlist**：linkedlist是一种链表结构，支持任意大小的列表。但其内存占用会随着列表长度的增加而增加。
- **quicklist**：Redis 3.2版本引入，quicklist是一种由多个ziplist组成的列表结构，既能保证性能，又能节省内存，适用于大型列表。

### 集合

集合是一系列无序的字符串集合，支持添加、删除和查询元素。集合在Redis中支持两种编码方式：

- **intset**：当集合中的元素都是整数时，Redis会采用intset编码方式存储。intset编码方式的优点是存储空间小，操作效率高。
- **hashtable**：当集合中的元素包含字符串时，Redis会采用hashtable编码方式存储。hashtable编码方式的优点是可以存储任意类型的元素，支持字符串操作。缺点是存储空间相对较大，操作效率相对较低。

### 有序集合

有序集合是一系列无序的字符串集合，每个元素关联一个分数，可以根据分数排序。有序集合在Redis中支持两种编码方式：

- **ziplist**：当集合中元素个数少于128个，并且每个元素的大小小于64字节时，使用此编码方式。这是因为ziplist在处理较小数据时，内存效率更高，性能更优。
- **skiplist**：skiplist是一种跳跃表结构，支持快速查询和排序。适用于大型有序集合。

### 哈希表

哈希表是一系列键值对集合，每个键关联一个值。哈希表在Redis中支持两种编码方式：

- **ziplist**：保存的所有键值的字符串长度小于64字节，并且键值对数量小于512个，Redis会采用ziplist编码方式存储。ziplist编码方式的优点是存储空间小，操作效率高。缺点是不支持快速的键查找操作。
- **hashtable**：除上述条件之外，Redis会采用hashtable编码方式存储。hashtable编码方式的优点是支持快速的键查找操作，缺点是存储空间相对较大，操作效率相对较低。

## Type与Encoding底层原理

了解Redis支持的数据类型和编码方式后，我们来看一下它们的底层实现原理。

### 编码转换

Redis中的每个键值对都有一个类型标识，表示该键值对的数据类型。当我们对一个键进行操作时，Redis会根据该键当前的编码方式以及操作所需的编码方式，对键值对进行编码转换。

例如，当我们向一个字符串中追加内容时，如果该字符串当前的编码方式为raw，但是新的内容可以使用embstr编码方式存储，那么Redis会将该字符串的编码方式从raw转换为embstr。

### 数据结构

除了编码方式外，Redis还使用了许多经典的数据结构来实现各种数据类型。例如，Redis的列表和哈希表都是采用链表结构实现的。而有序集合则采用了跳跃表(Skip List)这种高效的数据结构。

这些数据结构都经过了精心设计和优化，以满足各种场景下的应用需求。例如，链表结构适合频繁地添加和删除元素，而跳跃表结构则适合排序和查找。



本篇博客介绍了Redis支持的五种主要数据类型以及相应的编码方式。

Redis的数据类型和编码方式是为了在不同的场景下达到最佳的性能和内存占用。理解这些类型和编码机制，对于深化我们对Redis的认识，优化其性能，以及发挥其最大潜力是至关重要的。

虽然每个项目的需求和应用可能会有所不同，但通过精心选择和使用合适的类型和编码，我们都可以充分利用Redis为我们的应用带来的高效，快速和可靠。

总之，Redis的类型和编码是其核心功能的基石，理解这些可以帮助我们更好地使用Redis，解决实际问题。当你下次面临需要决定使用哪种数据结构或编码方式的时候，希望你可以记住今天的内容，并从中找到答案。感谢您抽出宝贵的时间阅读这篇文章，希望它对您有所帮助！
