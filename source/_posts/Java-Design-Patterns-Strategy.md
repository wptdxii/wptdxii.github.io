---
title: 策略模式(Strategy Pattern)
date: 2017-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

策略模式属于行为型模式

<!-- more -->

# 意图

* 定义一系列算法，并将每个算法封装起来，使它们之间可以相互替换。策略模式可以使算法独立于使用它的客户端而变化

# 场景

当遇到下面场景时可以考虑使用策略模式：

* 针对同一个问题有多种处理方式，仅仅是具体行为有差别，且需要动态切换处理方式
* 封装算法时，算法有用到但不应该暴露给客户端的数据时，可以使用策略模式，将数据封装到上下文中
* 当同一个抽象类有多个子类，且需要选择语句选择实现时

# UML 类图

策略模式类图如下：

![Java-Design-Pattern-Strategy.png](http://otg3f8t90.bkt.clouddn.com/2018/1/8/Java-Design-Pattern-Strategy.png)

类图说明：

* Strategy：策略接口，用来约束策略算法
* ConcreteStrategy：策略的具体实现，如果算法需要外部数据，可以将其封装到 Context，然后传递给具体的策略实现
* Context：上下文，持有策略对象，用来操作策略，可以封装策略算法需要的数据，将自身传递给策略对象，但这对于可能不需要这些数据的策略对象会造成冗余
* 策略的选择可以放到客户端，也可以放到上下文中

# 实现

定义 DragonSlayingStrategy，对应类图中的 Strategy：

```java
@FunctionalInterface
public interface DragonSlayingStrategy {
    void execute();
}
```

定义 MeleeStrategy、ProjectileStrategy、SpellStrategy，对应类图中的 ConcreteStrategy：

```java
public class MeleeStrategy implements DragonSlayingStrategy {
    @Override
    public void execute() {
        System.out.println("Sever the dragon's head in he melee!");
    }
}
```

```java
public class ProjectileStrategy implements DragonSlayingStrategy {
    @Override
    public void execute() {
        System.out.println("Shoot the dragon with the magical crossbow!");
    }
}
```

```java
public class SpellStrategy implements DragonSlayingStrategy{
    @Override
    public void execute() {
        System.out.println("Cast the spell of disintegration and the dragon vaporizes in a pile of dust!");
    }
}
```

定义 DragonSlayer，对应类图中的 Context：

```java
public class DragonSlayer {
    private DragonSlayingStrategy strategy;

    public DragonSlayer(DragonSlayingStrategy strategy) {
        this.strategy = strategy;
    }

    public void changeStrategy(DragonSlayingStrategy strategy) {
        this.strategy = strategy;
    }

    public void goToBattle() {
        this.strategy.execute();
    }
}
```

客户端调用：

```java
public class Client {

    public static void main(String[] args) {
        DragonSlayer slayer = new DragonSlayer(new MeleeStrategy());
        slayer.goToBattle();

        slayer.changeStrategy(new ProjectileStrategy());
        slayer.goToBattle();

        slayer.changeStrategy(new SpellStrategy());
        slayer.goToBattle();

        // Java 8 Strategy pattern，lambda 语法的匿名内部类
        slayer = new DragonSlayer(() -> System.out.println("Sever the dragon's head in he melee!"));
        slayer.goToBattle();

        slayer.changeStrategy(() -> System.out.println("Shoot the dragon with the magical crossbow!"));
        slayer.goToBattle();

        slayer.changeStrategy(() -> System.out.println("Cast the spell of disintegration and the dragon vaporizes in a pile of dust!"));
        slayer.goToBattle();
    }
}
```

# 总结

策略模式的本质是分离算法，选择实现。其重心不是如何实现算法，而是如何组合、调用这些算法，从而增加系统的灵活性。策略模式要求策略算法具有平等性，即策略算法应该是相同行为的的不同实现，这样才能实现动态选择不同的策略对象。策略模式在运行时，每个时刻只能使用一个具体的策略对象。

策略模式优点:

* 封装策略算法，实现动态替换
* 避免多重条件语句
* 容易增加新的策略，具有良好的扩展性

策略模式缺点:

* 客户端必须了解每种策略的不同，一定程度上暴露了策略对象的内部实现
* 只适合扁平的算法结构，策略算法要求具有平等性
* 随着策略的增加，子类会变得繁多

# Ref

* [Strategy](http://www.oodesign.com/strategy-pattern.html)
* [java-design-patterns-strategy](https://github.com/iluwatar/java-design-patterns/blob/master/strategy/README.md)