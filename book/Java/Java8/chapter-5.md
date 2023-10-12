---
sort: 5
---
# 第五章 使用流

## 1. 筛选和切片
### 1.1 Predicate筛选
Streams接口支持filter方法 。 该操作会接受一个Predicate作为参数， 并返回一个包括所有符合谓词的元素的流。 

### 1.2 筛选各异的元素
流还支持一个叫作distinct的方法（去重）， 它会返回一个元素各异（ 根据流所生成元素的hashCode和equals方法实现） 的流。 

### 1.3 截短流
流支持limit(n)方法， 该方法会返回一个不超过给定长度的流。 所需的长度作为参数传递给limit。 如果流是有序的， 则最多会返回前n个元素。 

### 1.4 跳过元素
流还支持skip(n)方法， 返回一个扔掉了前n个元素的流。 如果流中元素不足n个，则返回一个空流。 请注意， limit(n)和skip(n)是互补的！ 

## 2. 映射
一个非常常见的数据处理套路就是从某些对象中选择信息。比如在SQL里，你可以从表中选择一列。Stream API也通过map和flatMap方法提供了类似的工具。

### 2.1 对流中每一个元素应用函数
流支持map方法， 它会接受一个函数作为参数。 这个函数会被应用到每个元素上， 并将其映射成一个新的元素（ 使用映射一词， 是因为它和转换类似， 但其中的细微差别在于它是“创建一个新版本” 而不是去“修改” ） 。 
```java
List<String> dishNames = menu.stream()
.map(Dish::getName) 
.collect(toList());

// 输出： 
```
因为getName方法返回一个```String```， 所以map方法输出的流的类型就是```Stream<String>```。

### 2.2 流的扁平化
对于一张单词表， 如何返回一张列表， 列出里面各不相同的字符呢？ 例如， 给定单词列表\["Hello","World"\]， 你想要返回列表\["H","e","l", "o","W","r","d"\]。

你可能会认为这很容易， 你可以把每个单词映射成一张字符表， 然后调用distinct来过滤重复的字符。 第一个版本可能是这样的：
```java
words.stream().map(word -> word.split("")).distinct().collect(toList());
// 输出：[H, e, l, l, o], [W, o, r, l, d]
```
这个方法的问题在于， 传递给map方法的Lambda为每个单词返回了一个```String[]```（ String列表） 。 因此， map返回的流实际上是```Stream<String[]>```类型的。 你真正想要的是用```Stream<String>```来表示一个字符流。 
图5-5说明了这个问题。
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic5-5.png)

**图 5-5 不正确地使用map找出单词列表中各不相同的字符**

后续使用flatMap来解决这个问题！ 让我们一步步看看怎么解决它。
1. 尝试使用map和Arrays.stream()
首先， 你需要一个字符流， 而不是数组流。 有一个叫作Arrays.stream()的方法可以接受一个数组并产生一个流， 例如：
```java
String[] arrayOfWords = {"Goodbye", "World"};
Stream<String> streamOfwords = Arrays.stream(arrayOfWords);
```
把它用在前面的那个流水线里， 看看会发生什么：
```java
words.stream().map(word->word.split("")) // 将每个单词转换为由其字母构成的数组
.map(Arrays::stream) // 让每个数组变成一个单独的流
.distinct().collect(toList());
// 返回值为 List<Stream>
// 输出 [java.util.stream.ReferencePipeline$Head@1b28cdfa, java.util.stream.ReferencePipeline$Head@eed1f14]
```
当前的解决方案仍然搞不定！ 这是因为， 你现在得到的是一个流的列表（ 更准确地说是```Stream<String>```） ！ 的确， 你先是把每个单词转换成一个字母数组， 然后把每个数组变成了一个独立的流。

2. 使用flatMap
```java
List<String> uniqueCharacters = words.stream()
        .map(w -> w.split("")) // 将每个单词转换为由其字母构成的数组
        .flatMap(Arrays::stream) // 将各个生成流扁平化为单个流
        .distinct()
        .collect(toList());
// H
// e
// l
// o
// W
// r
// d
```
使用flatMap方法的效果是， 各个数组并不是分别映射成一个流， 而是映射成流的内容。 所有使用```map(Arrays::stream)```时生成的单个流都被合并起来， 即扁平化为一个流。 图5-6说明了使用```flatMap```方法的效果。 把它和图5-5中map的效果比较一下。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic5-6.png)

**图 5-6 使用flatMap找出单词列表中各不相同的字符**

一言以蔽之， flatmap方法让你把一个流中的每个值都换成另一个流， 然后把所有的流连接起来成为一个流。

## 3. 查找和匹配
另一个常见的数据处理套路是看看数据集中的某些元素是否匹配一个给定的属性。Stream API通过allMatch、 anyMatch、 noneMatch、 findFirst和findAny方法提供了这样的工具。

### 3.1 检查Predicate是否至少匹配一个元素
anyMatch方法可以回答“流中是否有一个元素能匹配给定的Predicate” 。 anyMatch方法返回一个boolean， 因此是一个终端操作。

### 3.2 检查Predicate是否匹配所有元素
allMatch方法的工作原理和anyMatch类似， 但它会看看流中的元素是否都能匹配给定的Predicate。 也是一个终端操作。
和allMatch相对的是noneMatch。 它可以确保流中没有任何元素与给定的Predicate匹配。 

anyMatch、 allMatch和noneMatch这三个操作都用到了我们所谓的短路， 这就是大家熟悉的Java中&&和||运算符短路在流中的版本。
limit也是一个短路操作。

短路求值: 有些操作不需要处理整个流就能得到结果。 例如， 假设你需要对一个用and连起来的大布尔表达式求值。 不管表达式有多长， 你只需找到一个表达式为false，就可以推断整个表达式将返回false， 所以用不着计算整个表达式。 这就是短路。

### 3.3 查找元素
findAny方法将返回当前流中的任意元素。 它可以与其他流操作结合使用。 
比如，你可能想找到一道素食菜肴。 你可以结合使用filter和findAny方法来实现这个查询：
```java
Optional<Dish> dish = menu.stream()
.filter(Dish::isVegetarian)
.findAny();
```

**Optional简介**
```Optional<T>```类（ ```java.util.Optional```） 是一个容器类， 代表一个值存在或不存在。 在上面的代码中， findAny可能什么元素都没找到。 
现在， 了解一下Optional里面几种可以迫使你显式地检查值是否存在或处理值不存在的情形的方法也不错。
* ```isPresent()```将在```Optional```包含值的时候返回```true```, 否则返回```false```。
* ```ifPresent(Consumer<T> block)```会在值存在的时候执行给定的代码块。 我们在[第3章](https://dxjeric.github.io/book/Java/Java8/chapter-3.html)介绍了```Consumer```函数式接口； 它让你传递一个接收T类型参数， 并返回void的Lambda表达式。
* ```T get()```会在值存在时返回值， 否则抛出一个```NoSuchElement```异常
* ```T orElse(T other)```会在值存在时返回值， 否则返回一个默认值

例如， 在前面的代码中你需要显式地检查Optional对象中是否存在一道菜可以访问其名称：
```java
menu.stream()
    .filter(Dish::isVegetarian)
    .findAny() // 返回一个Optional<Dish>
    .ifPresent(d -> System.out.println(d.getName()); // 如果包含一个值就打印它， 否则什么    都不做
```

### 3.4 查找第一个元素
有些流有一个出现顺序（ encounter order） 来指定流中项目出现的逻辑顺序（ 比如由List或排序好的数据列生成的流） 。 对于这种流， 你可能想要找到第一个元素。 为此有一个findFirst方法， 它的工作方式类似于findany。 
例如， 给定一个数字列表， 下面的代码能找出第一个平方能被3整除的数：
```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
.map(x -> x * x)
.filter(x -> x % 3 == 0)
.findFirst(); // 9
```
为什么会同时有findFirst和findAny呢？ 答案是并行。 找到第一个元素在并行上限制更多。 如果你不关心返回的元素是哪个， 请使用findAny， 因为它在使用并行流时限制较少。

## 4. 归约
在本节中， 你将看到如何把一个流中的元素组合起来， 使用reduce操作来表达更复杂的查询， 比如“计算菜单中的总卡路里” 或“菜单中卡路里最高的菜是哪一个” 。 此类查询需要将流中所有元素反复结合起来， 得到一个值， 比如一个Integer。 这样的查询可以被归类为归约操作（ 将流归约成一个值） 。 用函数式编程语言的术语来说， 这称为折叠（ fold） ， 因为你可以将这个操作看成把一张长长的纸（ 你的流） 反复折叠成一个小方块， 而这就是折叠操作的结果。

### 4.1 元素求和
使用reduce操作， 它对这种重复应用的模式做了抽象。 你可以像下面这样对流中所有的元素求和：
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

reduce接受两个参数：
* 一个初始值， 这里是0；
* 一个```BinaryOperator<T>```来将两个元素结合起来产生一个新值， 这里我们用的是lambda: ```(a, b) -> a + b```。

![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic5-7.png)

**图 5-7 使用reduce来对流中的数字求和**

reduce还有一个重载的变体， 它不接受初始值， 但是会返回一个Optional对象：```Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));```

为什么它返回一个```Optional<Integer>```呢？ 考虑流中没有任何元素的情况。 reduce操作无法返回其和， 因为它没有初始值。 这就是为什么结果被包裹在一个Optional对象里， 以表明和可能不存在。 

### 4.2 最大值和最小值
```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```
![](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic5-8.png)

**图 5-8 一个归约操作——计算最大值**

**归约方法的优势与并行化**
相比于前面写的逐步迭代求和， 使用reduce的好处在于， 这里的迭代被内部迭代抽象掉了， 这让内部实现得以选择并行执行reduce操作。 而迭代式求和例子要更新共享变量sum， 这不是那么容易并行化的。 如果你加入了同步， 很可能会发现线程竞争抵消了并行本应带来的性能提升！ 这种计算的并行化需要另一种办法： 将输入分块， 分块求和， 最后再合并起来。 但这样的话代码看起来就完全不一样了。 你在[第7章](https://dxjeric.github.io/book/Java/Java8/chapter-7.html)会看到使用分支/合并框架来做是什么样子。 但现在重要的是要认识到， 可变的累加器模式对于并行化来说是死路一条。 你需要一种新的模式， 这正是reduce所提供的。 你还将在第7章看到， 使用流来对所有的元素并行求和时， 你的代码几乎不用修改： stream()换成了parallelStream()。

流操作： 无状态和有状态 **TODO**
> 你已经看到了很多的流操作。 乍一看流操作简直是灵丹妙药， 而且只要在从集合生成流的时候把Stream换成parallelStream就可以实现并行。
> 当然， 对于许多应用来说确实是这样， 就像前面的那些例子。 你可以把一张菜单变成流， 用filter选出某一类的菜肴， 然后对得到的流做map来对卡路里求和， 最后reduce得到菜单的总热量。 这个流计算甚至可以并行进行。 但这些操作的特性并不相同。 它们需要操作的内部状态还是有些问题的。
> 诸如map或filter等操作会从输入流中获取每一个元素， 并在输出流中得到0或1个结果。 这些操作一般都是无状态的： 它们没有内部状态（ 假设用户提供的Lambda或方法引用没有内部可变状态） 。
> 但诸如reduce、 sum、 max等操作需要内部状态来累积结果。 在上面的情况下，内部状态很小。 在我们的例子里就是一个int或double。 不管流中有多少元素要处理， 内部状态都是有界的。> 相反， 诸如sort或distinct等操作一开始都和filter和map差不多——都是接受一个流， 再生成一个流（ 中间操作） ， 但有一个关键的区别。 从流中排序和删除重复项时都需要知道先前的历史。 例如， 排序要求所有元素都放入缓冲区后才能给输出流加入一个项目， 这一操作的存储要求是无界的。 要是流比较大或是无限的， 就可能会有问题（ 把质数流倒序会做什么呢？ 它应当返回最大的质数， 但数学告诉我们它不存在） 。 我们把这些操作叫作有状态操作。

**表5-1 中间操作和终端操作**

| 操作      | 类型              | 返回类型      | 使用的类型/函数式接口      | 函数描述符       |
| --------- | ----------------- | ------------- | -------------------------- | ---------------- |
| filter    | 中间              | Stream\<T\>   | Predicate\<T\>             | T -> boolean     |
| distinct  | 中间(有状态-无界) | Stream\<T\>   |                            |                  |
| skip      | 中间(有状态-有界) | Stream\<T\>   | long                       |                  |
| limit     | 中间(有状态-有界) | Stream\<T\>   | long                       |                  |
| map       | 中间              | Stream\<R\>   | Function\<T, R\>           | T -> R           |
| flatMap   | 中间              | Stream\<R\>   | Function\<T, Stream\<R\>\> | T -> Stream\<R\> |
| sorted    | 中间(有状态-无界) | Stream\<T\>   | Comparator\<T\>            | (T, T) -> int    |
| anyMatch  | 终端              | boolean       | Predicate\<T\>             | T -> boolean     |
| noneMatch | 终端              | boolean       | Predicate\<T\>             | T -> boolean     |
| allMatch  | 终端              | boolean       | Predicate\<T\>             | T -> boolean     |
| findAny   | 终端              | Optional\<T\> |                            |                  |
| findFirst | 终端              | Optional\<T\> |                            |                  |
| forEach   | 终端              | void          | Consumer\<T\>              | T -> void        |
| collect   | 终端              | R             | Collector\<T, A, R\>       |                  |
| reduce    | 终端(有状态-有界) | Optional\<T\> | BinaryOperator\<T\>        | (T, T) -> T      |
| count     | 终端              | long          |                            |                  |

## 6. 数值流

```java
int calories = menu.stream()
.map(Dish::getCalories)
.reduce(0, Integer::sum);
```
这段代码的问题是， 它有一个暗含的装箱成本。 每个Integer都必须拆箱成一个原始类型， 再进行求和。 要是可以直接像下面这样调用sum方法， 岂不是更好？

```java
int calories = menu.stream()
.map(Dish::getCalories)
.sum();
```
但这是不可能的。 问题在于map方法会生成一个```Stream<T>```。 虽然流中的元素是Integer类型， 但Streams接口没有定义sum方法。 

### 6.1 原始类型流特化
Java 8引入了三个原始类型特化流接口来解决这个问题： IntStream、 DoubleStream和LongStream， 分别将流中的元素特化为int、 long和double， 从而避免了暗含的装箱成本。 每个接口都带来了进行常用数值归约的新方法， 比如对数值流求和的sum， 找到最大元素的max。 此外还有在必要时再把它们转换回对象流的方法。 要记住的是， 这些特化的原因并不在于流的复杂性， 而是装箱造成的复杂性——即类似int和Integer之间的效率差异。

1. 映射到数值流
将流转换为特化版本的常用方法是mapToInt、 mapToDouble和mapToLong。 这些方法和前面说的map方法的工作方式一样， 只是它们返回的是一个特化流， 而不是```Stream<T>```。 
```java
    int calories = menu.stream() // 返回一个Stream<Dish>
    .mapToInt(Dish::getCalories) // 返回一个IntStream
    .sum()
```

这里， mapToInt会从每道菜中提取热量（ 用一个Integer表示） ， 并返回一个IntStream（ 而不是一个```Stream<Integer>```） 。

2. 转换回对象流
同样， 一旦有了数值流， 你可能会想把它转换回非特化流。 可以使用```boxed```接口。

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 将Stream 转换为数值流
Stream<Integer> stream = intStream.boxed(); // 将数值流转换为Stream
```

3. 默认值OptionalInt
求和的那个例子很容易， 因为它有一个默认值： 0。 但是， 如果你要计算IntStream中的最大元素， 就得换个法子了， 因为0是错误的结果。 如何区分没有元素的流和最大值真的是0的流呢？ 前面我们介绍了Optional类， 这是一个可以表示值存在或不存在的容器。 Optional可以用Integer、 String等参考类型来参数化。 对于三种原始流特化， 也分别有一个Optional原始类型特化版本： OptionalInt、 OptionalDouble和OptionalLong。
```java
OptionalInt maxCalories = menu.stream()
.mapToInt(Dish::getCalories)
.max();
```
现在， 如果没有最大值的话， 你就可以显式处理OptionalInt去定义一个默认值了：
```java
int max = maxCalories.orElse(1); // 如果没有最大值的话， 显式提供一个默认最大值
```

### 6.2 数值范围
Java 8引入了两个可以用于IntStream和LongStream的静态方法， 帮助生成这种范围： range和rangeClosed。 这两个方法都是第一个参数接受起始值， 第二个参数接受结束值。 但range是不包含结束值的， 而rangeClosed则包含结束值。 

### 6.3 数值流应用： 勾股数

## 7. 构建流

### 7.1 由值创建流
可以使用静态方法Stream.of， 通过显式值创建一个流。 它可以接受任意数量的参数。 

例如， 以下代码直接使用Stream.of创建了一个字符串流。 然后， 你可以将字符串转换为大写， 再一个个打印出来：
```java
Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);
// 你可以使用empty得到一个空流，
Stream<String> emptyStream = Stream.empty();
```

### 7.2 由数组创建流
你可以使用静态方法Arrays.stream从数组创建一个流。 它接受一个数组作为参数。 

### 7.3 由文件生成流
Java中用于处理文件等I/O操作的NIO API（ 非阻塞 I/O） 已更新， 以便利用Stream API。 java.nio.file.Files中的很多静态方法都会返回一个流。 

例如，一个很有用的方法是Files.lines， 它会返回一个由指定文件中的各行构成的字符串流。 使用你迄今所学的内容， 你可以用这个方法看看一个文件中有多少各不相同的词：
```java
    long uniqueWords = 0;
    try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset()))
    { // 流会自动关闭
        uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))) // 生成单词流
                .distinct() // 删除重复项
                .count(); // 数一数有多少各不相同的单词
    }catch(
    IOException e)
    {
        // 如果打开文件时出现异常则加以处理
    }
```

### 7.4 由函数生成流： 创建无限流
Stream API提供了两个静态方法来从函数生成流： Stream.iterate和Stream.generate。 这两个操作可以创建所谓的无限流： 不像从固定集合创建的流那样有固定大小的流。 由iterate和generate产生的流会用给定的函数按需创建值， 因此可以无穷无尽地计算下去！ 一般来说， 应该使用limit(n)来对这种流加以限制， 以避免打印无穷多个值。

1. 迭代
```java
Stream.iterate(0, n -> n + 2)
.limit(10)
.forEach(System.out::println);
```
iterate方法接受一个初始值（ 在这里是0） ， 还有一个依次应用在每个产生的新值上的Lambda（ ```UnaryOperator<t>```类型） 。 这里， 我们使用Lambda ```n -> n + 2```，返回的是前一个元素加上2。 因此， iterate方法生成了一个所有正偶数的流： 流的第一个元素是初始值0。 

2.  生成
与iterate方法类似， generate方法也可让你按需生成一个无限流。 但generate不是依次对每个新生成的值应用函数的。 它接受一个```Supplier<T>```类型的Lambda提供新的值。 
```java
Stream.generate(Math::random)
.limit(5)
.forEach(System.out::println);
```

使用iterate创建斐波纳契数列
```java
Stream.iterate(new int[]{0, 1}, t -> new int[]{t[1],t[0] + t[1]})
    .limit(10)
    .map(t -> t[0])
    .forEach(System.out::println);
```

我们使用的供应源（ 指向Math.random的方法引用） 是无状态的： 它不会在任何地方记录任何值， 以备以后计算使用。 但供应源不一定是无状态的。 你可以创建存储状态的供应源， 它可以修改状态， 并在为流生成下一个值时使用。 

使用generate创建测验5.4中的斐波纳契数列
```java
IntSupplier fib = new IntSupplier(){
    private int previous = 0;
    private int current = 1;
    public int getAsInt(){
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
    }
};
IntStream.generate(fib).limit(10).forEach(System.out::println);
```

## 8. 小结
* Streams API可以表达复杂的数据处理查询。 常用的流操作总结在表5-1中。
* 你可以使用filter、 distinct、 skip和limit对流做筛选和切片。
* 你可以使用map和flatMap提取或转换流中的元素。
* 你可以使用findFirst和findAny方法查找流中的元素。 你可以用allMatch、 noneMatch和anyMatch方法让流匹配给定的Predicate。
* 这些方法都利用了短路： 找到结果就立即停止计算； 没有必要处理整个流。
* 你可以利用reduce方法将流中所有的元素迭代合并成一个结果， 例如求和或查找最大元素。
* filter和map等操作是无状态的， 它们并不存储任何状态。 reduce等操作要存储状态才能计算出一个值。 sorted和distinct等操作也要存储状态， 因为它们需要把流中的所有元素缓存起来才能返回一个新的流。 这种操作称为有状态操作。
* 流有三种基本的原始类型特化： IntStream、 DoubleStream和LongStream。它们的操作也有相应的特化。
* 流不仅可以从集合创建， 也可从值、 数组、 文件以及iterate与generate等特定方法创建。
* 无限流是没有固定大小的流。