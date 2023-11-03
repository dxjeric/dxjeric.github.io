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
一个常见的数据库操作是根据一个或多个属性对集合中的项目进行分组。 就像前面讲到按货币对交易进行分组的例子一样， 如果用指令式风格来实现的话， 这个操作可能会很麻烦、 啰嗦而且容易出错。 但是， 如果用Java 8所推崇的函数式风格来重写的话， 就很容易转化为一个非常容易看懂的语句。 我们来看看这个功能的第二个例子： 假设你要把菜单中的菜按照类型进行分类， 有肉的放一组， 有鱼的放一组，其他的都放另一组。 用Collectors.groupingBy工厂方法返回的收集器就可以轻松地完成这项任务， 如下所示：
```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
// 其结果是下面的Map
// {FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken]}
```

如图6-4所示， 分组操作的结果是一个Map， 把分组函数返回的值作为映射的键， 把流中所有具有这个分类值的项目的列表作为对应的映射值。 在菜单分类的例子中， 键就是菜的类型， 值就是包含所有对应类型的菜肴的列表。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-4.png)

**图 6-4 在分组过程中对流中的项目进行分类**

但是， 分类函数不一定像方法引用那样可用， 因为你想用以分类的条件可能比简单的属性访问器要复杂。 例如， 你可能想把热量不到400卡路里的菜划分为“低热量” （ diet） ， 热量400到700卡路里的菜划为“普通” （ normal） ， 高于700卡路里的划为“高热量” （ fat） 。 由于Dish类的作者没有把这个操作写成一个方法，你无法使用方法引用， 但你可以把这个逻辑写成Lambda表达式：
```java
public enum CaloricLevel {
    DIET, NORMAL, FAT
}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(groupingBy(dish -> {
    if (dish.getCalories() <= 400)
        return CaloricLevel.DIET;
    else if (dish.getCalories() <= 700)
        return CaloricLevel.NORMAL;
    else
        return CaloricLevel.FAT;
}))
```

### 3.1 多级分组
要实现多级分组， 我们可以使用一个由双参数版本的Collectors.groupingBy工厂方法创建的收集器， 它除了普通的分类函数之外， 还可以接受collector类型的第二个参数。 
那么要进行二级分组的话， 我们可以把一个内层groupingBy传递给外层groupingBy， 并定义一个为流中项目分类的二级标准， 如代码清单6-2所示。

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
        groupingBy(Dish::getType, // 一级分类函数
                groupingBy(dish -> { // 二级分类函数
                    if (dish.getCalories() <= 400)
                        return CaloricLevel.DIET;
                    else if (dish.getCalories() <= 700)
                        return CaloricLevel.NORMAL;
                    else
                        return CaloricLevel.FAT;
                })));
// 这个二级分组的结果就是像下面这样的两级Map：
/*
{
    MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]},
    FISH={DIET=[prawns], NORMAL=[salmon]},
    OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}
}
*/
```

图6-5显示了为什么结构相当于 n 维表格， 并强调了分组操作的分类目的。
一般来说， 把groupingBy看作“桶” 比较容易明白。 第一个groupingBy给每个键建立了一个桶。 然后再用下游的收集器去收集每个桶中的元素， 以此得到 n 级分组。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-5.png)

**图 6-5 n 层嵌套映射和 n 维分类表之间的等价关系**

### 3.2 按子组收集数据
还要注意， 普通的单参数groupingBy(f)（ 其中f是分类函数） 实际上是groupingBy(f, toList())的简便写法。
```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));

// 其结果是下面的Map：
// {MEAT=3, FISH=2, OTHER=4}
```

普通的单参数groupingBy(f)（ 其中f是分类函数） 实际上是groupingBy(f, toList())的简便写法。

再举一个例子， 你可以把前面用于查找菜单中热量最高的菜肴的收集器改一改， 按照菜的类型分类：
```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = menu.stream().collect(groupingBy(Dish::getType, maxBy(comparingInt(Dish::getCalories))));

// 这个分组的结果显然是一个map， 以Dish的类型作为键， 以包装了该类型中热量最高的Dish的Optional<Dish>作为值：
// {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```

这个Map中的值是Optional， 因为这是maxBy工厂方法生成的收集器的类型， 但实际上， 如果菜单中没有某一类型的Dish， 这个类型就不会对应一个Optional. empty()值， 而且根本不会出现在Map的键中。 groupingBy收集器只有在应用分组条件后， 第一次在流中找到某个键对应的元素时才会把键加入分组Map中。 这意味着Optional包装器在这里不是很有用， 因为它不会仅仅因为它是归约收集器的返回类型而表达一个最终可能不存在却意外存在的值。

1. 把收集器的结果转换为另一种类型
因为分组操作的Map结果中的每个值上包装的Optional没什么用， 所以你可能想要把它们去掉。 要做到这一点， 或者更一般地来说， 把收集器返回的结果转换为另一种类型， 你可以使用Collectors.collectingAndThen工厂方法返回的收集器， 如下所示。

代码清单6-3 查找每个子组中热量最高的Dish
```java
Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
    .collect(groupingBy(Dish::getType, // 分类函数
            collectingAndThen(
                    maxBy(comparingInt(Dish::getCalories)), // 包装后的收集器
                    Optional::get))); // 转换函数

// 输出结果为
// {FISH=salmon, OTHER=pizza, MEAT=pork}
```

这个工厂方法接受两个参数——要转换的收集器以及转换函数， 并返回另一个收集器。 这个收集器相当于旧收集器的一个包装， collect操作的最后一步就是将返回值用转换函数做一个映射。 在这里， 被包起来的收集器就是用maxBy建立的那个， 而转换函数Optional::get则把返回的Optional中的值提取出来。 前面已经说过， 这个操作放在这里是安全的， 因为reducing收集器永远都不会返回Optional.empty()。 其结果是下面的Map：

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-6.png)

**图 6-6 嵌套收集器来获得多重效果**

图6-6可以直观地展示它们是怎么工作的。 从最外层开始逐层向里， 注意以下几点。
* 收集器用虚线表示， 因此groupingBy是最外层， 根据菜肴的类型把菜单流分组， 得到三个子流。
* groupingBy收集器包裹着collectingAndThen收集器， 因此分组操作得到的每个子流都用这第二个收集器做进一步归约。
* collectingAndThen收集器又包裹着第三个收集器maxBy。
* 随后由归约收集器进行子流的归约操作， 然后包含它的collectingAndThen收集器会对其结果应用Optional:get转换函数。
* 对三个子流分别执行这一过程并转换而得到的三个值， 也就是各个类型中热量最高的Dish， 将成为groupingBy收集器返回的Map中与各个分类键（ Dish的类型） 相关联的值。

2. 与groupingBy联合使用的其他收集器的例子

通过groupingBy工厂方法的第二个参数传递的收集器将会对分到同一组中的所有流元素执行进一步归约操作。 例如， 你还重用求出所有菜肴热量总和的收集器， 不过这次是对每一组Dish求和：
```java
Map<Dish.Type, Integer> totalCaloriesByType = menu.stream().collect(groupingBy(Dish::getType, summingInt(Dish::getCalories)));
```

## 4. 分区
分区是分组的特殊情况： 由一个Predicate（ 返回一个布尔值的函数） 作为分类函数， 它称分区函数。 分区是分组的特殊情况： 由一个Predicate（ 返回一个布尔值的函数） 作为分类函数， 它称分区函数。 
``` java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian)); // 分区函数
```
### 4.1 分区的优势
分区的好处在于保留了分区函数返回true或false的两套流元素列表。 在上一个例子中， 要得到非素食Dish的List， 你可以使用两个筛选操作来访问partitionedMenu这个Map中false键的值： 一个利用Predicate， 一个利用该Predicate的非。
```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream().collect(
        partitioningBy(Dish::isVegetarian, // 分区函数
                groupingBy(Dish::getType))); // 第二个收集器

// 输出结果
// {false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, true={OTHER=[french fries, rice, season fruit, pizza]}}
```

groupingBy和partitioningBy收集器之间的相似之处并不止于此； 你在下一个测验中会看到， 还可以按照和6.3.1节中分组类似的方式进行多级分区。

**测验6.2： 使用partitioningBy**
和groupingBy收集器类似， partitioningBy收集器也可以结合其他收集器使用。 尤其是它可以与第二个partitioningBy收集器一起使用来实现多级分区。 
```java
// 1
menu.stream().collect(partitioningBy(Dish::isVegetarian, partitioningBy (d -> d.getCalories() > 500)));
// 输出: { false={false=[chicken, prawns, salmon], true=[pork, beef]}, true={false=[rice, season fruit], true=[french fries, pizza]}}

// 2
menu.stream().collect(partitioningBy(Dish::isVegetarian, partitioningBy (Dish::getType)));
// 这无法编译， 因为partitioningBy需要一个Predicate， 也就是返回一个布尔值的函数。 方法引用Dish::getType不能用作Predicate。

// 3
menu.stream().collect(partitioningBy(Dish::isVegetarian, counting()));
// 输出: {false=5, true=4}

```

### 4.2 将数字按质数和非质数分区
假设你要写一个方法， 它接受参数int n， 并将前 n 个自然数分为质数和非质数。但首先， 找出能够测试某一个待测数字是否是质数的Predicate会很有帮助：

```java
public boolean isPrime(int candidate) {return IntStream.range(2, candidate) // 产生一个自然数范围， 从2开始， 直至但不包括待测数
    .noneMatch(i -> candidate % i == 0); // 如果待测数字不能被流中任何数字整除则返回true
    }
```
一个简单的优化是仅测试小于等于待测数平方根的因子：
```java
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
            .noneMatch(i -> candidate % i == 0);
}
```

为了把前n个数字分为质数和非质数， 只要创建一个包含这n个数的流， 用刚刚写的isPrime方法作为Predicate， 再给partitioningBy收集器归约就好了：
```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
            .collect(partitioningBy(candidate -> isPrime(candidate)));
}
```

我们已经讨论过了Collectors类的静态工厂方法能够创建的所有收集器， 并介绍了使用它们的实际例子。 表6-1将它们汇总到一起， 给出了它们应用到```Stream<T>```上返回的类型， 以及它们用于一个叫作menuStream的```Stream<Dish>```上的实际例子。
**表6-1 Collectors类的静态工厂方法**

| 工厂方法          | 返回类型                 | 用于                                                         | 使用示例：                                                   |
| ----------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| toList            | List\<T\>                | 把流中所有项目收集到一个List                                 | List\<Dish\>  dishes = menuStream.collect(toList());         |
| toSet             | Set\<T\>                 | 把流中所有项目收集到一个Set， 删除重复项                     | Set\<Dish\>  dishes = menuStream.collect(toSet());           |
| toCollection      | Collection\<T\>          | 把流中所有项目收集到给定的供应源创建的集合                   | Collection\<Dish\>  dishes = menuStream.collect(toCollection(),     ArrayList::new); |
| counting          | Long                     | 计算流中元素的个数                                           | long  howManyDishes = menuStream.collect(counting());        |
| summingInt        | Integer                  | 对流中项目的一个整数属性求和                                 | int  totalCalories = menuStream.collect(summingInt(Dish::getCalories)); |
| averagingInt      | Double                   | 计算流中项目Integer属性的平均值                              | double  avgCalories = menuStream.collect(averagingInt(Dish::getCalories)); |
| summarizingInt    | IntSummaryStatistics     | 收集关于流中项目Integer属性的统计值， 例如最大、 最小、 总和 | IntSummaryStatistics  menuStatistics = menuStream.collect(summarizingInt(Dish::getCalories)); |
| joining\`         | String                   | 连接对流中每个项目调用toString方法所生成的字符串             | String  shortMenu = menuStream.map(Dish::getName).collect(joining(", ")); |
| maxBy             | Optional\<T\>            | 一个包裹了流中按照给定比较器选出的最大元素的Optional， 或如果流为空则为Optional.empty() | Optional\<Dish\>  fattest = menuStream.collect(maxBy(comparingInt(Dish::getCalories))); |
| minBy             | Optional\<T\>            | 一个包裹了流中按照给定比较器选出的最小元素的Optional， 或如果流为空则为Optional.empty() | Optional\<Dish\>  lightest = menuStream.collect(minBy(comparingInt(Dish::getCalories))); |
| reducing          | 归约操作产生的类型       | 从一个作为累加器的初始值开始， 利用BinaryOperator与流中的元素逐个结合， 从而将流归约为单个值 | int  totalCalories = menuStream.collect(reducing(0, Dish::getCalories,  Integer::sum)); |
| collectingAndThen | 转换函数返回的类型       | 包裹另一个收集器，  对其结果应用转换函数                     | int  howManyDishes = menuStream.collect(collectingAndThen(toList(), List::size)); |
| groupingBy        | Map\<K,  List\<T\>\>     | 根据项目的一个属性的值对流中的项目作问组， 并将属性值作为结果`Map`的键 | Map\<Dish.Type,List\<Dish\>\>  dishesByType = menuStream.collect(groupingBy(Dish::getType)); |
| partitioningBy    | Map\<Boolean,List\<T\>\> | 根据对流中每个项目应用Predicate的结果来对项目进行分区             | Map\<Boolean,List\<Dish\>\>  vegetarianDishes = menuStream.collect(partitioningBy(Dish::isVegetarian)); |

## 5. 收集器接口
Collector接口包含了一系列方法， 为实现具体的归约操作（ 即收集器） 提供了范本。 我们已经看过了Collector接口中实现的许多收集器， 例如toList或groupingBy。 这也意味着， 你可以为Collector接口提供自己的实现， 从而自由地创建自定义归约操作。 

Collector接口的定义， 它列出了接口的签名以及声明的五个方法。
**代码清单6-4 Collector接口**
```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();

    BiConsumer<A, T> accumulator();

    Function<A, R> finisher();

    BinaryOperator<A> combiner();

    Set<Characteristics> characteristics();
}
```

本列表适用以下定义:
* T是流中要收集的项目的泛型。
* A是累加器的类型， 累加器是在收集过程中用于累积部分结果的对象。
* R是收集操作得到的对象（ 通常但并不一定是集合） 的类型。

例如， 你可以实现一个```ToListCollector<T>```类， 将```Stream<T>```中的所有元素收集到一个```List<T>```里， 它的签名如下：
```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
```

### 5.1 理解Collector接口声明的方法

你会注意到， 前四个方法都会返回一个会被collect方法调用的函数， 而第五个方法characteristics则提供了一系列特征， 也就是一个提示列表， 告诉collect方法在执行归约操作的时候可以应用哪些优化（ 比如并行化） 。

**1. 建立新的结果容器： supplier方法**
supplier方法必须返回一个结果为空的Supplier， 也就是一个无参数函数， 在调用时它会创建一个空的累加器实例， 供数据收集过程使用。 

很明显， 对于将累加器本身作为结果返回的收集器， 比如我们的ToListCollector， 在对空流执行操作的时候， 这个空的累加器也代表了收集过程的结果。 在我们的ToListCollector中， supplier返回一个空的List， 如下所示：
```java
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}

// 注意也可以只传递一个构造函数引用
public Supplier<List<T>> supplier() {
    return ArrayList::new;
}
```

**2. 将元素添加到结果容器： accumulator方法**
accumulator方法会返回执行归约操作的函数。 当遍历到流中第 n 个元素时， 这个函数执行时会有两个参数： 保存归约结果的累加器（ 已收集了流中的前 n-1 个项目） ， 还有第 n 个元素本身。 该函数将返回void， 因为累加器是原位更新， 即函数的执行改变了它的内部状态以体现遍历的元素的效果。 

对于ToListCollector，这个函数仅仅会把当前项目添加至已经遍历过的项目的列表：
```java
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}

// 也可以使用方法引用， 这会更为简洁：
public BiConsumer<List<T>, T> accumulator() {
    return List::add;
}
```

**3. 对结果容器应用最终转换： finisher方法**
在遍历完流后， finisher方法必须返回在累积过程的最后要调用的一个函数， 以便将累加器对象转换为整个集合操作的最终结果。 

通常， 就像ToListCollector的情况一样， 累加器对象恰好符合预期的最终结果， 因此无需进行转换。 所以finisher方法只需返回identity函数：
```java
public Function<List<T>, List<T>> finisher() {
    return Function.identity();
}
```

这三个方法已经足以对流进行顺序归约， 至少从逻辑上看可以按图6-7进行。 实践中的实现细节可能还要复杂一点， 一方面是因为流的延迟性质， 可能在collect操作之前还需要完成其他中间操作的流水线， 另一方面则是理论上可能要进行并行归约。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-7.png)

**图 6-7 顺序归约过程的逻辑步骤**

**4. 合并两个结果容器： combiner方法**
四个方法中的最后一个——combiner方法会返回一个供归约操作使用的函数， 它定义了对流的各个子部分进行并行处理时， 各个子部分归约所得的累加器要如何合并。 

对于toList而言， 这个方法的实现非常简单， 只要把从流的第二个部分收集到的项目列表加到遍历第一部分时得到的列表后面就行了：
```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1; 
    };
}
```

有了这第四个方法， 就可以对流进行并行归约了。 它会用到Java 7中引入的分支/合并框架和Spliterator抽象， 我们会在下一章中讲到。 这个过程类似于图6-8所示， 这里会详细介绍。
* 原始流会以递归方式拆分为子流， 直到定义流是否需要进一步拆分的一个条件为非（ 如果分布式工作单位太小， 并行计算往往比顺序计算要慢， 而且要是生成的并行任务比处理器内核数多很多的话就毫无意义了） 。
* 现在， 所有的子流都可以并行处理， 即对每个子流应用图6-7所示的顺序归约算法。
* 最后， 使用收集器combiner方法返回的函数， 将所有的部分结果两两合并。 这时会把原始流每次拆分时得到的子流对应的结果合并起来。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic6-8.png)

**图 6-8 使用combiner方法来并行化归约过程**

**5. characteristics方法**
最后一个方法——characteristics会返回一个不可变的Characteristics集合，它定义了收集器的行为——尤其是关于流是否可以并行归约， 以及可以使用哪些优化的提示。 Characteristics是一个包含三个项目的枚举。

* UNORDERED —— 归约结果不受流中项目的遍历和累积顺序的影响。
* CONCURRENT —— accumulator函数可以从多个线程同时调用， 且该收集器可以并行归约流。 如果收集器没有标为UNORDERED， 那它仅在用于无序数据源时才可以并行归约。
* IDENTITY_FINISH —— 这表明完成器方法返回的函数是一个恒等函数， 可以跳过。 这种情况下， 累加器对象将会直接用作归约过程的最终结果。 这也意味着， 将累加器A不加检查地转换为结果R是安全的。

迄今开发的ToListCollector是IDENTITY_FINISH的， 因为用来累积流中元素的List已经是我们要的最终结果， 用不着进一步转换了， 但它并不是UNORDERED，因为用在有序流上的时候， 我们还是希望顺序能够保留在得到的List中。 最后， 它是CONCURRENT的， 但我们刚才说过了， 仅仅在背后的数据源无序时才会并行处理

### 5.2 全部融合到一起
前一小节中谈到的五个方法足够我们开发自己的ToListCollector了。 你可以把它们都融合起来， 如下面的代码清单所示。

**代码清单6-5 ToListCollector**
```java

import java.util.*;
import java.util.function.*;
import java.util.stream.Collector;
import static java.util.stream.Collector.Characteristics.*;

public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new; // 创建集合操作的起始点
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return List::add; // 累积遍历过的项目， 原位修改累加器
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.indentity(); // 恒等函数
    }

    @Override
    public BinaryOperator<List<T>> combiner() {
        return (list1, list2) -> {
            list1.addAll(list2); // 修改第一个累加器， 将其与第二个累加器的内容合并
            return list1; // 返回修改后的第一个累加器
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(
                IDENTITY_FINISH, CONCURRENT)); // 为收集器添加IDENTITY_FINISH和CONCURRENT标志
    }
}
```

请注意， 这个实现与Collectors.toList方法并不完全相同， 但区别仅仅是一些小的优化。 这些优化的一个主要方面是Java API所提供的收集器在需要返回空列表时使用了Collections.emptyList()这个单例（ singleton） 。 这意味着它可安全地替代原生Java， 来收集菜单流中的所有Dish的列表：
```java
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());
// 这个实现和标准的
List<Dish> dishes = menuStream.collect(toList());
```
构造之间的其他差异在于toList是一个工厂， 而ToListCollector必须用new来实例化。

**进行自定义收集而不去实现Collector**
对于IDENTITY_FINISH的收集操作， 还有一种方法可以得到同样的结果而无需从头实现新的Collectors接口。 Stream有一个重载的collect方法可以接受另外三个函数——supplier、 accumulator和combiner， 其语义和Collector接口的相应方法返回的函数完全相同。 所以比如说， 我们可以像下面这样把菜肴流中的项目收集到一个List中：
```java
List<Dish> dishes = menuStream.collect(
    ArrayList::new, // 供应源
    List::add, // 累加器
    List::addAll); // 组合器
```
这第二种形式虽然比前一个写法更为紧凑和简洁， 却不那么易读。 此外， 以恰当的类来实现自己的自定义收集器有助于重用并可避免代码重复。 另外值得注意的是， 这第二个collect方法不能传递任何Characteristics， 所以它永远都是一个IDENTITY_FINISH和CONCURRENT但并非UNORDERED的收集器。

## 6. 开发你自己的收集器以获得更好的性能
我们用Collectors类提供的一个方便的工厂方法创建了一个收集器， 它将前 n 个自然数划分为质数和非质数， 如下所示。
代码清单6-6 将前 n 个自然数按质数和非质数分区
```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
            .collect(partitioningBy(candidate -> isPrime(candidate)));
}

// 通过限制除数不超过被测试数的平方根， 我们对最初的isPrime方法做了一些改进：
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
            .noneMatch(i -> candidate % i == 0);
}
```
通过自定义收集器获得更好的性能。

### 6.1 仅用质数做除数
一个可能的优化是仅仅看看被测试数是不是能够被质数整除。 要是除数本身都不是质数就用不着测了。 所以我们可以仅仅用被测试数之前的质数来测试。 然而我们目前所见的预定义收集器的问题， 也就是必须自己开发一个收集器的原因在于， 在收集过程中是没有办法访问部分结果的。 这意味着， 当测试某一个数字是否是质数的时候， 你没法访问目前已经找到的其他质数的列表。
假设你有这个列表， 那就可以把它传给isPrime方法， 将方法重写如下：
```java
public static boolean isPrime(List<Integer> primes, int candidate) {
    return primes.stream().noneMatch(i -> candidate % i == 0);
}
```
而且还应该应用先前的优化， 仅仅用小于被测数平方根的质数来测试。 因此， 你需要想办法在下一个质数大于被测数平方根时立即停止测试。 不幸的是， Stream API中没有这样一种方法。 你可以使用filter(p -> p <= candidateRoot)来筛选出小于被测数平方根的质数。 但filter要处理整个流才能返回恰当的结果。 如果质数和非质数的列表都非常大， 这就是个问题了。 你用不着这样做； 你只需在质数大于被测数平方根的时候停下来就可以了。 因此， 我们会创建一个名为takeWhile的方法， 给定一个排序列表和一个Predicate， 它会返回元素满足Predicate的最长前缀：

```java
public static <A> List<A> takeWhile(List<A> list, Predicate<A> p) {
    int i = 0;
    for (A item : list) {
        if (!p.test(item)) { // 检查列表中的当前项目是否满足谓词
            return list.subList(0, i); // 如果不满足， 返回该项目之前的前缀子列表
        }
        i++;
    }
    return list; // 列表中的所有项目都满足谓词， 因此返回列表本身}
}
```

利用这个方法， 你就可以优化isPrime方法， 只用不大于被测数平方根的质数去测试了：
```java
public static boolean isPrime(List<Integer> primes, int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return takeWhile(primes, i -> i <= candidateRoot)
            .stream()
            .noneMatch(p -> candidate % p == 0);
}
```
**请注意， 这个takeWhile实现是即时的。 理想情况下， 我们会想要一个延迟求值的takeWhile， 这样就可以和noneMatch操作合并。 不幸的是， 这样的实现超出了本章的范围， 你需要了解Stream API的实现才行。**

1. 第一步： 定义Collector类的签名
让我们从类签名开始吧， 记得Collector接口的定义是：
```java
public interface Collector<T, A, R>
```
其中T、 A和R分别是流中元素的类型、 用于累积部分结果的对象类型， 以及collect操作最终结果的类型。 这里应该收集Integer流， 而累加器和结果类型则都是```Map<Boolean, List<Integer>>```（ 和先前代码清单6-6中分区操作得到的结果Map相同） ， 键是true和false， 值则分别是质数和非质数的List：
```java
public class PrimeNumbersCollector
    implements Collector<Integer, // 流中元素的类型
    Map<Boolean, List<Integer>>, // 累加器类型
    Map<Boolean, List<Integer>>> // collect操作的结果类型
```

2. 第二步： 实现归约过程
需要实现Collector接口中声明的五个方法。 supplier方法会返回一个在调用时创建累加器的函数：
```java
public Supplier<Map<Boolean, List<Integer>>> supplier() {
    return () -> new HashMap<Boolean, List<Integer>>() {
        {
            put(true, new ArrayList<Integer>());
            put(false, new ArrayList<Integer>());
        }
    };
}
```
这里不但创建了用作累加器的Map， 还为true和false两个键下面初始化了对应的空列表。 在收集过程中会把质数和非质数分别添加到这里。 收集器中最重要的方法是accumulator， 因为它定义了如何收集流中元素的逻辑。 这里它也是实现前面所讲的优化的关键。 现在在任何一次迭代中， 都可以访问收集过程的部分结果， 也就是包含迄今找到的质数的累加器：
```java
public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
        acc.get(isPrime(acc.get(true), candidate)) // 根据isPrime的结果， 获取质数或非质数列表
                .add(candidate); // 将被测数添加到相应的列表中
    };
}
```
在这个方法中， 你调用了isPrime方法， 将待测试是否为质数的数以及迄今找到的质数列表（ 也就是累积Map中true键对应的值） 传递给它。 这次调用的结果随后被用作获取质数或非质数列表的键， 这样就可以把新的被测数添加到恰当的列表中。

3. 第三步： 让收集器并行工作（ 如果可能）
下一个方法要在并行收集时把两个部分累加器合并起来， 这里， 它只需要合并两个Map， 即将第二个Map中质数和非质数列表中的所有数字合并到第一个Map的对应列表中就行了：
```java
public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
    return (Map<Boolean, List<Integer>> map1,
            Map<Boolean, List<Integer>> map2) -> {
        map1.get(true).addAll(map2.get(true));
        map1.get(false).addAll(map2.get(false));
        return map1;
    };
}
```

实际上这个收集器是不能并行使用的， **因为该算法本身是顺序的。 这意味着永远都不会调用combiner方法， 你可以把它的实现留空（ 更好的做法是抛出一个UnsupportedOperationException异常） 。**
为了让这个例子完整， 我们还是决定实现它。

4. 第四步： finisher方法和收集器的characteristics方法

最后两个方法的实现都很简单。 前面说过， accumulator正好就是收集器的结果，用不着进一步转换， 那么finisher方法就返回identity函数：
```java
public Function<Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>> finisher() {
    return Function.identity();
}
```
就characteristics方法而言， 我们已经说过， 它既不是CONCURRENT也不是UNORDERED， 但却是IDENTITY_FINISH的：
```java
public Set<Characteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
}
```

**代码清单6-7 PrimeNumbersCollector**
```java

public class PrimeNumbersCollector
        implements Collector<Integer, Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>> {
    @Override
    public Supplier<Map<Boolean, List<Integer>>> supplier() {
        return () -> new HashMap<Boolean, List<Integer>>() {
            { // 从一个有两个空List的Map开始收集过程
                put(true, new ArrayList<Integer>());
                put(false, new ArrayList<Integer>());
            }
        };
    }

    @Override
    public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
        return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
            acc.get(isPrime(acc.get(true), // 将已经找到的质数列表传递给isPrime方法
                    candidate))
                    .add(candidate); // 根据isPrime方法的返回值， 从Map中取质数或非质数列表， 把当前的被测数加进去
        };
    }

    @Override
    public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
        return (Map<Boolean, List<Integer>> map1,
                Map<Boolean, List<Integer>> map2) -> { // 将第二个Map合并到第一个
            map1.get(true).addAll(map2.get(true));
            map1.get(false).addAll(map2.get(false));
            return map1;
        };
    }

    @Override
    public Function<Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>> finisher() {
        return Function.identity(); // 收集过程最后无需转换， 因此用identity 函数收尾
    }

    @Override
    public Set<Characteristics> characteristics() {
        // 这个收集器是IDENTITY_FINISH， 但既不是UNORDERED也不是CONCURRENT， 因为质数是按顺序发现的
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
    }
}
```

现在你可以用这个新的自定义收集器来代替6.4节中用partitioningBy工厂方法创建的那个， 并获得完全相同的结果了：
```java
public Map<Boolean, List<Integer>> partitionPrimesWithCustomCollector(int n) {
    return IntStream.rangeClosed(2, n).boxed()
            .collect(new PrimeNumbersCollector());
}
```

## 7. 小结
* collect是一个终端操作， 它接受的参数是将流中元素累积到汇总结果的各种方式（ 称为收集器） 。
* 预定义收集器包括将流元素归约和汇总到一个值， 例如计算最小值、 最大值或平均值。 这些收集器总结在表6-1中。
* 预定义收集器可以用groupingBy对流中元素进行分组， 或用partitioningBy进行分区。
* 收集器可以高效地复合起来， 进行多级分组、 分区和归约。
* 你可以实现Collector接口中定义的方法来开发你自己的收集器。
