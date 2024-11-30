# 位图法简述

面对海量数据问题时，数据的判重和基数统计是两个绕不开的常见问题。解决的方案我们也有，即布隆过滤器和HyperLogLog。虽然它们节省空间并且效率高，但也付出了一定的代价，即：

- 传统布隆过滤器只能插入元素，不能删除元素。
- HyperLogLog 不保证 100% 准确，提供的是基数的近似估算，存在误差。

这两个缺点可以说是所有概率性数据结构做出的 trade-off，毕竟鱼与熊掌不可兼得。

话说回来，有什么相对高效的能够保证绝对精确的方法呢？最朴素的思路是利用布隆过滤器和 HyperLogLog 的基础——位数组，也叫位图（bitmap）。不妨来看一道老生常谈的面试题：

> 给定含有 40 亿个不重复的位于 [0, 2^32 - 1] 区间内的整数集合，如何快速判定某个数是否在该集合内？

显然，如果我们将这 40 亿个数原样存储下来，需要耗费高达 14.9 GB 的内存，不可接受。

所以我们可以用位图来存储，即第 0 个比特表示数字 0，第 1 个比特表示数字 1，以此类推。如果某个数位于原集合内，就将它对应的位图内的比特置为 1，否则保持为 0。这样就能很方便地查询得出结果了，仅仅需要占用 512 MB 的内存，空间占用不到原来的 3.4%。

由于位图的这个特性，它经常被作为索引用在数据库、查询引擎和搜索引擎中，并且位操作（如 and 求交集、or 求并集）之间可以并行，效率更好。

## 稀疏存储

**但是，位图也不是完美无缺的，不管业务中实际的元素基数有多少，它占用的内存空间都恒定不变**。

位图最大的弊端是稀疏存储，如果数据很分散，基数很高，那么构建的 BitMap 将非常大，然后又会存储很多 bit 位是 0 的数据，造成严重的空间浪费。

举个例子，如果上文题目中的集合只存储了 0 这一个元素，那么该位图只有最低位是 1，其他位全为 0，但仍然占用了 512MB 内存，这就意味着大部分的空间都被浪费了。数据越稀疏，空间浪费越严重。

# Roaring Bitmap

Roaring Bitmap 简称为 RBM，它的出现解决了位图不适应稀疏存储的问题。下文将 Roaring Bitmap 简称为 RBM。

RBM 的历史并不长，它于 2016 年由 S. Chambi、D. Lemire、O. Kaser 等人在论文《Better bitmap performance with Roaring bitmaps》与《Consistently faster and smaller compressed bitmaps with Roaring》中提出。

官网：https://roaringbitmap.org/。

RBM 的主要思路是：将 32 位无符号整数按照高 16 位分桶，即最多可能有 2^16=65536 个桶，论文内称为Container。存储数据时，按照数据的高 16 位找到 Container（找不到就会新建一个），再将低 16 位放入 Container中。也就是说，一个 RBM 就是很多 Container 的集合。

如下所示：


![](https://files.mdnice.com/user/38894/f83b4981-2bc8-47f4-a7da-a04572243a8b.png)

Container 是 RBM 的核心概念，为了更高效地存储和查询数据，RBM 在不同情况下会采用不同类型的 Container。

## Container

RBM 的 Container 一共有三种。

- Array Container

Array Container 是 RBM 初始化默认的 Container。Array Container 适合存放稀疏的数据，其内部数据结构是一个有序的 Short 数组。数组初始容量为 4，最大容量为 4096，数组是动态变化的，当容量不够时，会进行扩容，并且当超过最大容量 4096 时，就会转换为 Bitmap Container。由于数组是有序的，存储和查询时都可以通过二分查找快速定位其在数组中的位置。


![](https://files.mdnice.com/user/38894/0112fe82-0f2d-4ae3-984e-c4e52af99025.png)


例如，0x00020032（十进制 131122 ）放入 RBM 的过程如下图所示：


![](https://files.mdnice.com/user/38894/84f52f15-6a38-47d8-8fa6-fdcdbf6fb98f.png)


第一步，先将该数字转换为 16 进制，131122 对应的十六进制为 0x00020032。其中，高十六位对应 0x0002，首先我们找到 0x0002 所在的桶，再将 131122 的低 16 位存入到对应的 Container 中，131122 的低 16 位转换为 10 进制就是 50，没有超过 Array Container 的容量 4096，所以将低 16 位直接放入到对应的 Array Container 中。

- Bitmap Container

Bitmap Container 底层实现为位图。使用 Long[] 存储位图数据。我们知道，每个 Container 处理 16 位整形的数据，也就是 0~65535，因此根据位图的原理，需要 65536 个比特来存储数据，每个比特位用 1 来表示有，0 来表示无。每个 Long 有 64 位，因此需要 1024 个 Long 来提供 65536 个比特。

因此，每个 Bitmap Container 在构建时就会初始化长度为 1024 的 Long[]，而一个 Long 值占 8 B，因此一个 Bitmap Container 固定占用 8 KB 空间。

Bitmap Container 不用像 Array Container 那样需要二分查找定位位置，而是可以直接通过下标直接寻址。


![](https://files.mdnice.com/user/38894/c0cbbce4-403e-44cc-803e-49604ac4d527.png)

例如，0xFFFF3ACB（十进制 4294916811 ）放入 RBM 的过程如下图所示：


![](https://files.mdnice.com/user/38894/84fdaa77-822a-4459-8d98-1d5b8348d092.png)

0xFFFF3ACB 的前 16 位是 FFFF，找到对应的桶 0xFFFF。在桶对应的 Container 中存储低 16 位，因为 Container 中元素个数已经超过 4096，因此是一个 Bitmap Container。低 16 位为 3ACB（十进制为 15051 ）, 因此在 Bitmap Container 中通过下标直接寻址找到相应的位置，将其置为 1 即可（如上图 15051 的位置）。

- Run Container

Run Container 在初始的 RBM 实现中并没有它，而是在前面介绍的第二篇论文，也就是《Consistently faster and smaller compressed bitmaps with Roaring》中新加入的。

Run Container 中的 Run 指的是行程长度压缩算法（Run Length Encoding），对连续数据有比较好的压缩效果。 它的原理是，对于连续出现的数字，只记录初始数字和后续数量。即：

- 对于数列 11，它会压缩为 11,0。
- 对于数列 11,12,13,14,15，它会压缩为 11,4。
- 对于数列 11,12,13,14,15,21,22，它会压缩为 11,4,21,1。

Run Container 的压缩效果和数据的连续性关系极为密切，考虑极端情况，如果所有数据都是连续的，那么最终只需要 4 字节。如果所有数据都不连续（比如全是奇数或全是偶数），那么不仅不会压缩，还会膨胀成原来的两倍大。所以，RBM 引入 Run Container 是作为其他两种 Container 的折衷方案。

在增删改查的时间复杂度方面，Bitmap Container 只涉及到位运算，显然为 O(1)。而 Array Container 和 Run Container 都需要用二分查找在有序数组中定位元素，故为 O(logN)。

## Container 的创建与转换

在创建一个新 Container 时，如果只插入一个元素，RBM 默认会用 Array Container 来存储。如果插入的是元素序列的话，则会先计算 Array Container 和 Run Container 的空间占用大小，并选择较小的那一种进行存储。

当 Array Container 的容量超过 4096 后，会自动转成 Bitmap Container 存储。4096 这个阈值很聪明，低于它时Array Container 比较省空间，高于它时 Bitmap Container 比较省空间。也就是说 Array Container 存储稀疏数据，Bitmap Container 存储密集数据，可以最大限度地避免内存浪费。


![](https://files.mdnice.com/user/38894/b6931583-9fee-44b1-8375-0e13d9402317.png)


# RBM 和 Bitmap 的区别

1. RBM 只为用到的容器分配空间，解决了稀疏数据浪费空间的问题，并且因为不会开辟大量不用的内存，参与计算的内存块比较少，提升计算速度。
2. 每个容器根据数据的稠密情况使用 array 或 bitmap 数据结构，节省每个容器占用的内存空间。

## 空间占用对比

### 连续数据

分别插入 1w、10w、100w、1000w 条连续数据，并且对比 Bitmap 和 RBM 占用空间的大小。比较结果如下表所示：


![](https://files.mdnice.com/user/38894/3e9ef18c-1c02-471f-8b2f-892981005342.png)


```Java
    @Test
    public void testSizeOfBitMap() {

        //对比占用空间大小 - 10w元素
        RoaringBitmap roaringBitmap3 = new RoaringBitmap();
        byte[] bits2 = new byte[100000];
        for (int i = 0; i < 100000; i++) {
                roaringBitmap3.add(i);
                bits2[i] = (byte) i;
        }
        System.out.println("10w数据 roaringbitmap byte size:"+ roaringBitmap3.getSizeInBytes());
        System.out.println("10w数据 位图数组 byte size:"+bits2.length);

        RoaringBitmap roaringBitmap4 = new RoaringBitmap();
        byte[] bits3 = new byte[1000000];
        for (int i = 0; i < 1000000; i++) {
            roaringBitmap4.add(i);
            bits3[i] = (byte) i;
        }
        System.out.println("100w数据 roaringbitmap byte size:"+ roaringBitmap4.getSizeInBytes());
        System.out.println("100w数据 位图数组 byte size:"+bits3.length);

        RoaringBitmap roaringBitmap5 = new RoaringBitmap();
        byte[] bits4 = new byte[10000000];
        for (int i = 0; i < 10000000; i++) {
            roaringBitmap5.add(i);
            bits4[i] = (byte) i;
        }
        System.out.println("1000w数据 roaringbitmap byte size:"+ roaringBitmap5.getSizeInBytes());
        System.out.println("1000w数据 位图数组 byte size:"+bits4.length);
    }
```

输出：

```Java
10w数据 roaringbitmap byte size:16396
10w数据 位图数组 byte size:100000
100w数据 roaringbitmap byte size:131112
100w数据 位图数组 byte size:1000000
1000w数据 roaringbitmap byte size:1253690
1000w数据 位图数组 byte size:10000000
```

### 稀疏数据

我们知道，位图所占用空间大小只和位图中索引的最大值有关系，现在我们向位图中插入 1 和 999w 两个偏移量位的元素，再次对比 BitMap 和 RBM 所占用空间大小。


![](https://files.mdnice.com/user/38894/fe409b4c-6176-424d-a18a-2ed2ca1196a7.png)


```Java
    @Test
    public void testSize() {
        RoaringBitmap roaringBitmap5 = new RoaringBitmap();
        byte[] bits4 = new byte[10000000];
        for (int i = 0; i < 10000000; i++) {
            if (i == 1 || i == 9999999) {
                roaringBitmap5.add(i);
                bits4[i] = (byte) i;
            }
        }
        System.out.println("两个稀疏数据 roaringbitmap byte size:"+ roaringBitmap5.getSizeInBytes());
        System.out.println("两个稀疏数据 位图数组 byte size:"+bits4.length);
    }
```

输出：

```Java
两个稀疏数据 roaringbitmap byte size:24
两个稀疏数据 位图数组 byte size:10000000
```

# RBM 的应用

RBM 的应用范围极广，在我们熟知的 Druid，Spark，Hive 等中均有所应用。

官方提供了 RBM 的多种语言实现，Java、C/C++、Python、Go、C# 等等一应俱全。

## 代码示例

Java版本见：https://github.com/lemire/RoaringBitmap。

```xml
                <dependency>
                        <groupId>org.roaringbitmap</groupId>
                        <artifactId>RoaringBitmap</artifactId>
                        <version>0.9.16</version>
                </dependency>
```

```java
import org.roaringbitmap.IntConsumer;
import org.roaringbitmap.RoaringBitmap;

public class Basic {

    public static void main(String[] args) {
        RoaringBitmap rr = RoaringBitmap.bitmapOf(1, 2, 3, 1000);
        RoaringBitmap rr2 = new RoaringBitmap();
        rr2.add(4000L, 4255L);
        rr.select(3); // 返回RBM中的第4个值,索引从0开始
        rr.rank(2); // 返回<=x的元素个数
        rr.contains(1000); // RBM是否包含1000,返回true
        rr.contains(7); // 返回false

        RoaringBitmap rror = RoaringBitmap.or(rr, rr2);// new bitmap
        rr.or(rr2); //两个RBM合并
        boolean equals = rror.equals(rr);// true
        if (!equals) throw new RuntimeException("bug");
        // 打印元素的个数
        long cardinality = rr.getLongCardinality();
        System.out.println(cardinality);
        rr.forEach((IntConsumer) System.out::println);
    }
}
```

# 巨人的肩膀

- https://blog.csdn.net/u011250186/article/details/109168967
- https://roaringbitmap.org/
- https://github.com/lemire/RoaringBitmap
- https://blog.csdn.net/SHWAITME/article/details/140742696
- https://cloud.tencent.com/developer/article/1481855
- https://blog.csdn.net/penriver/article/details/119736050
- https://www.cnblogs.com/Jcloud/p/17250205.html
- https://segmentfault.com/a/1190000044441208