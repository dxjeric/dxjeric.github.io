---
sort: 2
---
# 第二章 通过行为参数化传递代码

**内容简介: **
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
```
