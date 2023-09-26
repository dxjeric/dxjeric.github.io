---
sort: 2
---
# 第二章 通过行为参数化传递代码
内容简介: 
* 应对不断变化的需求
    行为参数化就是可以帮助你处理频繁变更的需求的一种软件开发模式。 一言以蔽之， 它意味着拿出一个代码块， 把它准备好却不去执行它。 这个代码块以后可以被你程序的其他部分调用， 这意味着你可以推迟这块代码的执行。
* 行为参数化
* 匿名类
* Lambda表达式预览
* 真实示例： Comparator、 Runnable和GUI

## 1. 应对不断变化的需求
需求:从一堆苹果选出符合条件的苹果。古老的代码方式如下
```java
// 选出绿色的苹果
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>(); // 累积苹果的列表
    for(Apple apple: inventory){
        if( "green".equals(apple.getColor() ) { // 仅仅选出绿苹果
            result.add(apple);
        }
    }
    return result;
}

// 进一步的方式，将颜色作为参数传入
public static List<Apple> filterApplesByColor(List<Apple> inventory, String color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory){
        if ( apple.getColor().equals(color) ) {
            result.add(apple);
        }
    }
    return result;
}

// 选出重量符合的苹果
public static List<Apple> filterApplesByColor(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory){
        if ( apple.ingetWeightt() > weight ) {
            result.add(apple);
        }
    }
    return result;
}

// 如果有更复杂的规则，比如多个属性组合，则需要重新完成新的代码实现
```

## 2. 行为参数化
我们从更高层次的抽象。 使用一种新解决方案，对选择标准建模： 你考虑的是苹果， 需要根据Apple的某些属性来返回一个boolean值。 
抽象接口如下:
```java
public interface ApplePredicate{
    boolean test (Apple apple);
}
```
通过具体的类来实现该接口，比如需要选择不同属性的苹果
```java
public class AppleHeavyWeightPredicate implements ApplePredicate { // 仅仅选出重的苹果
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate { // 仅仅选出绿苹果
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}
```
![选择苹果的不同策略](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic2-1.png)

你可以把这些标准看作filter方法的不同行为。 你刚做的这些和[“策略设计模式”](http://en.wikipedia.org/wiki/Strategy_pattern)相关， 它让你定义一族算法， 把它们封装起来（ 称为“策略” ） ， 然后在运行时选择一个算法。 在这里， 算法族就是ApplePredicate， 不同的策略就是AppleHeavyWeightPredicate和AppleGreenColorPredicate。

使用策略实现苹果筛选
```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) { // 对象封装了测试苹果的条件
            result.add(apple);
        }
    }
    return result;
}

// 传递代码/行为
public class AppleRedAndHeavyPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return "red".equals(apple.getColor())
                && apple.getWeight() > 150;
    }
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

在[第3节](#3-简化行为参数)（ [第3章](./chapter-3.md)中有更详细的内容） 中看到， 通过使用Lambda， 你可以直接把表达式"red".equals(apple.getColor()) &&apple.getWeight() >150传递给filterApples方法， 而无需定义多个ApplePredicate类， 从而去掉不必要的代码。

![参数化filterApples的行为并传递不同的筛选策略](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic2-3.png)

## 3. 简化行为参数
* 匿名类
* Lambda表达式

### 3.1 匿名类
匿名类和你熟悉的Java局部类（ 块中定义的类） 差不多， 但匿名类没有名字。 它允许你同时声明并实例化一个类。 换句话说， 它允许你随用随建。

### 3.2 使用匿名类 实现苹果筛选
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() { // 直接内联参数化filterapples方法的行为
    public boolean test(Apple apple) {
        return "red".equals(apple.getColor());
    }
});
```
GUI应用程序中经常使用匿名类来创建事件处理器对象（ 下面的例子使用的是JavaFX API， 一种现代的Java UI平台） ：
```java
button.setOnAction(new EventHandler<ActionEvent>() {
    public void handle(ActionEvent event) {
        System.out.println("Woooo a click!!");
    }
});
```
**匿名类的缺点**
1. 它往往很笨重， 因为它占用了很多空间。
2. 代码阅读理解成本较高(如下面的代码)

```java
public class MeaningOfThis {
    public final int value = 4;

    public void doIt() {
        int value = 6;
        Runnable r = new Runnable() {
            public final int value = 5;

            public void run() {
                int value = 10;
                System.out.println(this.value);
            }
        };
        r.run();
    }

    public static void main(String... args) {
        MeaningOfThis m = new MeaningOfThis();
        m.doIt(); // 这一行的输出是什么？
    }
}
// 答案是5， 因为this指的是包含它的Runnable， 而不是外面的类MeaningOfThis。
```

### 3.3 使用Lambda表达式 实现苹果筛选
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> "red".equals(apple.getColor()));
```
使用Lambda表达式的代码看上去比先前干净很多。

![行为参数化与值参数化不同方式比较](https://github.com/dxjeric/dxjeric.github.io/raw/master/pictures/Java/Java8/pic2-4.png)

### 3.4 将List类型抽象化 实现不同水果筛选
将List类型抽象化， 从而超越你眼前要处理的问题，这样可以实现不同水果的筛选
```java
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) { // 引入类型参数T
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```
## 4. 实例
行为参数化是一个很有用的模式， 它能够轻松地适应不断变化的需求。 这种模式可以把一个行为（ 一段代码） 封装起来， 并通过传递和使用创建的行为（ 例如对Apple的不同谓词） 将方法的行为参数化。 前面提到过， 这种做法类似于策略设计模式。 你可能已经在实践中用过这个模式了。 Java API中的很多方法都可以用不同的行为来参数化。 这些方法往往与匿名类一起使用。 我们会展示三个例子， 这应该能帮助你巩固传递代码的思想了： 用一个Comparator排序，用Runnable执行一个代码块， 以及GUI事件处理。

### 4.1 用Comparator来排序
在Java 8中， List自带了一个sort方法（ 你也可以使用Collections.sort） 。 sort的行为可以用java.util.Comparator对象来参数化， 它的接口如下：
```java
// java.util.Comparator
public interface Comparator<T> {
    public int compare(T o1, T o2);
}
```
在使用中，可以通过匿名类、用Lambda表达式来实现List数据的排序
```java
// 匿名类
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
// Lambda表达式
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

### 4.2 用Runnable执行代码块
线程就像是轻量级的进程： 它们自己执行一个代码块。 但是， 怎么才能告诉线程要执行哪块代码呢？ 多个线程可能会运行不同的代码。 我们需要一种方式来代表稍候执行的一段代码。 在Java里， 你可以使用Runnable接口表示一个要执行的代码块。请注意， 代码不会返回任何结果（ 即void） ：
```java
// java.lang.Runnable
public interface Runnable{
    public void run();
}

// 匿名类
Thread t = new Thread(new Runnable() {
    public void run(){
        System.out.println("Hello world");
    }
});

// Lambda表达式
Thread t = new Thread(() -> System.out.println("Hello world"));
```

### 4.3 GUI事件处理
GUI编程的一个典型模式就是执行一个操作来响应特定事件， 如鼠标单击或在文字上悬停。 同样可以使用匿名类和Lambda表达式实现

## 5. 小结
* 行为参数化， 就是一个方法接受多个不同的行为作为参数， 并在内部使用它们， 完成不同行为的能力。
* 行为参数化可让代码更好地适应不断变化的要求， 减轻未来的工作量。
* 传递代码， 就是将新行为作为参数传递给方法。 但在Java 8之前这实现起来很啰嗦。 为接口声明许多只用一次的实体类而造成的啰嗦代码， 在Java 8之前可以用匿名类来减少。
* Java API包含很多可以用不同行为进行参数化的方法， 包括排序、 线程和GUI处理。