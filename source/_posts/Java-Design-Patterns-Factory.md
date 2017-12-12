---
title: 工厂模式(Factory Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

工厂模式(Factory Pattern)又叫做简单工厂模式或者静态工厂模式，属于创建型模式(Creational Pattern)，可以看做是工厂方法模式(Factory Method Pattern)的弱化版本。工厂模式不是一个标准的设计模式，不属于　GoF 23 种设计模式之一

<!-- more -->

# 意图

* 在不暴露内部实现的情况下为外部调用创建接口实例
* 为外部调用提供创建接口实例的统一接口

# 场景

面向接口编程是面向对象编程的一个重要原则。接口的思想是“封装隔离”，“封装”指的是对被隔离体的行为和职责进行封装，“隔离”指的是对外部调用和内部实现进行隔离。只要接口不变，内部实现的变化就不会影响到外部调用，从而使系统具有更好的扩展性和可维护性。接口保证了系统的可插拔性。但在面向接口编程时会出现外部调用只知道接口而不知道具体实现的问题，工厂模式就是为了解决此类问题。

> 这里的接口在 Java 中通常指的是 接口(Interface)和抽象类(Abstract Class)

# UML 类图

工厂模式的类图如下：

![Java-Design-Patterns-Factory.png](http://otg3f8t90.bkt.clouddn.com/2017/12/5/Java-Design-Patterns-Factory.png)

类图说明：

* 工厂、枚举类型和接口是暴露给外部的
* 外部调用通过工厂获取接口实例而非自己创建，实现了面向接口编程
* 使用枚举关联接口实例的类型，在工厂中通过枚举类型选择实现

# 实现

定义 Shape 接口：

```java
public interface Shape {
    void draw();
}
```

实现 Shape 接口：

```java
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("draw:" + this);
    }

    @Override
    public String toString() {
        return getClass().getSimpleName();
    }
}

```

```java

public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("draw:" + this);
    }

    @Override
    public String toString() {
        return getClass().getSimpleName();
    }
}
```

根据 Shape 接口的具体实现定义枚举：

```java
public enum ShapeType {
    CIRCLE, RECTANGLE
}
```

创建工厂：

```java
public final class ShapeFactory {
    private ShapeFactory() {
        throw new UnsupportedOperationException("Can't be initialized");
    }

    public static Shape createShape(ShapeType type) {
        Shape shape;
        switch (type) {
            case CIRCLE:
                shape = new Circle();
                break;
            case RECTANGLE:
                shape = new Rectangle();
                break;
            default:
                shape = null;
        }
        return shape;
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Shape circle = ShapeFactory.createShape(ShapeType.CIRCLE);
        circle.draw();

        Shape rectangle = ShapeFactory.createShape(ShapeType.RECTANGLE);
        rectangle.draw();
    }
}
```

# 总结

工厂模式的本质是：选择实现。

工厂模式优点：

* 帮助外部调用和内部实现解耦，使外部调用实现面向接口编程

工厂模式缺点：

* 需要向外部调用暴露一定的内部实现细节，如外部调用需要知道工厂选择实现的条件的对应关系
* 不符合开闭原则，当需要扩展接口实例时，需要同时修改工厂和关联类型的枚举

# Ref

* [Factory Pattern](http://www.oodesign.com/factory-pattern.html)