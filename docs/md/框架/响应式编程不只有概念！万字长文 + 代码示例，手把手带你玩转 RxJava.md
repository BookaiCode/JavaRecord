# Reactive Streams 介绍

在聊 Reactive Streams 之前，先了解一下 Reactive Programming（反应式/响应式编程）。为了解决异步编程中出现的各种问题，程序员们提出了各种的思路去解决这些问题，这些解决问题的方式、方法，手段就可以叫做 Reactive Programming。

Reactive Programming 是一种编程思想，类似面向对象，函数式编程。

本质上是对数据流或某种变化做出的反应，这个变化什么时候触发是未知的，所以他是一种基于异步、回调的方式在处理问题。

当越来越多的程序员，开始使用这种编程思想时，需要一些大佬来统一一个思想规范。所以国外的几个大佬公司启动了 Reactive Streams 项目。Netflix、Pivotal、Lightbend 联合来为异步流处理提供标准，规范。

Reactive Streams 翻译过来就是响应式/反应式流。**其实是一种基于异步流处理的标准化规范，目的是在使用流处理时更加可靠，高效和响应式。**

# Java 层面的 Reactive Streams

基于这个规范的实现很多，比如三方库中比较出名的 RxJava，Reactor 等等。

但是 JDK8 版本中，Java 已经有了 CompletableFuture 的支撑，我们可以将大量的异步任务做好编排。但是在 JDK8 版本中的 CompletableFuture 依然有很多特性无法支撑。所以在 JDK9，CompletableFuture 做了很多的更新，比如支持延迟，超时，子类化之类的功能。

这时，咱们会发现，其实 CompletableFuture 已经可以去支撑做一些异步编程的操作了。但是为什么很多大公司依然还是使用 RxJava，Reactor 这种三方依赖库呢？

问题在于，大多数的时候，咱们采用异步编程处理的任务并不是非常复杂的。这个时候，咱们确实不需要去使用 Reactive Streams 反应流的框架。如果系统越来越复杂，或者你处理的业务本身就是及其复杂的那种，你就要去写一个让人头皮发麻的代码了。随着时间的推移，这种代码会变成非常难以维护。

其次 CompletableFuture 并不是真正的基于 Reactive Streams 去实现。CompletableFuture 描述的是单次执行的结果。尽管可以通过各种方法将异步任务之间构建成一串任务组成的流程图，本质上依然是单次的结果。

反应式流，面向的是 Stream。 咱们 Java 中的 Stream API 更类似 Reactive Streams 的思想。Stream API 是同步阻塞的。

最经典的就是 CompletableFuture 无法处理 Reactive Streams 中的一个核心概念，Back Pressure（背压，反压，回压），比如在上下游承载能力不同时，比如下游玩不转了，需要告知上游采取一些策略去解决。CompletableFuture 明显无法处理这种。

其次还有 Java 中提供的回调，Future 机制在实现响应式编程中，问题和缺点都比较难处理。有个比较出名的概念叫做 Callback Hell（回调地狱）。简单来说就是回调里面套回调，虽然将子过程做到解耦，但是随着业务的负责，回调代码的可读性、复杂性就大大的增加，这个就是回调地狱。

所以，咱们需要一套框架或者说类库来实现真正响应式流，大概需要几个特性：

* 支持将异步任务做封装以及组装，需要 API 对异步任务进行包装，并且需要很多子任务来对异步操作进行链式组装，过程中包括过滤，异常处理，超时等等操作。
* 减少异步任务的嵌套，减少代码的复杂性，增加可读性，避免 Callback Hell 这种及其复杂恶心的代码。
* 支持背压 Back Pressure，也就需要有上游和下游的概念，可以做到协商处理数据流的速度。

# Java 层面 Reactive Steams 的 API

首先 Reactive Steams 响应流实现方式其实是基于观察者模式的扩展，同时也能看到发布订阅模式，责任链模式等等。

整个 Reactive Steams 流程大致如下。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMTgibXaD43CfueC3XMT1iaUB1ovAJ9LveK6iaw8YUyCicia79IzCJWLHsPf9CS8o2bmxSOQt12GSDRKvA/640?wx_fmt=png&amp;from=appmsg)

直接在 JDK9 版本之上查看 Doug Lee 提供的 Flow 类。

在 Flow 类中，提供了核心的四个接口：Publisher，Subscriber，Subscription，Processor

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMTgibXaD43CfueC3XMT1iaUBTVSOkgjPOw4pnqTOrpwxic8X9o0Vtb09BUdP6jVCj41Yj1CVAqjexWA/640?wx_fmt=png&amp;from=appmsg)

Publisher：Publisher 是函数式接口，负责发布数据的。 Publisher 内部有一个方法 subscribe 方法，去和具体的订阅者绑定关系。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMTgibXaD43CfueC3XMT1iaUBxrb5DSib2xeqEeItcB7mpibkSCjZhseBSBHzSiciaNZa4BwKZYB5zmupFg/640?wx_fmt=png&amp;from=appmsg)

Subscriber：Subscriber 是订阅者，负责订阅，消费数据。四个方法：

- onSubscribe：订阅成功后触发，并且表明可以开始接收发布者的数据元素了。
- onNext：每次获取到发布者的数据元素都会执行 onNext。
- onError：接收数据元素时，出现异常等问题时，走 onError。
- onComplete：当指定接收的元素个数搞定后，触发 onComplete。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMTgibXaD43CfueC3XMT1iaUBJhzfo9CnDJicVedPtuy4Pp9Ivp8DcdgjH6v6dwvLoUwicSp88dfrP6rw/640?wx_fmt=png&amp;from=appmsg)

Subscription：发布者和订阅者是基于 Subscription 关联的。当建立了订阅的关系后，发布者会将 Subscription 传递给订阅者。订阅者指定获取元素的数量和取消订阅操作，都要基于 Subscription 去操作。提供了两个方法：

- request：订阅者要获取的元素个数。
- cancel：取消订阅，当前的订阅者不接收当前发布者的元素。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMTgibXaD43CfueC3XMT1iaUBhukW4Q6HRW1KPAuUONjNnYGDHDZNG1Cia3aaWSyxNHfNZu0zsTdZibWw/640?wx_fmt=png&amp;from=appmsg)

Processor：Processor 继承了 Publisher 和 Subscriber，即是发布者也是订阅者。Processor 一般作为数据的中转，订阅者处理完数据元素，可以再次发给下一个订阅者。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMTgibXaD43CfueC3XMT1iaUBGeW651DJBGcBsicB0QOgcMeBwYpE3zYb9Pbfk3vQhEibicNuvNFejlmqA/640?wx_fmt=png&amp;from=appmsg)

这四个接口很重要，是 Reactive Streams 的规范，但是可以明显的看到，内部没有具体的内容实现。

这里就类似 JDBC 这种规范，规范在 JDK9 中提出来了，想实现，可以基于当前的这四个接口再做具体的逻辑处理以及实现的细节。

# Java 层面 Reactive Steams 基本操作

咱们测试 Java 中的 Flow 里提供的 API 时，就是走最基本的操作。

其中 Processor 不需要重写，玩最基本的操作，不去做订阅者和发布者的转换。

其次 Subscription 也不需要重写，这东西就是提供了订阅者指定订阅的消息个数，以及取消的操作。

然后 Publisher 需要重写，但是 JDK 中已经提供了一个 Publisher 的实现，SubmissionPublisher，可以直接使用。

最后，Subscriber 需要咱们自己重写，指定好订阅消息的个数，已经消费的一些逻辑

```java
import java.util.concurrent.Flow;


public class MySubscriber implements Flow.Subscriber<Integer> {

    @Override
    // 绑定好订阅关系后，就会触发这个方法
    public void onSubscribe(Flow.Subscription subscription) {
        subscription.request(10);
    }

    @Override
    public void onNext(Integer item) {
        System.out.println(Thread.currentThread().getName() + "：接收到数据流：" + item);
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println(Thread.currentThread().getName() + "：接收消息出现异常：" + throwable.getMessage());
    }

    @Override
    public void onComplete() {
        System.out.println(Thread.currentThread().getName() + "：当前订阅者要求接收的消息全部处理完毕。");
    }
}
```

直接使用 SubmissionPublisher 测试整体效果

```java
public static void main(String[] args) {
    // 只有一个工作线程的线程池
    ExecutorService executor = Executors.newFixedThreadPool(1);
    // 指定缓冲区的大小
    int maxBufferCapacity = 5;

    // 需要指定两个参数
    // 第一个参数需要传递一个线程池，指定订阅者使用的线程
    // 第二个参数，需要指定一个缓冲区，发布者发布消息后，消息会扔到缓冲区里。
    SubmissionPublisher<Integer> publisher = new SubmissionPublisher<>(executor,maxBufferCapacity);

    // 绑定订阅者
    MySubscriber subscriber = new MySubscriber();
    publisher.subscribe(subscriber);

    // 发布消息
    for (int i = 0; i < 10; i++) {
        System.out.println(Thread.currentThread().getName() + "：发布消息：" + i);
        publisher.submit(i);
    }

    // 释放资源
    publisher.close();
    executor.shutdown();
}
```

结果输出如下：

```
main：发布消息：0
main：发布消息：1
main：发布消息：2
main：发布消息：3
main：发布消息：4
main：发布消息：5
main：发布消息：6
main：发布消息：7
main：发布消息：8
main：发布消息：9
pool-1-thread-1：接收到数据流：0
pool-1-thread-1：接收到数据流：1
pool-1-thread-1：接收到数据流：2
pool-1-thread-1：接收到数据流：3
pool-1-thread-1：接收到数据流：4
pool-1-thread-1：接收到数据流：5
pool-1-thread-1：接收到数据流：6
pool-1-thread-1：接收到数据流：7
pool-1-thread-1：接收到数据流：8
pool-1-thread-1：接收到数据流：9
pool-1-thread-1：当前订阅者要求接收的消息全部处理完毕。
```

- **缓冲区：** 缓冲区就是发布者和订阅者之间的一块内存，类似线程池中的阻塞队列，可以将消息扔到这个缓存区里。其次咱们设置的缓冲区大小是 5，但是发现 get 出来的时候，5 被替换为了 8。这是因为 SubmissionPublisher 为了更有效的使用内存，默认会基于 roundCapacity 方法将传递的缓冲区大小替换为 2 的 n 次幂。
- **背压效果：** 当订阅者指定的消息已经全部处理完毕后，发布者最多只能发布缓冲区大小个数的消息，剩下的内容会基于背压的效果直接暂时不发送。
- **onComplete：** 需要发布者做了close 操作，确认了发布者已经将消息全部发送，并且订阅者也已经将全部的消息处理完毕后，才会触发 onComplete。
- **Subscription：** 订阅者可以在 onNext 或者其他方法中动态的使用 subscription 去指定后续需要几个消息订阅，以及是否需要取消订阅消息等操作。

# Reactive Steams 落地体验

## 回调地狱问题

前面的方式大致了解了 JDK9 中更新的 Reactive Streams 的规范，咱们实现也仅仅是看到了发布订阅和回压的效果。并没有看到如何解决回调地狱的问题。咱们可以通过 Spring5 官网提供的一个例子，来体验一下 CallBack Hell 回调地狱带来的问题。后面再根据三方的实现来看一下基于 Reactive Streams 实现后效果如何。这里基本是根据伪代码走的。

例子：在用户的 UI 页面上，展示当前用户最喜欢的 Top5 的商品详情。这里会根据用户的 ID 去查询当前用户 Top5 商品的ID，如果 ID 可以查询到之后再根据商品的 ID 去查询商品的详情。如果当前用户 ID 查询的结果不存在喜欢的 Top 商品，没有的话，通过推荐服务查询 Top5 的商品信息。展示给用户。

当前例子需要三个服务的支撑：

* 根据用户 ID 查询用户的 Top5 商品ID。
* 根据 Top5 商品ID查询商品详情。
* 调用推荐服务，获取5个商品详情。

基于 Java 最原生的异步编程方式，实现上述操作，来看看到底什么是回调地狱。。。

商品详情实体类：

```java
@Data
public class Fav {

    private String itemId;

    private String itemName;

    private String itemDetail;

}
```

准备回调方法，拿到结果后触发

```java
public interface Callback<T> {

    void onSuccess(T t);

    void onError(Throwable throwable);

}
```

准备了访问三个服务的 Service 接口

```java
public interface UserService {

    /**
     * 根据用户Id查询用户的Top5商品Id
     * @param userId
     * @param list
     */
    void getFav(String userId, Callback<List<String>> list);

}

public interface ItemService {
    
    /**
     * 根据商品Id查询商品的详情
     *
     * @param itemId
     * @param callback
     */
    void getDetail(String itemId, Callback<Fav> callback);
}

public interface SuggestionService {

    /**
     * 调用推荐服务，获取推荐商品
     * @param favs
     */
    void getSuggestion(Callback<List<Fav>> favs);
}
```

准备了响应数据的 UI 线程工具以及响应方法

```java
public class UiUtils {

    public static void submitOnUiThread(Runnable runnable){
        // 线程池中的线程做响应的操作………………
    }


    public static void show(Object obj){
        // 利用UI线程展示具体数据
    }

    public static void error(Object obj){
        // 出现错误响应的内容
    }

}
```

完成了 Controller 中的异步编程效果

```java
@RestController
public class CallBackHellController {

    @Autowired
    private UserService userService;

    @Autowired
    private ItemService itemService;

    @Autowired
    private SuggestionService suggestionService;


    @GetMapping("/callbackhell")
    public void callbackHell(String userId){
        //1、调用用户服务，查询Top5商品Id
        userService.getFav(userId, new Callback<List<String>>() {
            @Override
            public void onSuccess(List<String> list) {
                // 已经查询到商品Id，但是不知道是否有值
                if (list.isEmpty()){
                    // 3、用户没有Top5商品Id，通过推荐服务查询推荐商品详情
                    suggestionService.getSuggestion(new Callback<List<Fav>>(){
                        @Override
                        public void onSuccess(List<Fav> favs) {
                            // 推荐服务查询到了商品详情，响应即可
                            UiUtils.submitOnUiThread(() -> {
                                favs.stream().limit(5).forEach(UiUtils::show);
                            });
                        }
                        @Override
                        public void onError(Throwable throwable) {
                            UiUtils.error(throwable);
                        }
                    });

                }
                else{
                    // 2、通过用户查询到了Top5商品Id，通过商品Id查询商品详情
                    list.stream().limit(5).forEach(itemId -> itemService.getDetail(itemId,new Callback<Fav>(){

                        @Override
                        public void onSuccess(Fav fav) {
                            // 查询到了商品详情，利用UI线程，给客户端响应数据
                            UiUtils.submitOnUiThread(() -> UiUtils.show(fav));
                        }

                        @Override
                        public void onError(Throwable throwable) {
                            // 出现异常了。
                            UiUtils.error(throwable);
                        }
                    }));
                }
            }
            @Override
            public void onError(Throwable throwable) {
                // 出现异常了。
                UiUtils.error(throwable);
            }
        });


    }

}
```

## 解决回调地狱问题

这里为了解决回调地狱问题，需要一个 Reactor 的依赖来帮助咱们实现异步编程。

需要导入依赖

```xml
<!--reactor的核心依赖-->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.7.7</version>
</dependency>
```

不能再使用之前的 Callback 方式了。需要使用 reactor 提供的 Flux，并且这种链式操作会更直观，也更好维护。就只需要修改三个服务对应的 Service。

```java
public interface UserService {

    /**
     * 根据用户ID查询Top5商品ID
     * @param userId
     * @return
     */
    Flux<List<String>> getFav(String userId);

}

public interface ItemService {


    /**
     * 根据商品ID查询商品详情
     * @param itemId
     * @return
     */
    Flux<Fav> getDetail(String itemId);

}

public interface SuggestionService {

    /**
     * 获取推荐的商品详情
     * @return
     */
    Flux<List<Fav>> getSuggestion();

}
```

然后就可以利用 Flux 提供的 API 来解决之前回调地狱的问题。

```java
@RestController
public class ReactorCallbackController {

    @Autowired
    private UserService userService;

    @Autowired
    private ItemService itemService;

    @Autowired
    private SuggestionService suggestionService;


    @GetMapping("reactorcallback")
    public void reactorCallback(String userId){
        userService
                .getFav(userId)   // 根据用户Id查询Top5商品Id
                .flatMap(itemService::getDetail)  // 根据商品ID查询商品详情
                .switchIfEmpty(suggestionService.getSuggestion())  // 如果前面为null，这里通过推荐服务查询商品详情
                .take(5)    // 获取前5个数据
                .publishOn(UiUtils.reactorOnUiThread())    // 使用Ui线程
                .subscribe(UiUtils::show,UiUtils::error);   // 成功走show，失败走error
    }

}
```

## CompletableFuture 的异步编程

Future 的形式相比 Callback Hell 效果要好一些，虽然 JDK8 和 9 都对 CompletableFuture 做了各种优化，但是他的表现还是不太好。多个Future在嵌套时，可读性还是比较差的。并且 CompletableFuture 不存在什么回压，或者是延迟调用的功能。

现在借助 CompletableFuture 来实现一个场景。

1. 获取一个用户ID的列表。
2. 通过用户ID分别获取他的名字以及统计信息。（希望这两个操作是并行执行的）
3. 当两个信息都获取到之后，封装成一个普通字符串即可。
4. 响应数据，最后拿到结果（输出一下）。

实现代码

```java
public class GetNameAndStatTestByCF {

    public static void main(String[] args) {
        // 1、获取一组用户ID列表
        CompletableFuture<List<String>> idList = getID();
        CompletableFuture<List<String>> dataCompletableFuture = idList.thenComposeAsync(ids -> {
            Stream<CompletableFuture<String>> resultStream = ids.stream().map(id -> {
                // 2、并行基于ID查询名称信息
                CompletableFuture<String> nameTask = getName();
                // 2、并行基于ID查询统计信息
                CompletableFuture<Integer> statTask = getStat();
                // 让两个查询名称信息和查询统计信息操作并行执行
                return nameTask.thenCombineAsync(statTask, (name, stat) -> {
                    // 3、拿到信息组装
                    return "Name：" + name + "，Stat：" + stat;
                });
            });
            // 将resultStream转换成一个数组
            List<CompletableFuture<String>> resultList = resultStream.toList();
            // 全部的任务封装起来
            CompletableFuture<Void> allDone = CompletableFuture.allOf(resultList.toArray(new CompletableFuture[]{}));
            // 执行全部任务
            return allDone.thenApplyAsync(v -> resultList.stream()
                    .map(CompletableFuture::join)
                    .collect(Collectors.toList()));
        });

        // 4、获取全部的组件信息后响应客户端（输出）
        List<String> data = dataCompletableFuture.join();
        System.out.println(data);
    }

    // 模拟zz服务获取统计信息
    private static CompletableFuture<Integer> getStat() {
        return CompletableFuture.supplyAsync(() -> 666);
    }

    // 模拟yy服务获取名称信息
    private static CompletableFuture<String> getName() {
        return CompletableFuture.supplyAsync(() -> "张三");
    }

    // 模拟xx服务，获取一组用户ID
    private static CompletableFuture<List<String>> getID() {
        return CompletableFuture.supplyAsync(() -> {
            // 模拟查询三方服务
            List<String> list = new ArrayList<>();
            list.add("1");
            list.add("2");
            list.add("3");
            return list;
        });
    }

}
```

## 解决 CompletableFuture 的问题

CompletableFuture 可以实现一些简单的异步编程，但是可看性和维护性以后后期的扩展都需要对整体代码做比较大成本的维护。依然采用 Reactor 来实现一个一模一样的逻辑，再看代码效果。

```java
public class GetNameAndStatByReactor {

    public static void main(String[] args) {
        // 1、获取一组用户ID列表
        Flux<String> idFlux = getId();

        Flux<String> result = idFlux.flatMap(id -> {
            // 2、并行基于ID查询名称信息
            Flux<String> nameFlux = getName(id);
            // 2、并行基于ID查询统计信息
            Flux<Integer> statFlux = getStat(id);
            // 俩任务并行处理完毕，触发3
            return nameFlux.zipWith(statFlux, (name, stat) -> {
                // 3、拿到信息组装
                return "Name：" + name + "，Stat：" + stat;
            });
        });
        Mono<List<String>> listMono = result.collectList();
        List<String> info = listMono.block();
        // 4、获取全部的组件信息后响应客户端（输出）
        System.out.println(info);
    }

    private static Flux<Integer> getStat(String id) {
        // 会查询三方服务，然后封装结果
        return Flux.just(888);
    }

    private static Flux<String> getName(String id) {
        // 会查询三方服务，然后封装结果
        return Flux.just("张三");
    }


    private static Flux<String> getId() {
        // 会查询三方服务，然后封装结果
        return Flux.just("1","2","3");
    }
}
```

# RxJava2 实现异步编程

RxJava 是一个小框架，或者是依赖库。在 RxJava 的1.x版本中，它并不基于 Reactive Streams 去实现的。没有关系，因为 RxJava 的 2 版本，就是基于Reactive Streams 实现的了。

使用 RxJava 巨简单，因为作者想将 RxJava 尽量做到轻量级，就一个依赖。

```xml
<!--RxJava2的依赖-->
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.21</version>
</dependency>
```

## RxJava2 的入门操作

获取一个 Person 对象的集合，将 Person 集合中的所有年龄大于10岁的 Person 对象筛选出来，并输出他的名字。

采用 RxJava 来实现一下：

```java
public class Demo {

    public static void main(String[] args) {
        //1、获取Person对象集合
        List<Person> personList = getPersonList();

        //2、完成上面要求的操作
        //2.1、将person集合转换为RxJava的流
        Flowable.fromArray(personList.toArray(new Person[]{}))
                //2.2 过滤年龄大于10岁的
                .filter(person -> person.getAge() > 10)
                //2.3 获取筛选后的Person名称
                .map(person -> person.getName())
                //2.4 输出Name
                .subscribe(System.out::println);
    }

    private static List<Person> getPersonList() {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("大娃",5));
        personList.add(new Person("二娃",7));
        personList.add(new Person("三娃",9));
        personList.add(new Person("四娃",11));
        personList.add(new Person("五娃",13));
        return personList;
    }
}
```

## RxJava2 的基础处理流程

在 RxJava 中有三个核心的角色

* 被观察者（Observable）
* 观察者（Observer）
* 订阅（Subscribe）

```java
public class Demo2 {

    public static void main(String[] args) {
        //1、构建Observable
        Observable<String> observable = Observable.create(emitter -> {
            emitter.onNext("Hello");
            emitter.onNext("World");
            emitter.onComplete();
        });

        //2、构建Observer
        Observer<String> observer = new Observer<>() {
            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("开始订阅");
            }

            @Override
            public void onNext(String s) {
                System.out.println("观察者：" + s);
            }

            @Override
            public void onError(Throwable e) {
                e.printStackTrace();
            }

            @Override
            public void onComplete() {
                System.out.println("订阅结束");
            }
        };

        //3、订阅
        observable.subscribe(observer);
    }

}
```

## 创建操作符

### create

Observable.create() 是手动创建 Observable 的方法，允许完全控制数据的发射、完成和错误处理。

```java
Observable<String> observable = Observable.create(emitter -> {
    emitter.onNext("Hello");
    emitter.onNext("World");
    emitter.onComplete();
});

observable.subscribe(
    item -> System.out.println("收到: " + item),
    error -> System.err.println("错误: " + error),
    () -> System.out.println("完成")
);

// 输出：
// 收到: Hello
// 收到: World
// 完成
```

### just

just 用于创建一个发射固定数据的 Observable，数据是预定义的，发射后立即完成。

```java
Observable.just("Hello")
    .subscribe(item -> 
        System.out.println("收到: " + item)
    );

// 输出：
// 收到: Hello
```

### fromArray

fromArray 用于从数组创建一个 Observable，按数组顺序发射所有元素。

```java
String[] fruits = {"Apple", "Banana", "Cherry", "Date"};
Observable.fromArray(fruits)
    .subscribe(fruit -> 
        System.out.print(fruit + " ")
    );

// 输出：
// Apple Banana Cherry Date
```

### fromCallable

fromCallable 用于从 Callable 创建 Observable，Callable 的返回值会被包装成 Observable 发射。

```java
    public static void main(String[] args) {
        Observable.fromCallable(() -> "计算结果: " + System.currentTimeMillis())
                .subscribe(System.out::println);

        // 输出：
        // 计算结果: 1620000000000
    }
```

### timer

timer 用于创建一个延迟指定时间后发射单个数据的 Observable，通常是 0L，然后结束。

```java
    public static void main(String[] args) throws InterruptedException {
        System.out.println("开始时间: " + System.currentTimeMillis());

        Observable.timer(2, TimeUnit.SECONDS)
                .subscribe(tick ->
                        System.out.println("触发时间: " + System.currentTimeMillis() + "，值: " + tick)
                );

        Thread.sleep(3000);
    }
```

默认使用 `Schedulers.computation()`线程池计算，线程池中的线程是守护线程，如果主线程结束守护线程也会随之终止。

### interval

interval() 方法用于创建一个 周期性定时发射的 Observable，它会按照指定的时间间隔无限期地发射递增的数字序列（从0开始）。

```java
    public static void main(String[] args) throws IOException {
        Observable.interval(2, TimeUnit.SECONDS)
                .subscribe(aLong -> System.out.println(Thread.currentThread().getName() + "：" + aLong));

        System.in.read();
    }
```

### intervalRange

intervalRange() 是 interval() 的增强版本，用于创建一个有限次数的周期性发射的 Observable。它允许你指定起始值、发射次数、初始延迟和间隔时间。

```java
    public static void main(String[] args) throws IOException {
        Observable.intervalRange(100, 4, 0, 2, TimeUnit.SECONDS)
                .subscribe(aLong -> System.out.println(Thread.currentThread().getName() + "：" + aLong));
        System.in.read();
    }
```

### range & rangeLong

这两个方法用于创建一个发射连续整数序列的 Observable：

- `range(start, count)`：发射 Integer 类型的连续整数。
- `rangeLong(start, count)`：发射 Long 类型的连续整数。

```java
    public static void main(String[] args) throws IOException {
        Observable.range(0, 10)
                .subscribe(integer -> System.out.println(Thread.currentThread().getName() + "：" + integer));
        System.in.read();
    }
```

### never、error、empty

这三个方法都是创建特殊的 Observable 的工厂方法，用于特定的场景。

* never：创建一个永远不会发射任何数据，也不会终止的 Observable。
* error：创建一个立即发射错误的 Observable。
* empty：创建一个立即完成但不发射任何数据的 Observable。

```java
    public static void main(String[] args) {
        // Observable.never()
        // Observable.error(new RuntimeException("error事件"))
        Observable.empty()
                .subscribe(new Observer() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        System.out.println("开始订阅");
                    }

                    @Override
                    public void onNext(Object s) {
                        System.out.println("观察者：" + s);
                    }

                    @Override
                    public void onError(Throwable e) {
                        System.out.println("出现异常：" + e.getMessage());
                    }

                    @Override
                    public void onComplete() {
                        System.out.println("订阅结束");
                    }
                });
    }
```

## 转换操作符

转换操作符是 RxJava 中用于对 Observable 发射的数据进行变换、处理、组合的操作符，用于实现数据流的各种转换逻辑。

### map

map() 是 RxJava 中最基本、最常用的转换操作符，用于对 Observable 发射的每个元素进行一对一转换。

简单来说：输入一个值，输出另一个值（1进1出）。

```java
    public static void main(String[] args) {
        Observable.just(1, 2, 3, 4, 5)
                .map(number -> "数字: " + number)
                .subscribe(text -> System.out.println(text));
    }
```

### flatMap

flatMap() 用于将每个发射项转换为 Observable，然后"扁平化"合并成一个 Observable。
简单来说：输入一个值，可以输出任意多个值（1进N出）。

```java
    public static void main(String[] args) {
        Observable.just("A", "B")
                .map(letter -> letter + "1")
                .subscribe(result -> System.out.print(result + " "));
        // 输出: A1 B1

        // flatMap: 1对多转换
        Observable.just("A", "B")
                .flatMap(letter ->
                        Observable.just(letter + "1", letter + "2", letter + "3")
                )
                .subscribe(result -> System.out.print(result + " "));
        // 输出: A1 A2 A3 B1 B2 B3
    }
```

### concatMap

concatMap() 是 flatMap() 的顺序保持版本，它会严格按照原始顺序依次处理每个元素，只有前一个元素的 Observable 完成之后，才会处理下一个。
简单来说：flatMap是并发处理，concatMap是串行处理。

```java
    public static void main(String[] args) throws IOException {
        Observable.just(3, 1, 2)  // 注意顺序：3, 1, 2
                .flatMap(num ->
                        Observable.just(num)
                                .delay(num, TimeUnit.SECONDS)  // 延迟对应的秒数
                )
                .subscribe(num -> System.out.println("flatMap: " + num));
        // 输出顺序：1 2 3（谁先完成谁先输出）

        Observable.just(3, 1, 2)
                .concatMap(num ->
                        Observable.just(num)
                                .delay(num, TimeUnit.SECONDS)
                )
                .subscribe(num -> System.out.println("concatMap: " + num));
        // 输出顺序：3 1 2（严格保持原始顺序）
        System.in.read();
    }
```

### buffer

buffer() 用于将 Observable 发射的数据项收集到集合中，然后一次性发射这些集合，而不是单个发射。
简单来说：把多个单独的数据"打包"成一批一起发射。

```java
    public static void main(String[] args) {
        Observable.range(1, 10)  // 发射1-10
                .buffer(3)           // 每3个一批
                .subscribe(batch ->
                        System.out.println("批次: " + batch)
                );

        // 输出：
        // 批次: [1, 2, 3]
        // 批次: [2, 4, 6]
        // 批次: [7, 8, 9]
        // 批次: [10]           ← 最后一批不足3个
    }
```

### scan

scan() 用于对 Observable 发射的数据进行累积计算，并发射每个中间结果。
简单来说：像 Excel 里的累计求和，每来一个新数据，就与前一个结果计算，并输出当前累计值。

```java
    public static void main(String[] args) {
        Observable.just(1, 2, 3, 4, 5)
                .scan(Integer::sum)
                .subscribe(result -> System.out.print(result + " "));

        // 输出：
        // 1 3 6 10 15

        // 计算过程：
        // 初始：无种子值，第一次直接发射1
        // 1 + 2 = 3
        // 3 + 3 = 6
        // 6 + 4 = 10
        // 10 + 5 = 15
    }
```

### window

window() 用于将 Observable 发射的数据分组到多个子 Observable 中，然后发射这些子 Observable 而不是单个数据项。
简单来说：创建多个"窗口"，每个窗口是一个 Observable，将数据分配到不同窗口中。

```java
    public static void main(String[] args) {
        // buffer: 直接发射List
        Observable.range(1, 5)
                .buffer(2)
                .subscribe(list ->
                        System.out.println("buffer输出List: " + list)
                );

        // window: 发射Observable，需要进一步处理
        Observable.range(1, 5)
                .window(2)
                .flatMapSingle(Observable::toList  // 需要手动转换
                )
                .subscribe(list ->
                        System.out.println("window输出List: " + list)
                );

        // 两者输出相同，但window更灵活：
        // 输出：
        // [1, 2]
        // [3, 4]
        // [5]
    }
```

window 和 buffer 的区别就是：

* window 返回的是被观察者的集合。
* buffer 返回的是数据的集合。

## 过滤操作符

过滤操作符用于从数据流中筛选出需要的数据，过滤掉不需要的数据。

### filter

filter() 是 RxJava 中最基本的过滤操作符，用于根据指定条件筛选 Observable 发射的数据，只让满足条件的数据通过，不满足条件的被过滤掉。

对每个数据项进行判断，返回 true则通过，返回 false则丢弃。

```java
    public static void main(String[] args) {
        Observable.range(1, 10)  // 发射1-10
                .filter(number -> number % 2 == 0)  // 只保留偶数
                .subscribe(even -> System.out.print(even + " "));

        // 输出：
        // 2 4 6 8 10
    }
```

### ofType

ofType() 是一个类型过滤操作符，用于过滤 Observable 发射的数据，只保留指定类型的数据，其他类型的数据会被过滤掉。
核心思想：只让指定类型的数据通过，相当于 filter(item -> item instanceof TargetType) 的简化版。

```java
    public static void main(String[] args) {
        Observable<Object> mixedObservable = Observable.just(
                "Hello",      // String
                123,          // Integer
                45.6,         // Double
                "World",      // String
                true,         // Boolean
                789           // Integer
        );

        // 只保留字符串类型
        mixedObservable
                .ofType(String.class)
                .subscribe(str ->
                        System.out.println("字符串: " + str)
                );

        // 输出：
        // 字符串: Hello
        // 字符串: World
    }
```

### distinct

distinct() 用于过滤 Observable 中重复的数据项，确保每个数据项只发射一次。
核心思想：去重，保证发射的数据序列中不包含重复元素。

```java
    public static void main(String[] args) {
        Observable.just(1, 2, 2, 3, 3, 3, 4, 5, 5, 1)
                .distinct()
                .subscribe(num -> System.out.print(num + " "));

        // 输出：
        // 1 2 3 4 5
        // 注意：最后的 1 也被去重了
    }
```

### skip & skipLast

- skip(n)：跳过 Observable 发射的前 n 个数据项。
- skipLast(n)：跳过 Observable 发射的后 n 个数据项。

核心思想：skip是"跳过开头"，skipLast是"跳过结尾"。

```java
    public static void main(String[] args) {
        Observable.create(emitter -> {
                    emitter.onNext(1);
                    emitter.onNext(2);
                    emitter.onError(new RuntimeException("中间出错"));
                })
                .skipLast(1) 
                .subscribe(
                        num -> System.out.println("数据: " + num),
                        error -> System.err.println("错误: " + error.getMessage()),
                        () -> System.out.println("完成")
                );

        // 输出：
        // 数据: 1
        // 错误: 中间出错
    }
```

### distinctUntilChanged

distinctUntilChanged() 用于过滤掉连续重复的数据，只保留发生变化的数据。
核心思想：去重，但只去重连续相同的值，不连续出现的相同值会被保留。

```java
    public static void main(String[] args) {
        Observable.just(1, 1, 2, 2, 1, 3, 3, 2, 2, 1)
                .distinctUntilChanged()
                .subscribe(num -> System.out.print(num + " "));

        // 输出：
        // 1 2 1 3 2 1

        // 解释：
        // 原始: 1, 1, 2, 2, 1, 3, 3, 2, 2, 1
        // 去重连续重复后: 1, 2, 1, 3, 2, 1
        // 注意：中间的 1 虽然出现过，但因为不连续，所以保留了
    }
```

### take

take() 用于从 Observable 的开头取出指定数量的数据，然后完成。
核心思想：只取前 N 个数据，之后的数据忽略，Observable 提前完成。

```java
    public static void main(String[] args) {
        Observable.range(1, 10)  // 发射1-10
                .take(3)             // 只取前3个
                .subscribe(
                        num -> System.out.print(num + " "),
                        error -> {
                        },
                        () -> System.out.println("\n已完成")
                );

        // 输出：
        // 1 2 3
        // 已完成
        // 注意：4-10 不会被发射
    }
```

### firstElement & lastElement & elementAt

这三个操作符都用于从 Observable 中取出特定位置的单个元素：

- **`firstElement()`**：取第一个元素
- **`lastElement()`**：取最后一个元素
- **`elementAt(index)`**：取指定索引位置的元素

```java
    public static void main(String[] args) {
        Observable.range(1, 5)  // 1,2,3,4,5
                .firstElement()      // 取第一个
                .subscribe(
                        num -> System.out.println("第一个: " + num),
                        error -> {
                        },
                        () -> System.out.println("没有第一个元素")
                );

        // 输出：
        // 第一个: 1
    }
```

## 组合操作符

组合操作符是 RxJava 中用于将多个 Observable 组合成一个 Observable 的操作符。它们处理多个数据流之间的关系，实现流的合并、连接、组合等操作。

### concat

concat() 用于顺序连接多个 Observable，前一个 Observable 完成后，才开始发射下一个 Observable 的数据。
核心思想：串行连接，保持顺序，像排队一样一个接一个。

```java
    public static void main(String[] args) {
        Observable<String> first = Observable.just("A", "B", "C");
        Observable<String> second = Observable.just("D", "E", "F");
        Observable<String> third = Observable.just("G", "H");

        Observable.concat(first, second, third)
                .subscribe(
                        letter -> System.out.print(letter + " "),
                        error -> {
                        },
                        () -> System.out.println("\n全部完成")
                );

        // 输出：
        // A B C D E F G H
        // 全部完成
        // 严格按照 first → second → third 的顺序
    }
```

### concatArray

concatArray() 是 concat() 的数组版本，用于顺序连接多个 Observable（以数组形式提供），功能与 concat() 完全相同，只是参数形式不同。

```java
    public static void main(String[] args) {
        // 创建Observable数组
        Observable<String>[] observables = new Observable[]{
                Observable.just("A", "B"),
                Observable.just("C", "D", "E"),
                Observable.just("F", "G")
        };

        // 使用 concatArray 连接
        Observable.concatArray(observables)
                .subscribe(letter -> System.out.print(letter + " "));

        // 输出：
        // A B C D E F G
        // 顺序连接所有数组中的Observable
    }
```

### merge

merge() 用于并发合并多个 Observable，将所有数据按实际到达时间顺序混合发射。

核心思想：并行执行，谁先到谁先出，像多车道合并成单车道。

```java
    public static void main(String[] args) throws InterruptedException {
        Observable<Long> fast = Observable.interval(100, TimeUnit.MILLISECONDS)
                .map(i -> i + 100)  // 100, 101, 102...
                .take(3);
        Observable<Long> slow = Observable.interval(200, TimeUnit.MILLISECONDS)
                .map(i -> i + 200)  // 200, 201, 202...
                .take(3);

        Observable.merge(fast, slow)
                .subscribe(num -> System.out.print(num + " "));

        Thread.sleep(1000);

        // 输出（实际时间顺序）：
        // 100 101 200 102 201 202
        // 解释：
        // 100ms: fast发射100
        // 200ms: slow发射200, fast发射101
        // 300ms: fast发射102
        // 400ms: slow发射201
        // 600ms: slow发射202
    }
```

### zip

zip() 用于将多个 Observable 的数据按索引配对组合，像拉链一样一一对应。

核心思想：等待所有Observable都有数据，然后配对发射，以最短的Observable为准。

```java
    public static void main(String[] args) {
        Observable<String> letters = Observable.just("A", "B", "C", "D");
        Observable<Integer> numbers = Observable.just(1, 2, 3);

        Observable.zip(
                        letters,
                        numbers,
                        (letter, number) -> letter + number
                )
                .subscribe(result -> System.out.print(result + " "));

        // 输出：
        // A1 B2 C3
        // 注意：D 被丢弃了，因为 numbers 只有3个

    }
```

### startWith & startWithArray

这两个操作符都用于在 Observable 发射数据之前，先发射一些指定的数据，就像给数据流添加"开场白"。

- startWith()：添加单个元素或 Observable。
- startWithArray()：添加多个元素（可变参数）。

```java
    public static void main(String[] args) {
        // startWith: 添加单个元素
        Observable.just("B", "C")
                .startWith("A")
                .subscribe(letter -> System.out.print("startWith: " + letter + " "));

        // startWithArray: 添加多个元素
        Observable.just("D", "E")
                .startWithArray("A", "B", "C")
                .subscribe(letter -> System.out.print("startWithArray: " + letter + " "));

        // 输出：
        // startWith: A B C
        // startWithArray: A B C D E
    }
```

### count

count() 用于统计 Observable 发射的元素数量，返回一个发射单个计数值的 Observable。

核心思想：数一数发射了多少个元素。

```java
    public static void main(String[] args) {
        Observable.just("A", "B", "C", "D", "E")
                .count()
                .subscribe(count ->
                        System.out.println("元素数量: " + count)
                );

        // 输出：
        // 元素数量: 5
    }
```

## 功能操作符

功能操作符是 RxJava 中用于辅助调试、监控、错误处理、资源管理和工具性功能的操作符。它们不直接处理数据流，而是提供辅助功能。

### delay

delay() 用于延迟发射 Observable 的数据，可以延迟整个流，也可以延迟每个元素。
核心思想：让数据"等一会儿"再发射。

```java
    public static void main(String[] args) throws InterruptedException {
        System.out.println("开始时间: " + System.currentTimeMillis());

        Observable.just("A", "B", "C")
                .delay(1, TimeUnit.SECONDS)  // 延迟1秒
                .subscribe(item ->
                        System.out.println(System.currentTimeMillis() + ": " + item)
                );

        Thread.sleep(2000);

        // 输出：
        // 开始时间: 1766569373459
        // 1766569374574: A
        // 1766569374580: B
        // 1766569374580: C
        // 注意：A,B,C几乎同时发射，都延迟了1秒
    }
```

### subscribeOn & observerOn

这两个操作符用于控制 Observable 在哪个线程上执行和观察，是 RxJava 线程调度的核心。

- **`subscribeOn()`**：指定 Observable 执行在哪个线程
- **`observeOn()`**：指定 Observer 观察在哪个线程

subscribeOn 示例：

```java
    public static void main(String[] args) throws InterruptedException {
        Observable.create(emitter -> {
                    System.out.println("执行线程: " + Thread.currentThread().getName());
                    emitter.onNext("数据");
                    emitter.onComplete();
                })
                .subscribeOn(Schedulers.io())  // 在IO线程执行
                .subscribe(data ->
                        System.out.println("接收线程: " + Thread.currentThread().getName())
                );

        Thread.sleep(1000);

        // 输出：
        // 执行线程: RxCachedThreadScheduler-1
        // 接收线程: RxCachedThreadScheduler-1
        // 注意：执行和接收都在IO线程
    }
```

observeOn 示例：

```java
    public static void main(String[] args) throws InterruptedException {
        Observable.create(emitter -> {
                    System.out.println("执行线程: " + Thread.currentThread().getName());
                    emitter.onNext("数据");
                    emitter.onComplete();
                })
                .observeOn(Schedulers.io())  // 在IO线程接收
                .subscribe(data ->
                        System.out.println("接收线程: " + Thread.currentThread().getName())
                );

        Thread.sleep(1000);

        // 输出：
        // 执行线程: main
        // 接收线程: RxCachedThreadScheduler-1
        // 注意：执行在主线程，接收在IO线程
    }
```

### retry

retry 用于在 Observable 发生错误时自动重试，实现错误恢复机制，直到成功或达到重试上限。

```java
    public static void main(String[] args) {
        AtomicInteger attempt = new AtomicInteger();

        Observable.create(emitter -> {
                    attempt.getAndIncrement();
                    System.out.println("第" + attempt + "次尝试");
                    if (attempt.get() < 3) {
                        emitter.onError(new RuntimeException("随机失败"));
                    } else {
                        emitter.onNext("成功");
                        emitter.onComplete();
                    }
                })
                .retry(2)  // 最多重试2次
                .subscribe(
                        data -> System.out.println("结果: " + data),
                        error -> System.err.println("最终失败: " + error.getMessage())
                );

    // 输出：
    // 第1次尝试
    // 第2次尝试
    // 第3次尝试
    // 结果: 成功
    // 尝试3次后成功（初始1次+重试2次）
    }
```

### retryUntil

retryUntil 用于自定义重试停止条件，当条件满足时停止重试。

```java
    public static void main(String[] args) {
        AtomicInteger attempt = new AtomicInteger(0);
        Observable.create(emitter -> {
                    int count = attempt.incrementAndGet();
                    System.out.println("第" + count + "次尝试");
                    if (count < 3) {
                        emitter.onError(new RuntimeException("失败"));
                    } else {
                        emitter.onNext("成功");
                        emitter.onComplete();
                    }
                })
                .retryUntil(() -> {
                    // 返回 true 时停止重试
                    return attempt.get() >= 3;  // 尝试3次后停止
                })
                .subscribe(
                        data -> System.out.println("结果: " + data),
                        error -> System.err.println("最终失败: " + error.getMessage())
                );

        // 输出：
        // 第1次尝试
        // 第2次尝试
        // 第3次尝试
        // 结果: 成功
    }
```

## 生命周期的功能操作符

很多功能操作符贯穿整个发布订阅的生命周期中。

### doOnSubscribe- 订阅时

```java
    public static void main(String[] args) {
        Observable.just("A", "B", "C")
                .doOnSubscribe(disposable ->
                        System.out.println("有人订阅了！")
                )
                .subscribe();
        // 输出：
        // 有人订阅了！
    }
```

### doOnNext - 发射每个元素时

```java
    public static void main(String[] args) {
        Observable.range(1, 3)
                .doOnNext(num ->
                        System.out.println("即将发射: " + num)
                )
                .map(num -> num * 2)
                .doOnNext(num ->
                        System.out.println("发射后: " + num)
                )
                .subscribe();

        // 输出：
        // 即将发射: 1
        // 发射后: 2
        // 即将发射: 2
        // 发射后: 4
        // 即将发射: 3
        // 发射后: 6
    }
```

### doAfterNext - 发射后执行

```java
Observable.just("A", "B", "C")
    .doOnNext(item -> 
        System.out.println("发射前: " + item)
    )
    .doAfterNext(item -> 
        System.out.println("发射后: " + item)
    )
    .subscribe(item -> 
        System.out.println("接收: " + item)
    );

// 输出：
// 发射前: A
// 接收: A
// 发射后: A
// 发射前: B
// 接收: B
// 发射后: B
// 发射前: C
// 接收: C
// 发射后: C
```

### doOnError - 出错时

```java
    public static void main(String[] args) {
        Observable.error(new RuntimeException("测试错误"))
                .doOnError(error ->
                        System.err.println("捕获到错误: " + error.getMessage())
                )
                .onErrorReturnItem("默认值")
                .subscribe();

        // 输出：
        // 捕获到错误: 测试错误
    }
```

### doOnComplete - 完成时

```java
    public static void main(String[] args) {
        Observable.just("任务1", "任务2")
                .doOnComplete(() ->
                        System.out.println("所有任务都完成了！")
                )
                .subscribe();

        // 输出：
        // 所有任务都完成了！
    }
```

### doOnTerminate - 完成/错误前执行

```java
    public static void main(String[] args) {
        // 正常完成的情况
        Observable.just("A", "B")
                .doOnTerminate(() ->
                        System.out.println("即将完成/出错")
                )
                .doOnComplete(() ->
                        System.out.println("doOnComplete")
                )
                .subscribe(
                        item -> System.out.println("接收: " + item),
                        error -> {
                        },
                        () -> System.out.println("onComplete")
                );

        // 输出：
        // 接收: A
        // 接收: B
        // 即将完成/出错
        // doOnComplete
        // onComplete

        // 出错的情况
        Observable.error(new RuntimeException("测试"))
                .doOnTerminate(() ->
                        System.out.println("即将完成/出错")
                )
                .doOnError(error ->
                        System.out.println("doOnError: " + error.getMessage())
                )
                .subscribe(
                        item -> {
                        },
                        error -> System.out.println("onError: " + error.getMessage())
                );

        // 输出：
        // 即将完成/出错
        // doOnError: 测试
        // onError: 测试

    }
```

### doAfterTerminate - 完成/错误后执行

```java
    public static void main(String[] args) {
        // 正常完成的情况
        Observable.just("A", "B")
                .doAfterTerminate(() ->
                        System.out.println("完成/出错后执行")
                )
                .doOnComplete(() ->
                        System.out.println("doOnComplete")
                )
                .subscribe(
                        item -> System.out.println("接收: " + item),
                        error -> {
                        },
                        () -> System.out.println("onComplete")
                );

        // 输出：
        // 接收: A
        // 接收: B
        // doOnComplete
        // onComplete
        // 完成/出错后执行

        // 出错的情况
        Observable.error(new RuntimeException("测试"))
                .doAfterTerminate(() ->
                        System.out.println("完成/出错后执行")
                )
                .doOnError(error ->
                        System.out.println("doOnError: " + error.getMessage())
                )
                .subscribe(
                        item -> {
                        },
                        error -> System.out.println("onError: " + error.getMessage())
                );

        // 输出：
        // doOnError: 测试
        // onError: 测试
        // 完成/出错后执行
    }
```

### doOnDispose - 取消订阅时

```java
    public static void main(String[] args) throws InterruptedException {
        Disposable disposable = Observable.interval(1, TimeUnit.SECONDS)
                .doOnDispose(() ->
                        System.out.println("订阅被取消了")
                )
                .subscribe(num ->
                        System.out.println("计数: " + num)
                );

        Thread.sleep(3500);
        disposable.dispose();

        // 输出：
        // 计数: 0
        // 计数: 1
        // 计数: 2
        // 订阅被取消了
    }
```

### doFinally - 最终执行

```java
    public static void main(String[] args) {
        Observable.just("数据")
                .doFinally(() ->
                        System.out.println("无论如何都会执行")
                )
                .subscribe();

        // 输出：
        // 无论如何都会执行
    }
```

### doOnEach - 每个事件时

```java
    public static void main(String[] args) {
        Observable.just("A", "B", "C")
                .doOnEach(notification -> {
                    if (notification.isOnNext()) {
                        System.out.println("发射: " + notification.getValue());
                    } else if (notification.isOnComplete()) {
                        System.out.println("完成");
                    } else if (notification.isOnError()) {
                        System.out.println("错误: " + notification.getError());
                    }
                })
                .subscribe();

        // 输出：
        // 发射: A
        // 发射: B
        // 发射: C
        // 完成
    }
```

### doOnRequest - 请求时（背压相关）

```java
    public static void main(String[] args) {
        Flowable.range(1, 100)
                .doOnRequest(requested ->
                        System.out.println("下游请求了 " + requested + " 个元素")
                )
                .subscribe(
                        num -> System.out.println("接收: " + num),
                        error -> {
                        },
                        () -> System.out.println("完成")
                );

    }
```

# 结束

响应式编程确实需要一些时间来掌握，但一旦理解核心概念，你就会发现它在处理异步数据流时的强大之处。

从回调地狱到流畅的链式调用，从手忙脚乱的资源管理到智能的背压控制，响应式编程让复杂异步操作变得清晰可控。虽然学习曲线存在，但投入是值得的。

记住，不是所有场景都需要响应式。在合适的业务场景下使用，才能发挥最大价值。建议从实际项目的小模块开始尝试，逐步积累经验。

希望这篇内容能为你打开响应式编程的大门。编程之路就是不断学习的过程，保持好奇，继续探索吧！
