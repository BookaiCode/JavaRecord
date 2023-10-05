在编程世界中，「**空指针异常（NullPointerException）**」无疑是我们最常遇到的"罪魁祸首"之一。它像一片隐蔽的地雷，静静地等待着我们不小心地踏入，给我们的代码带来潜在的威胁。这种问题虽然看似微小，但却无法忽视。甚至可能对整个程序的稳定性产生重大影响。

为了应对这个长久以来困扰开发者的问题，Java 8 版本引入了一个强大的工具——Optional 类。

Optional 不仅仅是一个容器，它更是一种编程理念的转变，让我们可以用更优雅的方式处理可能为空的情况。

在本篇博客中，我将向大家介绍 JDK Optional 类及其使用方法，帮助你从根本上杜绝空指针异常，提升代码质量。

## Optional 介绍

Optional 类是一个容器对象，它可以包含或不包含非空值。如果一个对象可能为空，那么我们就可以使用 Optional 类来代替该对象。

Optional 类型的变量可以有两种状态：**存在值和不存在值**。

Optional类有两个重要的方法：**of和ofNullable**：

- of方法用于创建一个非空的Optional对象，如果传入的参数为null，则会抛出**NullPointerException**异常。
- ofNullable方法用于创建一个可以为空的Optional对象。如果传入的参数为空，则返回一个空的Optional对象。当 Optional 对象存在值时，调用 get() 方法可以返回该值，当 Optional 对象不存在值时，调用 get() 方法会抛出 **NoSuchElementException** 异常。

下面是一个使用 Optional 类的例子：

```java
Optional<String> optional = Optional.of("Hello World");
System.out.println(optional.get()); //输出 Hello World
```

在上面的例子中，我们首先使用 of() 方法创建了一个包含字符串 "Hello World" 的 Optional 对象，然后使用 get() 方法获取该对象的值并将其打印出来。

注意，如果我们尝试创建一个 null 值的 Optional 对象，则会抛出 NullPointerException 异常。

在使用 Optional 类时，我们应该尽量避免使用 `isPresent()` 和 `get()` 方法，因为这些方法可能会引起空指针异常。比较推荐使用**Optional.ofNullable()**来创建一个Optional 对象。

## Optional 使用

### 创建 Optional 对象

我们可以使用以下几种方式来创建 Optional 对象：

- `Optional.of(value)`：创建一个包含非空值的 Optional 对象。
- `Optional.empty()`：创建一个不包含任何值的空 Optional 对象。
- `Optional.ofNullable(value)`：创建一个**可能包含 null 值的 Optional 对象**。如果 value 不为 null，则该方法会创建一个包含该值的 Optional 对象；否则，创建一个空 Optional 对象。

下面是一个使用 `Optional.ofNullable()` 方法的例子：

```java
String str = null;
Optional<String> optional = Optional.ofNullable(str);
System.out.println(optional.isPresent()); //输出 false
```

在上面的例子中，我们首先声明了一个空字符串 str，并将其赋值为 null。然后，我们使用 `ofNullable()` 方法创建了一个 Optional 对象。由于 str 是 null，因此创建的 Optional 对象也是空的。最后，我们使用 `isPresent()` 方法检查 Optional 对象是否包含值。

### orElse()与orElseGet()

`orElse()`方法接收一个参数，即为默认值。如果Optional对象中的值不为空，则返回该值，否则返回传入的默认值。具体用法如下：

```java
Optional<String> optional = Optional.ofNullable(null);
String result = optional.orElse("default");
System.out.println(result); // 输出 "default"
```

`orElseGet()`方法与`orElse()`方法类似，也是用于获取默认值的方法。但是，`orElseGet()`方法接收的参数是一个「**Supplier函数式接口**」，用于在需要返回默认值时生成该值。具体用法如下：

```java
Optional<String> optional = Optional.ofNullable(null);
String result = optional.orElseGet(() -> "default");
System.out.println(result); // 输出 "default"
```

#### orElse() 和 orElseGet()的区别

`orElse()` 和 `orElseGet()`特别相似，有必要抽离出来讲下他们之间的区别。

`orElse()` 方法无论 Optional 对象是否为空都会执行，因此它总是会创建一个新的对象。`orElseGet()` 方法只有在 Optional 对象为空时才会执行，因此它可以用来延迟创建新的对象。

用一个例子感受一下：

```java
    @Test
    void test() {
        System.out.println("--------不为null的情况----------");
        //不为 null
        String str1 = "hello";
        String result11 = Optional.ofNullable(str1).orElse(get(str1 + ":orElse()方法被执行了"));
        String result12 = Optional.ofNullable(str1).orElseGet(() -> get(str1 + ":orElseGet()方法被执行了"));
        System.out.println(result11);
        System.out.println(result12);
        System.out.println("--------为null的情况----------");
        //为 null
        String str2 = null;
        String result21 = Optional.ofNullable(str2).orElse(get(str1 + ":orElse()方法被执行了"));
        String result22 = Optional.ofNullable(str2).orElseGet(() -> get(str2 + ":orElseGet()方法被执行了"));
        System.out.println(result21);
        System.out.println(result22);
    }


    public String get(String name) {
        System.out.println(name);
        return name;
    }
```

输出结果如下：

```java
--------不为null的情况----------
hello:orElse()方法被执行了
hello
hello
--------为null的情况----------
hello:orElse()方法被执行了
null:orElseGet()方法被执行了
hello:orElse()方法被执行了
null:orElseGet()方法被执行了
```

因此，一般来说，如果你希望在 Optional 对象为空时才创建新的对象，可以使用 `orElseGet()` 方法。否则，如果你希望总是创建新的对象，无论 Optional 对象是否为空，可以使用 `orElse()` 方法，通常来说`orElseGet()`更佳，个人也是推荐使用`orElseGet()`。

### map()与flatMap() 

`map()` 方法接受一个函数作为参数，该函数将被应用于 Optional 对象中的值。**如果 Optional 对象存在值，则将该值传递给函数进行转换，否则返回一个空 Optional 对象**。

下面是一个使用 map() 方法的例子：

```java
String str = "Hello";
Optional<String> optional = Optional.of(str);
Optional<String> upperCaseOptional = optional.map(String::toUpperCase);
System.out.println(upperCaseOptional.get()); //输出 HELLO
```

在上面的例子中，我们首先使用 `of()` 方法创建了一个包含字符串 "Hello" 的 Optional 对象。然后，我们使用 `map()` 方法将该字符串转换为大写形式，并将结果存储到另一个 Optional 对象 upperCaseOptional 中。最后，我们使用 `get()` 方法获取 upperCaseOptional 对象中的值并打印出来。



`flatMap()` 方法与 `map()` 方法类似，都接受一个函数作为参数。但是，`flatMap()` 方法返回的是一个 Optional 类型的值。如果函数返回的是一个 Optional 对象，则该方法会将其"展开"，否则返回一个空 Optional 对象。

下面是一个使用 flatMap() 方法的例子：

```java
String str = "hello world";
Optional<String> optional = Optional.of(str);
Optional<Character> result = optional.flatMap(s -> {
    if (s.length() > 0)
        return Optional.of(s.charAt(0));
    else
        return Optional.empty();
});
System.out.println(result.get()); //输出 h
```

在上面的例子中，我们首先创建了一个包含字符串 "hello world" 的 Optional 对象。然后，我们使用 `flatMap()` 方法将该字符串转换为第一个字符，并将结果存储到另一个 Optional 对象 result 中。最后，我们使用 `get()` 方法获取 result 对象中的值并打印出来。

### filter() 

filter() 方法接受一个「**谓词（Predicate）**」作为参数，返回一个 Optional 类型的值。如果 Optional 对象存在值且满足谓词的条件，则返回该 Optional 对象，否则返回一个空 Optional 对象。

下面是一个使用 filter() 方法的例子：

```java
Integer number = 10;
Optional<Integer> optional = Optional.of(number);
Optional<Integer> result = optional.filter(n -> n % 2 == 0);
System.out.println(result.isPresent()); //输出 true
```

在上面的例子中，我们首先创建了一个包含整数 10 的 Optional 对象。然后，我们使用 `filter()` 方法过滤出该数字是否为偶数，并将结果存储到另一个 Optional 对象 result 中。最后，我们使用 `isPresent()` 方法检查 result 对象是否存在值。

## 常用方法

我们已经介绍了Optional类的几种常用方法。除此之外，我们这里再逐一列举和解析其他方法。

| 方法名                                                       | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `empty()`                                                    | 返回一个空的`Optional`实例。                                 |
| `of(T value)`                                                | 返回一个包含给定非null值的`Optional`。如果value为null，会抛出NullPointerException。 |
| `ofNullable(T value)`                                        | 返回一个可选的包含给定值的`Optional`，也可能为空。           |
| `get()`                                                      | 如果`Optional`有值则将其返回，否则抛出`NoSuchElementException`。 |
| `isPresent()`                                                | 如果值存在返回true，否则false。                              |
| `isEmpty()`                                                  | 如果值不存在返回true，否则false。(Java 11后新增)             |
| `ifPresent(Consumer<? super T> consumer)`                    | 如果值存在，则使用该值调用指定的消费者，否则不执行任何操作。 |
| `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)` | 如果值存在，则使用该值调用指定的消费者，否则执行指定的无参数的动作。(Java 9后新增) |
| `filter(Predicate<? super T> predicate)`                     | 如果一个值存在，并且这个值给定的predicate对其返回true，返回一个`Optional`描述的值，否则返回一个空的`Optional`。 |
| `map(Function<? super T,? extends U> mapper)`                | 如果有值，应用mapping函数并返回结果。否则返回空的`Optional`。 |
| `flatMap(Function<? super T,Optional<U>> mapper)`            | 如果值存在，返回从Optional中提取的值，否则返回一个空的`Optional`。 |
| `orElse(T other)`                                            | 如果存在该值，返回值， 否则返回`other`。                     |
| `orElseGet(Supplier<? extends T> other)`                     | 如果存在该值，返回值，否则触发`other`并返回。                |
| `orElseThrow()`                                              | 如果存在该值，返回包含的值，否则抛出`NoSuchElementException`。 |
| `orElseThrow(Supplier<? extends X> exceptionSupplier)`       | 如果存在该值，返回包含的值，否则抛出由`Supplier`产生的异常。 |



在这篇文章中，我们深入探讨了Java的Optional类及其在编程实践中的应用。通过使用Optional，我们可以更有效地处理可能存在的空值情况，从而避免运行时的NullPointException。虽然它引入了额外的复杂性，但如果正确使用，它可以提供更清晰、更易于维护的代码。

但是，重要的是要记住，Optional并不是解决所有问题的银弹。像所有工具一样，我们需要了解它的优点和局限性，并确保在适当的场景下使用它。编程始终是一个学习和探索的过程，Optional只是我们工具箱中的一个工具。希望通过本文，你对如何利用Java的Optional类有了更全面的理解。