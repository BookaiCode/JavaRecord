[TOC]

因为之后工作会接触到Spark离线任务相关的内容，所以肯定会接触到Scala，谈到大数据技术栈，Scala语言基本绕不开。像Spark。Flink和Kafka等，源码中都能看到Scala的影子。本文旨在对Scala语法进行扫盲，让初学者能够看懂Scala代码，目的就达到了，至于底层原理不会展开叙述。

先分享Scala官方的网站：https://docs.scala-lang.org/

大部分的学习资料都可以在这找到，语言可以切换为中文，非常友好。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPLktFJAvceDP9EU1YkQdjsgMYsLdO2QfiaJDgmyTxRvox43SR51ggRGN6BiaOhxpKMtRHkcYickiaC6Q/640?wx_fmt=png)

另外我们可以使用Scastie在浏览器上直接运行Scala代码进行调试：https://scastie.scala-lang.org/

## Scala跟Java的区别和联系

Scala语言和Java语言有许多相似之处，但也有一些明显的区别。**Scala语言来源于Java，它以Java的虚拟机（JVM）为运行环境，Scala源码 (.scala)会编译成.class文件。这意味着Scala程序可以与Java程序互操作，并且可以利用JVM的优化和性能**。

在语法上，Scala和Java也有一些区别。例如，在Scala中，一切皆为对象，而在Java中，基本类型、null、静态方法等不是对象。在Scala中，成员变量/属性必须显示初始化，而在Java中可以不初始化。此外，在Scala中，异常处理采用Try-catch {case-case}-finally的方式，而在Java中采用Try-catch-catch-finally的方式。

Scala还有一些特有的概念，例如：**惰性函数、伴生对象、特质、偏函数**等。这些概念都为Scala语言提供了更多的灵活性和表达能力。使得Scala语言非常适合用来开发大数据处理框架。此外，Scala语言的语法糖也非常甜，可以用更少的代码量来实现相同的功能。

## Scala安装

Scala安装很简单。

1. 首先Idea安装 Scala插件。

   ![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPLktFJAvceDP9EU1YkQdjsicYaEhvUYNkAdX79icagGAljRPGEmwtSJHGXjE4VgMk5ica0uayhxzK8A/640?wx_fmt=png)

2. 项目结构里点击全局库，添加 Scala SDK进行下载。

   ![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPLktFJAvceDP9EU1YkQdjsF2icxJlNs9bvIbxvrq7wvsjQAxYp3kQr2hfdUZQFZL8pWibGs7IA7W2w/640?wx_fmt=png)

3. 右键点击添加到你要使用Scala的项目的项目库，项目的库里就会多出Scala的SDK。

   ![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPLktFJAvceDP9EU1YkQdjsCia8djsBUP8pVvJWykpBtBPOSHwicmxluNWySOYxd0KSo3V8oq34jvicw/640?wx_fmt=png)

到这就结束了，然后我们就可以在项目里使用Scala了。

新建一个Scala项目，运行**Hello Wrold**试一下。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScPLktFJAvceDP9EU1YkQdjskHoia03xEOx0BO5X5IBeMeGQ8sRSiaagMABZf394mLkUWdUXSbXicsS9A/640?wx_fmt=png)

## Scala中的数据类型

![img](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNibrCwVLmuEcvHm7rwkFCvgiaWwQTEib93ibwxdsC8Ha727hS6U7osnPDuY5EShDY095mHKRric5SFXAA/640?wx_fmt=png)

Scala中的数据类型可以分为两大类：值类型（`AnyVal`）和引用类型（`AnyRef`）。这两种类型都是 `Any` 类型的子类。

值类型包括9种基本数据类型，分别是 `Byte`、`Short`、`Int`、`Long`、`Float`、`Double`、`Char`、`Boolean` 和 `Unit`。其中，前8种类型与Java中的基本数据类型相对应，而 `Unit` 类型表示无值，类似于Java中的 `void`。

引用类型包括所有非值类型的数据类型，例如字符串、数组、列表等。它们都是 `AnyRef` 类型的子类。

在Scala的数据类型层级结构的底部，还有两个特殊的数据类型： `Nothing` 和 `Null`。其中， `Nothing` 类型是所有类型的子类型，它没有任何实例。而 `Null` 类型是所有引用类型的子类型，它只有一个实例： `null`。

## Scala语法

主方法是一个程序的入口点。JVM要求一个名为`main`的主方法，接受一个字符串数组的参数。你可以如下所示来定义一个主方法。

```scala
object Main {
  def main(args: Array[String]): Unit =
    println("Hello, Scala developer!")
}
```

在Scala 2中，也可以通过创建一个扩展`App`类的对象来定义主程序。例如：

```scala
object Main extends App {
  println("Hello, Scala developer!")
}
```

需要注意的是，这种方法在Scala 3中不再推荐使用。它们被新的`@main`方法取代了，这是在Scala 3中生成可以从命令行调用的程序的推荐方法。`App`目前仍以有限的形式存在，但它不支持命令行参数，将来会被弃用。

### val和var

在 Scala 中，`val` 和 `var` 都可以用来定义变量，但它们之间有一些重要的区别。

`val` 用于定义不可变变量，也就是说，一旦定义了一个 `val` 变量，它的值就不能再被改变。例如：

```scala
val x = 1
// x = 2 // 这会报错，因为不能给 val 变量重新赋值
```

而 `var` 用于定义可变变量，它的值可以在定义后被改变。例如：

```scala
var y = 1
y = 2 // 这是合法的，因为 y 是一个 var 变量
```

val和var的类型可以被推断，或者你也可以显式地声明类型，例如：

```scala
val x: Int = 1 + 1
var x: Int = 1 + 1
```

在实际编程中，我们应该尽量使用 `val` 来定义不可变变量，这样可以提高代码的可读性和可维护性。只有在确实需要改变变量值的情况下，才应该使用 `var` 来定义可变变量。

### 泛型

在Scala 中，使用方括号 `[]` 来定义泛型类型。而在Java中是使用`<>`。

```scala
object Main extends App {
  trait Animal {
    def speak: String
  }

  class Dog extends Animal {
    def speak = "Woof!"
  }

  class Cat extends Animal {
    def speak = "Meow!"
  }

  class Parrot extends Animal {
    def speak = "Squawk!"
  }

  class AnimalShelter[A <: Animal] {
    private var animals: List[A] = Nil

    def addAnimal(animal: A): Unit = {
      animals = animal :: animals
    }

    def getAnimal: A = {
      val animal = animals.head
      animals = animals.tail
      animal
    }
  }

  val dogShelter = new AnimalShelter[Dog]
  dogShelter.addAnimal(new Dog)
  val dog: Dog = dogShelter.getAnimal
  println(dog.speak)

  val catShelter = new AnimalShelter[Cat]
  catShelter.addAnimal(new Cat)
  val cat: Cat = catShelter.getAnimal
  println(cat.speak)

  val parrotShelter = new AnimalShelter[Parrot]
  parrotShelter.addAnimal(new Parrot)
  val parrot: Parrot = parrotShelter.getAnimal
  println(parrot.speak)
}

输出：
Woof!
Meow!
Squawk!
```

这个示例中，我们定义了一个 `Animal` 特质和三个实现了该特质的类：`Dog`，`Cat` 和 `Parrot`。然后我们定义了一个 `AnimalShelter` 类，它使用了泛型类型参数 `A`，并且限制了 `A` 必须是 `Animal` 的子类型。这样我们就可以创建不同类型的动物收容所，比如 `dogShelter`，`catShelter` 和 `parrotShelter`，并且在添加和获取动物时保证类型安全。

### 包导入

`import` 语句用于导入其他包中的成员（类，特质，函数等）。 使用相同包的成员不需要 `import` 语句。 导入语句可以有选择性：

```scala
import users._  // 导入包 users 中的所有成员
import users.User  // 导入类 User
import users.{User, UserPreferences}  // 仅导入选择的成员
import users.{UserPreferences => UPrefs}  // 导入类并且设置别名
```

Scala 不同于 Java 的一点是 Scala 可以在任何地方使用导入：

```scala
def sqrtplus1(x: Int) = {
  import scala.math.sqrt
  sqrt(x) + 1.0
}
```

如果存在命名冲突并且你需要从项目的根目录导入，请在包名称前加上 `_root_`：

```scala
package accounts

import _root_.users._
```

注意：包 `scala` 和 `java.lang` 以及 `object Predef` 是默认导入的。

### 包对象

在 Scala 中，包对象（Package Object）是一种特殊的对象，它与包同名，并且可以在包中定义一些公共的成员和方法，供包中的其他类和对象直接使用。包对象可以解决在包级别共享常量、类型别名、隐式转换等问题。在 Scala 中，可以使用 `package` 关键字定义一个包对象。包对象的文件名必须为 `package.scala`，并与包名一致。

下面是关于包对象的解释和示例代码：

```scala
// File: com/example/myapp/package.scala

package com.example

package object myapp {
  val appName: String = "MyApp"

  def printAppName(): Unit = {
    println(appName)
  }
}
```

在上述示例中，定义了一个包对象 `myapp`，位于包 `com.example` 下。在包对象中，我们定义了一个名为 `appName` 的常量和一个名为 `printAppName` 的方法。

这样，我们就可以在包中的其他类和对象中直接使用 `appName` 和 `printAppName`，而无需导入或限定符。

下面是一个使用包对象的示例代码：

```scala
package com.example.myapp

object Main {
  def main(args: Array[String]): Unit = {
    println(myapp.appName)  // 直接访问包对象中的常量
    myapp.printAppName()    // 直接调用包对象中的方法
  }
}
```

在上述示例中，我们在 `Main` 对象中直接访问了包对象 `myapp` 中的常量 `appName` 和方法 `printAppName`。由于包对象与包同名且位于同一包中，因此可以直接使用它们。

### 特质

在Scala中，类是单继承的，但是特质（trait）可以多继承。这意味着，一个类只能继承一个父类，但可以继承多个特质。这样，从结果上看，就实现了多重继承。

下面是一个例子：

```scala
trait A {
  def printA() = println("A")
}

trait B {
  def printB() = println("B")
}

class C extends A with B

object Main extends App {
  val c = new C
  c.printA()
  c.printB()
}

输出：
A
B
```

例子中，定义了两个特质 `A` 和 `B`，它们分别有一个方法 `printA` 和 `printB`。然后我们定义了一个类 `C`，它继承了特质 `A` 和 `B`。这样，类 `C` 就可以使用特质 `A` 和 `B` 中定义的方法了。

特质也可以有默认的实现。

```scala
trait Greeter {
  def greet(name: String): Unit =
    println("Hello, " + name + "!")
}
```

你可以使用`extends`关键字来继承特质，使用`override`关键字来覆盖默认的实现。

```scala
class DefaultGreeter extends Greeter

class CustomizableGreeter(prefix: String, postfix: String) extends Greeter {
  override def greet(name: String): Unit = {
    println(prefix + name + postfix)
  }
}

val greeter = new DefaultGreeter()
greeter.greet("Scala developer") // Hello, Scala developer!

val customGreeter = new CustomizableGreeter("How are you, ", "?")
customGreeter.greet("Scala developer") // How are you, Scala developer?
```

凡是需要特质的地方，都可以由该特质的子类型来替换。

```scala
import scala.collection.mutable.ArrayBuffer

trait Pet {
  val name: String
}

class Cat(val name: String) extends Pet
class Dog(val name: String) extends Pet

val dog = new Dog("Harry")
val cat = new Cat("Sally")

val animals = ArrayBuffer.empty[Pet]
animals.append(dog)
animals.append(cat)
animals.foreach(pet => println(pet.name))  // Prints Harry Sally
```

在这里 `trait Pet` 有一个抽象字段 `name` ，`name` 由Cat和Dog的构造函数中实现。最后一行，我们能调用`pet.name`的前提是它必须在特质Pet的子类型中得到了实现。

### 运算符

在 Scala 中，运算符是用于执行特定操作的符号或标记。Scala 具有丰富的运算符，并且允许用户自定义运算符，以及在自定义类中使用运算符。下面是关于定义和使用运算符的解释和示例代码：

 在 Scala 中，可以使用 `def` 关键字定义自定义运算符。自定义运算符可以是任何由字母、数字或下划线组成的标识符，以及一些特殊字符，例如 `+`、`-`、`*` 等。要定义一个运算符，可以在方法名前面加上一个操作符，然后在方法体中实现相应的逻辑。

下面是一个示例代码：

```scala
class Vector2D(val x: Double, val y: Double) {
  def +(other: Vector2D): Vector2D = {
    new Vector2D(x + other.x, y + other.y)
  }
}

val v1 = new Vector2D(1.0, 2.0)
val v2 = new Vector2D(3.0, 4.0)
val sum = v1 + v2

println(sum.x) // 输出：4.0
println(sum.y) // 输出：6.0
```

在上述示例中，定义了一个 `Vector2D` 类，表示二维向量。我们通过 `val` 关键字定义了 `x` 和 `y` 作为向量的坐标。

然后，我们定义了一个自定义运算符 `+`，它接受另一个 `Vector2D` 对象作为参数，并返回一个新的 `Vector2D` 对象。在方法体内，我们实现了向量的加法操作。

在主程序中，我们创建了两个 `Vector2D` 对象 `v1` 和 `v2`。然后，我们使用自定义的运算符 `+` 来执行向量的加法，并将结果赋值给 `sum`。

最后，我们打印出 `sum` 的 `x` 和 `y` 坐标，验证加法操作的结果。

我们可以像使用内置运算符一样使用自定义运算符。它们可以用于相应类型的实例上，并按照定义的逻辑执行操作。

下面是一个示例代码：

```scala
val num1 = 10
val num2 = 5

val sum = num1 + num2
val difference = num1 - num2
val product = num1 * num2

println(sum)        // 输出：15
println(difference)  // 输出：5
println(product)     // 输出：50
```

在上述示例中，我们定义了两个整数变量 `num1` 和 `num2`。然后，我们使用内置的运算符 `+`、`-` 和 `*` 来执行加法、减法和乘法操作，并将结果分别赋值给 `sum`、`difference` 和 `product`。

### 传名参数

传名参数（Call-by-Name Parameters）是一种特殊的参数传递方式，它允许我们将表达式作为参数传递给函数，并在需要时进行求值。传名参数使用 `=>` 符号来定义，以表示传递的是一个表达式而不是具体的值。下面是关于传名参数的解释和示例代码：

传名参数的特点是，在每次使用参数时都会重新求值表达式，而不是在调用函数时进行求值。这样可以延迟表达式的求值，只在需要时才进行计算。传名参数通常用于需要延迟计算、惰性求值或者需要按需执行的场景。

下面是一个示例代码：

```scala
def callByName(param: => Int): Unit = {
  println("Inside callByName")
  println("Param 1: " + param)
  println("Param 2: " + param)
}

def randomNumber(): Int = {
  println("Generating random number")
  scala.util.Random.nextInt(100)
}

callByName(randomNumber())
```

在上述示例中，定义了一个名为 `callByName` 的函数，它接受一个传名参数 `param`。在函数体内，我们打印出两次参数的值。

另外，定义了一个名为 `randomNumber` 的函数，它用于生成随机数。在该函数内部，我们打印出生成随机数的消息，并使用 `scala.util.Random.nextInt` 方法生成一个介于 0 到 100 之间的随机数。

在主程序中，我们调用 `callByName` 函数，并将 `randomNumber()` 作为传名参数传递进去。

当程序执行时，会先打印出 "Inside callByName" 的消息，然后两次调用 `param`，即 `randomNumber()`。在每次调用时，都会重新生成一个新的随机数，并打印出相应的值。

这说明传名参数在每次使用时都会重新求值表达式，而不是在调用函数时进行求值。这样可以实现按需执行和延迟计算的效果。

### implicit

`implicit` 关键字用于定义隐式转换和隐式参数。它可以用来简化代码，让编译器自动执行一些操作。

下面是一些使用 `implicit` 关键字的示例：

- 隐式转换：可以使用 `implicit` 关键字定义隐式转换函数，让编译器自动将一种类型的值转换为另一种类型的值。

```scala
implicit def intToString(x: Int): String = x.toString

val x: String = 1
println(x) // 输出 "1"
```

在这个例子中，定义了一个隐式转换函数 `intToString`，它接受一个 `Int` 类型的参数，并返回它的字符串表示。由于这个函数被定义为 `implicit`，因此编译器会在需要时自动调用它。

在主程序中，我们将一个 `Int` 类型的值赋值给一个 `String` 类型的变量。由于类型不匹配，编译器会尝试寻找一个隐式转换函数来将 `Int` 类型的值转换为 `String` 类型的值。在这个例子中，编译器找到了我们定义的 `intToString` 函数，并自动调用它将 `1` 转换为 `"1"`。

- 隐式参数：可以使用 `implicit` 关键字定义隐式参数，让编译器自动为方法提供参数值。

```scala
implicit val x: Int = 1

def foo(implicit x: Int): Unit = println(x)

foo // 输出 1
```

在这个例子中，定义了一个隐式值 `x` 并赋值为 `1`。然后我们定义了一个方法 `foo`，它接受一个隐式参数 `x`。

在主程序中，我们调用了方法 `foo`，但没有显式地传入参数。由于方法 `foo` 接受一个隐式参数，因此编译器会尝试寻找一个隐式值来作为参数传入。在这个例子中，编译器找到了我们定义的隐式值 `x` 并将其作为参数传入方法 `foo`。

### Object和Class

在Scala中，`class` 和 `object` 都可以用来定义类型，但它们之间有一些重要的区别。`class` 定义了一个类，它可以被实例化。每次使用 `new` 关键字创建一个类的实例时，都会创建一个新的对象。

```scala
class MyClass(x: Int) {
  def printX(): Unit = println(x)
}

val a = new MyClass(1)
val b = new MyClass(2)
a.printX() // 输出 1
b.printX() // 输出 2
```

构造器可以通过提供一个默认值来拥有可选参数：

```scala
class Point(var x: Int = 0, var y: Int = 0)

val origin = new Point  // x and y are both set to 0
val point1 = new Point(1)
println(point1.x)  // prints 1
```

在这个版本的`Point`类中，`x`和`y`拥有默认值`0`所以没有必传参数。然而，因为构造器是从左往右读取参数，所以如果仅仅要传个`y`的值，你需要带名传参。

```scala
class Point(var x: Int = 0, var y: Int = 0)
val point2 = new Point(y=2)
println(point2.y)  // prints 2
```

而 `object` 定义了一个单例对象。它不能被实例化，也不需要使用 `new` 关键字创建。在程序中，一个 `object` 只有一个实例。此外，`object` 中定义的成员都是静态的，这意味着它们可以在不创建实例的情况下直接访问。而 `class` 中定义的成员只能在创建实例后访问。

```scala
object MyObject {
  val x = 1
  def printX(): Unit = println(x)
}

MyObject.printX() // 输出 1
```

另外，在Scala中，如果一个 `object` 的名称与一个 `class` 的名称相同，那么这个 `object` 被称为这个 `class` 的伴生对象。伴生对象和类可以相互访问彼此的私有成员：

```scala
class MyClass(x: Int) {
  private val secret = 42
  def printCompanionSecret(): Unit = println(MyClass.companionSecret)
}

object MyClass {
  private val companionSecret = 24
  def printSecret(c: MyClass): Unit = println(c.secret)
}

val a = new MyClass(1)
a.printCompanionSecret() // 输出 24
MyClass.printSecret(a) // 输出 42
```

在这个例子中，定义了一个类 `MyClass` 和它的伴生对象 `MyClass`。类 `MyClass` 中定义了一个私有成员变量 `secret` 和一个方法 `printCompanionSecret`，用于打印伴生对象中的私有成员变量 `companionSecret`。而伴生对象 `MyClass` 中定义了一个私有成员变量 `companionSecret` 和一个方法 `printSecret`，用于打印类 `MyClass` 的实例中的私有成员变量 `secret`。

在主程序中，创建了一个类 `MyClass` 的实例 `a`，并调用了它的 `printCompanionSecret` 方法。然后我们调用了伴生对象 `MyClass` 的 `printSecret` 方法，并将实例 `a` 作为参数传入。

这就是Scala中类和伴生对象之间互相访问私有成员的基本用法。

### 样例类

样例类（case class）是一种特殊的类，**常用于描述不可变的值对象**（Value Object）。它们非常适合用于不可变的数据。定义一个样例类非常简单，只需在类定义前加上`case`关键字即可。例如，下面是一个简单的样例类定义：

```scala
case class Person(var name: String, var age: Int)
```

创建样例类的实例时，不需要使用`new`关键字，直接使用类名即可。例如，下面是一个创建样例类实例并修改其成员变量的示例：

```scala
object Test01 {
  case class Person(var name: String, var age: Int)

  def main(args: Array[String]): Unit = {
    val z = Person("张三", 20)
    z.age = 23
    println(s"z = $z")
  }
}
```

### _(下划线)

在Scala中，下划线 `_` 是一个特殊的符号，它可以用在许多不同的地方，具有不同的含义。

- 作为通配符：下划线可以用作通配符，表示匹配任意值。例如，在模式匹配中，可以使用下划线来表示匹配任意值。

```scala
x match {
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
}
```

- 作为忽略符：下划线也可以用来忽略不需要的值。例如，在解构赋值时，可以使用下划线来忽略不需要的值。

```scala
val (x, _, z) = (1, 2, 3)
```

- 作为函数参数占位符：下划线还可以用作函数参数的占位符，表示一个匿名函数的参数。例如，在调用高阶函数时，可以使用下划线来简化匿名函数的定义。

```scala
val list = List(1, 2, 3)
list.map(_ * 2)
```

- 将方法转换为函数：在方法名称后加一个下划线，会将其转化为偏应用函数（partially applied function），就能直接赋值了。

```scala
def add(x: Int, y: Int) = x + y
val f = add _
```

这只是下划线在Scala中的一些常见用法。由于下划线在不同的上下文中具有不同的含义，因此在使用时需要根据具体情况进行判断。

### println

`println` 函数用于向标准输出打印一行文本。它可以接受多种不同类型的参数，并将它们转换为字符串进行输出。

下面是一些常见的使用 `println` 函数进行输出的方式：

- 输出字符串：直接将字符串作为参数传入 `println` 函数，它会将字符串原样输出。

```scala
println("Hello, world!")
```

- 输出变量：将变量作为参数传入 `println` 函数，它会将变量的值转换为字符串并输出。

```scala
val x = 1
println(x)
```

- 输出表达式：将表达式作为参数传入 `println` 函数，它会计算表达式的值并将其转换为字符串输出。

```scala
val x = 1
val y = 2
println(x + y)
```

- 使用字符串插值：可以使用字符串插值来格式化输出。在字符串前加上 `s` 前缀，然后在字符串中使用 `${expression}` 的形式来插入表达式的值。

```scala
val name = "Alice"
val age = 18
println(s"My name is $name and I am $age years old.")
```

这些是 `println` 函数的一些常见用法。你可以根据需要使用不同的方式来格式化输出。

### 集合

在Scala中，集合有三大类：**序列Seq、集Set、映射Map**，所有的集合都扩展自Iterable，所以Scala中的集合都可以使用 foreach方法。在Scala中集合有可变（mutable）和不可变（immutable）两种类型。

#### List

如我们可以使用如下方式定义一个List，其他集合类型的定义方式也差不多。

```scala
object Main {
  def main(args: Array[String]): Unit = {
    // 定义一个空的字符串列表
    var emptyList: List[String] = List()
    // 定义一个具有数据的列表
    var intList = List(1, 2, 3, 4, 5, 6)
    // 定义空列表
    var emptyList2 = Nil
    // 使用::运算符连接元素
    var numList = 1 :: (2 :: (3 :: Nil))
    println(emptyList)
    println(intList)
    println(emptyList2)
    println(numList)
  }
}

输出：
List()
List(1, 2, 3, 4, 5, 6)
List()
List(1, 2, 3)
```

下面是一些List的常用方法：

```scala
val list = List(1, 2, 3, 4)

// 获取列表的长度
val length = list.length

// 获取列表的第一个元素
val first = list.head

// 获取列表的最后一个元素
val last = list.last

// 获取列表除第一个元素外剩余的元素
val tail = list.tail

// 获取列表除最后一个元素外剩余的元素
val init = list.init

// 反转列表
val reversed = list.reverse

// 在列表头部添加元素
val newList1 = 0 +: list

// 在列表尾部添加元素
val newList2 = list :+ 5

// 连接两个列表
val list1 = List(1, 2)
val list2 = List(3, 4)
val concatenatedList = list1 ++ list2

// 检查列表是否为空
val isEmpty = list.isEmpty

// 检查列表是否包含某个元素
val containsElement = list.contains(1)

// 过滤列表中的元素
val filteredList = list.filter(_ > 2)

// 映射列表中的元素
val mappedList = list.map(_ * 2)

// 折叠列表中的元素（从左到右）
val sum1 = list.foldLeft(0)(_ + _)

// 折叠列表中的元素（从右到左）
val sum2 = list.foldRight(0)(_ + _)

// 拉链操作
val names = List("Alice", "Bob", "Charlie")
val ages = List(25, 32, 29)
val zipped = names.zip(ages) // List(("Alice", 25), ("Bob", 32), ("Charlie", 29))

// 拉链操作后解压缩
val (unzippedNames, unzippedAges) = zipped.unzip // (List("Alice", "Bob", "Charlie"), List(25, 32, 29))
```

更多方法不再赘述，网上很容易查阅到相关文章。

#### Map

```scala
object Main {
  def main(args: Array[String]): Unit = {
    // 定义一个空的映射
    val emptyMap = Map()
    // 定义一个具有数据的映射
    val intMap = Map("key1" -> 1, "key2" -> 2)
    // 使用元组定义一个映射
    val tupleMap = Map(("key1", 1), ("key2", 2))
    println(emptyMap)
    println(intMap)
    println(tupleMap)
  }
}

输出：
Map()
Map(key1 -> 1, key2 -> 2)
Map(key1 -> 1, key2 -> 2)
```

下面是map常用的一些方法：

```scala
val map = Map("key1" -> 1, "key2" -> 2)

// 获取映射的大小
val size = map.size

// 获取映射中的所有键
val keys = map.keys

// 获取映射中的所有值
val values = map.values

// 检查映射是否为空
val isEmpty = map.isEmpty

// 检查映射是否包含某个键
val containsKey = map.contains("key1")

// 获取映射中某个键对应的值
val value = map("key1")

// 获取映射中某个键对应的值，如果不存在则返回默认值
val valueOrDefault = map.getOrElse("key3", 0)

// 过滤映射中的元素
val filteredMap = map.filter { case (k, v) => v > 1 }

// 映射映射中的元素
val mappedMap = map.map { case (k, v) => (k, v * 2) }

// 遍历映射中的元素
map.foreach { case (k, v) => println(s"key: $k, value: $v") }
```

这里的`case`关键字起到匹配的作用。

#### Range

`Range`属于序列（`Seq`）这一类集合的子集。它表示一个整数序列，可以用来遍历一个整数区间内的所有整数。例如，`1 to 5`表示一个从1到5的整数序列，包括1和5。

Range常见于for循环中，如下可定义一个Range：

```scala
// 定义一个从1到5的整数序列，包括1和5
val range1 = 1 to 5

// 定义一个从1到5的整数序列，包括1但不包括5
val range2 = 1 until 5

// 定义一个从1到10的整数序列，步长为2
val range3 = 1 to 10 by 2

// 定义一个从10到1的整数序列，步长为-1
val range4 = 10 to 1 by -1
```

如果我们想把Range转为List，我们可以这样做：

```scala
val range = 1 to 5
val list = range.toList
```

`Range`继承自`Seq`，因此它拥有`Seq`的所有常用方法，例如`length`、`head`、`last`、`tail`、`init`、`reverse`、`isEmpty`、`contains`、`filter`、`map`、`foldLeft`和`foldRight`等。它还拥有一些特殊的方法，例如：

```scala
val range = 1 to 10 by 2

// 获取序列的起始值
val start = range.start

// 获取序列的结束值
val end = range.end

// 获取序列的步长
val step = range.step

// 获取一个包括结束值的新序列
val inclusiveRange = range.inclusive
```

### 迭代器

迭代器（Iterator）是一种用于遍历集合中元素的工具。它提供了一种方法来访问集合中的元素，而不需要暴露集合的内部结构。在 Scala 中，你可以使用 `iterator` 方法来获取一个集合的迭代器。

```scala
object Main {
  def main(args: Array[String]): Unit = {

    val list = List(1, 2, 3)
    val iterator = list.iterator

    // 1. 使用 hasNext 方法来检查迭代器中是否还有元素
    val hasMoreElements = iterator.hasNext
    println(s"Has more elements: $hasMoreElements")

    // 2. 使用 next 方法来获取迭代器中的下一个元素
    val nextElement = iterator.next()
    println(s"Next element: $nextElement")

    // 注意：上面的代码已经将迭代器移动到了第二个元素，因此下面的代码将从第二个元素开始执行

    // 3. 使用 size 方法来获取迭代器中元素的个数
    val size = iterator.size
    println(s"Size: $size")

    val size1 = iterator.size
    println(s"Size1: $size1")

    // 注意：上面的代码已经将迭代器移动到了末尾，因此下面的代码将不再有效

    // 4. 使用 contains 方法来检查迭代器中是否包含某个元素
    val containsElement = iterator.contains(2)
    println(s"Contains element: $containsElement")
  }
}

输出：
Has more elements: true
Next element: 1
Size: 2
Size1: 0
Contains element: false
```

特别注意：**迭代器是一次性的，所以在使用完毕后就不能再次使用。因此，在上面的代码中，我们在调用 `next` 方法后就不能再使用其他方法来访问迭代器中的元素了。所以 size1输出为0**。

### Tuple

把`Tuple`从集合中抽出来讲述是因为`Tuple`不属于集合。它是一种用来将多个值组合在一起的数据结构。一个`Tuple`可以包含不同类型的元素，每个元素都有一个固定的位置。Scala 中的元组包含一系列类：Tuple2，Tuple3等，直到 Tuple22。

示例如下：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    // 定义一个包含两个元素的Tuple
    val tuple1 = (1, "hello")
    println(tuple1)

    // 定义一个包含三个元素的Tuple
    val tuple2 = (1, "hello", true)
    println(tuple2)

    // 定义一个包含多个不同类型元素的Tuple
    val tuple3 = (1, "hello", true, 3.14)
    println(tuple3)

    // 访问Tuple中的元素
    val firstElement = tuple3._1
    val secondElement = tuple3._2
    println(s"first element: $firstElement, second element: $secondElement")
  }
}

输出：
(1,hello)
(1,hello,true)
(1,hello,true,3.14)
first element: 1, second element: hello
```

下面是一些Tuple的常用方法：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    val tuple = (1, "hello")
    // 交换二元组的元素
    // 输出：(hello,1)
    val swapped = tuple.swap

    // 使用 copy 方法来创建一个新的 Tuple，其中某些元素被替换为新值
    //输出：(1,world)
    val newTuple = tuple.copy(_2 = "world")

    // 遍历元素
    // 输出： 1 hello
    tuple.productIterator.foreach(println)

    // 转换为字符串
    // 输出： (1,hello)
    val stringRepresentation = tuple.toString

    // 使用 Tuple.productArity 方法来获取 Tuple 中元素的个数
    // 输出： 2
    val arity = tuple.productArity

    // 使用 Tuple.productElement 方法来访问 Tuple 中的元素
    // 输出： 1
    val firstElement = tuple.productElement(0)
  }
}
```

### 提取器对象

提取器对象是一个包含有 `unapply` 方法的单例对象。`apply` 方法就像一个构造器，接受参数然后创建一个实例对象，反之 `unapply` 方法接受一个实例对象然后返回最初创建它所用的参数。提取器常用在模式匹配和偏函数中。

下面是一个使用提取器对象（Extractor Object）的 Scala 代码示例：

```scala
object Email {
  def apply(user: String, domain: String): String = s"$user@$domain"
  
  def unapply(email: String): Option[(String, String)] = {
    val parts = email.split("@")
    if (parts.length == 2) Some(parts(0), parts(1))
    else None
  }
}

// 测试
val address = "john.doe@example.com"
address match {
  case Email(user, domain) => println(s"User: $user, Domain: $domain")
  case _ => println("Invalid email address")
}
```

在上述示例中，定义了一个名为`Email`的提取器对象。提取器对象具有两个方法：`apply`和`unapply`。

`apply`方法接收用户名和域名作为参数，并返回一个完整的电子邮件地址。在这个示例中，我们简单地将用户名和域名拼接成电子邮件地址的字符串。

`unapply`方法接收一个电子邮件地址作为参数，并返回一个`Option`类型的元组。在这个示例中，我们使用`split`方法将电子邮件地址分割为用户名和域名两部分，并通过`Some`将它们封装到一个`Option`中返回。如果分割后的部分不是两部分，即电子邮件地址不符合预期的格式，我们返回`None`。

在测试部分，我们创建了一个电子邮件地址字符串`address`。然后，我们使用`match`表达式将`address`与提取器对象`Email`进行匹配。如果匹配成功，我们提取出用户名和域名，并打印出对应的信息。如果匹配失败，即电子邮件地址无效，我们打印出相应的错误信息。

### 流程判断

#### while和if

```scala
object Main {
  def main(args: Array[String]): Unit = {
    println("----while----")
    var i = 0
    while (i < 5) {
      println(i)
      i += 1
    }

    println("----if----")
    val x = 3
    if (x > 0) {
      println("x大于0")
    } else {
      println("x小于0")
    }
  }
}

输出：
----while----
0
1
2
3
4
----if----
x大于0
```

Scala中的while和if跟Java中的方法几乎没有区别。

#### for

```scala
object Main {
  def main(args: Array[String]): Unit = {
  println("----for循环----")
    for (i <- 1 to 5) {
      println(i)
    }
  }
}

输出：
----for循环----
1
2
3
4
5
```

for循环跟Java略微有点区别。其中`i <- 1 to 5`是Scala中for循环的一种常见形式。它表示遍历一个序列，序列中的元素依次为1、2、3、4、5。

##### 多重for循环简写

Scala中对于多重for循环可以进行简写，例如我们要用Java写多重for循环是下面这样：

```java
public class Main {
  public static void main(String[] args) {
    // 多重for循环
    for (int i = 0; i < 3; i++) {
      for (int j = 0; j < 3; j++) {
        System.out.println(i + " " + j);
      }
    }
  }
}
```

而用Scala我们可以直接简写为下面这样：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    // 多重for循环
    for (i <- 0 until 3; j <- 0 until 3) {
      println(i + " " + j)
    }
  }
}

输出：
0 0
0 1
0 2
1 0
1 1
1 2
2 0
2 1
2 2
```

可以看出scala的for循环语法更加的精简。代码行数更少。

##### yield

在for循环的过程中我们可以使用 yield来对for循环的元素进行 操作收集：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    val numbers = for (i <- 1 to 5) yield i * 2
    println(numbers)
  }
}

输出：
Vector(2, 4, 6, 8, 10)
```

#### 模式匹配（pattern matching）

在Scala语言中，没有`switch`和`case`关键字。相反，我们可以使用模式匹配（pattern matching）来实现类似于`switch`语句的功能。它是Java中的`switch`语句的升级版，同样可以用于替代一系列的 if/else 语句。下面是一个简单的例子，它展示了如何使用模式匹配来实现类似于`switch`语句的功能：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    def matchTest(x: Any): String = x match {
      case 1 => "one"
      case "two" => "two"
      case y: Int => "scala.Int"
      case _ => "many"
    }

    println(matchTest(1))
    println(matchTest("two"))
    println(matchTest(3))
    println(matchTest("test"))
  }
}

输出：
one
two
scala.Int
many
```

在上面的例子中，定义了一个名为`matchTest`的函数，它接受一个类型为`Any`的参数`x`。在函数体中，我们使用了一个模式匹配表达式来匹配参数`x`的值。

在模式匹配表达式中，我们定义了四个`case`子句。第一个`case`子句匹配值为1的情况；第二个`case`子句匹配值为"two"的情况；第三个`case`子句匹配类型为`Int`的情况；最后一个`case`子句匹配所有其他情况。

##### 样例类（case classes）的匹配

样例类非常适合用于模式匹配。

```scala
abstract class Notification

case class Email(sender: String, title: String, body: String) extends Notification

case class SMS(caller: String, message: String) extends Notification

def showNotification(notification: Notification): String = {
  notification match {
    case Email(sender, title, _) =>
      s"You got an email from $sender with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
  }
}

val someSms = SMS("12345", "Are you there?")
val someEmail = Email("John Doe", "Meeting", "Are we still meeting tomorrow?")

println(showNotification(someSms))
println(showNotification(someEmail))

```

这段代码定义了一个抽象类 `Notification`，以及两个扩展自 `Notification` 的样例类 `Email` 和 `SMS`。然后定义了一个函数 `showNotification`，它接受一个 `Notification` 类型的参数，并使用模式匹配来检查传入的通知是 `Email` 还是 `SMS`，并相应地生成一条消息。

最后，我们创建了两个实例：一个 `SMS` 和一个 `Email`，并使用 `showNotification` 函数来显示它们的消息。

##### 模式守卫（Pattern guards）

为了让匹配更加具体，可以使用模式守卫，也就是在模式后面加上`if <boolean expression>`。

```scala
def checkNumberType(number: Int): String = number match {
  case n if n > 0 && n % 2 == 0 => "Positive even number"
  case n if n > 0 && n % 2 != 0 => "Positive odd number"
  case n if n < 0 && n % 2 == 0 => "Negative even number"
  case n if n < 0 && n % 2 != 0 => "Negative odd number"
  case _ => "Zero"
}

// 测试
println(checkNumberType(10))    // 输出: Positive even number
println(checkNumberType(15))    // 输出: Positive odd number
println(checkNumberType(-4))    // 输出: Negative even number
println(checkNumberType(-9))    // 输出: Negative odd number
println(checkNumberType(0))     // 输出: Zero
```

在上述示例中，我们定义了一个名为`checkNumberType`的方法，它接收一个整数参数`number`并返回一个描述数字类型的字符串。

通过使用模式守卫，我们可以对`number`进行多个条件的匹配，并根据条件来返回相应的结果。在每个`case`语句中，我们使用模式守卫来进一步过滤匹配的数字。

例如，`case n if n > 0 && n % 2 == 0` 表示当 `number` 大于 0 且为偶数时执行该分支。类似地，其他的 `case` 语句也使用了模式守卫来进行更精确的匹配。

在测试部分，我们调用了`checkNumberType`方法并传入不同的整数进行测试。根据不同的输入，方法将返回相应的字符串描述数字类型。

##### 仅匹配类型

当不同类型对象需要调用不同方法时，仅匹配类型的模式非常有用

```scala
def processValue(value: Any): String = value match {
  case str: String => s"Received a String: $str"
  case num: Int => s"Received an Int: $num"
  case lst: List[_] => s"Received a List: $lst"
  case _: Double => "Received a Double"
  case _ => "Unknown value"
}

// 测试
println(processValue("Hello"))                // 输出: Received a String: Hello
println(processValue(10))                     // 输出: Received an Int: 10
println(processValue(List(1, 2, 3)))           // 输出: Received a List: List(1, 2, 3)
println(processValue(3.14))                    // 输出: Received a Double
println(processValue(true))                    // 输出: Unknown value
```

在上述示例中，定义了一个名为`processValue`的方法，它接收一个任意类型的参数`value`，并返回一个描述值类型的字符串。

通过使用类型模式匹配，我们可以根据不同的值类型来执行相应的逻辑。在每个`case`语句中，我们使用类型模式匹配来匹配特定类型的值。

例如，`case str: String` 表示当 `value` 的类型为 `String` 时执行该分支，并将其绑定到变量 `str`。类似地，其他的 `case` 语句也使用了类型模式匹配来匹配不同的值类型。

在测试部分，我们调用了`processValue`方法并传入不同类型的值进行测试。根据值的类型，方法将返回相应的描述字符串。

**Scala的模式匹配是我觉得非常实用和灵活的一个功能，比Java的`switch`语句更加强大和灵活。Scala的模式匹配可以匹配不同类型的值，包括数字、字符串、列表、元组等。而Java的`switch`语句只能匹配整数、枚举和字符串类型的值**。

##### 密封类

特质（trait）和类（class）可以用`sealed`标记为密封的，这意味着其所有子类都必须与之定义在相同文件中，从而保证所有子类型都是已知的。密封类限制了可扩展的子类类型，并在模式匹配中确保所有可能的类型都被处理，提高了代码的安全性和可靠性。

下面是一个使用密封类（sealed class）和模式匹配的 Scala 代码示例：

```scala
sealed abstract class Shape

case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape
case class Square(side: Double) extends Shape

def calculateArea(shape: Shape): Double = shape match {
  case Circle(radius) => math.Pi * radius * radius
  case Rectangle(width, height) => width * height
  case Square(side) => side * side
}

// 测试
val circle = Circle(5.0)
val rectangle = Rectangle(3.0, 4.0)
val square = Square(2.5)

println(s"Area of circle: ${calculateArea(circle)}")         // 输出: Area of circle: 78.53981633974483
println(s"Area of rectangle: ${calculateArea(rectangle)}")   // 输出: Area of rectangle: 12.0
println(s"Area of square: ${calculateArea(square)}")         // 输出: Area of square: 6.25
```

在上述示例中，我们定义了一个密封类`Shape`，它是一个抽象类，不能直接实例化。然后，我们通过扩展`Shape`类创建了`Circle`、`Rectangle`和`Square`这三个子类。

在`calculateArea`方法中，我们使用模式匹配对传入的`shape`进行匹配，并根据不同的`Shape`子类执行相应的逻辑。在每个`case`语句中，我们根据具体的形状类型提取相应的属性，并计算出面积。

在测试部分，我们创建了一个`Circle`对象、一个`Rectangle`对象和一个`Square`对象，并分别调用`calculateArea`方法计算它们的面积。

### 嵌套方法

当在Scala中定义一个方法时，我们可以选择将其嵌套在另一个方法内部。这样的嵌套方法只在外部方法的作用域内可见，而对于外部方法以外的代码是不可见的。这可以帮助我们组织和封装代码，提高代码的可读性和可维护性。

```scala
def calculateDiscountedPrice(originalPrice: Double, discountPercentage: Double): Double = {
  def applyDiscount(price: Double, discount: Double): Double = {
    val discountedPrice = price - (price * discount)
    discountedPrice
  }

  def validateDiscount(discount: Double): Double = {
    val maxDiscount = 0.8 // 最大折扣为80%
    if (discount > maxDiscount) {
      maxDiscount
    } else {
      discount
    }
  }

  val validatedDiscount = validateDiscount(discountPercentage)
  val finalPrice = applyDiscount(originalPrice, validatedDiscount)
  finalPrice
}

// 调用外部方法
val price = calculateDiscountedPrice(100.0, 0.9)
println(s"The final price is: $price")
```

在上述示例中，定义了一个外部方法`calculateDiscountedPrice`，它接收原始价格`originalPrice`和折扣百分比`discountPercentage`作为参数，并返回最终价格。

在`calculateDiscountedPrice`方法的内部，我们定义了两个嵌套方法：`applyDiscount`和`validateDiscount`。`applyDiscount`方法用于计算折扣后的价格，它接收价格和折扣作为参数，并返回折扣后的价格。`validateDiscount`方法用于验证折扣百分比是否超过最大折扣限制，并返回一个有效的折扣百分比。

在外部方法中，我们首先调用`validateDiscount`方法来获取有效的折扣百分比，然后将其与原始价格一起传递给`applyDiscount`方法，计算最终价格。最后，我们打印出最终价格。

### 正则表达式模型

正则表达式是用来找出数据中的指定模式（或缺少该模式）的字符串。`.r`方法可使任意字符串变成一个正则表达式。

```scala
object Main extends App {
  val emailPattern = "([a-zA-Z0-9_.+-]+)@([a-zA-Z0-9-]+)\\.([a-zA-Z0-9-.]+)".r

  def validateEmail(email: String): Boolean = email match {
    case emailPattern(username, domain, extension) =>
      println(s"Valid email address: $email")
      true
    case _ =>
      println(s"Invalid email address: $email")
      false
  }

  // 测试
  validateEmail("john.doe@example.com")        // 输出: Valid email address: john.doe@example.com
  validateEmail("jane.doe@invalid")            // 输出: Invalid email address: jane.doe@invalid
}
```

在上述示例中，我们首先创建了一个名为`emailPattern`的正则表达式对象，用于匹配电子邮件地址的模式。

然后，定义了一个名为`validateEmail`的方法，它接收一个字符串类型的电子邮件地址作为参数，并使用正则表达式模式匹配来验证电子邮件地址的有效性。

在模式匹配的`case`语句中，我们使用`emailPattern`对传入的电子邮件地址进行匹配，并将匹配结果中的用户名、域名和扩展提取到相应的变量中。如果匹配成功，我们打印出验证通过的消息，并返回`true`表示电子邮件地址有效。如果没有匹配成功，则打印出验证失败的消息，并返回`false`表示电子邮件地址无效。

在测试部分，我们调用`validateEmail`方法分别传入一个有效的电子邮件地址和一个无效的电子邮件地址进行测试。根据匹配结果，我们打印出相应的验证消息。

### 型变

在 Scala 中，协变（covariance）和逆变（contravariance）是用来描述类型参数在子类型关系中的行为的概念。协变和逆变是用来指定泛型类型参数的子类型关系的方式，以确保类型安全性。

#### 协变

**协变（Covariance）：** 协变表示类型参数在子类型关系中具有相同的方向。如果一个泛型类的类型参数是协变的，那么子类型的关系将保持不变，即父类型可以被替换为子类型。在 Scala 中，可以使用 `+` 符号来表示协变。

下面是一个使用协变的示例代码，使用 `+` 符号表示类型参数 `A` 是协变的：

```scala
class Animal
class Dog extends Animal

class Cage[+A]

val dogCage: Cage[Dog] = new Cage[Dog]
val animalCage: Cage[Animal] = dogCage
```

在上述示例中，我们定义了一个协变类 `Cage[+A]`，它接受一个类型参数 `A`，并使用 `+` 符号来表示 `A` 是协变的。我们创建了一个 `dogCage`，它是一个 `Cage[Dog]` 类型的实例。然后，我们将 `dogCage` 赋值给一个类型为 `Cage[Animal]` 的变量 `animalCage`，这是合法的，因为 `Cage[+A]` 的协变性允许我们将子类型的 `Cage` 赋值给父类型的 `Cage`。

#### 逆变

**逆变（Contravariance）：** 逆变表示类型参数在子类型关系中具有相反的方向。如果一个泛型类的类型参数是逆变的，那么子类型的关系将反转，即父类型可以替换为子类型。在 Scala 中，可以使用 `-` 符号来表示逆变。

下面是一个使用逆变的示例代码，使用 `-` 符号表示类型参数 `A` 是逆变的：

```scala
class Animal
class Dog extends Animal

class Cage[-A]

val animalCage: Cage[Animal] = new Cage[Animal]
val dogCage: Cage[Dog] = animalCage
```

在上述示例中，定义了一个逆变类 `Cage[-A]`，它接受一个类型参数 `A`，并使用 `-` 符号来表示 `A` 是逆变的。我们创建了一个 `animalCage`，它是一个 `Cage[Animal]` 类型的实例。然后，我们将 `animalCage` 赋值给一个类型为 `Cage[Dog]` 的变量 `dogCage`，这是合法的，因为 `Cage[-A]` 的逆变性允许我们将父类型的 `Cage` 赋值给子类型的 `Cage`。
通过协变和逆变，我们可以在 Scala 中实现更灵活的类型关系，并确保类型安全性。这在处理泛型集合或函数参数时特别有用。下面是一个更具体的示例：

```scala
abstract class Animal {
  def name: String
}

class Dog(val name: String) extends Animal {
  def bark(): Unit = println("Woof!")
}

class Cat(val name: String) extends Animal {
  def meow(): Unit = println("Meow!")
}

class Cage[+A](val animal: A) {
  def showAnimal(): Unit = println(animal.name)
}

def printAnimalNames(cage: Cage[Animal]): Unit = {
  cage.showAnimal()
}

val dog: Dog = new Dog("Fido")
val cat: Cat = new Cat("Whiskers")

val dogCage: Cage[Dog] = new Cage[Dog](dog)
val catCage: Cage[Cat] = new Cage[Cat](cat)

printAnimalNames(dogCage) // 输出：Fido
printAnimalNames(catCage) // 输出：Whiskers
```

在上述示例中，定义了一个抽象类 `Animal`，以及它的两个子类 `Dog` 和 `Cat`。`Dog` 和 `Cat` 类都实现了 `name` 方法。

然后，定义了一个协变类 `Cage[+A]`，它接受一个类型参数 `A`，并使用协变符号 `+` 表示 `A` 是协变的。`Cage` 类有一个名为 `animal` 的属性，它的类型是 `A`，也就是动物的类型。我们定义了一个名为 `showAnimal()` 的方法，它打印出 `animal` 的名称。

接下来，定义了一个名为 `printAnimalNames()` 的函数，它接受一个类型为 `Cage[Animal]` 的参数，并打印出其中动物的名称。

我们创建了一个 `Dog` 类型的对象 `dog` 和一个 `Cat` 类型的对象 `cat`。然后，我们分别创建了一个 `Cage[Dog]` 类型的 `dogCage` 和一个 `Cage[Cat]` 类型的 `catCage`。

最后，我们分别调用 `printAnimalNames()` 函数，并传入 `dogCage` 和 `catCage`。由于 `Cage` 类是协变的，所以可以将 `Cage[Dog]` 和 `Cage[Cat]` 赋值给 `Cage[Animal]` 类型的参数，而不会产生类型错误。

### 类型限界

在 Scala 中，类型上界（Upper Bounds）和类型下界（Lower Bounds）是用于限制泛型类型参数的范围的概念。它们允许我们在泛型类或泛型函数中指定类型参数必须满足某种条件。下面是关于类型上界和类型下界的解释和示例代码：

#### 类型上界

**类型上界（Upper Bounds）：** 类型上界用于指定泛型类型参数必须是某个类型或其子类型。我们使用 `<:` 符号来定义类型上界。例如，`A <: B` 表示类型参数 `A` 必须是类型 `B` 或其子类型。

下面是一个使用类型上界的示例代码：

```scala
abstract class Animal {
  def name: String
}

class Dog(val name: String) extends Animal {
  def bark(): Unit = println("Woof!")
}

class Cage[A <: Animal](val animal: A) {
  def showAnimal(): Unit = println(animal.name)
}

val dog: Dog = new Dog("Fido")
val cage: Cage[Animal] = new Cage[Dog](dog)

cage.showAnimal() // 输出：Fido
```

在上述示例中，定义了一个抽象类 `Animal`，以及它的子类 `Dog`。`Dog` 类继承自 `Animal` 类，并实现了 `name` 方法。

然后，定义了一个泛型类 `Cage[A <: Animal]`，它接受一个类型参数 `A`，并使用类型上界 `A <: Animal` 来确保 `A` 是 `Animal` 类型或其子类型。`Cage` 类有一个名为 `animal` 的属性，它的类型是 `A`。我们定义了一个名为 `showAnimal()` 的方法，它打印出 `animal` 的名称。

创建了一个 `Dog` 类型的对象 `dog`。然后，我们创建了一个 `Cage[Animal]` 类型的 `cage`，并将 `dog` 对象作为参数传递给它。

最后，调用 `cage` 的 `showAnimal()` 方法，它成功打印出了 `Dog` 对象的名称。

#### 类型下界

**类型下界（Lower Bounds）：** 类型下界用于指定泛型类型参数必须是某个类型或其父类型。我们使用 `>` 符号来定义类型下界。例如，`A >: B` 表示类型参数 `A` 必须是类型 `B` 或其父类型。

下面是一个使用类型下界的示例代码：

```scala
class Animal {
  def sound(): Unit = println("Animal sound")
}

class Dog extends Animal {
  override def sound(): Unit = println("Dog barking")
}

class Cat extends Animal {
  override def sound(): Unit = println("Cat meowing")
}

def makeSound[A >: Dog](animal: A): Unit = {
  animal.sound()
}

val dog: Dog = new Dog
val cat: Cat = new Cat

makeSound(dog) // 输出：Dog barking
makeSound(cat) // 输出：Animal sound
```

在上述示例中，定义了一个基类 `Animal`，以及两个子类 `Dog` 和 `Cat`。这些类都有一个 `sound()` 方法，用于输出不同的动物声音。

接下来，定义了一个泛型函数 `makeSound[A >: Dog](animal: A)`，其中类型参数 `A` 的下界被定义为 `Dog`，即 `A >: Dog`。这意味着 `A` 必须是 `Dog` 类型或其父类型。

在 `makeSound()` 函数内部，我们调用传入的 `animal` 对象的 `sound()` 方法。

然后，创建了一个 `Dog` 对象 `dog` 和一个 `Cat` 对象 `cat`。

最后，分别调用 `makeSound()` 函数，并将 `dog` 和 `cat` 作为参数传递进去。由于类型下界被定义为 `Dog`，所以 `dog` 参数符合条件，而 `cat` 参数被隐式地向上转型为 `Animal`，也满足条件。因此，调用 `makeSound()` 函数时，输出了不同的声音。

通过类型上界和类型下界，我们可以对泛型类型参数的范围进行限制，以确保类型的约束和类型安全性。这使得我们能够编写更灵活、可复用且类型安全的代码。

### 内部类

在 Scala 中，内部类是一个定义在另一个类内部的类。内部类可以访问外部类的成员，并具有更紧密的关联性。下面是一个关于 Scala 中内部类的解释和示例代码：

在 Scala 中，内部类可以分为两种类型：成员内部类（Member Inner Class）和局部内部类（Local Inner Class）。

**成员内部类：** 成员内部类是定义在外部类的作用域内，并可以直接访问外部类的成员（包括私有成员）。成员内部类可以使用外部类的实例来创建和访问。

下面是一个示例代码：

```scala
class Outer {
  private val outerField: Int = 10

  class Inner {
    def printOuterField(): Unit = {
      println(s"Outer field value: $outerField")
    }
  }
}

val outer: Outer = new Outer
val inner: outer.Inner = new outer.Inner
inner.printOuterField() // 输出：Outer field value: 10
```

在上述示例中，定义了一个外部类 `Outer`，它包含一个私有成员 `outerField`。内部类 `Inner` 定义在 `Outer` 的作用域内，并可以访问外部类的成员。

在主程序中，创建了外部类的实例 `outer`。然后，我们使用 `outer.Inner` 来创建内部类的实例 `inner`。注意，我们需要使用外部类的实例来创建内部类的实例。

最后，调用内部类 `inner` 的 `printOuterField()` 方法，它成功访问并打印了外部类的私有成员 `outerField`。

**局部内部类：** 局部内部类是定义在方法或代码块内部的类。局部内部类的作用域仅限于所在方法或代码块内部，无法从外部访问。

下面是一个示例代码：

```scala
def outerMethod(): Unit = {
  val outerField: Int = 10

  class Inner {
    def printOuterField(): Unit = {
      println(s"Outer field value: $outerField")
    }
  }

  val inner: Inner = new Inner
  inner.printOuterField() // 输出：Outer field value: 10
}

outerMethod()
```

在上述示例中，定义了一个外部方法 `outerMethod`。在方法内部，我们定义了一个局部变量 `outerField` 和一个局部内部类 `Inner`。

在方法内部，创建了内部类 `Inner` 的实例 `inner`。注意，内部类的作用域仅限于方法内部。

最后，调用内部类 `inner` 的 `printOuterField()` 方法，它成功访问并打印了外部变量 `outerField`。

通过使用内部类，我们可以在 Scala 中实现更紧密的关联性和封装性，同时允许内部类访问外部类的成员。内部类在某些场景下可以提供更清晰和组织良好的。

### 复合类型

在 Scala 中，复合类型（Compound Types）允许我们定义一个类型，它同时具有多个特质（Traits）或类的特性。复合类型可以用于限制一个对象的类型，以便它同时具备多个特性。下面是关于复合类型的解释和示例代码：

复合类型使用 `with` 关键字将多个特质或类组合在一起，形成一个新的类型。

下面是一个示例代码：

```scala
trait Flyable {
  def fly(): Unit
}

trait Swimmable {
  def swim(): Unit
}

class Bird extends Flyable {
  override def fly(): Unit = println("Flying...")
}

class Fish extends Swimmable {
  override def swim(): Unit = println("Swimming...")
}

def action(obj: Flyable with Swimmable): Unit = {
  obj.fly()
  obj.swim()
}

val bird: Bird = new Bird
val fish: Fish = new Fish

action(bird) // 输出：Flying...
action(fish) // 输出：Swimming...
```

在上述示例中，定义了两个特质 `Flyable` 和 `Swimmable`，分别表示可飞行和可游泳的特性。然后，我们定义了两个类 `Bird` 和 `Fish`，分别实现了相应的特质。

接下来，定义了一个方法 `action`，它接受一个类型为 `Flyable with Swimmable` 的参数。这表示参数必须同时具备 `Flyable` 和 `Swimmable` 的特性。

在主程序中，创建了一个 `Bird` 对象 `bird` 和一个 `Fish` 对象 `fish`。

最后，分别调用 `action` 方法，并将 `bird` 和 `fish` 作为参数传递进去。由于它们都同时具备 `Flyable` 和 `Swimmable` 的特性，所以可以成功调用 `fly()` 和 `swim()` 方法。

通过使用复合类型，可以在 Scala 中定义一个类型，它同时具备多个特质或类的特性，从而实现更灵活和精确的类型约束。这有助于编写更可靠和可复用的代码。

### 多态方法

在 Scala 中，多态方法（Polymorphic Methods）允许我们定义可以接受多种类型参数的方法。这意味着同一个方法可以根据传入参数的类型执行不同的逻辑。下面是关于多态方法的解释和示例代码：

多态方法使用类型参数来定义方法的参数类型，并使用泛型来表示可以接受多种类型参数。在方法内部，可以根据类型参数的实际类型执行不同的逻辑。

下面是一个示例代码：

```scala
def printType[T](value: T): Unit = {
  value match {
    case s: String => println("String: " + s)
    case i: Int => println("Int: " + i)
    case d: Double => println("Double: " + d)
    case _ => println("Unknown type")
  }
}

printType("Hello") // 输出：String: Hello
printType(123) // 输出：Int: 123
printType(3.14) // 输出：Double: 3.14
printType(true) // 输出：Unknown type
```

在上述示例中，定义了一个多态方法 `printType`，它接受一个类型参数 `T`。根据传入参数的类型，我们使用模式匹配来判断其实际类型，并执行相应的逻辑。

在方法内部，使用 `match` 表达式对传入的参数 `value` 进行模式匹配。对于不同的类型，我们分别输出相应的类型信息。

在主程序中，多次调用 `printType` 方法，并传入不同类型的参数。根据传入的参数类型，方法会执行相应的逻辑并输出对应的类型信息。

### 函数

Scala中一个简单的函数定义如下，我们可以在Scala中使用JDK的类：

```scala
import java.util.Date
object Main {
  def main(args: Array[String]): Unit = {
    printCurrentDate() // 输出当前日期和时间
  }
  def printCurrentDate(): Unit = {
    val currentDate = new Date()
    println(currentDate.toString)
  }
}
```

#### 函数默认值

在 Scala 中，可以为函数参数指定默认值。这样，当调用函数时如果没有提供参数值，将使用默认值。下面是一个简单的示例：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    greet() // 输出 "Hello, World!"
    greet("Alice") // 输出 "Hello, Alice!"
  }

  def greet(name: String = "World"): Unit = {
    println(s"Hello, $name!")
  }
}
```

#### 高阶函数

高阶函数是指使用其他函数作为参数、或者返回一个函数作为结果的函数。在Scala中函数是“一等公民”，所以允许定义高阶函数。这里的术语可能有点让人困惑，我们约定，使用函数值作为参数，或者返回值为函数值的“函数”和“方法”，均称之为“高阶函数”。

```scala
def applyFuncToList(list: List[Int], f: Int => Int): List[Int] = {
  list.map(f)
}

val numbers = List(1, 2, 3, 4)
val double = (x: Int) => x * 2
val doubledNumbers = applyFuncToList(numbers, double) // List(2, 4, 6, 8)
```

在这个例子中，`applyFuncToList` 函数接受一个整数列表和一个函数 `f`，该函数将一个整数作为输入并返回一个整数。然后，`applyFuncToList` 函数使用 `map` 方法将函数 `f` 应用于列表中的每个元素。在上面的代码中，我们定义了一个 `double` 函数，它将输入乘以2，并将其传递给 `applyFuncToList` 函数以对数字列表中的每个元素进行加倍。

#### 匿名函数

在 Scala 中，匿名函数是一种没有名称的函数，可以用来创建简洁的函数字面量。它们通常用于传递给高阶函数，或作为局部函数使用。

例如，下面是一个简单的匿名函数，它接受两个整数参数并返回它们的和：

```scala
object Main {
  def main(args: Array[String]): Unit = {
    val add = (x: Int, y: Int) => x + y
    println(add(1, 2))  //输出： 3
  }
}
```

#### 偏应用函数

简单来说，偏应用函数就是一种只对输入值的某个子集进行处理的函数。它只会对符合特定条件的输入值进行处理，而对于不符合条件的输入值则会抛出异常。

举个例子：

```scala
object Main {
  def main(args: Array[String]): Unit = {

    println(divide.isDefinedAt(0)) // false
    println(divideSafe.isDefinedAt(0)) // true

    println(divide(1)) // 42
    println(divideSafe(1)) // Some(42)

    // println(divide(0)) // 抛出异常
    println(divideSafe(0)) // None
  }

  val divide: PartialFunction[Int, Int] = {
    case d: Int if d != 0 => 42 / d
  }

  val divideSafe: PartialFunction[Int, Option[Int]] = {
    case d: Int if d != 0 => Some(42 / d)
    case _ => None
  }

}
```

这个例子中，`divide` 是一个偏应用函数，它只定义了对非零整数的除法运算。如果我们尝试用 `divide` 函数去除以零，它会抛出一个异常。其中`isDefinedAt` 是一个方法，它用于检查偏应用函数是否在给定的输入值上定义。如果偏应用函数在给定的输入值上定义，那么 `isDefinedAt` 方法会返回 `true`，否则返回 `false`。

为了避免这种情况，我们可以使用 `divideSafe` 函数，它返回一个 `Option` 类型的结果。如果除数为零，它会返回 `None` 而不是抛出异常。

#### 柯里化函数

柯里化（Currying）是一种将多参数函数转换为一系列单参数函数的技术。我们可以使用柯里化来定义函数，例如：

```scala
def add(a: Int)(b: Int): Int = a + b
```

这个 `add` 函数接受两个参数 `a` 和 `b`，并返回它们的和。由于它是一个柯里化函数，所以我们可以将它看作是一个接受单个参数 `a` 的函数，它返回一个接受单个参数 `b` 的函数。

我们可以这样调用这个函数：

```scala
val result = add(1)(2) // 3
```

或者这样：

```scala
val addOne = add(1) _
val result = addOne(2) // 3
```

在上面的例子中，我们首先调用 `add` 函数并传入第一个参数 `1`，然后我们得到一个新的函数 `addOne`，它接受一个参数并返回它与 1 的和。最后，我们调用 `addOne` 函数并传入参数 `2`，得到结果 3。

柯里化函数可以帮助我们实现参数复用和延迟执行等功能。

柯里化函数的好处之一是它可以让我们给一个函数传递较少的参数，得到一个已经记住了某些固定参数的新函数。这样，我们就可以在不同的地方使用这个新函数，而不需要每次都传递相同的参数²。

此外，柯里化函数还可以帮助我们实现函数的延迟计算。当我们传递部分参数时，它会返回一个新的函数，可以在新的函数中继续传递后面的参数。这样，我们就可以根据需要来决定何时执行这个函数。

#### 惰性函数

可以使用 `lazy` 关键字定义惰性函数。惰性函数的执行会被推迟，直到我们首次对其取值时才会执行。

下面是一个简单的例子，展示了如何定义和使用惰性函数：

```scala
def sum(x: Int, y: Int): Int = {
  println("sum函数被执行了...")
  x + y
}

lazy val res: Int = sum(1, 2)

println("-----")
println(res)
```

在这个例子中，我们定义了一个函数 `sum`，它接受两个参数并返回它们的和。然后我们定义了一个惰性值 `res` 并将其赋值为 `sum(1, 2)`。

在主程序中，我们首先打印了一行分隔符。然后我们打印了变量 `res` 的值。由于 `res` 是一个惰性值，因此在打印它之前，函数 `sum` 并没有被执行。只有当我们首次对 `res` 取值时，函数 `sum` 才会被执行。

这就是Scala中惰性函数的基本用法。你可以使用 `lazy` 关键字定义惰性函数，让函数的执行被推迟。
