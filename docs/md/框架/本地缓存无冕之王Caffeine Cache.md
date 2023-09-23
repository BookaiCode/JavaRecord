在常规的软件开发流程中，缓存的重要性日益凸显。它不仅为用户带来了更迅速的反馈时间，还能在大多数情况下有效减轻系统负荷。

本篇文章将详述一个本地缓存框架：「**Caffeine Cache**」。

Caffeine Cache以其高性能和可扩展性赢得「**本地缓存之王**」的称号，它是一个Java缓存库。它的设计目标是优化计算速度、内存效率和实用性，以符合现代软件开发者的需求。

Spring Boot 1.x版本中的默认本地缓存是Guava Cache。在 Spring5 （SpringBoot 2.x）后，Spring 官方放弃了 Guava Cache 作为缓存机制，而是使用性能更优秀的 Caffeine 作为默认缓存组件，这对于Caffeine来说是一个很大的肯定。

接下来，我们会详细介绍 Caffeine Cache 的特性和应用，并将这个高效的缓存工具无缝集成到你的项目中。

## 淘汰算法

在解析Caffeine Cache之前，我们首先要理解缓存淘汰算法。一个优秀的淘汰算法能使效率大幅提升，因为缓存过程总会伴随着数据的淘汰。

- **FIFO（First In First Out）先进先出：**

以时序为基准，先进入缓存的数据会被先淘汰。当缓存满时，把最早放入缓存的数据淘汰掉。

优点：实现简单，对于某些不常重复请求的应用效果较好。 

缺点：并未考虑到数据项的访问频率和访问时间，可能淘汰的是最近和频繁访问的数据。

- **LRU（Least Recently Used）最近最久未使用：**

此算法根据数据的历史访问记录来进行决策，最久未被访问的数据将被淘汰。LRU通过维护一个所有缓存项的链表，新数据插入到链表的头部，如果缓存满了，就会从链表尾部开始移除数据。

优点：LRU考虑了最近的数据访问模式，对于局部性原理的表现优秀，简单实用。 

缺点：不能体现数据的访问频率，如果数据最近被访问过，即使访问频度低也不会被淘汰。比如，如果一个数据在一分钟的前59秒被频繁访问，而在最后一秒无任何访问，但是有一批冷门数据在最后一秒进入缓存，那么热点数据可能会被淘汰掉。

- **LFU（Least Frequently Used）最近最少频率使用：**

其基本原理是对每个在缓存中的对象进行计数，记录其被访问的次数。当缓存满了需要淘汰某些对象时，LFU算法会优先淘汰那些被访问次数最少的对象。

优点：LFU能够较好地处理长期访问稳定、频率较高的情况，因为这样可以确保频繁访问的对象不容易被淘汰。

缺点：对于一些暂时高频访问但之后不再访问的对象，LFU无法有效处理。因为这些对象的访问次数已经非常高，之后即使不再访问，也不容易被淘汰，可能造成缓存空间的浪费。并且LFU需要维护所有对象的访问计数，这可能会消耗比较多的存储空间和计算资源。

- **W-TinyLFU（ Window Tiny Least Frequently Used）：**

**Caffeine 使用的就是  Window TinyLfu 淘汰策略，此策略提供了一个近乎最佳的命中率。**

看名字就能大概猜出来，它是 LFU 的变种，它在面临缓存换页（即缓存空间不足而需要替换旧缓存项）问题时，通过统计频率信息来选择最佳的候选项进行替换。

工作原理：

1. 频率滤波：W-TinyLFU 使用一个小型的滑动窗口记录最近访问过的对象，以捕获对象的使用频率。这个窗口内的数据被插入到一个 LFU 计数器中，该计数器基于频率清除最少使用的对象。
2. 突发性适应：W-TinyLFU 还包含一个 Admission Window，用于对新添加到缓存的项进行跟踪，以便能够处理突然出现的新热点数据。
3. 替换策略：当缓存满了，且有新的元素需要加入时，W-TinyLFU 使用频率信息选择最少使用的条目进行替换。如果新条目的使用频率较高，那么将替换掉使用频率较低的老条目；如果新项的使用频率较低，则可能会被拒绝。



相较于传统的 LRU 和 LFU 策略，W-TinyLFU具有以下优点：

1. 平衡了最近性和频率：与 LRU 相比，W-TinyLFU 不仅考虑了最近使用的情况，还计算了缓存的热门程度。与 LFU 相比，它不会让长时间以前非常热门但现在很少使用的数据占据大量的空间。
2. 计数器限制：TinyLFU 使用一个固定大小的计数滤波器来跟踪访问频率，这使得其内存占用远低于传统的 LFU 策略。
3. 适应性强：W-TinyLFU 可以更好地适应工作负载的变化，因为它对频率的计数有一个时间窗口。这使得 W-TinyLFU 能够避免过多地重视早期的访问模式，并能更快地适应最近的访问模式。
4. 避免缓存污染：由于它维护了一个 admission window，它可以避免一次性的、大规模的请求可能带来的缓存污染。

缺点：

1. 需要维护额外的频率信息，增加了一些开销。
2. 不如 LRU 算法实现简单。对于不同的使用场景，需要调整参数以获得最佳性能。

总的来说，W-TinyLFU 是一个复杂性高、灵活性强的缓存算法，对于识别和处理长期和突发的热数据表现良好，但相比于更简单的算法如 LRU，它需要更多的资源和精细的配置。

## Cache类型

Caffeine共提供了四种类型的Cache，对应着四种加载策略。

### Cache

最普通的一种缓存，无需指定加载方式，需要手动调用`put()`进行加载。需要注意的是，put()方法对于已存在的key将进行覆盖。

在获取缓存值时，如果想要在缓存值不存在时，原子地将值写入缓存，则可以调用`get(key, k -> value)`方法，该方法将避免写入竞争。

多线程情况下，当使用`get(key, k -> value)`时，如果有另一个线程同时调用本方法进行竞争，则后一线程会被阻塞，直到前一线程更新缓存完成；而若另一线程调用`getIfPresent()`方法，则会立即返回null，不会被阻塞。

```java
Cache<String, String> cache = Caffeine.newBuilder().build();

cache.getIfPresent("1");    // null
cache.get("1", k -> 1);    // 1
cache.getIfPresent("1");   //1
cache.set("1", "2");
cache.getIfPresent("1");   //2
```

### Loading Cache

LoadingCache是一种自动加载的缓存。其和普通缓存不同的地方在于，**当缓存不存在或已过期时，若调用get()方法，则会自动调用CacheLoader.load()方法加载最新值，调用getAll()方法将遍历所有的key调用get()，除非实现了CacheLoader.loadAll()方法。**

**使用LoadingCache时，需要指定CacheLoader，并实现其中的load()方法供缓存缺失时自动加载。**

多线程情况下，当两个线程同时调用`get()`，则后一线程将被阻塞，直至前一线程更新缓存完成。

```java
LoadingCache<String, Object> cache = Caffeine.newBuilder().build(new CacheLoader<String, Object>() {
            @Override
            public @Nullable Object load(@NonNull String s) {
                return "从数据库读取";
            }

            @Override
            public @NonNull Map<@NonNull String, @NonNull Object> loadAll(@NonNull Iterable<? extends @NonNull String> keys) {
                return null;
            }
        });

cache.getIfPresent("1"); // null
cache.get("1");          // 从数据库读取
cache.getAll(keys);      // null
```

**LoadingCache特别实用，我们可以在load方法里配置逻辑，缓存不存在的时候去数据库加载，可以实现多级缓存。**

### Async Cache

AsyncCache是Cache的一个变体，其响应结果均为`CompletableFuture`。

通过这种方式，AsyncCache对异步编程进行了适配。默认情况下，缓存计算使用`ForkJoinPool.commonPool()`作为线程池，如果想要指定线程池，则可以覆盖并实现`Caffeine.executor(Executor)`方法。

多线程情况下，当两个线程同时调用get(key, k -> value)，则会返回「**同一个CompletableFuture对象**」。由于返回结果本身不进行阻塞，可以根据业务设计自行选择阻塞等待或者非阻塞。

```java
import com.github.benmanes.caffeine.cache.AsyncCache;
import com.github.benmanes.caffeine.cache.CacheLoader;
import com.github.benmanes.caffeine.cache.Caffeine;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

public class CaffeineExample {
    private final Executor executor = Executors.newSingleThreadExecutor();

    CacheLoader<String, String> loader = new CacheLoader<>() {
        @Override
        public String load(String key) throws Exception {
            // 模拟数据加载过程
            Thread.sleep(1000);
            return key.toUpperCase();
        }

        @Override
        public CompletableFuture<String> asyncLoad(String key, Executor executor) {
            return CompletableFuture.supplyAsync(() -> {
                try {
                    return load(key);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }, executor);
        }
    };

    AsyncCache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .executor(executor)
            .buildAsync(loader);

    public void test() throws Exception {
        // 异步获取
        CompletableFuture<String> future = cache.get("hello");
        System.out.println(future.get());  // 输出 HELLO
    }

    public static void main(String[] args) throws Exception {
        new CaffeineExample().test();
    }
}
```

### Async Loading Cache

看名字就知道，显然这是Loading Cache和Async Cache的功能组合。**Async Loading Cache支持以异步的方式，对缓存进行自动加载**。

以下是如何创建一个Async Loading Cache的缓存示例：

```java
import com.github.benmanes.caffeine.cache.AsyncLoadingCache;
import com.github.benmanes.caffeine.cache.Caffeine;

// 创建 AsyncLoadingCache
AsyncLoadingCache<String, Data> cache = Caffeine.newBuilder()
    .maximumSize(10000)
    .expireAfterWrite(5, TimeUnit.MINUTES)
    .buildAsync(this::loadData);

// 定义加载方法
private CompletableFuture<Data> loadData(String key) {
    // 这只是一个示例，可以根据实际情况来执行数据加载操作
    return CompletableFuture.supplyAsync(() -> {
        // 从数据库或其他地方加载数据
        Data data = ... ;
        return data;
    });
}

// 使用缓存
public void useCache(String key) {
    cache.get(key).thenAccept(data -> {
        // 在这里，data 是从 loadData 方法返回的对象
        // 可以对 data 进行处理
       ...
    });
}
```

**Async Loading Cache也特别实用，有些业务场景我们Load数据的时间会比较长，这时候就可以使用Async Loading Cache，避免Load数据阻塞。**

## 驱逐策略

Caffeine提供了3种回收策略：基于大小回收，基于时间回收，基于引用回收。

### 基于大小的过期方式

基于大小的回收策略有两种方式：一种是基于缓存大小，一种是基于权重。

```java
// 根据缓存的计数进行驱逐
LoadingCache<String, Object> cache = Caffeine.newBuilder()
    .maximumSize(10000)
    .build(new CacheLoader<String, Object>() {

            @Override
            public @Nullable Object load(@NonNull String s) {
                return "从数据库读取";
            }

            @Override
            public @NonNull Map<@NonNull String, @NonNull Object> loadAll(@NonNull Iterable<? extends @NonNull String> keys) {
                return null;
            }
        });


// 根据缓存的权重来进行驱逐，权重低的会被优先驱逐
LoadingCache<String, Object> cache1 = Caffeine.newBuilder()
    .maximumWeight(10000)
    .weigher(new Weigher<String, Object>() {
                    @Override
                    public @NonNegative int weigh(@NonNull String s, @NonNull Object o) {
                        return 0;
                    }
                })
    .build(new CacheLoader<String, Object>() {
            @Override
            public @Nullable Object load(@NonNull String s) {
                return "从数据库读取";
            }

            @Override
            public @NonNull Map<@NonNull String, @NonNull Object> loadAll(@NonNull Iterable<? extends @NonNull String> keys) {
                return null;
            }
        });
```

注意：**maximumWeight与maximumSize不可以同时使用，这是因为它们都是用来限制缓存大小的机制。二者之间需要做出选择。**

### 基于时间的过期方式

```java
// 基于不同的到期策略进行退出
LoadingCache<String, Object> cache = Caffeine.newBuilder()
    .expireAfter(new Expiry<String, Object>() {
        @Override
        public long expireAfterCreate(String key, Object value, long currentTime) {
            return TimeUnit.SECONDS.toNanos(seconds);
        }

        @Override
        public long expireAfterUpdate(@Nonnull String s, @Nonnull Object o, long l, long l1) {
            return 0;
        }

        @Override
        public long expireAfterRead(@Nonnull String s, @Nonnull Object o, long l, long l1) {
            return 0;
        }
    }).build(key -> function(key));
```

Caffeine提供了三种定时驱逐策略：

- **expireAfterAccess（long, TimeUnit）**：在最后一次访问或者写入后开始计时，在指定的时间后过期。假如一直有请求访问该key，那么这个缓存将一直不会过期。

- **expireAfterWrite（long, TimeUnit）**：在最后一次写入缓存后开始计时，在指定的时间后过期。

- **expireAfter（Expiry）**：自定义策略，过期时间由Expiry实现独自计算。

### 基于引用的过期方式

Java中四种引用类型

| 引用类型                 | 被垃圾回收时间 | 用途                                                         | 生存时间          |
| ------------------------ | -------------- | ------------------------------------------------------------ | ----------------- |
| 强引用 Strong Reference  | 从来不会       | 对象的一般状态                                               | JVM停止运行时终止 |
| 软引用 Soft Reference    | 在内存不足时   | 对象缓存                                                     | 内存不足时终止    |
| 弱引用 Weak Reference    | 在垃圾回收时   | 对象缓存                                                     | gc运行后终止      |
| 虚引用 Phantom Reference | 从来不会       | 可以用虚引用来跟踪对象被垃圾回收器回收的活动，当一个虚引用关联的对象被垃圾收集器回收之前会收到一条系统通知 | JVM停止运行时终止 |

```java
// 当key和value都没有引用时驱逐缓存
LoadingCache<String, Object> cache = Caffeine.newBuilder()
    .weakKeys()
    .weakValues()
    .build(key -> function(key));

// 当垃圾收集器需要释放内存时驱逐
LoadingCache<String, Object> cache1 = Caffeine.newBuilder()
    .softValues()
    .build(key -> function(key));
```

注意：**AsyncLoadingCache不支持弱引用和软引用，并且Caffeine.weakValues()和Caffeine.softValues()不可以一起使用。**

## 写入外部存储

CacheWriter 方法可以将缓存中所有的数据写入到第三方，避免数据在服务重启的时候丢失。

```java
LoadingCache<String, Object> cache = Caffeine.newBuilder()
    .writer(new CacheWriter<String, Object>() {
        @Override public void write(String key, Object value) {
            // 写入到外部存储
        }
        @Override public void delete(String key, Object value, RemovalCause cause) {
            // 删除外部存储
        }
    })
    .build(key -> function(key));
```

## 统计

Caffeine内置了数据收集功能，通过`Caffeine.recordStats()`方法，可以打开数据收集。这样`Cache.stats()`方法将会返回当前缓存的一些统计指标，例如：

- **hitRate()**：查询缓存的命中率。
- **evictionCount()**：被驱逐的缓存数量。
- **averageLoadPenalty()**：新值被载入的平均耗时。

```java
Cache<String, String> cache = Caffeine.newBuilder().recordStats().build();
cache.stats(); // 获取统计指标
```

## SpringBoot集成Caffeine Cache

在Caffeine Cache的介绍结束后，接下来介绍如何在项目中顺利集成Caffeine Cache。

话不多说，直接开始上手实践吧。

首先在pom.xml文件中添加Spring Boot Starter Cache和Caffeine的Maven依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
		<dependency>
     		<groupId>com.github.ben-manes.caffeine</groupId>
     		<artifactId>caffeine</artifactId>
     		<version>3.1.1</version>
		</dependency>
</dependencies>
```

其次，创建一个配置类，并创建一个CacheManager Bean：

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
@EnableCaching
public class CachingConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(caffeineCacheBuilder());
        return cacheManager;
    }

    Caffeine<Object, Object> caffeineCacheBuilder() {
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(500)
                .expireAfterAccess(10, TimeUnit.MINUTES)
                .weakKeys()
                .recordStats();
    }
}
```

Caffeine 类使用了建造者模式，主要有如下配置参数：

- **initialCapacity**：缓存初始容量。
- **maximumSize**：设置缓存的最大条目数。当缓存达到这个大小时，它会开始进行清除。
- **maximumWeight**：设置缓存的最大权重。需要同时定义一个`Weigher<K,V>`来如何计算缓存条目的权重。
- **weigher**：定义了如何计算每个缓存条目的权重。
- **expireAfterAccess**：设置在特定时间段后访问缓存项后，会使其过期。
- **expireAfterWrite**：设置在特定时间段后写入（或修改）缓存项后，会使其过期。
- 此方法定义了写入缓存项后的特定时间段，之后该缓存项将被异步刷新。
- **refreshAfterWrite**：此方法定义了写入缓存项后的特定时间段，之后该缓存项将被刷新。
- **weakKeys**：设置缓存key为弱引用，在 GC 时可以直接淘汰。
- **weakValues**：设置value为弱引用，在 GC 时可以直接淘汰。
- **softValues**：设置缓存value为软引用，在内存溢出前可以直接淘汰。
- **recordStats**：启用缓存的统计数据，比如命中率等。
- **removalListener**：设置缓存淘汰监听器。当缓存中的某个条目被移除时会被调用。可以用来记录日志、触发某个操作。

除了配置Bean的方式，还可以用配置文件的方式：

```yaml
spring:
  cache:
    type: caffeine
    cache-names:
    - userCache
    caffeine:
      spec: maximumSize=1024,refreshAfterWrite=60s
```

配置完毕之后，就可以直接配合注解进行使用了：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class SomeService {

    @Autowired
    private SomeRepository someRepository;

    @Cacheable("items")
    public Item getItem(int id) {
        return someRepository.findById(id);
    }
}
```

在这个例子中，我们在getItem方法上添加了@Cacheable注解，每次调用该方法时，Spring首先查找名`item`的cache中是否有对应id的条目。如果有，就返回缓存的值，否则调用方法并将结果存入cache。

### 注解使用方式

注解使用方式，相关的常用注解包括：

- **@Cacheable**：表示该方法支持缓存。当调用被注解的方法时，如果对应的键已经存在缓存，则不再执行方法体，而从缓存中直接返回。当方法返回null时，将不进行缓存操作。
- **@CachePut**：表示执行该方法后，其值将作为最新结果更新到缓存中。**每次都会执行该方法**。
- **@CacheEvict**：表示执行该方法后，将触发缓存清除操作。
- **@Caching**：用于组合前三个注解，例如：

```java
@Caching(cacheable = @Cacheable("users"),
         evict = {@CacheEvict("cache2"), @CacheEvict(value = "cache3", allEntries = true)})
public User find(Integer id) {
    return null;
}
```

这类注解也可以使用在类上，表示该类的所有方法都支持对应的缓存注解。

其中，最常用的是@Cacheable，@Cacheable注解常用的属性如下：

- **cacheNames/value**：缓存组件的名字，即cacheManager中缓存的名称。
- **key**：缓存数据时使用的key。默认使用方法参数值，也可以使用SpEL表达式进行编写。
- **keyGenerator**：和key二选一使用。
- **cacheManager**：指定使用的缓存管理器。
- **condition**：在方法执行开始前检查，在符合condition的情况下，进行缓存。
- **unless**：在方法执行完成后检查，在符合unless的情况下，不进行缓存。
- **sync**：是否使用同步模式。若使用同步模式，在多个线程同时对一个key进行load时，其他线程将被阻塞。



Spring Cache还支持 Spring Expression Language (SpEL) 表达式。你可以通过 SpEL 在缓存名称或键中插入动态值。

举几个例子：

```java
@Cacheable(value = "books", key = "#isbn")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

它将使用传递给 `findBook` 方法的 `isbn` 参数的值。

你也可以用更复杂的 SpEL 表达式，例如：

```java
@Cacheable(value = "books", key = "#root.args[0]")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

`#root.args[0]` 指的是方法调用的第一个参数，也就是 `isbn`。

还有包含条件表达式的例子：

```java
@Cacheable(value = "books", key = "#isbn", condition = "#checkWarehouse == true")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

只有当 `checkWarehouse` 参数为 `true` 时，才会应用缓存。

### 缓存同步模式

`@Cacheable`默认的行为模式是不同步的。这意味着如果你在多线程环境中使用它，并且有两个或更多的线程同时请求相同的数据，那么可能会出现缓存击穿的情况。也就是说，所有请求都会达到数据库，因为在第一个请求填充缓存之前，其他所有请求都不会发现缓存项。

Spring 4.1引入了一个新属性`sync`来解决这个问题。如果设置`@Cacheable(sync=true)`，则只有一个线程将执行该方法并将结果添加到缓存，其他线程将等待。

以下是代码示例：

```java
@Service
public class BookService {

    @Cacheable(cacheNames="books", sync=true)
    public Book findBook(ISBN isbn) {
        // 这里是一些查找书籍的慢速方法，如数据库查询，API调用等
    }
}
```

在这个例子中，无论有多少线程尝试使用相同的ISBN查找相同的书，只有一个线程会实际执行`findBook`方法并将结果存储在名为"books"的缓存中。其他线程将等待，然后从缓存中获取结果，而不需要执行`findBook`方法。

在不同的Caffeine配置下，同步模式表现不同：

| Caffeine缓存类型 | 是否开启同步 | 多线程读取不存在/已驱逐的key                         | 多线程读取待刷新的key                                        |
| :--------------- | :----------- | :--------------------------------------------------- | :----------------------------------------------------------- |
| Cache            | 否           | 各自独立执行被注解方法。                             | -                                                            |
| Cache            | 是           | 线程1执行被注解方法，线程2被阻塞，直至缓存更新完成。 | -                                                            |
| LoadingCache     | 否           | 线程1执行`load()`，线程2被阻塞，直至缓存更新完成。   | 线程1使用老值立即返回，并异步更新缓存值；线程2立即返回，不进行更新。 |
| LoadingCache     | 是           | 线程1执行被注解方法，线程2被阻塞，直至缓存更新完成   | 线程1使用老值立即返回，并异步更新缓存值；线程2立即返回，不进行更新。 |



本篇文章的内容至此告一段落，最后做个小总结，希望这篇文章能够给你带来收获和思考。

在这篇文章中，我们深入探讨了Caffeine Cache以及其淘汰算法的内部工作原理。我们还详细介绍了如何在SpringBoot应用程序中集成Caffeine Cache。希望读者通过本文能深入理解Caffeine Cache的优势并在实践中有效应用。

总的来说，Caffeine Cache不仅提供了强大的缓存功能，还有一个高效的淘汰策略。这使得它在处理大量数据或高并发请求时成为非常好的选择。而且，由于其与SpringBoot的良好兼容性，你可以方便快捷地在SpringBoot项目中使用它。

最后，我们需要记住，虽然Caffeine Cache是一个强大的工具，但正确有效的使用确实需要对其淘汰算法有深入的理解。因此，建议持续关注并研究这个领域的最新进展，以便更好地利用Caffeine Cache提升你的应用性能。
