[TOC]

## 摘要

Redis是一款开源的高性能key-value数据库，广泛应用于各种场景。在Redis中，**数据类型（type）和编码（encoding）**是非常重要的概念。本篇博客将详细介绍Redis支持的数据类型以及相应的编码方式和底层实现原理。



要查看Redis某个key的内部编码，可以使用Redis命令`OBJECT ENCODING key`。其中，`key`是你想要查询的键名。例如，如果你想要查询名为`mykey`的键的内部编码，可以执行以下命令：

```sh
127.0.0.1:6379> object encoding mykey  // 查看某个Redis键值的编码
```

## redisObject

在 Redis 中，redisObject 是 Redis 中最基本的数据结构之一。redisObject 用于表示 Redis 中的键值对中的值，它可以是字符串、整数、列表、哈希表等任意一种 Redis 数据类型。

redisObject 的定义如下：

```c
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码方式
    unsigned encoding:4;

    // 引用计数
    int refcount;

    // 指向实际值的指针
    void *ptr;

} robj;
```

- type：表示 redisObject 的类型。
- encoding：表示 redisObject 的编码方式。
- refcount：表示当前 redisObject 被引用的次数。
- ptr： ptr字段则是一个指针，指向实际的 Redis 对象。

Redis源码encoding取值有如下几种：

```c
#define OBJ_ENCODING_RAW 0        /* Raw representation */
#define OBJ_ENCODING_INT 1        /* Encoded as integer */
#define OBJ_ENCODING_HT 2         /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3     /* Encoded as zipmap */
#define OBJ_ENCODING_LINKEDLIST 4 /* No longer used: old list encoding. */
#define OBJ_ENCODING_ZIPLIST 5    /* Encoded as ziplist */
#define OBJ_ENCODING_INTSET 6     /* Encoded as intset */
#define OBJ_ENCODING_SKIPLIST 7   /* Encoded as skiplist */
#define OBJ_ENCODING_EMBSTR 8     /* Embedded sds string encoding */
#define OBJ_ENCODING_QUICKLIST 9  /* Encoded as linked list of ziplists */
```

## 类型与编码介绍

Redis支持五种主要的数据类型：字符串(string)、列表(list)、集合(set)、有序集合(sorted set)和哈希(hash)。每种数据类型都有对应的编码方式。

数据类型与编码方式总览如下：

| 数据类型 | 编码方式                       |
| :------- | :----------------------------- |
| 字符串   | int、embstr、raw               |
| 哈希表   | ziplist、hashtable             |
| 列表     | ziplist、linkedlist、quicklist |
| 集合     | intset、hashtable              |
| 有序集合 | ziplist、skiplist              |

### 字符串

字符串是Redis中最基本的数据类型，通常用于存储文本或二进制数据。Redis支持两种编码方式：

- int：当字符串可以表示为整数时，Redis会将其转换为整数，并采用int编码方式存储。int编码方式的优点是存储空间小，操作效率高。缺点是只能存储整数，不支持字符串操作。
- embstr(embstr-encoded string)：**保存长度小于44字节的字符串**，当一个字符串比较短，采用此编码方式存储，可以减少内存占用。
- raw(raw-encoded string)：**保存长度大于44字节的字符串**，当一个字符串比较长时，采用此编码方式存储。

### 列表

列表是一系列有序的字符串集合，可以添加、修改和删除元素。Redis支持三种编码方式：

- ziplist：在Redis3.2版本之前，**当List列表中每个字符串的长度都小于64字节并且List列表中元素数量小于512个时**，List对象使用ziplist编码，其他情况使用linkedlist编码。ziplist是一种紧凑的、压缩的列表结构，可以节省内存。适用于小型列表。
- linkedlist：linkedlist是一种链表结构，支持任意大小的列表。但其内存占用会随着列表长度的增加而增加。
- quicklist：**Redis 3.2版本引入**，quicklist是一种由多个ziplist组成的列表结构，既能保证性能，又能节省内存。适用于大型列表。

### 集合

集合是一系列无序的字符串集合，支持添加、删除和查询元素。Redis支持两种编码方式：

- intset：**当集合中的元素都是整数时，Redis会采用intset编码方式存储**。intset编码方式的优点是存储空间小，操作效率高。
- hashtable：**当集合中的元素包含字符串时，Redis会采用hashtable编码方式存储**。hashtable编码方式的优点是可以存储任意类型的元素，支持字符串操作。缺点是存储空间相对较大，操作效率相对较低。

### 有序集合

有序集合是一系列无序的字符串集合，每个元素关联一个分数，可以根据分数排序。Redis支持两种编码方式：

- ziplist：**保存的元素少于128个并且所有元素大小都小于64字节使用ziplist编码**，ziplist是一种紧凑的、压缩的列表结构，适用于小型有序集合。
- skiplist：skiplist是一种跳跃表结构，支持快速查询和排序。适用于大型有序集合。

### 哈希表

哈希表是一系列键值对集合，每个键关联一个值。Redis支持两种编码方式：

- ziplist：**哈希对象保存的所有键值的字符串长度小于64字节并且键值对数量小于512个**，Redis会采用ziplist编码方式存储。ziplist编码方式的优点是存储空间小，操作效率高。缺点是不支持快速的键查找操作。
- hashtable：除上述条件之外，Redis会采用hashtable编码方式存储。hashtable编码方式的优点是支持快速的键查找操作。缺点是存储空间相对较大，操作效率相对较低。

## 类型与编码底层原理

了解Redis支持的数据类型和编码方式后，我们来看一下它们的底层实现原理。

### 编码转换

Redis中的每个键值对都有一个类型标识，表示该键值对的数据类型。当我们对一个键进行操作时，Redis会根据该键当前的编码方式以及操作所需的编码方式，对键值对进行编码转换。

例如，当我们向一个字符串中追加内容时，如果该字符串当前的编码方式为raw，但是新的内容可以使用embstr编码方式存储，那么Redis会将该字符串的编码方式从raw转换为embstr。

### 数据结构

除了编码方式外，Redis还使用了许多经典的数据结构来实现各种数据类型。例如，Redis的列表和哈希表都是采用链表结构实现的。而有序集合则采用了跳跃表(Skip List)这种高效的数据结构。

这些数据结构都经过了精心设计和优化，以满足各种场景下的应用需求。例如，链表结构适合频繁地添加和删除元素，而跳跃表结构则适合排序和查找。

## 总结

本篇博客介绍了Redis支持的五种主要数据类型以及相应的编码方式。Redis的数据类型和编码方式是为了在不同的场景下达到最佳的性能和内存占用。在使用Redis时，需要根据实际情况选择合适的数据类型和编码方式，以达到最佳的效果。同时，需要注意不同数据类型和编码方式的优缺点，以便在实际使用中做出合理的选择。

------

本篇文章就到这里，感谢阅读，如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。  ![](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)
