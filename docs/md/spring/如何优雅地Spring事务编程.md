在开发中，有时候我们需要对 Spring 事务的生命周期进行监控，比如在事务提交、回滚或挂起时触发特定的逻辑处理。那么如何实现这种定制化操作呢？

Spring 作为一个高度灵活和可扩展的框架，早就提供了一个强大的扩展点，即事务同步器 TransactionSynchronization 。通过 TransactionSynchronization ，我们可以轻松地控制事务生命周期中的关键阶段，实现自定义的业务逻辑与事务管理的结合。

```java
package org.springframework.transaction.support;

import java.io.Flushable;

public interface TransactionSynchronization extends Flushable {
  	/** 事务提交状态 */
    int STATUS_COMMITTED = 0;
  	/** 事务回滚状态 */
    int STATUS_ROLLED_BACK = 1;
  	/**系统异常状态 */
    int STATUS_UNKNOWN = 2;
    //挂起该事务同步器
    default void suspend() {
    }
    //恢复事务同步器
    default void resume() {
    }
    //flush底层的session到数据库
    default void flush() {
    }
	  // 事务提交之前
    default void beforeCommit(boolean readOnly) {
    }
		// 操作完成之前(包含commit/rollback)
    default void beforeCompletion() {
    }
	  // 事务提交之后
    default void afterCommit() {
    }
	  // 操作完成之后(包含commit/rollback)
    default void afterCompletion(int status) {
    }
}
```

TransactionSynchronization 是一个接口，它里面定义了一系列与事务各生命周期阶段相关的方法。比如，我们可以这样使用： 

```java
public class UserService {

    @Transactional(rollbackFor = Exception.class)
    public void saveUser(User user) {
        TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronization() {
            @Override
            public void afterCommit() {
                System.out.println("saveUser事务已提交...");
            }
        });
        userDao.saveUser(user);
    }
}
```

在 Spring 事务刚开始的时候，我们向 TransactionSynchronizationManager 事务同步管理器注册了一个事务同步器，事务提交前/后，会遍历执行事务同步器中对应的事务同步方法（一个 Spring 事务可以注册多个事务同步器）。

需要注意的是注册事务同步器必须得在一个 Spring 事务中才能注册，否则会抛出 **Transaction synchronization is not active** 这个错误。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNWlTdUWx3D2HR1r9d1zIrK9UbI49tE8gY779mYRj7icJyXiatddU1HjXrb85Zedo1ibVcKNQrQxEiaBA/640)

`isSynchronizationActive()` 方法用来判断当前是否存在事务（判断线程共享变量，是否存在 TransactionSynchronization）

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNWlTdUWx3D2HR1r9d1zIrKHlqfichM1zaxpmWkmyia25SZD24n4cLnAiaTvTC6n4EKyr91aUQLNqLgQ/640)

Spring 在创建事务的时候，会初始化一个空集合放到 synchronizations 属性中，所以只要当前存在事务，`isSynchronizationActive()`  就为 true。

## TransactionSynchronizationManager 解析

Spring 对于事务的管理都是基于 `TransactionSynchronizationManager` 这个类，先看下 TransactionSynchronizationManager 的一些属性：

```java
    private static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal("Transactional resources");
    private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations = new NamedThreadLocal("Transaction synchronizations");
    private static final ThreadLocal<String> currentTransactionName = new NamedThreadLocal("Current transaction name");
    private static final ThreadLocal<Boolean> currentTransactionReadOnly = new NamedThreadLocal("Current transaction read-only status");
    private static final ThreadLocal<Integer> currentTransactionIsolationLevel = new NamedThreadLocal("Current transaction isolation level");
    private static final ThreadLocal<Boolean> actualTransactionActive = new NamedThreadLocal("Actual transaction active");
```

- resources：保存连接资源，因为一个方法里面可能包含多个事务，所以就用 Map 来保存资源， key为 DataSource，value 为connectionHolder。线程可以通过该属性获取到同一个 Connection 对象。
- synchronizations：事务同步器，是 Spring 交由程序员进行扩展的代码，每个线程可以注册N个事务同步器。
- currentTransactionName：事务的名称。
- currentTransactionReadOnly：事务是否是只读。
- currentTransactionIsolationLevel：事务的隔离级别。
- actualTransactionActive：用于保存当前事务是否还是 Active 状态（事务是否开启）。

Spring 创建事务时，DataSourceTransactionManager.doBegin 方法中，将新创建的 connection 包装成 connectionHolder ，通过 TransactionSynchronizationManager#bindResource 方法存入 resources 中。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNWlTdUWx3D2HR1r9d1zIrKvbHKZDuU5UhotKmVIU0ONUPZTr11jJFODIgeofM7ZLc64IhBYibtPoA/640)

然后标注到一个事务当中的其它数据库操作就可以通过 TransactionSynchronizationManager#getResource 方法获取到这个连接。

```java
    @Nullable
    public static Object getResource(Object key) {
        Object actualKey = TransactionSynchronizationUtils.unwrapResourceIfNecessary(key);
        Object value = doGetResource(actualKey);
        if (value != null && logger.isTraceEnabled()) {
            logger.trace("Retrieved value [" + value + "] for key [" + actualKey + "] bound to thread [" + Thread.currentThread().getName() + "]");
        }

        return value;
    }

    @Nullable
    private static Object doGetResource(Object actualKey) {
        Map<Object, Object> map = (Map)resources.get();
        if (map == null) {
            return null;
        } else {
            Object value = map.get(actualKey);
            if (value instanceof ResourceHolder && ((ResourceHolder)value).isVoid()) {
                map.remove(actualKey);
                if (map.isEmpty()) {
                    resources.remove();
                }

                value = null;
            }

            return value;
        }
    }
```

从上面我们也能看到，Spring 对于多个数据库操作的事务实现是基于 ThreadLocal 的，所以 Spring 事务操作是无法使用多线程的。

## 应用场景

`TransactionSynchronization`  可以用于一些需要在事务结束后执行清理操作或其他相关任务的场景。

应用场景举例：

- **资源释放**：在事务提交或回滚后释放资源，如关闭数据库连接、释放文件资源等。
- **日志记录**：在事务结束后记录相关日志信息，例如记录事务的执行结果或异常情况。
- **缓存更新**：在事务完成后更新缓存数据，保持缓存和数据库数据的一致性。
- **消息通知**：在事务结束后发送消息通知相关系统或用户，如发送邮件或短信通知。

举例： 假设一个电商系统中存在订单支付的业务场景，当用户支付订单时，需要在事务提交后发送订单支付成功的消息通知给用户。

由于事务是和数据库连接相绑定的，如果把发送消息和数据库操作放在一个事务里面。当发送消息时间过长时会占用数据库连接，所以就要把数据库操作与发送消息到 MQ 解耦开来。

这时就可以通过 `TransactionSynchronization` 来实现在事务提交后发送消息通知的功能。具体示例代码如下：

```java
@Component
public class OrderPaymentNotification implements TransactionSynchronization {

    private String orderNo;

    public OrderPaymentNotification(String orderNo) {
        this.orderNo = orderNo;
    }

    @Override
    public void beforeCommit(boolean readOnly) {
        // 在事务提交前不执行任何操作
    }

    @Override
    public void beforeCompletion() {
        // 在事务即将完成时不执行任何操作
    }

    @Override
    public void afterCommit() {
        // 在事务提交后发送订单支付成功的消息通知
        sendMessage("订单支付成功", orderNo);
    }

    @Override
    public void afterCompletion(int status) {
        // 在事务完成后不执行任何操作
    }

    private void sendMessage(String message, String orderNo) {
        // 发送消息通知的具体实现逻辑
        System.out.println(message + ": " + orderNo);
    }
}
```

```java
    @Transactional
    public void finishOrder(String orderNo) {
        // 修改订单成功
        updateOrderSuccess(orderNo);
        // 发送消息到 MQ
        TransactionSynchronizationManager.registerSynchronization(new OrderPaymentNotification(orderNo));
    }
```
这样当事务成功提交之后，就会把消息发送给 MQ，并且不会占用数据库连接资源。

## @TransactionalEventListener

在 Spring Framework 4.2版本后还可以使用 @TransactionalEventListener 注解处理数据库事务提交成功后的执行操作。

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@EventListener
public @interface TransactionalEventListener {
    TransactionPhase phase() default TransactionPhase.AFTER_COMMIT;

    // 表明若没有事务的时候，对应的event是否需要执行，默认值为false表示，没事务就不执行了。
    boolean fallbackExecution() default false;

    @AliasFor(
        annotation = EventListener.class,
        attribute = "classes"
    )
    Class<?>[] value() default {};

    @AliasFor(
        annotation = EventListener.class,
        attribute = "classes"
    )
    Class<?>[] classes() default {};

    String condition() default "";
}



public enum TransactionPhase {
    // 在事务commit之前执行
    BEFORE_COMMIT,
    // 在事务commit之后执行
    AFTER_COMMIT,
    // 在事务rollback之后执行
    AFTER_ROLLBACK,
    // 在事务完成后执行（包括commit/rollback）
    AFTER_COMPLETION;

    private TransactionPhase() {
    }
}

```

从命名上可以直接看出，它就是个 `EventListener`，效果跟 TransactionSynchronization 一样，但比 TransactionSynchronization 更加优雅。它的使用方式如下：

```Java
@Data
public class Order {

    private Long orderId;
    private String orderNumber;
    private BigDecimal totalAmount;
}

@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Transactional
    public void createOrder(Order order) {
        // 保存订单逻辑
        System.out.println("Creating order: " + order.getOrderNumber());
        
        orderRepository.save(order);
        
        // 发布订单创建事件
        OrderCreatedEvent orderCreatedEvent = new OrderCreatedEvent(order);
        eventPublisher.publishEvent(orderCreatedEvent);
    }
}

@Getter
@Setter
public class OrderCreatedEvent {

    private Order order;

    public OrderCreatedEvent(Order order) {
        this.order = order;
    }
}

@Component
@Slf4j
public class OrderEventListener {

    @Autowired
    private EmailService emailService;

    /*
     * @Async加了就是异步监听,没加就是同步（启动类要开启@EnableAsync注解）
     * 可以使用@Order定义监听者顺序，默认是按代码书写顺序
     * 可以使用SpEL表达式来设置监听器生效的条件
     * 监听器可以看做普通方法,如果监听器抛出异常,在publishEvent里处理即可
     */

    @Async
    @Order(1)
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT, classes = OrderCreatedEvent.class)
    public void onOrderCreatedEvent(OrderCreatedEvent event) {
        // 处理订单创建事件，例如发送邮件通知
        log.info("Received OrderCreatedEvent for order: " + event.getOrder().getOrderNumber());
        emailService.sendOrderConfirmationEmail(event.getOrder());
    }
}
```

都看到这里了，如果觉得有帮助，还请您给我个小小的鼓励，动动手指，帮忙点个赞或收藏！谢谢喽，如果觉得文章有误，欢迎在评论区留言指正，我会第一时间讨论并改正！！！
