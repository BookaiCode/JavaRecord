> 不知道自己的无知，乃是双倍的无知。——柏拉图

[TOC]

## 跨代引用问题

**跨代引用是指新生代中存在对老年代对象的引用，或者老年代中存在对新生代的引用。**

假如要现在进行一次只局限于新生代区域内的收集（Minor GC），但新生代中的对象是完全有可能被老年代所引用的，为了找出该区域中的存活对象，不得不在固定的 GC Roots 之外，再额外遍历整个老年代中所有对象来确保可达性分析结果的正确性，反过来也是一样。无疑会为内存回收带来很大的性能负担。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScPeYWz4ibOQDZE50x2UUm1EoL6HmoNx2NNHQ7oneBDNlYBHWSUPAFZeAh4LZXvlUdfY82dN7DsHZpQ/0?wx_fmt=jpeg)

别慌，JVM的设计者已经考虑了这个场景，并想到了解决办法，那就是使用一种叫做：`记忆集（Remembered Set）`的数据结构。

## 记忆集

**记忆集位于新生代中。用以避免把整个老年代加进GC Roots扫描范围**。

**记忆集的作用和我们之前讲的OopMap很相似，维护了类似一种映射表的关系，避免了全局扫描，本质是用空间换时间**。

记忆集是一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构。**注意这里的说辞：抽象。意思就是说记忆集是一种逻辑上的概念，并没有规定具体的实现，类似方法区。下文我们会说到卡表，可以把记忆集和卡表的关系理解为Map跟HashMap**。

## 卡表

**卡表可以理解为是记忆集的具体实现。英文叫：Card Table**

垃圾收集器只需要通过记忆集判断出某一块非收集区域是否存在有指向了收集区域的指针就可以了，并不需要了解这些跨代指针的全部细节。那设计者在实现记忆集的时候，便可以选择更为粗犷的记录粒度来节省记忆集的存储和维护成本，下面列举了一些可供选择（当然也可以选择这个范围以外的）的记录精度：

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPeYWz4ibOQDZE50x2UUm1EobEzN6ammIY6mzfRr6fWiam070osjwXAOgvVVb48fNZ05mxdVUasibZLQ/0?wx_fmt=png)

**其中，第三种“卡精度”所指的就是“卡表”的方式去实现记忆集 ，这也是目前最常用的一种记忆集实现形式，HotSpot采用的就是卡表**。

在HotSpot虚拟机里面，卡表采用的是字节数组的形式。以下这行代码是HotSpot默认的卡表标记逻辑 ：

```
CARD_TABLE [this address >> 9] = 0;
```

**字节数组CARD_TABLE的每一个元素都对应着其标识的内存区域中一块特定大小的内存块，这个内存块被称作“卡页”（Card Page）**。一般来说，卡页大小都是以2的N次幂的字节数，通过上面代码可以看出HotSpot中使用的卡页是`2的9次幂`，即512字节。那如果卡表标识内存区域的起始地址是0x0000的话，数组CARD_TABLE的第0、1、2号元素，分别对应了地址范围为0x0000～0x01FF、0x0200～0x03FF、0x0400～0x05FF的卡页内存块 ，如图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPeYWz4ibOQDZE50x2UUm1EoQQ9kgkhRO0wibG8MPgiawaAKTxTD6pL29qqhIQ5aIaT4EfhyhaFdcjcg/0?wx_fmt=png)

**一个卡页的内存中通常包含不止一个对象，只要卡页内有一个（或更多）对象的字段存在着跨代指针，那就将对应卡表的数组元素的值标识为1，称为这个元素变脏（Dirty），没有则标识为0。在垃圾收集发生时，只要筛选出卡表中变脏的元素，就能轻易得出哪些卡页内存块中包含跨代指针，把它们加入GC Roots中一并扫描。**



**简单来说，就是卡页的字节数组只有0和1两种状态，1表示哪些内存区域存在跨代指针，那么只要把1的加入GC Roots中一并扫描，就能知道哪些进行跨代引用了，这样就不用挨个去扫描了**。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScPeYWz4ibOQDZE50x2UUm1Eo47iay5u4oZXlA3MXm7Xa9yfK46bXNmsOqlJgVqAssibWzXnlLwkpOxtQ/0?wx_fmt=jpeg)

**OK，我们还剩下一个问题，这个问题OopMap也遇到过。卡表元素如何维护？何时变脏、谁来把它们变脏等。**

**HotSpot解决的办法是使用写屏障**。

## 写屏障

先来解决何时变脏的问题，这个问题很简单，即**其他分代区域中对象引用了本区域对象时，其对应的卡表元素就应该变脏，变脏时间点原则上应该发生在引用类型字段赋值的那一刻**。

但问题是如何变脏，即如何在对象赋值的那一刻去更新维护卡表，在HotSpot虚拟机里是通过`写屏障（Write Barrier）`解决的。

> 注意：这里提到的 写屏障 和 volatile 的写屏障不是一回事。

**写屏障可以看作在虚拟机层面对“引用类型字段赋值”这个动作的AOP切面，在引用对象赋值时会产生一个环形（Around）通知。用过Spring的弟兄们对AOP肯定不陌生。**

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScPeYWz4ibOQDZE50x2UUm1EoaiatnrDldCqF0tKpn4pN5WpEkgyibgkQjfbGFw9v6K00pyiaQAzEfjElQ/0?wx_fmt=jpeg)

在赋值前的部分的写屏障叫作`写前屏障（Pre-Write Barrier）`，在赋值后的则叫作`写后屏障（Post-Write Barrier）`。HotSpot虚拟机的许多收集器中都有使用到写屏障，**但直至G1收集器出现之前，其他收集器都只用到了写后屏障。**

应用写屏障后，虚拟机就会为所有赋值操作生成相应的指令，一旦收集器在写屏障中增加了更新卡表操作，无论更新的是不是老年代对新生代对象的引用，每次只要对引用进行更新，就会产生额外的开销，不过这个开销与Minor GC时扫描整个老年代的代价相比还是低得多的。

## 写屏障的伪共享问题

卡表在高并发场景下还面临着**“伪共享”（False Sharing）**问题。伪共享是处理并发底层细节时一种经常需要考虑的问题，号称并发的`隐形杀手`，现代中央处理器的缓存系统中是以缓存行（Cache Line）为单位存储的，当多线程修改互相独立的变量时，如果这些变量恰好共享同一个缓存行，就会彼此影响（写回、无效化或者同步）而导致性能降低，这就是伪共享问题。

**为了避免伪共享问题，一种简单的解决方案是不采用无条件的写屏障，而是先检查卡表标记，只有当该卡表元素未被标记过时才将其标记为变脏**，即将卡表更新的逻辑变为以下代码所示：

相当于说就是多了一个if判断条件。

```c++
if (CARD_TABLE [this address >> 9] != 0)
CARD_TABLE [this address >> 9] = 0;
```

在JDK 7之后，HotSpot虚拟机增加了一个新的参数`-XX：+UseCondCardMark`（默认是关闭的），用来决定是否开启卡表更新的条件判断。开启会增加一次额外判断的开销，但能够避免伪共享问题，两者各有性能损耗，是否打开要根据应用实际运行情况来进行测试权衡。

------

如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScPibyOvOuNiasKa7qicaZgo5DIcDAickDKoU6KZUmLyibpnRc6ibzTxT9WAnkfPhFcq6iamGRo2ITZlPPczA/0?wx_fmt=jpeg)
