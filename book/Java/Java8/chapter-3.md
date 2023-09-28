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

