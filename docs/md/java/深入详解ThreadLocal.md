在我们日常的并发编程中，有一种神奇的机制在静悄悄地为我们解决着各种看似棘手的问题，它就是「**ThreadLocal**」。

这个朴素却强大的工具，许多Java开发者可能并没有真正了解过其内部运作原理和应用场景。

本篇文章，我将和大家一起探索 JDK 中这个独特而又强大的类——ThreadLocal。

透过本文，我们将揭开它神秘的面纱，并深入理解它是如何优雅处理线程级别的数据隔离，以及在实际开发中如何有效地利用它。

话不多说，我们进入正题。

## 什么是ThreadLocal

ThreadLocal是Java中的一个类，**它提供了一种线程绑定机制，可以将状态与线程（Thread）关联起来。每个线程都会有自己独立的一个ThreadLocal变量，因此对该变量的读写操作只会影响到当前执行线程的这个变量，而不会影响到其他线程的同名变量**。

我们先来看一个简单的ThreadLocal使用示例：

```java
public class ThreadLocalTest {
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();
 
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            threadLocal.set("本地变量1");
            print("thread1");
            System.out.println("线程1的本地变量的值为:"+threadLocal.get());
        });
 
        Thread thread2 = new Thread(() -> {
            threadLocal.set("本地变量2");
            print("thread2");
            System.out.println("线程2的本地变量的值为:"+threadLocal.get());
        });
 
        thread1.start();
        thread2.start();
    }
 
    public static void print(String s){
        System.out.println(s+":"+threadLocal.get());
     
    }

```

执行结果如下：

```
thread2:本地变量2
thread1:本地变量1
线程2的本地变量的值为:本地变量2
线程1的本地变量的值为:本地变量1
```

通过上面的例子，我们可以很轻易的看出，ThreadLocal消除了不同线程间共享变量的需求，可以用来实现「**线程局部变量**」，从而避免了多线程同步（synchronization）的问题。

OK，下面开始讲解ThreadLocal，讲ThreadLocal之前，我们得先从 `Thread` 类讲起。

在 `Thread` 类中有维护两个 `ThreadLocal.ThreadLocalMap` 对象，分别是：`threadLocals` 和`inheritableThreadLocals`。

源码如下：

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

/*
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

初始它们都为 null，只有在调用 `ThreadLocal` 类的 set 或 get 时才创建它们。ThreadLocalMap可以理解为线程私有的HashMap。

**ThreadLoalMap是ThreadLocal中的一个静态内部类**，是一个类似HashMap的数据结构，但并没有实现Map接口。

ThreadLoalMap中初始化了一个「**大小16的Entry数组**」，Entry对象用来保存每一个key-value键值对。key是ThreadLocal对象。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCB0rB5SDdAcNen8DvZ91ian9XnQwhjjICPsMGtKNfY9fh2CcJFsBVVuew/640)

有图有真相，源码中的定义如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCBvjfA3nLiaCtcQjoInJ9Gs5iaUlVAOuxdrkXIias6kcLk4JCPQ3iaezWiaQQ/640)

细心的你肯定发现了，Entry继承了「**弱引用（WeakReference）**」。在Entry内部使用ThreadLocal作为key，使用我们设置的value作为value。

## ThreadLocal 原理

ThreadLocal中我们最常用的肯定是`set()`和`get()`方法了，所以先从这两个方法入手。

### set方法

**当我们调用 ThreadLocal 的 `set()` 方法时实际是调用了当前线程的 ThreadLocalMap 的 set() 方法。**

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPIlWKwgbQ65sKiaib8iapt1NbvgPzggrd3O6mYhN1XOPJ6ic2Gdyl8Cleec7iaG5owyWtmh7qPY8BZ1vA/0)

ThreadLocal 的 set() 方法中，会进一步调用`Thread.currentThread()` 获得当前线程对象 ，然后获取到当前线程对象的ThreadLocalMap，判断是不是为空。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPIlWKwgbQ65sKiaib8iapt1NbpibZMdAMY6HWBJ7jQBAMEfDUHDiaDRruZKWqbXTicTicf6uUraG3rRbD8g/0)

为空就先调用`creadMap()`创建 ThreadLocalMap 对象，在构造参数里set进变量。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPIlWKwgbQ65sKiaib8iapt1Nbia6lr2ic09icJoicDhgZ6uB6xyZZpmSgAYicm3ibHQWaI80HZ47vROSRl3Lg/0)

不为空就直接`set(value)` 。

这种保证线程安全的方式有个专业术语，称为「**线程封闭**」，线程只能看到自己的ThreadLocal变量。线程之间是互相隔离的。

### get方法

`get()`方法用来获取与当前线程关联的ThreadLocal的值。

如果当前线程没有该ThreadLocal的值，则调用「**initialValue函数**」获取初始值返回，所以一般我们使用时需要继承该函数，给出初始值（不重写的话默认返回Null）。

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

get方法的流程主要是以下几步：

1. 获取当前的Thread对象，通过getMap获取Thread内的ThreadLocalMap。
2. 如果map已经存在，以当前的ThreadLocal为键，获取Entry对象，并从从Entry中取出值。
3. 否则，调用setInitialValue进行初始化。

我们可以重写`initialValue()`，设置初始值，具体写法如下：

```java
private static final ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>(){
     @Override
     protected Integer initialValue() {
     return Integer.valueOf(0);
     }
}
```

**推荐设置初始值，如果不设置为null，在某些情况下会引发空指针的问题。**

### remove方法

最后一个需要探究的就是`remove()`方法，它用于在map中移除一个不用的Entry。

先计算出hash值，若是第一次没有命中，就循环直到null，在此过程中也会调用「**expungeStaleEntry**」清除空key节点。代码如下：

```java
public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
}


/**
 * Remove the entry for key.
 */
private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
}
```

上面我们看了ThreadLocal的源码，我们知道 ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，**弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉**。

所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

出现「**内存泄漏**」的问题。

**其实在执行 ThreadLocal 的 set、remove、rehash 等方法时，它都会扫描 key 为 null 的 Entry，如果发现某个 Entry 的 key 为 null，则代表它所对应的 value 也没有作用了，所以它就会把对应的 value 置为 null，这样，value 对象就可以被正常回收了。**

但是假设 ThreadLocal 已经不被使用了，那么实际上 set、remove、rehash 方法也不会被调用，与此同时，如果这个线程又一直存活、不终止的话，那么刚才的那个调用链就一直存在，也就导致了内存泄漏。

## ThreadLocal 的Hash算法

`ThreadLocalMap`类似HashMap，它有自己的Hash算法。

```java
private final int threadLocalHashCode = nextHashCode();

private static final int HASH_INCREMENT = 0x61c88647;
  
private static int nextHashCode() {
  return nextHashCode.getAndAdd(HASH_INCREMENT);
}
    
public final int getAndAdd(int delta) {
  return unsafe.getAndAddInt(this, valueOffset, delta);
}

```

`HASH_INCREMENT`这个数字被称为「**斐波那契数**」 也叫 「**黄金分割数**」，其中的数学原理我们不去纠结，

我们只需知道用斐波那契数去散列，带来的好处就是 hash分布非常均匀。

每当创建一个`ThreadLocal`对象，这个`ThreadLocal.nextHashCode` 这个值就会增长 `0x61c88647` 。

讲到Hash就会涉及到Hash冲突，跟HashMap通过「**链地址法**」不同的是，ThreadLocal是通过「**线性探测法/开放地址法**」来解决hash冲突。

## ThreadLocal 1.7和1.8的区别

ThreadLocal 1.7版本的时候，entry对象的key是Thread。到了1.8版本entry的key是ThreadLocal。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCBCeo69p8WuBjzmSlhicCbS3k03WmgM7lXC5pGWfjUY7HgBekx9posy6w/640)

1.8版本的好处是当Thread销毁的时候，ThreadLocalMap也会随之销毁，减少内存的使用。因为ThreadLocalMap是Thread的内部类，所以只要Thread消失了，那ThreadLocalMap就不复存在了。

## ThreadLocal 的问题

### ThreadLocal 内存泄露问题

在 ThreadLocalMap 中的 Entry 的 key 是对 ThreadLocal 的 `WeakReference` 弱引用，而 value 是强引用。

注意构造函数里的第一行代码super(k)，这意味着ThreadLocal对象是一个弱引用

```java
/**
 * The entries in this hash map extend WeakReference, using
 * its main ref field as the key (which is always a
 * ThreadLocal object).  Note that null keys (i.e. entry.get()
 * == null) mean that the key is no longer referenced, so the
 * entry can be expunged from table.  Such entries are referred to
 * as "stale entries" in the code that follows.
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

其实要解决也简单，只需要使用完 ThreadLocal 后手动调用 `remove()` 方法。

但其实在 ThreadLocalMap 的实现中以及考虑到这种情况，因此在调用 `set()`、`get()`、`remove()` 方法时，也会清理 key 为 null 的记录。

### 为什么使用弱引用而不是强引用?

为什么采用了弱引用的实现而不是强引用呢？

这个问题在源码的注释上有说明，我们来瞅瞅。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPIlWKwgbQ65sKiaib8iapt1NbkRof7HnHK3alZP02QCxiaxyAulEHzNb5Ndoc4hVSxKq3MPZNr8miaULg/0)

原谅我英语不好，这段话使用的某道翻译，翻译如下：

**为了协助处理数据比较大并且生命周期比较长的场景，hash table的entry使用了WeakReference作为key。**

**所以，貌似看起来弱引用反而是为了解决内存存储问题而专门使用的？**

仔细思考一下，实际上，采用弱引用反而多了一层保障。

如果ThreadLocal作为key使用强引用，那么只要ThreadLocal实例本身在内存中，它的entry（包括value）就会始终存在于ThreadLocalMap中，即使线程已经不再需要它的ThreadLocal变量。它们也不会被回收，这导致了一种形式的内存泄漏。

**而如果我们使用WeakReference作为key。这意味着当对ThreadLocal实例的所有强引用都被垃圾收集器清除后，它的entry（包括value）也可以从ThreadLocalMap中清除，防止了潜在的内存泄漏。**

这样设计让ThreadLocal生命周期的控制权交给了用户，用户可以选择什么时候结束ThreadLocal实例的生命周期。

**ThreadLocal被清理后key为null，对应的value在下一次ThreadLocalMap调用set、get时清除，这样就算忘记调用 remove 方法，弱引用比强引用可以多一层保障。**

所以，内存泄露的根本原因在于是否手动清除操作，而不是弱引用。

## ThreadLocal 父子线程继承

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

/*
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

Thread类里一共有两个ThreadLocal变量，一个是threadLocals，也就是我们常说的threadlocal，还有一个是inheritableThreadLocals，这个是干啥用的呢？

**inheritableThreadLocals是用于父子线程之间继承的，这个变量用来存储那些需要被子线程继承的变量。**

**并且子线程可以访问和修改这个变量的值，而不会影响到父线程对应变量的值。**

异步场景下无法给子线程共享父线程的线程副本数据，可以通过 「**InheritableThreadLocal**」类解决这个问题。

它的原理就是当新建一个线程对象时，子线程是通过在父线程中调用 `new Thread()` 创建的，在 Thread 的构造方法中调用了 Thread的`init() `方法。

在` init() `方法中父线程的「**inheritableThreadLocals**」数据会复制到子线程：

```java
ThreadLocal.createInheritedMap(parent.inheritableThreadLocals) 
```

下面是一个简单的示例代码：

```
public class Main {
    public static void main(String[] args) {
        // 在主线程中设置 InheritableThreadLocal
        InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();
        inheritableThreadLocal.set("Hello from the main thread");

        // 创建新的线程
        Thread thread = new Thread(() -> {
            // 子线程尝试获取 InheritableThreadLocal 的值
            String value = inheritableThreadLocal.get();
            System.out.println("In child thread, value from InheritableThreadLocal: " + value);
        });

        thread.start();  // 启动子线程
        
        try {
            thread.join();  // 等待子线程结束
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("In main thread, value from InheritableThreadLocal: " + inheritableThreadLocal.get());
    }
}
```

以上代码首先在主线程中创建了一个 `InheritableThreadLocal` 对象，并设置了其值。然后，该代码创建并启动了一个新的线程，在新线程中尝试读取 `InheritableThreadLocal` 的值。因为 `InheritableThreadLocal` 的值是从父线程继承的，所以新线程能够读取到在主线程中设置的值。

输出如下：

```
In child thread, value from InheritableThreadLocal: Hello from the main thread
In main thread, value from InheritableThreadLocal: Hello from the main thread
```

**请注意，这种继承是一次性的，只在创建新线程的那一刻发生，之后父子线程对 `InheritableThreadLocal` 的修改就互不影响了。**

同时，由于使用的是浅拷贝，所以如果 `InheritableThreadLocal` 的值是可变对象，那么依然可能存在多个线程共享数据的情况。

但是我们做异步处理一般使用线程池，线程池会复用线程，所以`InheritableThreadLocal` 在线程池场景中会失效。

不过网上有开源框架，我们可以使用阿里巴巴的TTL解决这个问题：https://github.com/alibaba/transmittable-thread-local。

## 探测式清理 & 启发式清理

能把上面那些知识消化完，足够应付90%的面试和工作场景了。

但是也只能让面试官觉得你有点东西，但不是很多。如果想让面试官直呼牛B，那咱就得来聊聊「**探测式清理**」和「 **启发式清理**」了。

ThreadLocal 使用了两种清理无效条目（即键为 null 的条目）的方式：探测式清理和启发式清理。

- 探测式清理（源码中：expungeStaleEntry() 方法 ）
- 启发式清理（源码中：cleanSomeSlots() 方法 ）

### 探测式清理

这种清理方法基于一个事实：在查找特定键时，如果遇到无效条目（即键为null的条目），可以安全地删除它，因为它肯定不是正在寻找的键。

以下是`expungeStaleEntry()` 方法的源码：

```java
private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }
```

简单叙述下源码说了什么：

遍历散列数组，从开始位置（hash得到的位置）向后探测清理过期数据，如果遇到过期数据，则置为null。

如果碰到的是未过期的数据，则将此数据`rehash`，然后重新在 table 数组中定位。

如果定位的位置已经存在数据，则往后顺延，直到遇到没有数据的位置。

**说白了就是**：从当前节点开始遍历数组，将key等于null的entry置为null，key不等于null则rehash重新分配位置，若重新分配上的位置有元素则往后顺延。

### 启发式清理

启发式清理需要接收两个参数：

1. 探测式清理后返回的数字下标。
2. 数组总长度。

`cleanSomeSlots()`源码：

```java
private boolean cleanSomeSlots(int i, int n) {
            boolean removed = false;
            Entry[] tab = table;
            int len = tab.length;
            do {
                i = nextIndex(i, len);
                Entry e = tab[i];
                if (e != null && e.get() == null) {
                    n = len;
                    removed = true;
                    i = expungeStaleEntry(i);
                }
            } while ( (n >>>= 1) != 0);
            return removed;
        }
```

根据源码可以看出，启动式清理会从传入的下标 `i` 处，向后遍历。如果发现过期的Entry则再次触发探测式清理，并重置 `n`。

这个n是用来控制 `do while` 循环的跳出条件。如果遍历过程中，连续 `m` 次没有发现过期的Entry，就可以认为数组中已经没有过期Entry了。

这个 `m` 的计算是 `n >>>= 1` ，可以理解为是数组长度的2的几次幂。

例如：数组长度是16，那么2^4=16，也就是连续4次没有过期Entry。

**说白了就是：** 从当前节点开始，进行do-while循环检查清理过期key，结束条件是连续`n`次未发现过期key就跳出循环，n是经过位运算计算得出的，可以简单理解为数组长度的2的多少次幂次。

### 触发时机

这两种清理方式会在源码中多个位置被触发。

下面的触发场景中，我都从源码中找到了对应的位置，直接对号入座即可，有兴趣的可以去深入阅读这部分的源码。

- set() 方法中，遇到key=null的情况会触发一轮探测式清理流程。
- set() 方法最后会执行一次启发式清理流程。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCByzK7TMc93APXibrpCoCK3WIWu409KibkE9ibuEovEd1H94urfLNXokWfQ/640)

- rehash() 方法中会调用一次探测式清理流程。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCBUiavgtgTT8epVRI7z921dcYSiaLnCS4mZOxuqpkV7CBmLHJqKjxBgEjQ/640)

- get() 方法中遇到key过期的时候会触发一次探测式清理流程。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCBLJQAvJz0YNgfeATcx6WV0uRS5f8nvLEVkvdiaibHPHWjZlxiceg0oCl3A/640)

- 启发式清理流程中遇到key=null的情况也会触发一次探测式清理流程。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOt4Libmz1bw1WwRF0kfXiaCBbpLcXHcjKibPee9IzLTdTsbUNQBmSbia8LD50ObaIqlRFo7q2aticu8RQ/640)



最后，给本篇文章做个总结。

## 总结

ThreadLocal是Java提供的一种非常有用的工具，它可以帮助我们在每个线程中存储并管理各自独立的数据副本。这种特性使得ThreadLocal在处理多线程编程中的某些问题时极为高效且易于使用，例如实现线程安全、维护线程间的数据隔离等。

然而，ThreadLocal也要谨慎使用，因为不正确的使用可能会导致内存泄漏。特别是在使用完ThreadLocal后，我们需要记住及时调用其remove()方法清理掉线程局部变量，防止对已经不存在的对象的长时间引用，引发内存泄漏。

总的来说，ThreadLocal具有强大的功能，但必须了解其工作原理和可能的风险，才能充分利用它而不会产生意料之外的问题。因此, 深入理解并合理使用ThreadLocal是每个Java开发者的必备技能。



本篇文章到这结束了~，那就下次再见吧👋🏻，觉得有收获点个赞哦。