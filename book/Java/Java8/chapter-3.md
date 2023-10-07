---
sort: 3
---
# 第三章 Lambda表达式

内容简介:
本章会展示如何构建Lambda， 它的使用场合， 以及如何利用它使代码更简洁。 我们还会介绍一些新的东西， 如类型推断和Java 8 API中重要的新接口。 最后， 我们将介绍方法引用（ method reference） ， 这是一个常常和Lambda表达式联用的有用的新功能。

## 1. Lambda
可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式： 它没有名称， 但它有参数列表、 函数主体、 返回类型， 可能还有一个可以抛出的异常列表。
* 匿名 —— 我们说匿名， 是因为它不像普通的方法那样有一个明确的名称： 写得少而想得多！
* 函数 —— 我们说它是函数， 是因为Lambda函数不像方法那样属于某个特定的类。 但和方法一样， Lambda有参数列表、 函数主体、 返回类型， 还可能有可以抛出的异常列表。
* 传递 —— Lambda表达式可以作为参数传递给方法或存储在变量中。
* 简洁 —— 无需像匿名类那样写很多模板代码。
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic3-1.png)

**上图: Lambda表达式由参数、 箭头和主体组成**

相比于匿名函数，Lambda表达式，使得代码更加简洁。Lambda表达式有三个部分(如上图)
* 参数列表 —— 这里它采用了Comparator中compare方法的参数， 两个Apple。 
* 箭头 —— 箭头->把参数列表与Lambda主体分隔开。
* Lambda主体 —— 比较两个Apple的重量。 表达式就是Lambda的返回值了。

下面给出了Java 8中五个有效的Lambda表达式的例子。
```java
// 第一个Lambda表达式具有一个String类型的参数并返回一个int。Lambda没有return语句，因为已经隐含了return
(String s) -> s.length()

// 第二个Lambda表达式有一个Apple 类型的参数并返回一个boolean（ 苹果的重量是否超过150克）
(Apple a) -> a.getWeight() > 150

// 第三个Lambda表达式具有两个int类型的参数而没有返回值（ void返回）。 注意Lambda表达式可以包含多行语句，这里是两行
(int x, int y) -> {
    System.out.println("Result:");
    System.out.println(x+y);
}

// 第四个Lambda表达式没有参数，返回一个int
() -> 42

// 第五个Lambda表达式具有两个Apple类型的参数，返回一个int：比较两个Apple的重量
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
```

Lambda的基本语法是 ```(parameters) -> expression``` 或者 ```(parameters) -> { statements; }```

**PS: ```(Integer i) -> return "Alan" + i;```这个是错误的写法，因为return是一个控制流语句。 要使此Lambda有效， 需要使花括号， 如下所示：```(Integer i) -> {return "Alan" + i;}```** 

**Lambda示例**

| 使用案例              | Lambda示例                                                   |
| --------------------- | ------------------------------------------------------------ |
| 布尔表达式            | (List\<String\> list) -> list.isEmpty()                      |
| 创建对象              | () -> new Apple(10)                                          |
| 消费一个对象          | (Apple a) -> { System.out.println(a.getWeight()); }          |
| 从一个对象中选择/抽取 | (String s) -> s.length()                                     |
| 组合两个值            | (int a, int b) -> a * b                                      |
| 比较两个对象          | (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) |

## 2. Lambda使用场景
### 2.1 函数式接口
函数式接口就是只定义**一个**抽象方法的接口。 Java API中的一些其他函数式接口， 如[第二章](./chapter-2.md)中谈到的Comparator和Runnable。

Lambda表达式允许你直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例（ 具体说来， 是函数式接口一个具体实现的实例） 。 

### 2.2 函数描述符
函数式接口的抽象方法的签名基本上就是Lambda表达式的签名。 我们将这种抽象方法叫作函数描述符。 例如， Runnable接口可以看作一个什么也不接受什么也不返回（ void） 的函数的签名， 因为它只有一个叫作run的抽象方法， 这个方法什么也不接受， 什么也不返回（ void） 。

本章中使用了一个特殊表示法来描述Lambda和函数式接口的签名。 () -> void代表了参数列表为空， 且返回void的函数。 这正是Runnable接口所代表的。举另一个例子， (Apple, Apple) -> int代表接受两个Apple作为参数且返回int的函数。 

## 3. 把Lambda付诸实践： 环绕执行模式
资源处理（ 例如处理文件或数据库） 时一个常见的模式就是打开一个资源， 做一些处理， 然后关闭资源。 这个设置和清理阶段总是很类似， 并且会围绕着执行处理的那些重要代码。 这就是所谓的环绕执行（ execute around） 模式，如下图所示。 例如， 在以下代码中， 高亮显示的就是从一个文件中读取一行所需的模板代码（ 注意你使用了Java 7中的带资源的try语句， 它已经简化了代码， 因为你不需要显式地关闭资源了） ：
```java
public static String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine(); // 这就是做有用工作的那行代码
    }
}
```
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic3-2.png)

**任务A和任务B周围都环绕着进行准备/清理的同一段冗余代码**

### 3.1 第1步： 记得行为参数化
```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
### 3.2 第2步： 使用函数式接口来传递行为
```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

public static String processFile(BufferedReaderProcessor p) throws IOException {
    // do some thing
}
```

### 3.3 第3步： 执行一个行为
任何BufferedReader -> String形式的Lambda都可以作为参数来传递， 因为它们符合BufferedReaderProcessor接口中定义的process方法的签名。 现在你只需要一种方法在processFile主体内执行Lambda所代表的代码。 Lambda表达式允许你直接内联， 为函数式接口的抽象方法提供实现， 并且将整个表达式作为函数式接口的一个实例。 因此， 你可以在processFile主体内， 对得到的BufferedReaderProcessor对象调用process方法执行处理：
```java
public static String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br); // 处理BufferedReader对象
    }
}
```

### 3.4 第4步： 传递Lambda
通过传递不同的Lambda重用processFile方法， 并以不同的方式处理文件了。
```java
// 处理一行
String oneLine = processFile((BufferedReader br) -> br.readLine());
// 处理两行
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

下图总结了所采取的使pocessFile方法更灵活的四个步骤。
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic3-3.png)

**应用环绕执行模式所采取的四个步骤**

## 4. 使用函数式接口

### 4.1 Predicate
```java.util.function.Predicate<T>```接口定义了一个名叫test的抽象方法， 它接受泛型T对象， 并返回一个boolean。 

### 4.2 Consumer
```java.util.function.Consumer<T>```定义了一个名叫accept的抽象方法， 它接受泛型T的对象， 没有返回（ void）。

### 4.3 Function
```java.util.function.Function<T, R>```接口定义了一个叫作apply的方法， 它接受一个泛型T的对象， 并返回一个泛型R的对象。 如果你需要定义一个Lambda， 将输入对象的信息映射到输出， 就可以使用这个接口（ 比如提取苹果的重量， 或把字符串映射为它的长度）。
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T s : list) {
        result.add(f.apply(s));
    }
    return result;
}

// l的结果为 [7, 2, 6]
List<Integer> l = map(
        Arrays.asList("lambdas", "in", "action"),
        (String s) -> s.length() // Lambda是Function接口的apply方法的实现
);
```

**Java类型要么是引用类型（比如Byte、 Integer、 Object、 List）， 要么是原始类型（比如int、double、byte、char）。 但是泛型（比如Consumer\<T\>中的T）只能绑定到引用类型。 这是由泛型内部的实现方式造成的。 因此， 在Java里有一个将原始类型转换为对应的引用类型的机制。 这个机制叫作装箱（boxing）。 相反的操作， 也就是将引用类型转换为对应的原始类型， 叫作拆箱（unboxing）。 Java还有一个自动装箱机制来帮助程序员执行这一任务：装箱和拆箱操作是自动完成的。 如此处理在性能方面是要付出代价的， 装箱后的值本质上就是把原始类型包裹起来， 并保存在堆里。 因此， 装箱后的值需要更多的内存， 并需要额外的内存搜索来获取被包裹的原始值。**

Java 8为我们前面所说的函数式接口带来了一个专门的版本， 以便在输入和输出都是原始类型时避免自动装箱的操作。 比如， 在下面的代码中， 使用IntPredicate就避免了对值1000进行装箱操作， 但要是用Predicate\<Integer\>就会把参数1000装箱到一个Integer对象中：
```java
public interface IntPredicate {
    boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> { return (i%2) == 0; }
evenNumbers.test(1000); // true（ 无装箱）

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 == 1;
oddNumbers.test(1000); // false（ 装箱）
```

一般来说， 针对专门的输入参数类型的函数式接口的名称都要加上对应的原始类型前缀， 比如DoublePredicate、 IntConsumer、 LongBinaryOperator、 IntFunction等。 Function接口还有针对输出参数类型的变种： ToIntFunction\<T\>、 IntToDoubleFunction等。

下面的表格中列出了Java 8中的常用函数式接口:

| 函数式接口          | 函数描述符     | 原始类型特化                                                 |
| ------------------- | -------------- | ------------------------------------------------------------ |
| Predicate\<T\>      | T->boolean     | IntPredicate,LongPredicate, DoublePredicate                  |
| Consumer\<T\>       | T->void        | IntConsumer,LongConsumer, DoubleConsumer                     |
| Function\<T,R\>     | T->R           | IntFunction\<R\>, IntToDoubleFunction, IntToLongFunction, LongFunction<R>, LongToDoubleFunction, LongToIntFunction, DoubleFunction<R>, ToIntFunction\<T\>, ToDoubleFunction\<T\>, ToLongFunction\<T\> |
| Supplier\<T\>       | ()->T          | BooleanSupplier,IntSupplier, LongSupplier, DoubleSupplier    |
| UnaryOperator\<T\>  | T->T           | IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator     |
| BinaryOperator\<T\> | (T,T)->T       | IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator  |
| BiPredicate<L,R\>   | (L,R)->boolean |                                                              |
| BiConsumer\<T,U\>   | (T,U)->void    | ObjIntConsumer\<T\>, ObjLongConsumer\<T\>, ObjDoubleConsumer\<T\> |
| BiFunction\<T,U,R\> | (T,U)->R       | ToIntBiFunction\<T,U\>, ToLongBiFunction\<T,U\>, ToDoubleBiFunction\<T,U\> |

下面表格中列出了Lambdas及函数式接口的例子:

| 使用案例               | Lambda的例子                                                 | 对应的函数式接口                                             |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 布尔表达式             | (List\<String\> list) -> list.isEmpty()                        | Predicate\<List\<String\>\>                                      |
| 创建对象               | () -> new Apple(10)                                          | Supplier\<Apple\>                                              |
| 消费一个对象           | (Apple a) -> System.out.println(a.getWeight())               | Consumer\<Apple\>                                              |
| 从一个对象中选择/提 取 | (String s) -> s.length()                                     | Function\<String, Integer\>或 ToIntFunction\<String\>            |
| 合并两个值             | (int a, int b) -> a * b                                      | IntBinaryOperator                                            |
| 比较两个对象           | (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) | Comparator\<Apple\>或 BiFunction\<Apple, Apple, Integer\>或 ToIntBiFunction\<Apple, Apple\> |


**请注意， 任何函数式接口都不允许抛出受检异常（ checked exception） 。 如果你需要Lambda表达式来抛出异常， 有两种办法： 定义一个自己的函数式接口， 并声明受检异常， 或者把Lambda包在一个try/catch块中。**
**比如， 在3.3节我们介绍了一个新的函数式接口BufferedReaderProcessor，它显式声明了一个IOException**

但是你可能是在使用一个接受函数式接口的API， 比如```Function<T, R>```， 没有办法自己创建一个（ 你会在下一章看到， Stream API中大量使用表3-2中的函数式接口） 。 这种情况下， 你可以显式捕捉受检异常：
```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
};
```

## 5. 类型检查、 类型推断以及限制
### 5.1 类型检查
Lambda的类型是从使用Lambda的上下文推断出来的。 上下文（ 比如， 接受它传递的方法的参数， 或接受它的值的局部变量） 中Lambda表达式需要的类型称为目标类型。

下图概述了下列代码的类型检查过程。
```java
List<Apple> heavierThan150g = filter(inventory, (Apple a) -> a.getWeight() > 150);
```

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic3-4.png)
**解读Lambda表达式的类型检查过程**

这段代码是有效的， 因为我们所传递的Lambda表达式也同样接受Apple为参数， 并返回一个boolean。**请注意， 如果Lambda表达式抛出一个异常， 那么抽象方法所声明的throws语句也必须与之匹配。**

### 5.2 同样的Lambda， 不同的函数式接口

有了目标类型的概念， 同一个Lambda表达式就可以与不同的函数式接口联系起来，只要它们的抽象方法签名能够兼容。 比如， 前面提到的Callable和PrivilegedAction， 这两个接口都代表着什么也不接受且返回一个泛型T的函数。因此， 下面两个赋值是有效的：

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

下面展示了一个类似的例子； 同一个Lambda可用于多个不同的函数式接口：
```java
Comparator<Apple> c1 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
BiFunction<Apple, Apple, Integer> c3 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

**菱形运算符**
Java 7中已经引入了菱形运算符（ <>） ， 利用泛型推断从上下文推断类型的思想（ 这一思想甚至可以追溯到更早的泛型方法） 。 一个类实例表达式可以出现在两个或更多不同的上下文中， 并会像下面这样推断出适当的类型参数：
```java
List<String> listOfStrings = new ArrayList<>();
List<Integer> listOfIntegers = new ArrayList<>();
```

**特殊的void兼容规则**
如果一个Lambda的主体是一个语句表达式， 它就和一个返回void的函数描述符兼容（ 当然需要参数列表也兼容） 。 例如， 以下两行都是合法的， 尽管List的add方法返回了一个boolean， 而不是Consumer上下文（ T -> void） 所要求的void：
```java
// Predicate返回了一个boolean
Predicate<String> p = s -> list.add(s);
// Consumer返回了一个void
Consumer<String> b = s -> list.add(s);
```
**错误表达**
```java
    Object o = () -> {System.out.println("Tricky example"); };
```
Lambda表达式的上下文是Object（ 目标类型） 。 **但Object不是一个函数式接口。** 为了解决这个问题， 你可以把目标类型改成Runnable， 它的函数描述符是() -> void：

### 5.3 类型推断
Java编译器会从上下文（ 目标类型） 推断出用什么函数式接口来配合Lambda表达式， 这意味着它也可以推断出适合Lambda的签名， 因为函数描述符可以通过目标类型来得到。 这样做的好处在于， 编译器可以了解Lambda表达式的参数类型， 这样就可以在Lambda语法中省去标注参数类型。 

Lambda表达式有多个参数， 代码可读性的好处就更为明显。 例如， 你可以这样来创建一个Comparator对象：
```java
// 没有类型推断
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

// 有类型推断
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```
有时候显式写出类型更易读， 有时候去掉它们更易读。 没有什么法则说哪种更好； 对于如何让代码更易读， 程序员必须做出自己的选择。

### 5.4 使用局部变量
前面介绍的Lambda表达式都只用到了其主体里面的参数。其实Lambda表达式也允许使用自由变量（ 不是参数， 而是在外层作用域中定义的变量） ， 就像匿名类一样。 它们被称作捕获Lambda。 
如下： 下面的Lambda捕获了portNumber变量
```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

关于能对这些变量做什么有一些限制。 Lambda可以没有限制地捕获（ 也就是在其主体中引用） 实例变量和静态变量。 但局部变量必须显式声明为final， 或事实上是final。 换句话说， Lambda表达式只能捕获指派给它们的局部变量一次。 （ 注： 捕获实例变量可以被看作捕获最终局部变量this。 ） 
例如， 下面的代码无法编译， 因为portNumber变量被赋值两次：

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber); // 错误： Lambda表达式引用的局部变量必须是最终的（ final） 或事实上最终的
portNumber = 31337;
```

**对局部变量的限制**
1. 实例变量和局部变量背后的实现有一个关键不同。 实例变量都存储在堆中， 而局部变量则保存在栈上。 如果Lambda可以直接访问局部变量， 而且Lambda是在一个线程中使用的， 则使用Lambda的线程， 可能会在分配该变量的线程将这个变量收回之后， 去访问该变量。因此， Java在访问自由局部变量时， 实际上是在访问它的副本， 而不是访问原始变量。 如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。
2. 这一限制不鼓励你使用改变外部变量的典型命令式编程模式

**闭包**
Java 8的Lambda和匿名类可以做类似于闭包的事情： 它们可以作为参数传递给方法， 并且可以访问其作用域之外的变量。 但有一个限制： 它们不能修改定义Lambda的方法的局部变量的内容。 这些变量必须是隐式最终的。 可以认为Lambda是对值封闭， 而不是对变量封闭。 如前所述， 这种限制存在的原因在于局部变量保存在栈上， 并且隐式表示它们仅限于其所在线程。 如果允许捕获可改变的局部变量， 就会引发造成线程不安全的新的可能性， 而这是我们不想看到的（ 实例变量可以， 因为它们保存在堆中， 而堆是在线程之间共享的） 。

## 6. 方法引用
方法引用让你可以重复使用现有的方法定义， 并像Lambda一样传递它们。 在一些情况下， 比起使用Lambda表达式， 它们似乎更易读， 感觉也更自然。 

### 6.1 如何构建方法引用， 方法引用主要有三类
1. 指向静态方法的方法引用（ 例如Integer的parseInt方法， 写作Integer::parseInt） 。
2. 指向任意类型实例方法的方法引用（ 例如String的length方法， 写作String::length） 。
3. 指向现有对象的实例方法的方法引用（ 假设你有一个局部变量expensiveTransaction用于存放Transaction类型的对象， 它支持实例方法getValue， 那么你就可以写expensiveTransaction::getValue） 。
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic3-5.png)
**为三种不同类型的Lambda表达式构建方法引用的办法**

### 6.2 构造函数引用
对于一个现有构造函数， 你可以利用它的名称和关键字new来创建它的一个引用： ClassName::new。 它的功能与指向静态方法的引用类似。 例如， 假设有一个构造函数没有参数。 它适合Supplier的签名() -> Apple。 你可以这样做：
```java
Supplier<Apple> c1 = Apple::new; // 构造函数引用指向默认的Apple()构造函数
Apple a1 = c1.get(); // 调用Supplier的get方法将产生一个新的Apple

// 这就等价于
Supplier<Apple> c1 = () -> new Apple(); // 利用默认构造函数创建Apple的Lambda表达式
Apple a1 = c1.get(); // 调用Supplier的get方法将产生一个新的Apple
```

如果你的构造函数的签名是Apple(Integer weight)， 那么它就适合Function接口的签名， 于是你可以这样写：
```java
Function<Integer, Apple> c2 = Apple::new; // 指向Apple(Integer weight)的构造函数引用
Apple a2 = c2.apply(110); // 调用该Function函数的apply方法， 并给出要求的重量， 将产生一个Apple

// 这就等价于：
Function<Integer, Apple> c2 = (weight) -> new Apple(weight); // 用要求的重量创建一个Apple的Lambda表达式
Apple a2 = c2.apply(110); // 调用该Function函数的apply方法， 并给出要求的重量， 将产生一个新的Apple对象

// 一个由Integer构成的List中的每个元素都通过我们前面定义的类似的map方法传递给了Apple的构造函数， 得到了一个具有不同重量苹果的List：
List<Integer> weights = Arrays.asList(7, 3, 4, 10);
List<Apple> apples = map(weights, Apple::new); // 将构造函数引用传递给map方法

public static List<Apple> map(List<Integer> list, Function<Integer, Apple> f) {
    List<Apple> result = new ArrayList<>();
    for (Integer e : list) {
        result.add(f.apply(e));
    }
    return result;
}
```

如果你有一个具有两个参数的构造函数Apple(String color, Integerweight)， 那么它就适合BiFunction接口的签名， 于是你可以这样写：

```java
BiFunction<String, Integer, Apple> c3 = Apple::new; // 指向Apple(Stringcolor,Integerweight)的构造函数引用
Apple c3 = c3.apply("green", 110); // 调用该BiFunction函数的apply方法， 并给出要求的颜色和重量， 将产生一个新的Apple对象

// 这就等价于：
BiFunction<String, Integer, Apple> c3 = (color, weight) -> new Apple(color, weight); // 用要求的颜色和重量创建一个Apple的Lambda表达式
Apple c3 = c3.apply("green", 110); // 调用该BiFunction函数的apply方法， 并给出要求的颜色和重量， 将产生一个新的Apple对象
```

不将构造函数实例化却能够引用它， 这个功能有一些有趣的应用。 例如， 你可以使用Map来将构造函数映射到字符串值。 你可以创建一个giveMeFruit方法， 给它一个String和一个Integer， 它就可以创建出不同重量的各种水果：
```java
    static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
    static {
        map.put("apple", Apple::new);
        map.put("orange", Orange::new);
        // etc...
    }

    public static Fruit giveMeFruit(String fruit, Integer weight) {
        return map.get(fruit.toLowerCase()) // 你用map 得到了一个Function<Integer,Fruit>
                .apply(weight); // 用Integer类型的weight参数调用Function的apply()方法将提 供所要求的Fruit
    }
```

## 7. Lambda和方法引用实战
```java
// 第1步： 传递代码
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
        }
    } 
inventory.sort(new AppleComparator());

// 第2步： 使用匿名类
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
});

// 第3步： 使用Lambda表达式
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
// 或者
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));

//  Comparator具有一个叫作comparing的静态辅助方法， 它可以接受一个Function来提取Comparable键值， 并生成一个Comparator对象
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
// 或者
inventory.sort(comparing((a) -> a.getWeight())); 

// 第4步： 使用方法引用
nventory.sort(comparing(Apple::getWeight));
// 
```

## 8. 复合Lambda表达式的有用方法
Java 8的好几个函数式接口都有为方便而设计的方法。 具体而言， 许多函数式接口， 比如用于传递Lambda表达式的Comparator、 Function和Predicate都提供了允许你进行复合的方法。  在实践中， 这意味着你可以把多个简单的Lambda复合成复杂的表达式。 比如， 你可以让两个Predicate之间做一个or操作， 组合成一个更大的Predicate。 而且， 你还可以让一个函数的结果成为另一个函数的输入。 

### 8.1 比较器复合
使用静态方法Comparator.comparing， 根据提取用于比较的键值的Function来返回一个Comparator， 如下所示
```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```
1. 逆序
将苹果按照重量逆序排序，可以使用下面的方法实现
```java
inventory.sort(comparing(Apple::getWeight).reversed()); // 按重量递减排序
```
2. 比较器链
但如果发现有两个苹果一样重怎么办？ 哪个苹果应该排在前面呢？ 你可能需要再提供一个Comparator来进一步定义这个比较。 比如， 在按重量比较两个苹果之后， 你可能想要按原产国排序。 thenComparing方法就是做这个用的。 它接受一个函数作为参数（ 就像comparing方法一样） ， 如果两个对象用第一个Comparator比较之后是一样的， 就提供第二个Comparator。 
```java
inventory.sort(comparing(Apple::getWeight)
.reversed() // 按重量递减排序
.thenComparing(Apple::getCountry)); // 两个苹果一样重时， 进一步按国家排序
```

### 8.2 Predicate复合
Predicate接口包括三个方法： negate、 and和or， 让你可以重用已有的Predicate来创建更复杂的Predicate。 比如， 你可以使用negate方法来返回一个Predicate的非， 比如苹果不是红的：
```java
// 非红苹果
Predicate<Apple> notRedApple = redApple.negate(); // 产生现有Predicate对象redApple的非
// 比如一个苹果既是红色又比较重：
Predicate<Apple> redAndHeavyApple =redApple.and(a -> a.getWeight() > 150); // 链接两个谓词来生成另一个Predicate对象
// 要么是重（ 150克以上） 的红苹果， 要么是绿苹果：
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150).or(a -> "green".equals(a.getColor())); // 链接Predicate的方法来构造更复杂Predicate对象
```
**从简单Lambda表达式出发， 你可以构建更复杂的表达式， 但读起来仍然和问题的陈述差不多！ 请注意， and和or方法是按照在表达式链中的位置， 从左向右确定优先级的。 因此， a.or(b).and(c)可以看作(a || b) && c。**

### 8.3 函数复合
可以把Function接口所代表的Lambda表达式复合起来。 Function接口为此配了andThen和compose两个默认方法， 它们都会返回Function的一个实例。
andThen方法会返回一个函数， 它先对输入应用一个给定函数， 再对输出应用另一个函数。 比如， 假设有一个函数f给数字加1 (x -> x + 1)， 另一个函数g给数字乘2， 你可以将它们组合成一个函数h， 先给数字加1， 再给结果乘2：
```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g); // 数学上会写作g(f(x))或(g o f)(x)
int result = h.apply(1); // 这将返回4
```

使用compose方法， 先把给定的函数用作compose的参数里面给的那个函数， 然后再把函数本身用于结果。 比如在上一个例子里用compose的话， 它将意味着f(g(x))， 而andThen则意味着g(f(x))：
```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g); // 数学上会写作f(g(x))或(f o g)(x)
int result = h.apply(1); // 这将返回3
```
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic3-6.png)
**andThen和compose之间的区别**


## 9. 数学中的类似思想

## 10. 小结
* Lambda表达式可以理解为一种匿名函数： 它没有名称， 但有参数列表、 函数主体、 返回类型， 可能还有一个可以抛出的异常的列表。
* Lambda表达式让你可以简洁地传递代码。
* 函数式接口就是仅仅声明了一个抽象方法的接口。
* 只有在接受函数式接口的地方才可以使用Lambda表达式。
* Lambda表达式允许你直接内联， 为函数式接口的抽象方法提供实现， 并且将整个表达式作为函数式接口的一个实例。
* Java 8自带一些常用的函数式接口， 放在java.util.function包里， 包括Predicate\<T\>、 Function\<T,R\>、 Supplier\<T\>、 Consumer\<T\>和BinaryOperator\<T\>
* 为了避免装箱操作， 对Predicate\<T\>和Function\<T, R\>等通用函数式接口的原始类型特化： IntPredicate、 IntToLongFunction等。
* 环绕执行模式（ 即在方法所必需的代码中间， 你需要执行点儿什么操作， 比如资源分配和清理） 可以配合Lambda提高灵活性和可重用性。
* Lambda表达式所需要代表的类型称为目标类型。
* 方法引用让你重复使用现有的方法实现并直接传递它们。
* Comparator、 Predicate和Function等函数式接口都有几个可以用来结合Lambda表达式的默认方法。
