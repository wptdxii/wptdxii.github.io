---
title: 工厂方法模式(Factory Method Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

工厂方法模式(Factory Method Pattern)又叫做虚拟构造(Virtual Constructor)，属于创建型模式(Creational Pattern)

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

定义 Shape 接口，对应类图中的 Product：

```java
public interface Shape {
    void draw();
}
```

定义 Shape 接口的实现：

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

定义 Drawer，对应类图中的 Creator：

```java
public abstract class Drawer {
    // 定义工厂方法
    protected abstract Shape factoryMethod();

    public void draw() {
        // 调用工厂方法
        Shape shape = factoryMethod();
        shape.draw();
    }
}
```

定义 Drawer 的子类实现：

```java
public class CircleDrawer extends Drawer {
    @Override
    protected Shape factoryMethod() {
        return new Circle();
    }
}
```

```java
public class RectangleDrawer extends Drawer {
    @Override
    protected Shape factoryMethod() {
        return new Rectangle();
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        // 这里可以结合工厂模式使用
        Drawer circleDrawer = new CircleDrawer();
        circleDrawer.draw();

        Drawer rectangleDrawer = new RectangleDrawer();
        rectangleDrawer.draw();
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