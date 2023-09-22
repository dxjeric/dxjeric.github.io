---
sort: 1
---

# 第一章 基础知识

## 1. 变化
### 1.1. 流处理
第一个编程概念是流处理。介绍一下，流是一系列数据项，一次只生成一项。程序可以从输入流中一个一个读取数据项，然后以同样的方式将数据项写入输出流。一个程序的输出流很可能是另一个程序的输入流。

### 1.2. 用行为参数化把代码传递给方法
Java 8中增加的另一个编程概念是通过API来传递代码的能力。这听起来实在太抽象了。

### 1.3. 并行与共享的可变数据
第三个编程概念更隐晦一点， 它来自我们前面讨论流处理能力时说的“几乎免费的并行” 。 你需要放弃什么吗？ 你可能需要对传给流方法的行为的写法稍作改变。 这些改变可能一开始会让你感觉有点儿不舒服， 但一旦习惯了你就会爱上它们。 你的行为必须能够同时对不同的输入安全地执行。 一般情况下这就意味着， 你写代码时不能访问共享的可变数据。 这些函数有时被称为“纯函数” 或“无副作用函数” 或“无状态函数” ， 这一点我们会在第7章和第13章详细讨论。 前面说的并行只有在假定你的代码的多个副本可以独立工作时才能进行。 但如果要写入的是一个共享变量或对象， 这就行不通了： 如果两个进程需要同时修改这个共享变量怎么办？（ 1.3节配图给出了更详细的解释。 ） 你在本书中会对这种风格有更多的了解。

Java 8的流实现并行比Java现有的线程API更容易， 因此， 尽管可以使用synchronized来打破“不能有共享的可变数据” 这一规则， 但这相当于是在和整个体系作对， 因为它使所有围绕这一规则做出的优化都失去意义了。 在多个处理器内核之间使用synchronized， 其代价往往比你预期的要大得多， 因为同步迫使代码按照顺序执行， 而这与并行处理的宗旨相悖。

这两个要点（ 没有共享的可变数据， 将方法和函数即代码传递给其他方法的能力）是我们平常所说的函数式编程范式的基石， 我们在第13章和第14章会详细讨论。 与此相反， 在命令式编程范式中， 你写的程序则是一系列改变状态的指令。 “不能有共享的可变数据” 的要求意味着， 一个方法是可以通过它将参数值转换为结果的方式完全描述的； 换句话说， 它的行为就像一个数学函数， 没有可见的副作用。

## 2. Java中的函数
编程语言中的函数一词通常是指方法， 尤其是静态方法； 这是在数学函数， 也就是没有副作用的函数之外的新含义。 幸运的是， 你将会看到， 在Java 8谈到函数时，这两种用法几乎是一致的。

Java 8中新增了函数——值的一种新形式。 它有助于使用1.3节中谈到的流， 有了它， Java 8可以进行多核处理器上的并行编程。 我们首先来展示一下作为值的函数本身的有用之处。

想想Java程序可能操作的值吧。 首先有原始值， 比如42（ int类型） 和3.14（ double类型） 。 其次， 值可以是对象（ **更严格地说是对象的引用**） 。 获得对象的唯一途径是利用new， 也许是通过工厂方法或库函数实现的； 对象引用指向类的一个实例。 例子包括"abc"（ String类型） ， new Integer(1111)（ Integer类型） ， 以及new HashMap<Integer,String>(100)的结果——它显然调用了HashMap的构造函数。 甚至数组也是对象。 

为了帮助回答这个问题， 我们要注意到， 编程语言的整个目的就在于操作值， 要是按照历史上编程语言的传统， 这些值因此被称为一等值（ 或一等公民， 这个术语是从20世纪60年代美国民权运动中借用来的） 。 编程语言中的其他结构也许有助于我们表示值的结构， 但在程序执行期间不能传递， 因而是二等公民。 前面所说的值是Java中的一等公民， 但其他很多Java概念（ 如方法和类等） 则是二等公民。 用方法来定义类很不错， 类还可以实例化来产生值， 但方法和类本身都不是值。 这又有什么关系呢？ 还真有， 人们发现， 在运行时传递方法能将方法变成一等公民。 这在编程中非常有用， 因此Java 8的设计者把这个功能加入到了Java中。 顺便说一下， 你可能会想， 让类等其他二等公民也变成一等公民可能也是个好主意。 有很多语言，如Smalltalk和JavaScript， 都探索过这条路。

### 2.1 方法和Lambda作为一等公民
Scala和Groovy等语言的实践已经证明， 让方法等概念作为一等值可以扩充程序员的工具库， 从而让编程变得更容易。 一旦程序员熟悉了这个强大的功能， 他们就再也不愿意使用没有这一功能的语言了。 因此， Java 8的设计者决定允许方法作为值， 让编程更轻松。 此外， 让方法作为值也构成了其他若干Java 8功能（ 如Stream） 的基础。

### 2.2 传递代码： 一个例子
假设你有一个Apple类， 它有一个getColor方法， 还有一个变量inventory保存着一个Apples的列表。 你可能想要选出所有的绿苹果， 并返回一个列表。 通常我们用筛选（ filter） 一词来表达这个概念。 在Java 8之前， 你可能会写这样一个方法filterGreenApples:

```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
    List<Apple> result = new ArrayList<>(); // result是用来累积结果的List， 开始为空，然后一个个加入绿苹果
    for (Apple apple: inventory){
        if ("green".equals(apple.getColor())) { // 高亮显示的代码会仅仅选出绿苹果
            result.add(apple);
        }
    }
    return result;
}
```
有人可能想要选出重的苹果， 比如超过150克，需要重写接口如下:
```java
public static List<Apple> filterHeavyApples(List<Apple> inventory){
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory){
        if (apple.getWeight() > 150) { // 这里高亮显示的代码会仅仅选出重的苹果
            result.add(apple);
        }
    } 
    return result;
}
```

根据前面提过的方法， Java 8会把条件代码作为参数传递进去， 这样可以避免filter方法出现重复的代码。 现在你可以写：
``` java
public static boolean isGreenApple(Apple apple) {
    return "green".equals(apple.getColor());
} 
public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
} 
public interface Predicate<T>{ // 写出来是为了清晰（ 平常只要从java.util.function导入就可以了）
    boolean test(T t);
} 
static List<Apple> filterApples(List<Apple> inventory,Predicate<Apple> p) { // 方法作为Predicate参数p传递进去（ 见附注栏“什么是谓词？ ”）
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory){
        if (p.test(apple)) { // 苹果符合p所代表的条件吗
        result.add(apple);
        }
    }
    return result;
}

// 然后可以这样实现
filterApples(inventory, Apple::isGreenApple);
// 或者
filterApples(inventory, Apple::isHeavyApple);
```

### 2.3 从传递方法到Lambda
把方法作为值来传递显然很有用， 但要是为类似于isHeavyApple和isGreenApple这种可能只用一两次的短方法写一堆定义有点儿烦人。 不过Java 8也解决了这个问题， 它引入了一套新记法（ 匿名函数或Lambda） ， 让你可以写:
``` java
filterApples(inventory, (Apple a) -> "green".equals(a.getColor()));
// 或者
filterApples(inventory, (Apple a) -> a.getWeight() > 150 );
// 甚至
filterApples(inventory, (Apple a) -> a.getWeight() < 80 ||"brown".equals(a.getColor()) );
```
## 3.流
几乎每个Java应用都会制造和处理集合。 但集合用起来并不总是那么理想。 比方说， 你需要从一个列表中筛选金额较高的交易， 然后按货币分组。 你需要写一大堆套路化的代码来实现这个数据处理命令， 如下所示：

```java
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>(); // 建立累积交易分组的Map

for (Transaction transaction : transactions) { // 遍历交易的List
    if(transaction.getPrice() > 1000){ // 筛选金额较高的交易
        Currency currency = transaction.getCurrency(); // 提取交易货币
        List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
        if (transactionsForCurrency == null) { // 如果这个货币的分组Map是空的， 那就建立一个
            transactionsForCurrency = new ArrayList<>();
            transactionsByCurrencies.put(currency,
            transactionsForCurrency);
        } 
        transactionsForCurrency.add(transaction); // 将当前遍历的交易添加到具有同一货币的交易List中
    }
}
```
有了Stream API， 你现在可以更加简洁的解决问题
``` java
import static java.util.stream.Collectors.toList;
Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream().filter((Transaction t) -> t.getPrice() > 1000).collect(groupingBy(Transaction::getCurrency));
    // filter 筛选金额较高的交易
    // collect 按货币分组
```
Java 8也用Stream API（ java.util.stream） 解决了这两个问题： 集合处理时的套路和晦涩， 以及难以利用多核。这样设计的第一个原因是， 有许多反复出现的数据处理模式， 类似于前一节所说的filterApples或SQL等数据库查询语言里熟悉的操作， 如果在库中有这些就会很方便： 根据标准筛选数据（ 比如较重的苹果） ， 提取数据（ 例如抽取列表中每个苹果的重量字段） ， 或给数据分组（ 例如， 将一个数字列表分组， 奇数和偶数分别列表） 等。 第二个原因是， 这类操作常常可以并行化。 例如， 如图1-6所示， 在两个CPU上筛选列表， 可以让一个CPU处理列表的前一半， 第二个CPU处理后一半， 这称为分支步骤(1)。 CPU随后对各自的半个列表做筛选(2)。 最后(3)， 一个CPU会把两个结果合并起来（ Google搜索这么快就与此紧密相关， 当然他们用的CPU远远不止两个了） 。

我们只是说新的Stream API和Java现有的集合API的行为差不多： 它们都能够访问数据项目的序列。 不过， 现在最好记得， Collection主要是为了存储和访问数据， 而Stream则主要用于描述对数据的计算。 这里的关键点在于， Stream允许并提倡并行处理一个Stream中的元素。 虽然可能乍看上去有点儿怪， 但筛选一个Collection（ 将上一节的filterApples应用在一个List上） 的最快方法常常是将其转换为Stream， 进行并行处理， 然后再转换回List， 下面举的串行和并行的例子都是如此。 我们这里还只是说“几乎免费的并行” ， 让你稍微体验一下， 如何利用Stream和Lambda表达式顺序或并行地从一个列表里筛选比较重的苹果。
**顺序处理如下:**
```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples = inventory.stream().filter((Apple a) -> a.getWeight() > 150).collect(toList());
```
![将filter分支到两个CPU上并聚合结果](picture/Java/Java8/pic1-6.jpg)
![将filter分支到两个CPU上并聚合结果](https://github.com/dxjeric/dxjeric.github.io/blob/master/picture/Java/Java8/pic1-6.jpg)

**并行处理如下:**
```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples = inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150).collect(toList());
```

## 4. 默认方法
Java 8中加入默认方法主要是为了支持库设计师， 让他们能够写出更容易改进的接口。 这一点会在第9章中详谈。 

## 5. 来自函数式编程的其他好思想
Java中从函数式编程中引入的两个核心思想： 将方法和Lambda作为一等值， 以及在没有可变共享状态时， 函数或方法可以有效、 安全地并行执行。 前面说到的新的Stream API把这两种思想都用到了

在Java 8里有一个Optional<T>类， 如果你能一致地使用它的话， 就可以帮助你避免出现NullPointer异常。 它是一个容器对象， 可以包含， 也可以不包含一个值。 Optional<T>中有方法来明确处理值不存在的情况， 这样就可以避免NullPointer异常了。 换句话说， 它使用类型系统， 允许你表明我们知道一个变量可能会没有值。 我们会在第10章中详细讨论Optional<T>。

第二个想法是（ 结构）模式匹配 。 
> 这个术语有两个意思， 这里我们指的是数学和函数式编程上所用的， 即函数是分情况定义的， 而不是使用ifthen-else。 它的另一个意思类似于“在给定目录中找到所有类似于IMG*.JPG形式的文件” ， 和所谓的正则表达式有关。

**Java 8对模式匹配的支持并不完全， 虽然我们会在[第14章]()中介绍如何对其进行表达。**
与此同时， 我们会用一个以Scala语言（ 另一个使用JVM的类Java语言， 启发了Java在一些方面的发展； 请参阅[第15章]()） 表达的例子加以描述。 

## 6. 总结
* 请记住语言生态系统的思想， 以及语言面临的“要么改变， 要么衰亡” 的压力。 虽然Java可能现在非常有活力， 但你可以回忆一下其他曾经也有活力但未能及时改进的语言的命运， 如COBOL。
* Java 8中新增的核心内容提供了令人激动的新概念和功能， 方便我们编写既有效又简洁的程序。
* 现有的Java编程实践并不能很好地利用多核处理器。
* 函数是一等值； 记得方法如何作为函数式值来传递， 还有Lambda是怎样写的。
* Java 8中Streams的概念使得Collections的许多方面得以推广， 让代码更为易读， 并允许并行处理流元素。
* 你可以在接口中使用默认方法， 在实现类没有实现方法时提供方法内容。
* 其他来自函数式编程的有趣思想， 包括处理null和使用模式匹配。