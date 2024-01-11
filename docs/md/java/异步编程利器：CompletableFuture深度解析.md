## 摘要

在异步编程中，我们经常需要处理各种异步任务和操作。Java 8引入的 CompletableFuture 类为我们提供了一种强大而灵活的方式来处理异步编程需求。CompletableFuture 类提供了丰富的方法和功能，能够简化异步任务的处理和组合。

本文将深入解析 CompletableFuture，希望对各位读者能有所帮助。

**CompletableFuture 适用于以下场景**

- 并发执行多个异步任务，等待它们全部完成或获取其中任意一个的结果。
- 对已有的异步任务进行进一步的转换、组合和操作。
- 异步任务之间存在依赖关系，需要按照一定的顺序进行串行执行。
- 需要对异步任务的结果进行异常处理、超时控制或取消操作。

## 如何使用

下面是一个演示 CompletableFuture 如何使用的代码示例：

```java
public class CompletableFutureExample {

    public static void main(String[] args) {
        // 创建CompletableFuture对象，并定义异步任务
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            // 异步任务的逻辑代码
            // 在这里执行耗时操作或其他需要异步执行的任务
            try {
                TimeUnit.SECONDS.sleep(2); // 模拟耗时操作
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Hello, ";
        });

        // 添加任务完成后的回调方法
        CompletableFuture<String> resultFuture = future.thenApplyAsync(result -> {
            // 任务完成后的处理逻辑
            // result为上一步任务的结果
            return result + "World!";
        });

        // 组合多个CompletableFuture对象
        CompletableFuture<String> combinedFuture = future.thenCombine(resultFuture, (result1, result2) -> {
            // 对多个CompletableFuture的结果进行组合处理
            return result1 + result2 + " Welcome to the CompletableFuture world!";
        });

        // 异常处理
        CompletableFuture<String> exceptionHandledFuture = combinedFuture.exceptionally(ex -> {
            // 异常处理逻辑
            System.out.println("任务执行出现异常：" + ex.getMessage());
            return "Fallback Result";
        });

        // 等待并获取任务的结果
        try {
            String result = exceptionHandledFuture.get();
            System.out.println("任务的最终结果为：" + result);
        } catch (InterruptedException | ExecutionException e) {
            // 处理异常情况
            e.printStackTrace();
        }
    }
}
```

结果输出：

```java
任务的最终结果为：Hello, Hello, World! Welcome to the CompletableFuture world!
```

首先，我们创建了一个`CompletableFuture`对象`future`。在`future`中，我们使用`supplyAsync`方法定义了一个异步任务，其中 lambda表达式 中的代码会在另一个线程中执行。在这个例子中，我们模拟了一个耗时操作，通过`TimeUnit.SECONDS.sleep(2)`暂停了2秒钟。

然后，我们添加了一个回调方法`resultFuture`。在这个回调方法中，将前一个异步任务的结果作为参数进行处理，并返回处理后的新结果。在这个例子中，我们将前一个任务的结果与字符串 "World!" 连接起来，形成新的结果。

接下来，我们使用`thenCombine`方法组合了两个`CompletableFuture`对象：`future`和`resultFuture`。在这个组合任务中，我们将两个任务的结果进行组合处理，返回最终的结果。在这个例子中，我们将前两个任务的结果与字符串 " Welcome to the CompletableFuture world!" 连接起来。

此外，我们还处理了异常情况。通过`exceptionally`方法，我们定义了一个异常处理回调方法。如果在任务执行过程中发生了异常，我们可以在这里对异常进行处理，并返回一个默认值作为结果。

最后，我们使用`get`方法等待并获取最终的任务结果。需要注意的是，`get`方法可能会阻塞当前线程，直到任务完成并返回结果。在这个例子中，我们使用`try-catch`块捕获可能的异常情况，并打印出最终的任务结果。

这个例子只是部分展示了`CompletableFuture`的功能，实际上它比你想象的还要强大！

## 源码解析

CompletableFuture 的源码非常庞大和复杂，涉及到并发、线程池、同步机制等多方面的知识。在这里，我们只重点介绍 CompletableFuture 的核心实现原理。

### 基本结构

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScO4YMibB6Xu8xMZ8QkQz8VE6zdA0AMoo6OHiaowFoic9ZRSrbyuKPbdxeW04bgE8tzasNFibSOLKDzehA/640)

CompletableFuture 的作者是大名鼎鼎的 Doug Lea。CompletableFuture 类是实现了 Future 和 CompletionStage 接口的一个关键类。它可以表示异步计算的结果，并提供了一系列方法来操作和处理这些结果。

CompletableFuture 内部使用了一个属性`result`来保存计算结果，以及若干个属性`waiters`来保存等待结果的任务。当计算完成后，CompletableFuture将会通知所有等待结果的任务，并将结果传递给它们。

为了实现链式操作，CompletableFuture还定义了内部类：`Completion`, `UniCompletion`, 和 `BiCompletion`。

`Completion`, `UniCompletion`, 和 `BiCompletion` 是 `CompletableFuture` 内部用于处理异步任务完成的辅助类。

- `Completion` 是一个通用的辅助类，它包含了任务完成后的回调方法，以及处理异常的方法。
- `UniCompletion` 是 `Completion` 的子类，是一元依赖的基类，用于处理单个任务的完成情况，并提供了更多的方法来处理结果和异常。
- `BiCompletion` 是 `UniCompletion` 的子类，是二元依赖的基类，同时也是多元依赖的基类，用于处理两个任务的完成情况，并提供了更多的方法来组合和处理这两个任务的结果和异常。

这些辅助类在 `CompletableFuture` 的内部被使用，以实现异步任务的执行、结果的处理和组合等操作。它们提供了一种灵活的方式来处理异步任务的完成情况，并通过回调方法或其他一些方法来处理任务的结果和异常。

### 内部原理

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOmicjMp7XdT2Oaykl41iaArlvhpkDr9kJCKR64WTnEEgLPTzXuKo57wvOXib42AicRlVPYSh1pb4cavw/640)

CompletableFuture中包含两个字段：**result** 和 **stack**。result 用于存储当前CF的结果，stack (Completion）表示当前CF完成后需要触发的依赖动作(Dependency Actions)，去触发依赖它的CF的计算，依赖动作可以有多个(表示有多个依赖它的CF)，以栈（Treiber stack）的形式存储，stack表示栈顶元素。

CompletableFuture 在设计思想上类似 “观察者模式，每个 CompletableFuture 都可以被看作一个被观察者，其内部有一个Completion类型的链表成员变量stack，用来存储注册到其中的所有观察者。当被观察者执行完成后会弹栈stack属性，依次通知注册到其中的观察者。

### 执行流程

CompletableFuture 的执行流程如下：

1. 创建CompletableFuture对象：通过调用`CompletableFuture`类的构造方法或静态工厂方法创建一个新的CompletableFuture对象。
2. 定义异步任务：使用`supplyAsync()`、`runAsync()`等方法定义需要在后台线程中执行的异步任务，这些方法接受一个 lambda表达式 或 Supplier/Runnable 接口作为参数。
3. 启动异步任务：一旦CompletableFuture对象创建并定义了异步任务，任务会立即在后台线程中开始执行，并返回一个代表异步计算结果的CompletableFuture对象。
4. 异步任务执行过程：
   - 当异步任务完成时，它会设置自己的结果值，将状态标记为已完成。
   - 如果有其他线程在此之前调用了`complete()`、`completeExceptionally()`、`cancel()`等方法，可能会影响任务的最终状态。
5. 注册回调方法：
   - 使用`thenApply()`, `thenAccept()`, `thenRun()`等方法来注册回调函数，当异步任务完成或异常时，这些回调函数会被触发。
   - 回调函数也可以是异步的，通过`thenApplyAsync()`, `thenAcceptAsync()`, `thenRunAsync()`等方法注册。
6. 组合多个CompletableFuture：
   - 使用`thenCompose()`, `thenCombine()`, `allOf()`, `anyOf()`等方法，可以将多个CompletableFuture对象进行组合，形成更复杂的异步任务处理流程。
7. 处理异常：
   - 通过使用`exceptionally()`, `handle()`, `whenComplete()`等方法，可以注册异常处理函数，当异步任务出现异常时，这些处理函数会被触发。
8. 等待结果：
   - 使用`get()`或`join()`方法来阻塞当前线程，并等待CompletableFuture对象的完成并获取最终的结果。
   - `get()`方法会抛出可能的异常（InterruptedException, ExecutionException）。
   - `join()`方法与`get()`类似，但不会抛出 checked 异常。
9. 取消任务：通过调用CompletableFuture对象的`cancel()`方法取消异步任务的执行。

请注意，以上步骤的顺序和具体实现可能略有不同，但大致上反映了CompletableFuture的执行流程。在实际应用中，我们可以根据需求选择适合的方法来处理异步任务的完成情况、结果、异常以及任务之间的关系。

## 方法介绍

CompletableFuture类提供了一系列用于处理和组合异步任务的方法。以下是这些方法的介绍：

### 创建对象

创建一个 `CompletableFuture` 对象有以下几种方法：

- 使用 `CompletableFuture` 的构造方法

```java
CompletableFuture<String> future = new CompletableFuture<>();
```

- 使用 `CompletableFuture` 的静态工厂方法

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 异步任务逻辑
    return "Result";
});

CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    // 异步任务逻辑
});
```

- 使用转换方法

```java

CompletableFuture<Integer> transformedFuture = originalFuture.thenApply(result -> {
    // 转换逻辑
    return result.length();
});

originalFuture.thenAccept(result -> {
    // 处理结果逻辑
    System.out.println("Result: " + result);
});

CompletableFuture<Void> runnableFuture = originalFuture.thenRun(() -> {
    // 在结果完成后执行的操作
});
```

- 直接创建一个已完成状态的CompletableFuture

```java
//CompletableFuture.completedFuture()直接创建一个已完成状态的CompletableFuture
CompletableFuture<String> cf2 = CompletableFuture.completedFuture("result");

//先初始化一个未完成的CompletableFuture，然后通过complete()、completeExceptionally()，也完成该CompletableFuture
CompletableFuture<String> cf = new CompletableFuture<>();
cf.complete("success");
```

- toCompletableFuture

```JAVA
CompletionStage<Integer> stage = CompletableFuture.supplyAsync(() -> 42);

CompletableFuture<Integer> future = stage.toCompletableFuture();
```

用于将当前的 `CompletionStage` 对象转换为一个 `CompletableFuture` 对象。

### 异步执行任务

以下是在 `CompletableFuture` 对象上异步执行任务的一些方法示例：

- `supplyAsync(Supplier<U> supplier)`：异步执行一个有返回值的供应商（Supplier）任务。

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 异步任务逻辑
    return "Result";
});
```

- `runAsync(Runnable runnable)`：异步执行一个没有返回值的任务。

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    // 异步任务逻辑
});
```

### 链式操作

CompletableFuture提供了不同的方式来对异步任务进行链式操作。

- thenRun

```java
CompletableFuture<Void> executedFuture = future.thenRun(() -> executeTask());
```

`thenRun`方法用于在CompletableFuture完成后执行一个Runnable任务。它返回一个新的CompletableFuture对象，该对象没有返回值。

- thenAccept

```java
CompletableFuture<Void> acceptedFuture = future.thenAccept(result -> processResult(result));
```

`thenAccept`方法用于在CompletableFuture完成后对结果进行处理。它接收一个Consumer函数作为参数，并返回一个新的CompletableFuture对象。

- thenApply

```java
CompletableFuture<U> appliedFuture = future.thenApply(result -> transformResult(result));
```

`thenApply`方法用于在CompletableFuture完成后对结果进行转换。它接收一个Function函数作为参数，并返回一个新的CompletableFuture对象。

- thenCompose

```java
CompletableFuture<U> composedFuture = future.thenCompose(result -> executeAnotherTask(result));
```

用于对异步任务的结果进行处理，并返回一个新的异步任务。

- whenComplete

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 42);

CompletableFuture<Void> whenCompleteFuture = future.whenComplete((result, exception) -> {
    if (exception != null) {
        System.out.println("Exception occurred: " + exception.getMessage());
    } else {
        System.out.println("Result: " + result);
    }
});

whenCompleteFuture.join();
```

用于在异步任务完成后执行指定的动作。它允许你在任务完成时处理结果或处理异常。

- `thenCompose()` 用于对异步任务的结果进行处理，并返回一个新的异步任务。它接受一个函数式接口参数，根据原始任务的结果创建并返回一个新的 `CompletionStage` 对象。
- `whenComplete()` 用于在异步任务完成后执行指定的动作。它接受一个消费者函数式接口参数，用于处理任务的结果或异常，但没有返回值。

### 异步任务组合

CompletableFuture还提供了一系列方法来组合和处理多个异步任务的结果。

- allOf

```java
CompletableFuture<Void> allFuture = CompletableFuture.allOf(future1, future2, future3);
```

`allOf`方法接收一组CompletableFuture对象作为参数，并返回一个新的CompletableFuture对象，该对象在所有给定的CompletableFuture都完成时完成。这样我们可以等待所有任务都完成后再进行下一步操作。

- anyOf

```java
CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(future1, future2, future3);
```

`anyOf`方法与`allOf`类似，不同之处在于它返回的CompletableFuture对象在任何一个给定的CompletableFuture完成时就完成。这样我们可以获取最先完成的任务的结果。

- thenCombine

```java
CompletableFuture<U> combinedFuture = future1.thenCombine(future2, (result1, result2) -> combineResults(result1, result2));
```

`thenCombine`方法接收两个CompletableFuture对象和一个函数作为参数，用于指定当这两个CompletableFuture都完成时如何处理它们的结果。返回的新的CompletableFuture对象将接收到计算后的结果。

- applyToEither

```java
CompletableFuture<U> resultFuture = future1.applyToEither(future2, result -> processResult(result));
```

`applyToEither`方法用于获取两个CompletableFuture中任意一个完成的结果，并对该结果进行处理。它接收一个Function函数作为参数，并返回一个新的CompletableFuture对象。

- acceptEither

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);

future1.acceptEither(future2, result -> {
    System.out.println("Result: " + result);
});
```

用于在两个 `CompletableFuture` 对象中任意一个完成时执行指定的操作。该方法接收两个参数：另一个 `CompletableFuture` 对象和一个消费者函数（`Consumer`）。当其中任何一个 `CompletableFuture` 完成时，将其结果作为参数传递给消费者函数进行处理。

- runAfterBoth 

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 42);
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Void> combinedFuture = future1.runAfterBoth(future2, () -> {
    System.out.println("Both futures completed");
});

combinedFuture.join();
```

用于在两个异步任务都完成后执行指定的动作，需要注意的是，`runAfterBoth()` 方法是一个非阻塞方法，动作将在两个异步任务都完成后立即执行。

- runAfterEither

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(500);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 42;
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "Hello";
});

CompletableFuture<Void> eitherFuture = future1.runAfterEither(future2, () -> {
    System.out.println("One of the futures completed");
});

eitherFuture.join();
```

用于在两个异步任务中任意一个完成后执行指定的动作。

- thenAcceptBoth

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 42);
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Void> thenAcceptBothFuture = future1.thenAcceptBoth(future2, (result1, result2) -> {
    System.out.println("Action executed with thenAcceptBoth(): " + result1 + ", " + result2);
});

thenAcceptBothFuture.join();
```

用于在两个异步任务都完成后执行指定的动作。它的作用是接收两个异步任务的结果，并将结果作为参数传递给指定的消费者函数。

### 异常处理

CompletableFuture提供了多种方式来处理异步任务的异常情况。

- exceptionally

```java
CompletableFuture<U> exceptionHandledFuture = future.exceptionally(ex -> handleException(ex));
```

通过`exceptionally`方法，我们可以对CompletableFuture的异常情况进行处理。它接收一个Function函数作为参数，用于处理异常并返回一个新的CompletableFuture对象。

- handle

```java
CompletableFuture<U> handledFuture = future.handle((result, ex) -> handleResult(result, ex));
```

`handle`方法可以同时处理正常结果和异常情况。它接收一个BiFunction函数作为参数，用于处理结果和异常，并返回一个新的CompletableFuture对象。

- completeExceptionally

```
future.completeExceptionally();
```

异常地完成 `CompletableFuture`，将结果设置为一个异常。

- isCompletedExceptionally

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException("Something went wrong");
});

boolean completedExceptionally = future.isCompletedExceptionally();
System.out.println("Is completed exceptionally: " + completedExceptionally);
```

该方法返回一个布尔值，表示当前异步任务是否已经异常完成。

- obtrudeException

```java
CompletableFuture<Integer> future = new CompletableFuture<>();

future.obtrudeException(new RuntimeException("Something went wrong"));

boolean completedExceptionally = future.isCompletedExceptionally();
System.out.println("Is completed exceptionally: " + completedExceptionally);
```

用于强制将指定的异常作为异步任务的结果，调用 `obtrudeException(Throwable ex)` 方法后，异步任务将立即完成，并将指定的异常作为结果返回。

### 取值与状态

- join

```java
future.join()
```

`join()` 方法不会抛出已检查异常，因为它是基于 `CompletableFuture` 类设计的，如果异步任务抛出异常，`join()` 方法会将该异常包装在 `CompletionException` 中并抛出。

- get

```java
future.get()
```

`get()` 方法会抛出一个 `InterruptedException` 异常和一个 `ExecutionException` 异常，前者表示获取结果时被中断，后者表示获取结果时任务本身抛出了异常。

```java
future.get(1,TimeUnit.Hours)
```

 有异常则抛出异常，最长等待一个小时，一个小时之后，如果还没有数据，则异常。

- getNow

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // 异步任务逻辑
    return 42;
});

int result = future.getNow(0); // 获取异步操作的结果，如果尚未完成，则返回默认值0

System.out.println("Result: " + result);
```

`getNow(T value)` 是 `CompletableFuture` 类的一个方法，用于获取异步操作的结果，如果异步操作尚未完成，则返回给定的默认值，该方法会立即返回结果，不会阻塞当前线程。

### 超时控制与取消操作

CompletableFuture也支持超时控制和取消操作，以便更好地管理异步任务的执行。

- completeOnTimeout

```java
CompletableFuture<U> timeoutFuture = future.completeOnTimeout(defaultResult, timeout, timeUnit);
```

`completeOnTimeout`方法在指定的超时时间内等待CompletableFuture的完成，如果超时则将其设置为默认结果。它返回一个新的CompletableFuture对象。

- cancel

```java
boolean isCancelled = future.cancel(true);
```

`cancel`方法可用于取消CompletableFuture的执行。它接收一个boolean参数，指示是否中断正在执行的任务。返回值表示是否成功取消了任务。

- isCancelled

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // 异步任务逻辑
    return 42;
});

future.cancel(true); // 取消异步任务

boolean isCancelled = future.isCancelled();
System.out.println("Is cancelled: " + isCancelled);
```

`isCancelled()` 是 `CompletableFuture` 类的一个方法，用于判断当前异步任务是否已被取消。如果异步任务已被取消，则返回 `true`；否则返回 `false`。

### 依赖

- getNumberOfDependents

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);

CompletableFuture<Integer> combinedFuture = future1.thenCombine(future2, (result1, result2) -> result1 + result2);

int numberOfDependents = combinedFuture.getNumberOfDependents();
System.out.println("Number of dependents: " + numberOfDependents);
```

`getNumberOfDependents()` 用于获取当前 `CompletableFuture` 对象所依赖的其他异步任务的数量。如果没有任何依赖任务，或者所有依赖任务已经完成，则返回的数量为0。

### 完成

- complete

```java
future.complete("米饭");
```

`complete(T value)`：该方法返回布尔值，表示是否成功地将结果设置到 `CompletableFuture` 中。如果 `CompletableFuture` 未完成，则将结果设置，并返回 `true`；如果 `CompletableFuture` 已经完成，则不进行任何操作并返回 `false`。

- obtrudeValue 

```java
CompletableFuture<Integer> future = new CompletableFuture<>();

future.obtrudeValue(42);

boolean completedNormally = future.isDone() && !future.isCompletedExceptionally();
System.out.println("Is completed normally: " + completedNormally);
```

用于强制将指定的值作为异步任务的结果，调用 `obtrudeValue(T value)` 方法后，异步任务将立即完成，并将指定的值作为结果返回。

与 `complete()` 不同，`obtrudeValue()` 必须在任务已经完成的情况下调用，否则会引发 `IllegalStateException` 异常。并且`complete()` 方法对于已经完成的任务会忽略额外的完成操作，并返回 `false`。而`obtrudeValue()` 方法即使任务已经完成，仍然会强制使用新的结果值，并返回 `true`。

- isDone

```java
CompletableFuture<Integer> future = CompletableFuture.completedFuture(42);

boolean done = future.isDone();
System.out.println("Is done: " + done);
```

用于判断当前异步任务是否已经完成（无论是正常完成还是异常完成）。

### 并发限制

CompletableFuture也支持并发限制，以控制同时执行的异步任务数量。

```java
Executor executor = Executors.newFixedThreadPool(10);
CompletableFuture<U> future = CompletableFuture.supplyAsync(() -> doSomething(), executor);
```

我们可以通过使用线程池来限制CompletableFuture的并发执行数量。通过创建一个固定大小的线程池，并将其作为参数传递给CompletableFuture，就可以控制并发执行任务的数量。

### 记忆窍门

CompletableFuture类提供了许多方法，但实际上常用的方法只有几个。为了方便记忆，以下是一些总结的规律：

- 方法名带**Async**的都是异步方法，对应的没有Async则是同步方法，比如 `thenAccept`  与 `thenAcceptAsync` 。
- 方法名带**run**的入参为Runnable，且无返回值。
- 方法名带**supply**的入参为Supplier，且有返回值。
- 方法名带**Accept**的入参为Consumer，且无返回值。
- 方法名带**Apply**的入参为Function，且有返回值。
- 方法名带**Either**的方法表示谁先完成就消费谁。
- 方法名带**Both**的方法表示两个任务都完成才消费。

掌握以上规律后，就可以基本记住大部分方法，剩下的其他方法可以单独记忆。

## 总结

本文详细探讨了 CompletableFuture 的原理和方法，学习了如何在任务完成后执行操作、处理结果和转换结果。

CompletableFuture是Java中强大的异步编程工具之一，合理利用它的方法和策略可以更好地处理异步任务和操作。

希望本文对读者有所启发和帮助。