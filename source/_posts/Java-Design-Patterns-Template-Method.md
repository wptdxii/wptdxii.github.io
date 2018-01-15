---
title: 模板方法模式(Template Method Pattern)
date: 2017-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

模板方法模式属于行为型模式

<!-- more -->

# 意图

* 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中，使子类可以不改变一个算法的结构即可重新定义该算法的某些特定的步骤

# 场景

当遇到下面场景时可以考虑使用模板方法模式：

* 需要固定算法骨架，实现一个算法固定不变的部分，并把需要变化的部分留给子类实现的情况
* 各个子类有公共的行为，应该抽取出来，集中在父类中实现，避免避免代码重复
* 当需要控制子类子类扩展时，因为模板方法会在某些特定的点调用子类的方法，这样只允许在这些点进行扩展

# UML 类图

模板方法模式类图如下：

![Java-Design-Pattern-Template-Method.png](http://otg3f8t90.bkt.clouddn.com/2018/1/10/Java-Design-Pattern-Template-Method.png)

类图说明：

* AbstractClass：抽象类，用来定义算法骨架、原语操作和钩子操作，其中算法骨架指的是模板方法(Template Method)，原语操作(Primitive Operation)指的是需要子类实现的抽象方法，钩子操作(Hook Operation)指的是提供默认操作并且可以被子类根据需要重写的方法。该抽象类中也可以包含工厂方法，即模板方法模式可以与工厂模式结合使用
* ConcreteClass：具体实现类，用来实现算法骨架中的某些步骤，完成与特定子类相关的功能

# 实现

定义 StealingMethod，对应类图中的 AbstractClass：

```java
public abstract class StealingMethod {
    // factory method
    protected abstract String pickTarget();

    // primitive method，即原语操作
    protected abstract void confuseTarget(String target);

    // primitive method，即原语操作
    protected abstract void stealTheItem(String target);

    // hook method，即钩子操作，子类可以根据需要重写
    protected void wander() {
        System.out.println("The thief wanders around the street.");
    }

    // template method，即模板方法，封装骨架算法，需要使用 final 修饰防止子类修改
    public final void steal() {
        wander();
        String target = pickTarget();
        confuseTarget(target);
        stealTheItem(target);
    }
}
```

定义 ClumsyMethod、SubtleMethod，对应类图中的 ConcreteClass：

```java
public class ClumsyMethod extends StealingMethod{
    @Override
    protected String pickTarget() {
        return "old goblin woman";
    }

    @Override
    protected void confuseTarget(String target) {
        System.out.println("Approach the " + target + " from behind.");
    }

    @Override
    protected void stealTheItem(String target) {
        System.out.println("Grab the handbag and run away fast!");
    }
}
```

```java
public class SubtleMethod extends StealingMethod {
    @Override
    protected String pickTarget() {
        return "shop keeper";
    }

    @Override
    protected void confuseTarget(String target) {
        System.out.println("Approach the " + target + " with tears running and hug him!");
    }

    @Override
    protected void stealTheItem(String target) {
        System.out.println("While in close contact grab the " + target + "'s wallet.");
    }

    // 重写父类钩子方法
    @Override
    protected void wander() {
        System.out.println("The thief wanders around the shop.");
    }
}
```

定义 Thief，管理模板：

```java
public class Thief {
    private StealingMethod method;

    public Thief(StealingMethod method) {
        this.method = method;
    }

    public void changeMethod(StealingMethod method) {
        this.method = method;
    }

    public void steal() {
        this.method.steal();
    }
}
```

客户端调用：

```java
public class Client {

    public static void main(String[] args) {
        Thief thief = new Thief(new ClumsyMethod());
        thief.steal();

        thief.changeMethod(new SubtleMethod());
        thief.steal();
    }
}
```

# 总结

模板方法模式的本质是固定算法骨架。模板方法模式和工厂方法模式的类图结构类似，但侧重不同，工厂方法作为抽象方法是用于创建对象，由子类选择实现，而模板方法用来封装算法骨架，其本身并不是抽象的，而其调用的算法步骤是抽象方法，需由子类实现。算法步骤可以包含工厂方法，所以模板方法模式和工厂方法模式可以结合使用

模板方法模式优点：

* 实现代码复用，将子类的公共功能提炼和抽取，将其放到模板中实现
* 可以根据需求扩展原语操作，有较强的扩展性
* 提现了开闭原则和里式替换原则

模板方法模式缺点：

* 算法骨架不易修改，当需要更改算法骨架时，需要重构所有的子类

# Ref

* [Template Method](http://www.oodesign.com/template-method-pattern.html)
* [java-design-patterns-template-method](https://github.com/iluwatar/java-design-patterns/blob/master/template-method/README.md)