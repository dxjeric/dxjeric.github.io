---
sort: 6
---
# 第六章 用流收集数据
你可以把
Java 8的流看作花哨又懒惰的数据集迭代器。 它们支持两种类型的操作： 中间操作（ 如filter或map） 和终端操作（ 如count、 findFirst、 forEach和reduce） 。 中间操作可以链接起来， 将一个流转换为另一个流。 这些操作不会消耗流， 其目的是建立一个流水线。 与此相反， 终端操作会消耗流， 以产生一个最终结果， 例如返回流中的最大元素。 它们通常可以通过优化流水线来缩短计算时间。

你可以把Java 8的流看作花哨又懒惰的数据集迭代器。 它们支持两种类型的操作： 中间操作（ 如filter或map） 和终端操作（ 如count、 findFirst、 forEach和reduce） 。 中间操作可以链接起来， 将一个流转换为另一个流。 这些操作不会消耗流， 其目的是建立一个流水线。 与此相反， 终端操作会消耗流， 以产生一个最终结果， 例如返回流中的最大元素。 它们通常可以通过优化流水线来缩短计算时间。

## 1. 收集器简介

### 1.1 收集器用作高级归约

```java
// 使用几个分组货物
Map<Currency, List<Transaction>> transactionsByCurrencies =
transactions.stream().collect(groupingBy(Transaction::getCurrency));
```
图6-1所示的归约操作所做的工作和代码清单6-1中的指令式代码一样。 它遍历流中的每个元素， 并让Collector进行处理。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-1.png)

**图 6-1 按货币对交易分组的归约过程**

一般来说， Collector会对元素应用一个转换函数（ 很多时候是不体现任何效果的恒等转换， 例如toList） ， 并将结果累积在一个数据结构中， 从而产生这一过程的最终输出。 

### 1.2 预定义收集器
在本章剩下的部分中， 我们主要探讨预定义收集器的功能， 也就是那些可以从Collectors类提供的工厂方法（ 例如groupingBy） 创建的收集器。 它们主要提供了三大功能：
* 将流元素归约和汇总为一个值
* 元素分组
* 元素分区

## 2. 归约和汇总
在需要将流项目重组成集合时， 一般会使用收集器（ Stream方法collect的参数） 。 再宽泛一点来说， 但凡要把流中所有的项目合并成一个结果时就可以用。 这个结果可以是任何类型， 可以复杂如代表一棵树的多级映射， 或是简单如一个整数——也许代表了菜单的热量总和。 

我们先来举一个简单的例子， 利用counting工厂方法返回的收集器， 数一数菜单里有多少种菜：
```java
long howManyDishes = menu.stream().collect(Collectors.counting());
// 这还可以写得更为直接
long howManyDishes = menu.stream().count();
```

### 2.1 查找流中的最大值和最小值
假设你想要找出菜单中热量最高的菜。 你可以使用两个收集器， Collectors.maxBy和Collectors.minBy， 来计算流中的最大或最小值。 这两个收集器接收一个Comparator参数来比较流中的元素。 你可以创建一个Comparator来根据所含热量对菜肴进行比较， 并把它传递给Collectors.maxBy：
```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

### 2.2 汇总
Collectors类专门为汇总提供了一个工厂方法： Collectors.summingInt。 它可接受一个把对象映射为求和所需int的函数， 并返回一个收集器； 该收集器在传递给普通的collect方法后即执行我们需要的汇总操作。 举个例子来说， 你可以这样求出菜单列表的总热量：
```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
这里的收集过程如图6-2所示。 在遍历流时， 会把每一道菜都映射为其热量， 然后把这个数字累加到一个累加器（ 这里的初始值0） 。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-2.png)

**图 6-2 summingInt收集器的累积过程**

但汇总不仅仅是求和； 还有Collectors.averagingInt， 连同对应的averagingLong和averagingDouble可以计算数值的平均数：
```java
double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
```

例如， 通过一次summarizing操作你可以就数出菜单中元素的个数， 并得到菜肴热量总和、 平均值、 最大值和最小值:
```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
// 输出结果为
IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}
```

### 2.3 连接字符串

joining工厂方法返回的收集器会把对流中每一个对象应用toString方法得到的所有字符串连接成一个字符串。 这意味着你把菜单中所有菜肴的名称连接起来， 如下所示：

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
```

joining在内部使用了StringBuilder来把生成的字符串逐个追加起来。此外还要注意， 如果Dish类有一个toString方法来返回菜肴的名称， 那你无需用提取每一道菜名称的函数来对原流做映射就能够得到相同的结果：

```java
String shortMenu = menu.stream().collect(joining());

// 输出结果为 porkbeefchickenfrench friesriceseason fruitpizzaprawnssalmon
```

但该字符串的可读性并不好。 幸好， joining工厂方法有一个重载版本可以接受元素之间的分界符， 这样你就可以得到一个逗号分隔的菜肴名称列表：
```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));

// 输出结果为 pork, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon
```

### 2.4 广义的归约汇总
事实上， 我们已经讨论的所有收集器， 都是一个可以用reducing工厂方法定义的归约过程的特殊情况而已。 Collectors.reducing工厂方法是所有这些特殊情况的一般化。 可以说， 先前讨论的案例仅仅是为了方便程序员而已。 （ 但是， 请记得方便程序员和可读性是头等大事！ ） 例如， 可以用reducing方法创建的收集器来计算你菜单的总热量， 如下所示：
```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```

**它需要三个参数。**

* 第一个参数是归约操作的起始值， 也是流中没有元素时的返回值， 所以很显然对于数值和而言0是一个合适的值。
* 第二个参数就是你在6.2.2节中使用的函数， 将菜肴转换成一个表示其所含热量的int。
* 第三个参数是一个BinaryOperator， 将两个项目累积成一个同类型的值。 这里它就是对两个int求和。

同样， 你可以使用下面这样单参数形式的reducing来找到热量最高的菜， 如下所示：
```java
Optional<Dish> mostCalorieDish = menu.stream().collect(reducing(
(d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

你可以把单参数reducing工厂方法创建的收集器看作三参数方法的特殊情况， 它把流中的第一个项目作为起点， 把恒等函数（ 即一个函数仅仅是返回其输入参数） 作为一个转换函数。 这也意味着， 要是把单参数reducing收集器传递给空流的collect方法， 收集器就没有起点； 正如我们在6.2.1节中所解释的， 它将因此而返回一个```Optional<Dish>```对象。

**收集与归约**
在上一章和本章中讨论了很多有关归约的内容。 你可能想知道， Stream接口的collect和reduce方法有何不同， 因为两种方法通常会获得相同的结果。 例如， 你可以像下面这样使用reduce方法来实现toListCollector所做的工作：
```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
List<Integer> numbers = stream.reduce(
        new ArrayList<Integer>(),
        (List<Integer> l, Integer e) -> {
            l.add(e);
            return l;
        },
        (List<Integer> l1, List<Integer> l2) -> {
            l1.addAll(l2);
            return l1;
        });
```
**这个解决方案有两个问题： 一个语义问题和一个实际问题。 语义问题在于， reduce方法旨在把两个值结合起来生成一个新值， 它是一个不可变的归约。 与此相反， collect方法的设计就是要改变容器， 从而累积要输出的结果。** 这意味着， 上面的代码片段是在滥用reduce方法， 因为它在原地改变了作为累加器的List。 你在下一章中会更详细地看到， 以错误的语义使用reduce方法还会造成一个实际问题： 这个归约过程不能并行工作， 因为由多个线程并发修改同一个数据结构可能会破坏List本身。 在这种情况下， 如果你想要线程安全， 就需要每次分配一个新的List， 而对象分配又会影响性能。 这就是collect方法特别适合表达可变容器上的归约的原因， 更关键的是它适合并行操作， 本章后面会谈到这一点。

1. 收集框架的灵活性： 以不同的方法执行同样的操作
你还可以进一步简化前面使用reducing收集器的求和例子——引用Integer类的sum方法， 而不用去写一个表达同一操作的Lambda表达式。 这会得到以下程序：
```java
int totalCalories = menu.stream().collect(reducing(0, // 初始值
    Dish::getCalories, // 转换函数
    Integer::sum)); // 累积函数
```
从逻辑上说， 归约操作的工作原理如图6-3所示： 利用累积函数， 把一个初始化为起始值的累加器， 和把转换函数应用到流中每个元素上得到的结果不断迭代合并起来。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-3.png)

**图 6-3 计算菜单总热量的归约过程**

我们在第5章已经注意到， 还有另一种方法不使用收集器也能执行相同操作——将菜肴流映射为每一道菜的热量， 然后用前一个版本中使用的方法引用来归约得到的流:
```java
int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get();
```
请注意， 就像流的任何单参数reduce操作一样， reduce(Integer::sum)返回的不是int而是```Optional<Integer>```， 以便在空流的情况下安全地执行归约操作。 然后你只需用Optional对象中的get方法来提取里面的值就行了。 请注意， 在这种情况下使用get方法是安全的， 只是因为你已经确定菜肴流不为空。 
你在[第10章](https://dxjeric.github.io/book/Java/Java8/chapter-10.html)还会进一步了解到， 一般来说， 使用允许提供默认值的方法， 如orElse或orElseGet来解开Optional中包含的值更为安全。 最后， 更简洁的方法是把流映射到一个IntStream， 然后调用sum方法， 你也可以得到相同的结果：
```java
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

2. 根据情况选择最佳解决方案
这再次说明了， 函数式编程（ 特别是Java 8的Collections框架中加入的基于函数式风格原理设计的新API） 通常提供了多种方法来执行同一个操作。 这个例子还说明， 收集器在某种程度上比Stream接口上直接提供的方法用起来更复杂， 但好处在于它们能提供更高水平的抽象和概括， 也更容易重用和自定义。
我们的建议是， 尽可能为手头的问题探索不同的解决方案， 但在通用的方案里面，始终选择最专门化的一个。 无论是从可读性还是性能上看， 这一般都是最好的决定。 

例如， 要计菜单的总热量， 我们更倾向于最后一个解决方案（ 使用IntStream） ， 因为它最简明， 也很可能最易读。 同时， 它也是性能最好的一个， 因为IntStream可以让我们避免自动拆箱操作， 也就是从Integer到int的隐式转换， 它在这里毫无用处。
请看看测验6.1， 测试一下你对于reducing作为其他收集器的概括的理解程度如何。

测验6.1： 用reducing连接字符串
```java
// 以下哪一种reducing收集器的用法能够合法地替代joining收集器
String shortMenu = menu.stream().map(Dish::getName).collect(joining());

// (1)
String shortMenu = menu.stream().map(Dish::getName).collect( reducing ( (s1, s2) -> s1 + s2 ) ).get();

// (2)
String shortMenu = menu.stream().collect( reducing( (d1, d2) -> d1.getName() + d2.getName() ) ).get();

// (3)
String shortMenu = menu.stream().collect( reducing( "",Dish::getName, (s1, s2) -> s1 + s2 ) );
```
答案： 语句1和语句3是有效的， 语句2无法编译
(1) 这会将每道菜转换为菜名， 就像原先使用joining收集器的语句一样。 然后用一个String作为累加器归约得到的字符串流， 并将菜名逐个连接在它后面。
(2) 这无法编译， 因为reducing接受的参数是一个```BinaryOperator<t>```， 也就是一个```BiFunction<T,T,T>```。 这就意味着它需要的函数必须能接受两个参数， 然后返回一个相同类型的值， 但这里用的Lambda表达式接受的参数是两个菜， 返回的却是一个字符串。
(3) 这就意味着它需要的函数必须能接受两个参数， 然后返回一个相同类型的值， 但这里用的Lambda表达式接受的参数是两个菜， 返回的却是一个字符串。

## 3. 分组
