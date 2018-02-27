---
title: 工厂模式(Factory Pattern)
date: 2016-11-29 15:06:0
tags: Creational Pattern
categories: Java Design Patterns
---

工厂模式又叫做简单工厂模式或者静态工厂模式，属于创建型模式，不是一个标准的设计模式，不属于　GoF 23 种设计模式之一

<!-- more -->

# 意图

* 在不暴露内部实现的情况下为外部调用创建接口实例
* 为外部调用提供创建接口实例的统一接口

# 场景

面向接口编程是面向对象编程的一个重要原则。接口的思想是“封装隔离”，“封装”指的是对被隔离体的行为和职责进行封装，“隔离”指的是对外部调用和内部实现进行隔离。只要接口不变，内部实现的变化就不会影响到外部调用，从而使系统具有更好的扩展性和可维护性。接口保证了系统的可插拔性。但在面向接口编程时会出现外部调用只知道接口而不知道具体实现的问题，工厂模式就是为了解决此类问题。

> 这里的接口在 Java 中通常指的是 接口(Interface)和抽象类(Abstract Class)

# UML 类图

工厂模式的类图如下：

![Java-Design-Patterns-Factory.png](http://otg3f8t90.bkt.clouddn.com/2017/12/27/Java-Design-Patterns-Factory.png)

类图说明：

* Product：抽象的产品类
* ConcreteProduct：具体的产品实现
* Type：具体产品的标识，工厂类通过该标识创建具体的产品对象
* Factory：工厂，根据条件选择合适的产品实现
* Client：客户端
* 类图中 Type、Product、Factory 是暴露给客户端的，而 ConcreteProduct 对客户端是透明的，但是客户端需要根据 Type 来选择实现，所以 ConcreteProduct 对客户端还是有一定程度的暴露

# 实现

定义 Warcraft，对应类图中的 Product：

```java
public interface Warcraft {
    void fight();
}
```

定义 Warcraft 的实现，对应类图中的 ConcreteProduct：

```java
public class Thief implements Warcraft {
    @Override
    public void fight() {
        System.out.println("Thief fight");
    }
}
```

```java
public class Warrior implements Warcraft {
    @Override
    public void fight() {
        System.out.println("Warrior fight");
    }
}
```

定义 WarcraftType，对应类图中的 Type：

```java
public enum WarcraftType {
    THIEF, WARRIOR
}
```

定义 WarcraftFactory，对应类图中的 Factory：

```java
public class WarcraftFactory {
    private WarcraftFactory() {
        throw new UnsupportedOperationException("Can't be instantiated");
    }

    public static Warcraft createWarcraft(WarcraftType type) {
        switch (type) {
            case THIEF:
                return new Thief();
            case WARRIOR:
                return new Warrior();
            default:
                throw new IllegalArgumentException("WarcraftType not supported.");
        }
    }
}
```

客户端调用：

```java
public class Client {

    public static void main(String[] args) {
        Warcraft warrior = WarcraftFactory.createWarcraft(WarcraftType.WARRIOR);
        warrior.fight();

        Warcraft thief = WarcraftFactory.createWarcraft(WarcraftType.THIEF);
        thief.fight();
    }
}
```

# 总结

工厂模式可以看做是工厂方法模式的弱化版本，其本质是：选择实现。

工厂模式优点：

* 帮助外部调用和内部实现解耦，使外部调用实现面向接口编程

工厂模式缺点：

* 需要向外部调用暴露一定的内部实现细节，如外部调用需要知道工厂选择实现的条件的对应关系
* 不符合开闭原则，当需要扩展接口实例时，需要同时修改工厂和关联类型的枚举

# Ref

* [Factory Pattern](http://www.oodesign.com/factory-pattern.html)