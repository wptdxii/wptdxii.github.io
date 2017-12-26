---
title: 抽象工厂模式(Abstract Factory Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Pattern) 
categories: Java Design Patterns
---

抽象工厂模式属于创建型模式

<!-- more  -->

# 意图

* 抽象工厂为创建一组相关或相互依赖的对象提供接口，而不需要知道它们的具体实现。

# 场景

当遇到下列场景时可以考虑使用抽象工厂模式：

* 需要创建一系列对象，但只知道接口而不知道具体实现
* 并且一系列对象之间是有约束关系的，即只有特定的对象之间才能被同时创建

# UML 类图

抽象工厂模式类图如下：

![Java-Design-Patterns-Abstract-Factory.png](http://otg3f8t90.bkt.clouddn.com/2017/12/7/Java-Design-Patterns-Abstract-Factory.png)

类图说明：

* AbstractProduct：抽象的产品类，供客户端依赖
* ConcreteProduct：抽象产品的实现类，只暴露给抽象工厂的实现类。不同的对象之间是有约束关系的。如类图中 ConcreteProductA1 只能和 ConcreteProductB1 一起被创建，ConcreteProductA2 只能和 ConcreteProductB2 一起被创建
 AbstractFactory：抽象工厂，定义创建一系列相互约束的产品对象的接口
* ConcreteActory：抽象工厂的具体实现类，每个实现提供一个产品簇
* Client： 客户端，属于外部调用方，主要用来获取抽象工厂创建的产品簇。产品的实现类是不暴露给客户端的，客户端只依赖抽象的产品类。

# 实现

定义 Army、Castle、King 接口，对应类图中的 AbstractProduct：

```java
public interface Army {
    String getDescription();
}
```

```java
public interface Castle {
    String getDescription();
}
```

```java
public interface King {
    String getDescription();
}
```

实现 Army、Castle、King 接口，对应类图中的 ConcreteProduct：

```java
public class ElfArmy implements Army{
    @Override
    public String getDescription() {
        return "This is the Elven army!";
    }
}
```

```java
public class ElfCastle implements Castle {
    @Override
    public String getDescription() {
        return "This is the Elven castle!";
    }
}
```

```java
public class ElfKing implements King {
    @Override
    public String getDescription() {
        return "This is the Elven king!";
    }
}
```

```java
public class OrcArmy implements Army {
    @Override
    public String getDescription() {
        return "This is the Orc army!";
    }
}
```

```java
public class OrcCastle implements Castle {
    @Override
    public String getDescription() {
        return "This is the Orc castle!";
    }
}
```

```java
public class OrcKing implements King {
    @Override
    public String getDescription() {
        return "This is the Orc king!";
    }
}
```

定义 KingdomFactory，对应类图中的 AbstractFActory：

```java
public interface KingdomFactory {
    Army createArmy();

    Castle createCastle();

    King createKing();
}
```

实现 KingdomFactory 接口，对应类图中的 ConcreteFactory：

```java
public class ElfKingdomFactory implements KingdomFactory {
    @Override
    public Army createArmy() {
        return new ElfArmy();
    }

    @Override
    public Castle createCastle() {
        return new ElfCastle();
    }

    @Override
    public King createKing() {
        return new ElfKing();
    }
}
```

```java
public class OrcKingdomFactory implements KingdomFactory {
    @Override
    public Army createArmy() {
        return new OrcArmy();
    }

    @Override
    public Castle createCastle() {
        return new OrcCastle();
    }

    @Override
    public King createKing() {
        return new OrcKing();
    }
}
```

定义 KingdomType 枚举，用于标识抽象工厂类型：

```java
public enum KingdomType {
    ELF, ORC
}
```

定义 FactoryMaker，用于选择对应的抽象工厂实现，这里用到了简单工厂模式：

```java
public final class FactoryMaker {

    private FactoryMaker() {
        throw new UnsupportedOperationException("Can't be instantiated");
    }

    public static KingdomFactory makeFactory(KingdomType type) {
        switch (type) {
            case ELF:
                return new ElfKingdomFactory();
            case ORC:
                return new OrcKingdomFactory();
            default:
                throw new IllegalArgumentException("KingdomType not supported.");
        }
    }
}
```

定义 Kingdom，用于封装产品对象：

```java
public final class Kingdom {
    private Army army;
    private Castle castle;
    private King king;

    public void createKingdom(KingdomFactory kingdomFactory) {
        setArmy(kingdomFactory.createArmy());
        setCastle(kingdomFactory.createCastle());
        setKing(kingdomFactory.createKing());
    }

    public Army getArmy() {
        return army;
    }

    public void setArmy(Army army) {
        this.army = army;
    }

    public Castle getCastle() {
        return castle;
    }

    public void setCastle(Castle castle) {
        this.castle = castle;
    }

    public King getKing() {
        return king;
    }

    public void setKing(King king) {
        this.king = king;
    }
}
```

客户端调用:

```java
public class Client {
    public static void main(String[] args) {
        Kingdom kingdom = new Kingdom();
        kingdom.createKingdom(FactoryMaker.makeFactory(KingdomType.ELF));
        getDescription(kingdom);

        kingdom.createKingdom(FactoryMaker.makeFactory(KingdomType.ORC));
        getDescription(kingdom);
    }

    private static void getDescription(Kingdom kingdom) {
        System.out.println(kingdom.getArmy().getDescription());
        System.out.println(kingdom.getCastle().getDescription());
        System.out.println(kingdom.getKing().getDescription());
    }
}
```

# 相关模式

* 抽象工厂模式和工厂方法模式

    当抽象工厂的产品簇只有一个产品的时候，就退化成了工厂方法模式。

* 抽象工厂模式和单例模式

    在抽象工厂模式中，具体的工厂实现在整个应用中通常只需要一个实例就够了，因此可以把具体的工厂实现为单例模式

# 总结

抽象工厂模式的本质是：选择产品簇的实现。主要用来约束客户端对产品对象的使用。客户端只能通过抽象工厂的实现来选择产品对象，而不能直接自己创建产品对象，以免破坏产品簇之间的约束，造成不同产品对象组合时的不兼容。

抽象工厂模式优点：

* 客户端使用抽象工厂的实现来创建需要的产品对象，只知道产品接口而不知道其具体实现，两者完成解耦，实现了面向接口编程

* 客户端只能通过抽象工厂的实现创建需要的产品对象，而不能自己随意创建，保证了产品对象间的约束关系

* 客户端选用不同抽象工厂的实现，就可以完成不同产品簇的切换

抽象工厂模式缺点：

* 不易扩展产品类，当扩展产品类时，抽象工厂及其实现类都要更改

# Ref

* [Abstract Factory](http://www.oodesign.com/abstract-factory-pattern.html)
* [java-design-patterns-abstract-factory](https://github.com/iluwatar/java-design-patterns/blob/fb5c2a03246c1863e487cac2d1583ad04e1c4e4a/abstract-factory/README.md)