---
title: 桥接模式(Bridge Pattern)
date: 2017-11-29 15:06:00
tags: Structural Pattern
categories: Java Design Patterns
---

桥接模式属于结构型模式

<!-- more -->

# 意图

* 将抽象部分与实现部分分离，使它们都可以独立地变化

# 场景

当遇到下面场景时可以考虑使用桥接模式：

* 当一个类存在两个独立变化的维度，且这两个维度都需要扩展
* 不希望通过多层次继承实现扩展导致类的数量膨胀时
* 不希望抽象部分和实现部分固定绑定，需要实现部分可以在运行时选择
* 希望修改实现时不能影响客户端，实现和客户端解耦

# UML 类图

桥接模式类图如下：

![Java-Design-Patterns-Bridge.png](http://otg3f8t90.bkt.clouddn.com/2018/1/25/Java-Design-Patterns-Bridge.png)

类图说明：

* Abstraction：抽象部分的接口，通常定义的是业务相关的方法，该类持有一个实现部分对象的引用，业务方法需要调用实现对象来完成，该类通常定义为抽象对象，当然了也可以定义为接口
* RefinedAbstraction：扩展抽象部分接口，通常会扩展业务相关的方法
* Implementor：实现部分的接口，通常用于定义基本操作，供抽象部分调用
* ConcreteImplementor：实现部分的具体实现

# 实现

定义 Enchantment，对应类图中的 Implementor：

```java
public interface Enchantment {
    void onActivate();

    void apply();

    void onDeactivate();
}
```

定义 FlyingEnchantment、SoulEatingEnchantment，对应类图中的 ConcreteImplementor：

```java
public class FlyingEnchantment implements Enchantment {
    @Override
    public void onActivate() {
        System.out.println("The item begins to glow faintly.");
    }

    @Override
    public void apply() {
        System.out.println("The item flies and strikes the enemies finally returning to owner's hand.");

    }

    @Override
    public void onDeactivate() {
        System.out.println("The item's glow fades.");
    }
}
```

```java
public class SoulEatingEnchantment implements Enchantment {
    @Override
    public void onActivate() {
        System.out.println("The item spreads bloodlust.");
    }

    @Override
    public void apply() {
        System.out.println("The item eats the soul of enemies.");
    }

    @Override
    public void onDeactivate() {
        System.out.println("Bloodlust slowly disappears.");
    }
}
```

定义 Weapon，对应类图中的 Abstraction，该类通常被定义为抽象类，这里定义为接口：

```java
public interface Weapon {
    void wield();

    void swing();

    void unwield();

    Enchantment getEnchantment();
}
```

定义 Hammer、Sword，对应类图中的 RefinedAbstraction：

```java
public class Hammer implements Weapon {
    private Enchantment enchantment;

    public Hammer(Enchantment enchantment) {
        this.enchantment = enchantment;
    }

    @Override
    public void wield() {
        enchantment.onActivate();
    }

    @Override
    public void swing() {
        enchantment.apply();
        hit();
    }

    @Override
    public void unwield() {
        enchantment.onDeactivate();
    }

    @Override
    public Enchantment getEnchantment() {
        return enchantment;
    }

    // 扩展的业务方法
    public void hit() {
        System.out.println("Hit with a hammer.");
    }
}
```

```java
public class Sword implements Weapon {
    private Enchantment enchantment;

    public Sword(Enchantment enchantment) {
        this.enchantment = enchantment;
    }

    @Override
    public void wield() {
        enchantment.onActivate();
    }

    @Override
    public void swing() {
        enchantment.apply();
        stab();
    }

    @Override
    public void unwield() {
        enchantment.onDeactivate();
    }

    @Override
    public Enchantment getEnchantment() {
        return enchantment;
    }

    public void stab() {
        System.out.println("Stab with a sword.");
    }
}
```

在客户端调用：

```java
// 抽象部分子类和实现部分子类可以排列组合，可以有 m x n 种组合方法，比单纯的通过继承扩展减少很多子类
public class Client {
    public static void main(String[] args) {
        Sword flyingSword = new Sword(new FlyingEnchantment());
        invoke(flyingSword);

        Sword soulEatingSword = new Sword(new SoulEatingEnchantment());
        invoke(soulEatingSword);

        Hammer flyingHammer = new Hammer(new FlyingEnchantment());
        invoke(flyingHammer);

        Hammer soulEatingHammer = new Hammer(new SoulEatingEnchantment());
        invoke(soulEatingHammer);
    }

    private static void invoke(Weapon weapon) {
        weapon.wield();
        weapon.swing();
        weapon.unwield();
    }
}
```

# 总结

桥接模式的本质是分离抽象和实现。桥接模式桥接的是被抽离的抽象部分和实现部分，桥接体现在抽象部分拥有实现部分接口对象，维护桥接就是维护这个关系。

在 Java 中桥接模式无处不在，通常所说的“面向接口编程”就是广义桥接模式的提现。

桥接模式优点：

* 分离抽象部分和实现部分，使两者可以分别扩展并组合使用，具有较好的扩展性
* 对于一个抽象部分，可以动态选择实现部分
* 桥接模式主要是为了解决多维度变化的情况，相对于继承解决一维变化的情况，可以减少子类的定义

桥接模式缺点：

* 不易划分抽象部分和实现部分

# Ref

* [Bridge Pattern](http://www.oodesign.com/bridge-pattern.html)
* [java-design-patterns-bridge](https://github.com/iluwatar/java-design-patterns/blob/master/bridge/pom.xml)