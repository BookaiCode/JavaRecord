本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

[TOC]

## 摘要

ExecutorCompletionService 是Java并发编程中的一个有用的工具类，它实现了 CompletionService 接口。ExecutorCompletionService 将 Executor 和BlockingQueue 功能融合在一起，使用它可以提交我们的任务。这个任务委托给 Executor 执行，可以使用 ExecutorCompletionService 对象的 take() 和 poll() 方法获取结果。

本文将深入讲解 ExecutorCompletionService 的使用以及源码解析。

## ExecutorCompletionService适用场景

ExecutorCompletionService在以下场景中特别有用：

- **并行任务处理**：当需要同时执行多个任务，并按照完成的顺序获取它们的结果时，可以使用ExecutorCompletionService来简化任务提交和结果获取的流程。
- **高性能计算**：在需要进行大规模计算或复杂计算的场景中，可以将任务拆分成多个子任务，并使用ExecutorCompletionService来管理和获取子任务的结果。

假设现在有一批需要进行计算的任务，为了提高整批任务的执行效率，我们可以使用线程池来异步计算这些任务。通过向线程池中不断提交任务并保留与每个任务关联的Future对象。最后，我们可以遍历这些Future对象，并通过调用 get() 方法获取每个任务的计算结果。

**Future的不足**

Future 没有办法回调，只能手动去调用，当通过 get() 方法获取线程的返回值时，会导致阻塞，也就是和当前这个 Future 关联的计算任务执行完成的时候才返回结果，新任务必须等待已完成任务的结果才能继续进行处理。

这样会浪费很多时间，因为我们不知道哪个线程先执行完了，只能挨个去获取结果，这样已经完成的线程会因为前面未完成的线程的耗时而无法提前进行汇总，最好是谁先执行完成，谁先返回。

而 ExecutorCompletionService 可以实现这样的效果，节省获取完成结果的时间，它的内部有一个先进先出的阻塞队列，用于保存已经执行完成的 Future，通过调用它的 take() 方法或 poll() 方法可以获取到一个已经执行完成的 Future，进而通过调用 Future 接口实现类的 get() 方法获取最终的结果。

**CompletionService的目标是任务谁先完成谁先获取，即结果按照完成先后顺序排序**

## ExecutorCompletionService使用

ExecutorCompletionService 提供了一种方便的方式来处理一组异步任务，并按照完成的顺序获取它们的结果。它内部使用了Executor框架来执行任务，并且内部管理着一个已完成任务的阻塞队列，在结果获取上提供了更加灵活和高效的机制。

下面是一个简单的例子来演示ExecutorCompletionService的基本使用：

```java
public class ExecutorCompletionServiceExample {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        CompletionService<String> completionService = new ExecutorCompletionService<>(executor);

        // 提交任务
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            completionService.submit(() -> {
                double sleepTime = Math.random() * 1000;
                Thread.sleep((long) sleepTime); // 模拟耗时操作
                return "Task " + taskId + " completed,cost time: " + sleepTime;
            });
        }

        // 获取结果
        for (int i = 0; i < 10; i++) {
            Future<String> future = completionService.take();
            String result = future.get();
            System.out.println(result);
        }

        executor.shutdown();
    }
}
```

输出：

```java
Task 2 completed,cost time: 170.01927312611775
Task 3 completed,cost time: 460.9622858036789
Task 1 completed,cost time: 563.24738180643
Task 0 completed,cost time: 595.938819219159
Task 5 completed,cost time: 480.4473056068137
Task 4 completed,cost time: 748.2343208613524
Task 6 completed,cost time: 370.4679098376097
Task 7 completed,cost time: 270.45945981324905
Task 9 completed,cost time: 336.5536570760892
Task 8 completed,cost time: 577.5774464801026
```

在上述代码中，我们创建了一个固定大小的线程池，并使用 ExecutorCompletionService 来提交和获取任务的结果。通过调用`completionService.submit()`方法来提交任务，并随机指定睡眠时间，来模拟任务执行的耗时，然后通过`completionService.take()`方法来获取已完成的任务结果。

可以看到是按照任务的执行耗时顺序去获取结果的。

## ExecutorCompletionService原理解析

ExecutorCompletionService 提供了两个构造函数，一个可以指定阻塞队列，另一个使用内部默认的阻塞队列，两个构造函数都需要传进线程池参数。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP4AeGiaIibscmL8AGee4KDbTHIeYd1RmMCn9PDAd6thkGaoquiaR0GB7jnbQbFk8AKMCib9ZV8IjuLRQ/640)

提供了三个获取方法，可以看到都是从队列中获取。

- take()/poll() 方法的工作都委托给内部的已完成任务队列 completionQueue。
- 如果队列中有已完成的任务, take() 方法就返回任务的结果，否则阻塞等待任务完成。
- poll() 与 take() 方法不同，poll() 有两个版本:
  - 无参的 poll() 方法：如果完成队列中有数据就返回，否则返回null。
  - 有参数的 poll() 方法：如果完成队列中有数据就直接返回，否则等待指定的时间，到时间后如果还是没有数据就返回null。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP4AeGiaIibscmL8AGee4KDbT3NyPwDRreuZeLk0y68NBwN09211oDT8KSO4chIlhIc5z8QVMQj9BFQ/640)

两个提交任务方法，可以看到 submit() 方法最终会委托给内部的 executor 去执行任务，提交任务的时候会将任务封装成 QueueingFuture 对象。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP4AeGiaIibscmL8AGee4KDbTsYcu0NAwW2DgcqTfp4DdHc3WRbVGgZI3auWJJzBChM75ibL7jexTvtg/640)

ExecutorCompletionService内部维护了 `QueueingFuture` 类，`QueueingFuture` 继承了 `FutureTask`，并重写了 `done(`) 方法，

可以看到 done() 方法在任务完成的时候会将结果存进 已完成任务队列 completionQueue 中。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP4AeGiaIibscmL8AGee4KDbTObydfiaiboKvlvKsXyGCic7RzekzHgGbYSdxb9ibKAMkJtOiby6G29O9LJA/640)

Futuretask 的 done() 方法是用来标记一个任务已经完成的方法。当一个 Futuretask 中的任务完成后，就会调用 done() 方法通知。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP4AeGiaIibscmL8AGee4KDbT5A6I07gXibMZwaqCcesF3T4rX8ObECdZA7eO3P1q9MyCkoAt9o2wVbQ/640)

默认是空方法，不会执行任何动作。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScP4AeGiaIibscmL8AGee4KDbTmdtibiaq3OibxPiaWfveAafqLdIzqJBBMXrOCJXgrKFAlaLu2ovfdjGoXw/640)



**执行流程**

当我们使用ExecutorCompletionService类时，它能够按照任务完成的顺序获取它们的结果，这是因为ExecutorCompletionService类内部结合了QueueingFuture类和done()方法的机制。以下是源码流程步骤解释：

1. 提交任务：
   - 我们通过submit方法将任务提交给ExecutorCompletionService。在提交任务时，ExecutorCompletionService会使用自定义的QueueingFuture类来包装任务，并将其交给底层线程池执行。
2. QueueingFuture类：
   - QueueingFuture类是ExecutorCompletionService的内部类，继承自FutureTask。它的构造方法接收一个Callable对象作为参数。
   - 在QueueingFuture类中，它重写了done()方法。done()方法会在任务执行完成后被调用。
3. 任务执行完成时的处理：
   - 当任务执行完成后，在底层线程池的Worker线程中，会调用QueueingFuture的done()方法。
   - 在done()方法中，QueueingFuture会首先调用父类FutureTask的done()方法，以触发对计算结果的获取。然后，它会将任务的结果存储到一个内部的BlockingQueue队列中（即completionQueue）。
4. 获取任务结果：
   - 当我们调用take方法获取任务结果时，它会从completionQueue队列中取出已完成的任务结果，并返回该结果。如果队列为空，则会阻塞等待，直到有任务完成并返回结果。
   - take方法内部会调用QueueingFuture的get()方法，从而触发对应任务的计算结果的获取。
5. 保证按顺序获取结果：
   - 由于completionQueue是一个阻塞队列，并且在done()方法中将任务结果按照完成的顺序放入队列中，因此我们可以通过按顺序获取队列中的任务结果，来保证按照任务完成的顺序获取它们的结果。

通过以上源码流程步骤，ExecutorCompletionService类能够按照任务完成的顺序获取结果。它利用QueueingFuture类包装任务并存储结果到阻塞队列中，在任务执行完成后，按照完成的顺序将结果放入队列，从而实现了按顺序获取结果的功能。

## 注意事项

在使用ExecutorCompletionService时，需要注意以下事项：

- **合理选择线程池大小**：根据任务的数量和复杂性，合理选择线程池的大小，以充分利用系统资源并避免资源浪费。
- **及时处理异常**：在任务执行过程中，如果发生异常，需要及时处理和记录异常信息，以保证程序的稳定性和可靠性。
- **使用Future对象进行任务取消和超时控制**：通过使用Future对象的cancel方法，可以取消正在执行的任务。同时，可以通过调整 poll 方法的参数来设置超时时间，避免长时间等待任务结果而导致阻塞。

## 总结

ExecutorCompletionService是一个强大且灵活的工具类，能够简化异步任务的处理和结果获取过程。通过使用ExecutorCompletionService，我们可以更加高效地处理一组异步任务，并按照完成的顺序获取它们的结果。

本文介绍了ExecutorCompletionService的基本使用方法，并对其源码进行了解析。希望通过这篇文章能够帮助读者更好地理解和应用ExecutorCompletionService。