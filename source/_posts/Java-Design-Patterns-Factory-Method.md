---
title: 工厂方法模式(Factory Method Pattern)
date: 2016-11-29 15:06:00
tags: [Creational Pattern, GoF]
categories: Java Design Patterns
---

工厂方法模式又叫做虚拟构造(Virtual Constructor)，属于创建型模式

<!-- more -->

# 意图

* 定义一个用于创建对象的接口，让子类决定实例化哪个类，也就是将一个类的实例化延迟到其子类。
* 通过统一的接口获取创建的对象

# 场景

当遇到下列场景时可以考虑使用工厂方法模式：

* 一个类在实现功能的过程中需要依赖一个接口实例，但不知道其具体实现
* 或者一个类本身就希望由它的子类来创建所需对象

# UML 类图

工厂方法模式的类图如下：

![Java-Design-Patterns-Factory-Method.png](http://otg3f8t90.bkt.clouddn.com/2017/12/6/Java-Design-Patterns-Factory-Method.png)

类图说明：

* Product：工厂方法所创建的对象的接口
* ConcreteProduct：Product 接口的实现类
* Creator：创建器，声明了工厂方法。工厂方法返回 Product 对象，一般会被声明为抽象方法，但也可以提供缺省实现，特别是如果工厂方法接受类型参数的话，一般都会提供缺省实现。工厂方法通常被 Creator 自身的其他方法调用以实现功能。
* ConcreteCreator：Creator 的具体实现类，覆盖实现工厂方法，返回具体的 Product 实例。

# 实现

定义 Weapon，对应类图中的 Product：

```java
public interface Weapon {
    void attack();
}
```

定义 Weapon 的实现，对应类图中的 ConcreteProduct：

```java
public class Bow implements Weapon {
    @Override
    public void attack() {
        System.out.println("Attack with bow");
    }
}
```

```java
public class Sword implements Weapon {
    @Override
    public void attack() {
        System.out.println("Attack with sword");
    }
}
```

定义 Warcraft，其中定义了工厂方法，对应类图中的 Creator：

```java
public abstract class Warcraft {
    // 工厂方法
    protected abstract Weapon manufacture();

    public void fight() {
        // 调用工厂方法
        Weapon weapon = manufacture();
        weapon.attack();
    }
}
```

定义 Warcraft 的实现，对应类图中的 ConcreteCreator：

```java
public class Thief extends Warcraft {
    @Override
    protected Weapon manufacture() {
        return new Bow();
    }
}
```

```java
public class Warrior extends Warcraft{
    @Override
    protected Weapon manufacture() {
        return new Sword();
    }
}
```

定义 WarcraftType，用于标识不同的 ConcreteCreator：

```java
public enum WarcraftType {
    WARRIOR, THIEF
}
```

定义 WarcraftFactory，这里用到了简单工厂模式：

```java
public final class WarcraftFactory {
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

工厂方法模式的本质是延迟到子类来选择实现。

工厂方法模式中，客户端可以使用 Creator 对象也可以使用 Creator 工厂方法创建的对象，但多数情况下应该是 Creator 的其他方法在使用工厂方法提供的对象，工厂方法不提供给 Creator 外部使用。如果工厂方法有接受类型参数，那么 Creator 的工厂方法一般会提供缺省实现。

工厂模式优点：

* 符合开闭原则和依赖倒置原则
* 工厂方法为子类提供了挂钩，具有良好的扩展性
* 可以连接平行的类层次

工厂模式缺点：

* 需要为被延迟创建的对象提供抽象层
* 扩展产品类(Product)的同时也要扩展相应的创建器(Creator)

# Ref

* [Factory Method Pattern](http://www.oodesign.com/factory-method-pattern.html)
* [java design patterns-factory method](https://github.com/iluwatar/java-design-patterns/blob/master/factory-method/README.md)