---
title: 原型模式(Prototype Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

原型模式(Prototype Pattern)是创建型模式(Creational Pattern)

<!-- more -->

# 意图

* 用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象

# 场景

* 类初始化需要消耗非常多的资源，可以通过原型拷贝避免这些消耗
* 通过 new 创建一个对象需要非常繁琐的数据准备或访问权限，这时可以使用原型模式直接拷贝对象
* 一个对象需要提供给其他多个对象访问，而各个调用者可能都需要修改其值的时，可以考虑使用原型模式拷贝多个对象供各个调用者使用，即保护性拷贝

# UML 类图

原型模式的类图如下：

************

类图说明：

* Prototype：声明克隆自身的接口，用来约束想要用来克隆自己的类。在 Java 语言中，可以不定义该接口，直接使用 Cloneable 接口，该接口是个标识接口，没有需要实现的方法，使用的时候需要重写 Object 类中的 clone() 方法
* ConcreteType：具体的原型类
* Client：客户端，首先获取原型实例，然后通过原型实例的克隆方法获取新的对象

# 实现

## 自定义实现

定义 Prototype 接口：

```java
public interface Prototype<T> {
    T clonePrototype();
}
```

定义 Graph，对应类图中的 ConcretePrototype：

```java
public class Graph implements Prototype<Graph> {
    private int id;
    private Shape shape = new Shape();
    private Color color = new Color();

    public void setShape(String shape) {
        this.shape.setShape(shape);
    }

    public void setColor(String color) {
        this.color.setColor(color);
    }

    public void setId(int id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return "Id:" + id + "/" + "Shape:" + shape + "/" + "Color:" + color;
    }

    @Override
    public Graph clonePrototype() {
        Graph graph = new Graph();
        // 浅拷贝
//        graph.id = id;
//        graph.shape = shape;
//        graph.color = color;

        // 深拷贝
        graph.id = id;
        graph.shape = shape.clonePrototype();
        graph.color = color.clonePrototype();
        return graph;
    }
}
```

## Cloneable 实现

# 总结

# Ref