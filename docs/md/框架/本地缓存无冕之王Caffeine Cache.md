这篇文章来介绍一个本地缓存框架：**Caffeine Cache**。被称为现代缓存之王。Spring Boot 1.x版本中的默认本地缓存是Guava Cache。在 Spring5 (SpringBoot 2.x) 后，Spring 官方放弃了 Guava Cache 作为缓存机制，而是使用性能更优秀的 Caffeine 作为默认缓存组件，这对于Caffeine来说是一个很大的肯定。

[TOC]



## Caffeine Cache介绍

Caffeine 是基于 JAVA 8 的高性能缓存库， 因使用 **Window TinyLfu** 回收策略，提供了一个近乎最佳的命中率。

### 淘汰算法

**FIFO(First In First Out)：先进先出**

优先淘汰掉最先缓存的数据、是最简单的淘汰算法。缺点是如果先缓存的数据使用频率比较高的话，那么该数据就不停地进出，导致它的缓存命中率比较低。

**LRU(Least Recently Used)：最近最久未使用**

优先淘汰掉最久未访问到的数据。缺点是不能很好地应对突发流量。比如一个数据在一分钟内的前59秒访问很多次，而在最后1秒没有访问，但是有一批冷门数据在最后一秒进入缓存，那么热点数据就会被淘汰掉。

**LFU(Least Frequently Used)：最近最少频率使用**

先淘汰掉最不经常使用的数据，需要维护一个表示使用频率的字段。如果访问频率比较高的话，频率字段会占据一定的空间。并且如果数据访问模式随时间有变，LFU的频率信息无法随之变化，因此早先频繁访问的记录可能会占据缓存，而后期访问较多的记录则无法被命中。

**W-TinyLFU 算法**

Caffeine 使用了 W-TinyLFU 算法，看名字就能大概猜出来，它是 LFU 的变种，解决了 LRU 和LFU上述的缺点 。

W-TinyLFU具体是如何实现的，怎么解决 LRU 和LFU的缺点的，有兴趣可以网上搜索相关文章。这里不发散开来细讲，本篇文章重点是介绍具体使用。

## SpringBoot集成Caffeine Cache

首先导入依赖

```xml
        <!-- https://mvnrepository.com/artifact/com.github.ben-manes.caffeine/caffeine -->
        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
            <version>3.1.1</version>
        </dependency>
```

Caffeine 类使用了建造者模式，有如下配置参数：

- **expireAfterWrite**：写入间隔多久淘汰；
- **expireAfterAccess**：最后访问后间隔多久淘汰；
- **refreshAfterWrite**：写入后间隔多久刷新，支持异步刷新和同步刷新，如果和 expireAfterWrite 组合使用，能够保证即使该缓存访问不到、也能在固定时间间隔后被淘汰，否则如果单独使用容易造成OOM，**使用refreshAfterWrite时必须指定一个CacheLoader**；
- **expireAfter**：自定义淘汰策略，该策略下 Caffeine 通过时间轮算法来实现不同key 的不同过期时间；
- **maximumSize**：缓存 key 的最大个数；
- **weakKeys**：key设置为弱引用，在 GC 时可以直接淘汰；
- **weakValues**：value设置为弱引用，在 GC 时可以直接淘汰；
- **softValues**：value设置为软引用，在内存溢出前可以直接淘汰；
- **executor**：选择自定义的线程池，默认的线程池实现是 ForkJoinPool.commonPool()；
- **maximumWeight**：设置缓存最大权重；
- **weigher**：设置具体key权重；
- **recordStats**：缓存的统计数据，比如命中率等；
- **removalListener**：缓存淘汰监听器；

### Cache类型

Caffeine共提供了四种类型的Cache，对应着四种加载策略。

#### Cache

最普通的一种缓存，无需指定加载方式，需要手动调用put()进行加载。需要注意的是，put()方法对于已存在的key将进行覆盖。

在获取缓存值时，如果想要在缓存值不存在时，原子地将值写入缓存，则可以调用get(key, k -> value)方法，该方法将避免写入竞争。

调用invalidate()方法，将手动移除缓存。

多线程情况下，当使用get(key, k -> value)时，如果有另一个线程同时调用本方法进行竞争，则后一线程会被阻塞，直到前一线程更新缓存完成；而若另一线程调用getIfPresent()方法，则会立即返回null，不会被阻塞。

```java
Cache<String, String> cache = Caffeine.newBuilder().build();

cache.getIfPresent("1");    // null
cache.get("1", k -> 1);    // 1
cache.getIfPresent("1");    //1
cache.set("1", "2");
cache.getIfPresent("1");    //2
```

#### Loading Cache

LoadingCache是一种自动加载的缓存。其和普通缓存不同的地方在于，**当缓存不存在/缓存已过期时，若调用get()方法，则会自动调用CacheLoader.load()方法加载最新值**。调用getAll()方法将遍历所有的key调用get()，除非实现了CacheLoader.loadAll()方法。

**使用LoadingCache时，需要指定CacheLoader，并实现其中的load()方法供缓存缺失时自动加载**。

多线程情况下，当两个线程同时调用get()，则后一线程将被阻塞，直至前一线程更新缓存完成。

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
cache.getAll(keys);        // null
```

**LoadingCache特别实用，我们可以在load方法里配置逻辑，缓存不存在的时候去我们的数据库加载，可以实现多级缓存**。

#### Async Cache

AsyncCache是Cache的一个变体，其响应结果均为CompletableFuture，通过这种方式，AsyncCache对异步编程模式进行了适配。默认情况下，缓存计算使用ForkJoinPool.commonPool()作为线程池，如果想要指定线程池，则可以覆盖并实现Caffeine.executor(Executor)方法。

多线程情况下，当两个线程同时调用get(key, k -> value)，则会返回**同一个CompletableFuture**对象。由于返回结果本身不进行阻塞，可以根据业务设计自行选择阻塞等待或者非阻塞。

```java
AsyncCache<String, String> cache = Caffeine.newBuilder().buildAsync();

CompletableFuture<String> completableFuture = cache.get(key, k -> "1");
completableFuture.get(); // 阻塞，直至缓存更新完成
```

#### Async Loading Cache

显然这是Loading Cache和Async Cache的功能组合。AsyncLoadingCache支持以异步的方式，对缓存进行自动加载。

类似LoadingCache，同样需要指定CacheLoader，并实现其中的load()方法供缓存缺失时自动加载，该方法将自动在ForkJoinPool.commonPool()线程池中提交。如果想要指定Executor，则可以实现AsyncCacheLoader().asyncLoad()方法。

```java
AsyncLoadingCache<String, String> cache = Caffeine.newBuilder()
        .buildAsync(new AsyncCacheLoader<String, String>() {
            @Override
            // 自定义线程池加载
            public @NonNull CompletableFuture<String> asyncLoad(@NonNull String key, @NonNull Executor executor) {
                return null;
            }
        })
    //或者使用默认线程池加载（和上面方式二者选其一）
        .buildAsync(new CacheLoader<String, String>() {
            @Override  
            public String load(@NonNull String key) throws Exception {
                return "456";
            }
        });

CompletableFuture<String> completableFuture = cache.get(key); // CompletableFuture<String>
completableFuture.get(); // 阻塞，直至缓存更新完成
```

**Async Loading Cache也特别实用，有些业务场景我们Load数据的时间会比较长，这时候就可以使用Async Loading Cache，避免Load数据阻塞**。

### 驱逐策略

Caffeine提供了3种回收策略：基于大小回收，基于时间回收，基于引用回收

#### 基于大小的过期方式

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

**maximumWeight与maximumSize不可以同时使用**。

#### 基于时间的过期方式

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

**expireAfterAccess(long, TimeUnit)**：在最后一次访问或者写入后开始计时，在指定的时间后过期。假如一直有请求访问该key，那么这个缓存将一直不会过期。
**expireAfterWrite(long, TimeUnit)**：在最后一次写入缓存后开始计时，在指定的时间后过期。
**expireAfter(Expiry)**：自定义策略，过期时间由Expiry实现独自计算。

#### 基于引用的过期方式

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

**注意：AsyncLoadingCache不支持弱引用和软引用。**

**Caffeine.weakKeys()**： 使用弱引用存储key。如果没有其他地方对该key有强引用，那么该缓存就会被垃圾回收器回收。

**Caffeine.weakValues()** ：使用弱引用存储value。如果没有其他地方对该value有强引用，那么该缓存就会被垃圾回收器回收。

**Caffeine.softValues()** ：使用软引用存储value。当内存满了过后，软引用的对象以将使用最近最少使用(least-recently-used ) 的方式进行垃圾回收。由于使用软引用是需要等到内存满了才进行回收，所以我们通常建议给缓存配置一个使用内存的最大值。 

**Caffeine.weakValues()和Caffeine.softValues()不可以一起使用。**

### 写入外部存储

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


### 被动刷新机制

试想这样一种情况：当缓存运行过程中，有些缓存值我们需要定期进行刷新变更。

使用刷新机制**refreshAfterWrite()**，刷新机制只支持**LoadingCache和AsyncLoadingCache**。

refreshAfterWrite()方法是一种被动更新，它必须设置CacheLoad，key过期后并不立即刷新value：

1、当过期后第一次调用get()方法时，得到的仍然是过期值，同时异步地对缓存值进行刷新；

2、当过期后第二次调用get()方法时，才会得到更新后的值。

通过覆写CacheLoader.reload()方法，将在刷新时使得旧缓存值参与其中。

```java
Caffeine.newBuilder()
                .refreshAfterWrite(1,TimeUnit.MINUTES)
                .build(new CacheLoader<String, Object>() {
                    @Override
                    public @Nullable Object load(@NonNull String s) throws Exception {
                        return null;
                    }

                    @Override
                    public @Nullable Object reload(@NonNull String key, @NonNull Object oldValue) throws Exception {
                        return null;
                    }
                });
```

### 统计

Caffeine内置了数据收集功能，通过**Caffeine.recordStats()**方法，可以打开数据收集。这样Cache.stats()方法将会返回当前缓存的一些统计指标，例如：

- **hitRate()**：查询缓存的命中率
- **evictionCount()**：被驱逐的缓存数量
- **averageLoadPenalty()**：新值被载入的平均耗时

```java
Cache<String, String> cache = Caffeine.newBuilder().recordStats().build();
cache.stats(); // 获取统计指标
```

### SpringBoot中集成Caffeine

我们可以直接在SpringBoot中通过创建Bean的方法进行应用。除此以外，SpringBoot还自带了一些更加方便的缓存配置与管理功能。

除了上述的Caffeine依赖，我们还需导入：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-cache -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

**记得在启动类上添加@EnableCaching注解**

#### 配置文件的方式注入相关参数

```yaml
spring:
  cache:
    type: caffeine
    cache-names:
    - userCache
    caffeine:
      spec: maximumSize=1024,refreshAfterWrite=60s
```

#### Bean的方式注入相关参数

```java
@Configuration
public class CacheConfig {


    /**
     * 创建基于Caffeine的Cache Manager
     * 初始化一些key存入
     * @return
     */
    @Bean
    @Primary
    public CacheManager caffeineCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        ArrayList<CaffeineCache> caches = Lists.newArrayList();
        List<CacheBean> list = setCacheBean();
        for(CacheBean cacheBean : list){
            caches.add(new CaffeineCache(cacheBean.getKey(),
                    Caffeine.newBuilder().recordStats()
                            .expireAfterWrite(cacheBean.getTtl(), TimeUnit.SECONDS)
                            .maximumSize(cacheBean.getMaximumSize())
                            .build()));
        }
        cacheManager.setCaches(caches);
        return cacheManager;
    }


    /**
     * 初始化一些缓存的 key
     * @return
     */
    private List<CacheBean> setCacheBean(){
        List<CacheBean> list = Lists.newArrayList();
        CacheBean userCache = new CacheBean();
        userCache.setKey("userCache");
        userCache.setTtl(60);
        userCache.setMaximumSize(10000);

        CacheBean deptCache = new CacheBean();
        deptCache.setKey("deptCache");
        deptCache.setTtl(60);
        deptCache.setMaximumSize(10000);

        list.add(userCache);
        list.add(deptCache);

        return list;
    }

    class CacheBean {
        private String key;
        private long ttl;
        private long maximumSize;

        public String getKey() {
            return key;
        }

        public void setKey(String key) {
            this.key = key;
        }

        public long getTtl() {
            return ttl;
        }

        public void setTtl(long ttl) {
            this.ttl = ttl;
        }

        public long getMaximumSize() {
            return maximumSize;
        }

        public void setMaximumSize(long maximumSize) {
            this.maximumSize = maximumSize;
        }
    }

}
```


使用这种方式，可以同时在缓存管理器中添加多个缓存。需要注意的是，**SimpleCacheManager只能使用Cache和LoadingCache，异步缓存将无法支持**。

#### 注解使用姿势

还可以通过注解的方式进行使用，相关的常用注解包括：

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

这类注解也同时可以标记在一个类上，表示该类的所有方法都支持对应的缓存注解。

@Cacheable常用的注解属性如下：

- **cacheNames/value**：缓存组件的名字，即cacheManager中缓存的名称。
- **key**：缓存数据时使用的key。默认使用方法参数值，也可以使用SpEL表达式进行编写。
- **keyGenerator**：和key二选一使用。
- **cacheManager**：指定使用的缓存管理器。
- **condition**：在方法执行开始前检查，在符合condition的情况下，进行缓存。
- **unless**：在方法执行完成后检查，在符合unless的情况下，不进行缓存。
- **sync**：是否使用同步模式。若使用同步模式，在多个线程同时对一个key进行load时，其他线程将被阻塞。

Spring Cache提供了一些供我们使用的SpEL上下文数据，下表直接摘自Spring官方文档：

| 名称          | 位置       | 描述                                                         | 示例                   |
| ------------- | ---------- | ------------------------------------------------------------ | ---------------------- |
| methodName    | root对象   | 当前被调用的方法名                                           | `#root.methodname`     |
| method        | root对象   | 当前被调用的方法                                             | `#root.method.name`    |
| target        | root对象   | 当前被调用的目标对象实例                                     | `#root.target`         |
| targetClass   | root对象   | 当前被调用的目标对象的类                                     | `#root.targetClass`    |
| args          | root对象   | 当前被调用的方法的参数列表                                   | `#root.args[0]`        |
| caches        | root对象   | 当前方法调用使用的缓存列表                                   | `#root.caches[0].name` |
| Argument Name | 执行上下文 | 当前被调用的方法的参数，如findArtisan(Artisan artisan),可以通过#artsian.id获得参数 | `#artsian.id`          |
| result        | 执行上下文 | 方法执行后的返回值（仅当方法执行后的判断有效，如 unless cacheEvict的beforeInvocation=false） | `#result`              |

#### 缓存同步模式

@Cacheable注解支持配置同步模式。在不同的Caffeine配置下，对是否开启同步模式进行观察。

| Caffeine缓存类型 | 是否开启同步 | 多线程读取不存在/已驱逐的key                       | 多线程读取待刷新的key                                        |
| :--------------- | :----------- | :------------------------------------------------- | :----------------------------------------------------------- |
| Cache            | 否           | 各自独立执行被注解方法                             | -                                                            |
| Cache            | 是           | 线程1执行被注解方法，线程2被阻塞，直至缓存更新完成 | -                                                            |
| LoadingCache     | 否           | 线程1执行`load()`，线程2被阻塞，直至缓存更新完成   | 线程1使用老值立即返回，并异步更新缓存值；线程2立即返回，不进行更新。 |
| LoadingCache     | 是           | 线程1执行被注解方法，线程2被阻塞，直至缓存更新完成 | 线程1使用老值立即返回，并异步更新缓存值；线程2立即返回，不进行更新。 |

从上面的总结可以看到，sync开启或关闭，在Cache和LoadingCache中的表现是不一致的：

- Cache中，sync表示是否需要所有线程同步等待
- LoadingCache中，sync表示在读取不存在/已驱逐的key时，是否执行被注解方法

------

本篇文章就到这里，感谢阅读，如果本篇博客有任何错误和建议，欢迎给我留言指正。文章持续更新，可以关注公众号第一时间阅读。 ![img](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

