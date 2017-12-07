---
title: 抽象工厂模式(Abstract Factory Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

抽象工厂模式(Abstract Factory Pattern)是创建型模式(Creational Pattern)

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

定义 Shape 接口， 对应类图中的 AbstractProductA：

```java
public interface Shape {
    void draw();
}
```

定义 Shape 接口的实现：

```java
public class CircleShape implements Shape {
    @Override
    public void draw() {
        System.out.println("draw:" + getClass().getSimpleName());
    }
}
```

```java
public class RectangleShape implements Shape {
    @Override
    public void draw() {
        System.out.println("draw:" + getClass().getSimpleName());
    }
}
```

定义 Area 接口，对应类图中的 AbstractProductB：

```java
public interface Area {
    void calculate();
}
```

定义 Area 接口的实现：

```java
public class CircleArea implements Area {
    private static final String AREA = "S = πr²";

    @Override
    public void calculate() {
        System.out.println("Area:" + AREA);
    }
}
```

```java
public class RectangleArea implements Area {
    private static final String AREA = "S = ab";

    @Override
    public void calculate() {
        System.out.println("Area:" + AREA);
    }
}
```

定义抽象工厂，对应类图中的 AbstractFactory：

```java
public interface AbstractFactory {
    Shape createShape();

    Area createArea();
}
```

定义抽象工厂的实现：

```java
public class CircleFactory implements AbstractFactory {
    @Override
    public Shape createShape() {
        return new CircleShape();
    }

    @Override
    public Area createArea() {
        return new CircleArea();
    }
}
```

```java
public class RectangleFactory implements AbstractFactory {
    @Override
    public Shape createShape() {
        return new RectangleShape();
    }

    @Override
    public Area createArea() {
        return new RectangleArea();
    }
}
```

定义客户端：

```java
public class Client {
    public static void main(String[] args) {
        // 可以结合简单工厂
        AbstractFactory circleFactory = new CircleFactory();
        assemble(circleFactory.createShape(),circleFactory.createArea());

        AbstractFactory rectangleFactory = new RectangleFactory();
        assemble(rectangleFactory.createShape(), rectangleFactory.createArea());
    }

    private static void assemble(Shape shape, Area area) {
        shape.draw();
        area.calculate();
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

* 分离接口与实现

    客户端使用抽象工厂的实现来创建需要的产品对象，只知道产品接口而不知道其具体实现，两者完成解耦，实现了面向接口编程

* 约束产品对象

    客户端只能通过抽象工厂的实现创建需要的产品对象，而不能自己随意创建，保证了产品对象间的约束关系

* 容易切换产品簇

    客户端选用不同抽象工厂的实现，就可以完成不同产品簇的切换

抽象工厂模式缺点：

* 不易扩展产品类

    当扩展产品类时，抽象工厂及其实现类都要更改

# Ref

* [Abstract Factory](http://www.oodesign.com/abstract-factory-pattern.html)
* [java-design-patterns-abstract-factory](https://github.com/iluwatar/java-design-patterns/blob/fb5c2a03246c1863e487cac2d1583ad04e1c4e4a/abstract-factory/README.md)