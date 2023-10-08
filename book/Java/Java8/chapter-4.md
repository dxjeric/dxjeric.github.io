---
sort: 4
---
# 第四章 引入流

## 1. 流是什么
流是Java API的新成员， 它允许你以声明性方式处理数据集合（ 通过查询语句来表达， 而不是临时编写一个实现） 。 也可以把它们看成遍历数据集的高级迭代器。 此外， 流还可以透明地并行处理， 你无需写任何多线程代码了！ 

下面两段代码都是用来返回低热量的菜肴名称的， 并按照卡路里排序， 一个是用Java 7写的， 另一个是用Java 8的流写的。
```java
// java7 版本代码
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish d:menu)
{
    if (d.getCalories() < 400) { // 用累加器筛选元素
        lowCaloricDishes.add(d);
    }
}Collections.sort(lowCaloricDishes,new Comparator<Dish>(){ // 用匿名类对菜肴排序

public int compare(Dish d1, Dish d2) {
    return Integer.compare(d1.getCalories(), d2.getCalories());
}});

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish d:lowCaloricDishes)
{
    lowCaloricDishesName.add(d.getName()); // 处理排序后的菜名列表
}

// Java8
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDishesName = menu.stream()
    .filter(d -> d.getCalories() < 400) // 选出400卡路里以下的菜肴
    .sorted(comparing(Dish::getCalories)) // 按照卡路里排序
    .map(Dish::getName) // 提取菜肴的名称
    .collect(toList()); // 将所有名称保存在List中

// java8可以将stream()换成parallelStream()， 实现多核架构并行执行这段代码
List<String> lowCaloricDishesName = menu.parallelStream()
    .filter(d -> d.getCalories() < 400)
    .sorted(comparing(Dishes::getCalories))
    .map(Dish::getName)
    .collect(toList())
```

java7使用了一个“垃圾变量” lowCaloricDishes。 它唯一的作用就是作为一次性的中间容器。在Java 8中， 实现的细节被放在它本该归属的库里了。 

新的方法有几个显而易见的好处：
* 代码是以声明性方式写的： 说明想要完成什么（ 筛选热量低的菜肴） 而不是说明如何实现一个操作（ 利用循环和if条件等控制流语句） 。 
* 可以把几个基础操作链接起来， 来表达复杂的数据处理流水线（ 在filter后面接上sorted、 map和collect操作， 如图4-1所示） ， 同时保持代码清晰可读。 filter的结果被传给了sorted方法， 再传给map方法， 最后传给collect方法。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic4-1.png)

**图 4-1 将流操作链接起来构成流的流水线**

Java 8中的Stream API可以让你写出这样的代码：
* 声明性 —— 更简洁， 更易读
* 可复合 —— 更灵活
* 可并行 —— 性能更好

后续内容使用的基础代码, 一个menu， 它只是一张菜肴列表。
```java
// Dish类的定义
public class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return name;
    }

    public enum Type {
        MEAT, FISH, OTHER
    }
}

List<Dish> menu = Arrays.asList(
    new Dish("pork", false, 800, Dish.Type.MEAT),
    new Dish("beef", false, 700, Dish.Type.MEAT),
    new Dish("chicken", false, 400, Dish.Type.MEAT),
    new Dish("french fries", true, 530, Dish.Type.OTHER),
    new Dish("rice", true, 350, Dish.Type.OTHER),
    new Dish("season fruit", true, 120, Dish.Type.OTHER),
    new Dish("pizza", true, 550, Dish.Type.OTHER),
    new Dish("prawns", false, 300, Dish.Type.FISH),
    new Dish("salmon", false, 450, Dish.Type.FISH) );
```
## 2. 流简介
Java 8中的集合支持一个新的stream方法， 它会返回一个流（ 接口定义在java.util.stream.Stream里） 。

**流就是“从支持数据处理操作的源生成的元素序列”**
1. 元素序列 —— 就像集合一样， 流也提供了一个接口， 可以访问特定元素类型的一组有序值。  因为集合是数据结构， 所以它的主要目的是以特定的时间/空间复杂度存储和访问元素（ 如ArrayList 与 LinkedList） 。 但流的目的在于表达计算， 比如你前面见到的filter、 sorted和map。 集合讲的是数据， 流讲的是计算。 
2. 源 —— 流会使用一个提供数据的源， 如集合、 数组或输入/输出资源。 **从有序集合生成流时会保留原有的顺序。**
3. 数据处理操作 —— 流的数据处理功能支持类似于数据库的操作， 以及函数式编程语言中的常用操作， 如filter、 map、 reduce、 find、 match、 sort等。 流操作可以顺序执行， 也可并行执行。

**流操作有两个重要的特点:**
1. 流水线 —— 很多流操作本身会返回一个流， 这样多个操作就可以链接起来， 形成一个大的流水线。 
2. 内部迭代 —— 与使用迭代器显式迭代的集合不同， 流的迭代操作是在背后进行的。 

让我们来看一段能够体现所有这些概念的代码：
```java
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames = menu.stream() // 从menu获得流（ 菜肴列表）
    .filter(d -> d.getCalories() > 300) // 建立操作流水线： 首先选出高热量的菜肴
    .map(Dish::getName) // 获取菜名
    .limit(3) // 只选择头三个
    .collect(toList()); // 将结果保存在另一个List中

System.out.println(threeHighCaloricDishNames); // 结果是[pork, beef,chicken]
```

图4-2显示了流操作的顺序： filter、 map、 limit、 collect， 每个操作简介如下。
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic4-2.png)

**图 4-2 使用流来筛选菜单， 找出三个高热量菜肴的名字**

## 3. 流与集合
集合与流之间的差异就在于什么时候进行计算。 **集合**是一个内存中的数据结构， 它包含数据结构中目前所有的值——集合中的每个元素都得先算出来才能添加到集合中。 相比之下， **流**则是在概念上固定的数据结构（ 你不能添加或删除元素） ， 其元素则是按需计算的。 
图4-3用DVD对比在线流媒体的例子展示了流和集合之间的差异。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic4-3.png)
**图 4-3 流与集合**

### 3.1 只能遍历一次
和迭代器类似， 流只能遍历一次。 遍历完之后， 我们就说这个流已经被消费掉了。 你可以从原始数据源那里再获得一个新的流来重新遍历一遍， 就像迭代器一样（ 这里假设它是集合之类的可重复的源， 如果是I/O通道就没戏了） 。
```java
List<String> title = Arrays.asList("Java8", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println); // 打印标题中的每个单词
s.forEach(System.out::println); // java.lang.IllegalStateException:流已被操作或关闭
```

### 3.2 外部迭代与内部迭代
使用Collection接口需要用户去做迭代（ 比如用for-each） ， 这称为外部迭代。相反， Streams库使用内部迭代——它帮你把迭代做了， 还把得到的流值存在了某个地方， 你只要给出一个函数说要干什么就可以了。 
代码清单4-1 集合： 用for-each循环外部迭代
```java
List<String> names = new ArrayList<>();
for(Dish d:menu)
{ // 显式顺序迭代菜单列表
    names.add(d.getName()); // 提取名称并将其添加到累加器
}
```

代码清单4-2 集合： 用背后的迭代器做外部迭代
```java
List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext())
{ // 显式迭代
    Dish d = iterator.next();
    names.add(d.getName());
}
```

代码清单4-3 流： 内部迭代
```java
List<String> names = menu.stream()
    .map(Dish::getName) // 用getName 方法参数化map， 提取菜名
    .collect(toList()); // 开始执行操作流水线； 没有迭代！
```
内部迭代时， 项目可以透明地并行处理， 或者用更优化的顺序进行处理。 要是用Java过去的那种外部迭代方法， 这些优化都是很困难的。 这似乎有点儿鸡蛋里挑骨头， 但这差不多就是Java 8引入流的理由了——Streams库的内部迭代可以自动选择一种适合你硬件的数据表示和并行实现。 与此相反， 一旦通过写for-each而选择了外部迭代， 那你基本上就要自己管理所有的并行问题了（ 自己管理实际上意味着“某个良辰吉日我们会把它并行化” 或“开始了关于任务和synchronized的漫长而艰苦的斗争” ） 。 Java 8需要一个类似于Collection却没有迭代器的接口， 于是就有了Stream！

图4-4说明了流（ 内部迭代） 与集合（ 外部迭代） 之间的差异。
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic4-4.png)
**图 4-4 内部迭代与外部迭代**

## 4. 流操作
java.util.stream.Stream中的Stream接口定义了许多操作。 它们可以分为两大类。 
* 中间操作: filter、 map和limit等可以连成一条计算流水线；
* 终端操作: collect等触发流水线执行并关闭它。

代码示例：
```java
List<String> names = menu.stream() // 从菜单获得流
        .filter(d -> d.getCalories() > 300) // 中间操作
        .map(Dish::getName) // 中间操作
        .limit(3) // 中间操作
        .collect(toList()); // 将Stream转换为List
```
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic4-5.png)
**图 4-5 中间操作与终端操作**

### 4.1 中间操作
诸如filter或sorted等中间操作会返回另一个流。 这让多个操作可以连接起来形成一个查询。 重要的是， 除非流水线上触发一个终端操作， 否则中间操作不会执行任何处理——它们很懒。 这是因为中间操作一般都可以合并起来， 在终端操作时一次性全部处理。

为了搞清楚流水线中到底发生了什么， 我们把代码改一改， 让每个Lambda都打印出当前处理的菜肴（ 就像很多演示和调试技巧一样， 这种编程风格要是搁在生产代码里那就吓死人了， 但是学习的时候却可以直接看清楚求值的顺序） ：
```java
List<String> names = menu.stream()
    .filter(d -> {
        // 打印当前筛选的菜肴
        System.out.println("filtering" + d.getName());
        return d.getCalories() > 300;
    })
    .map(d -> {
        // 提取菜名时打印出来
        System.out.println("mapping" + d.getName());
        return d.getName();
    })
    .limit(3)
    .collect(toList());

System.out.println(names);
```
流操作执行显示:
```result
filtering pork
mapping pork
filtering beef
mapping beef
filtering chicken
mapping chicken
[pork, beef, chicken]
```

有好几种优化利用了流的延迟性质。 第一， 尽管很多菜的热量都高于300卡路里， 但只选出了前三个！ 这是因为limit操作和一种称为短路的技巧， 我们会在[下一章](https://dxjeric.github.io/book/Java/Java8/chapter-5.html)中解释。 第二， 尽管filter和map是两个独立的操作， 但它们合并到同一次遍历中了（ 我们把这种技术叫作循环合并） 。

### 4.2 终端操作

终端操作会从流的流水线生成结果。 其结果是任何不是流的值。

### 4.3 使用流
总而言之， 流的使用一般包括三件事：
* 一个数据源（ 如集合） 来执行一个查询
* 一个中间操作链， 形成一条流的流水线
* 一个终端操作， 执行流水线， 并能生成结果

流的流水线背后的理念类似于构建器模式。 在构建器模式中有一个调用链用来设置一套配置（ 对流来说这就是一个中间操作链） ， 接着是调用built方法（ 对流来说就是终端操作） 。

## 5. 小结
* 流是“从支持数据处理操作的源生成的一系列元素”
* 流利用内部迭代： 迭代通过filter、 map、 sorted等操作被抽象掉了
* 流操作有两类： 中间操作和终端操作
* filter和map等中间操作会返回一个流， 并可以链接在一起。 可以用它们来设置一条流水线， 但并不会生成任何结果
* forEach和count等终端操作会返回一个非流的值， 并处理流水线以返回结果
* 流中的元素是按需计算的