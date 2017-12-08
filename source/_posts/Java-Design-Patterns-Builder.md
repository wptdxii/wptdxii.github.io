---
title: 建造者模式(Builder Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

建造者模式(Builder Pattern)也叫做生成器模式，是创建型模式(Creational Pattern)

<!-- more -->

# 意图

* 将一个复杂对象的构建和它的表示分离，使得同样的构建过程可以创建不同的表示

# 场景

当遇到下列场景时可以考虑使用建造者模式：

* 创建对象的过程(算法)，应该独立于该对象的组成部分和它们的装配方式
* 相同的构建过程有着不同的表示
* 对象的构造比较复杂，有着众多的参数和配置

# UML 类图

建造者模式的类图如下：

![Java-Design-Patterns-Builder.png](http://otg3f8t90.bkt.clouddn.com/2017/12/8/Java-Design-Patterns-Builder.png)

类图说明：

* Product：产品类，一般为具体类，不必要抽象，其由 PartA、PartB 组成。
* Builder：建造者接口，定义创建一个 Product 对象所需的各个部件的接口和返回 Product 对象的接口
* ConcreteBuilder：Builder 接口的具体实现，负责 Product 对象所需的各个部件的创建和组装，同时提供一个获取组装完成的 Product 对象的方法。切换不同的实现，即可通过相同的构建过程构建不同的 Product 对象
* Director：指导者，主要用来使用 Builder 接口，以统一的过程构建所需的 Product 对象。为不同的 Builder 实现提供相同的构建过程
* Client：客户端，通过 Director 和具体的 Builder 实现来构建复杂的 Product 对象

# 实现

## 标准实现

定义 枚举 Shape、Color、Size，对应类图中的 PartA、PartB：

```java
public enum Shape {
    CIRCLE, RECTANGLE;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

```java
public enum Color {
    WHITE,BLACK;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

```java
public enum Size {
    LARGE, SMALL;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

定义 Graph，对应类图中的 Product：

```java
public final class Graph {
    private Shape shape;
    private Color color;
    private Size size;

    // 防止在包外通过 new 创建，修饰符设置为 default
    Shape getShape() {
        return shape;
    }

    public void setShape(Shape shape) {
        this.shape = shape;
    }

    public Color getColor() {
        return color;
    }

    // setter 同时设置为 default
    void setColor(Color color) {
        this.color = color;
    }

    public Size getSize() {
        return size;
    }

    void setSize(Size size) {
        this.size = size;
    }

    @Override
    public String toString() {
        return size + " " + color + " " + shape;
    }
}
```

定义 Builder：

```java
public interface Builder {
    void buildShape(Shape shape);

    void buildColor(Color color);

    void buildSize(Size size);

    Graph build();
}
```

定义 Builder 的实现：

```java
public class ConcreteBuilder implements Builder {
    private Graph graph = new Graph();

    @Override
    public void buildShape(Shape shape) {
        graph.setShape(shape);
    }

    @Override
    public void buildColor(Color color) {
        graph.setColor(color);
    }

    @Override
    public void buildSize(Size size) {
        graph.setSize(size);
    }

    @Override
    public Graph build() {
        // 可以在这里添加约束规则
        return graph;
    }
}
```

定义 Director：

```java
public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    // 这里只是简单的顺序调用，实际应用中通常都是定义的比较复杂的构建算法
    public void construct(Shape shape, Color color, Size size) {
        builder.buildShape(shape);
        builder.buildColor(color);
        builder.buildSize(size);
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        // 可以结合简单工厂使用
        Builder builder = new ConcreteBuilder();
        Director director = new Director(builder);
        director.construct(Shape.CIRCLE, Color.WHITE, Size.SMALL);
        Graph graph = builder.build();
        System.out.println(graph);
    }
}
```

## 简化实现

# 总结

# Ref