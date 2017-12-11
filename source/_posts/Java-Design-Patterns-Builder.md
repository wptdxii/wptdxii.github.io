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

* Product：被构建的对象，一般为具体类，不必要定义抽象接口，其由 PartA、PartB 组成。
* Builder：建造者接口，定义创建一个 Product 对象所需的各个部件的接口和返回 Product 对象的接口。如果有需要，可以在这两个地方添加约束规则
* ConcreteBuilder：Builder 接口的具体实现，负责 Product 对象所需的各个部件的创建和组装，同时提供一个获取组装完成的 Product 对象的方法。切换不同的实现，即可通过相同的构建过程构建不同的 Product 对象
* Director：主要用来使用 Builder 接口，以统一的过程构建所需的 Product 对象，代表可重用的构建过程。在构建的过程中在需要创建和组装具体部件的时候，将这些功能通过委托，交给　Builder 去完成。Ｄirector 通过　Builder 方法的参数和返回值与　Builder 进行交互
* Client：外部调用，通过 Director 和具体的 Builder 实现来构建复杂的 Product 对象

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

    // 该方法在　Ｂuilder 接口中不是必须的
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

建造者模式更常用的是其简化形式，就是将　Client 和　Ｄirector 融合，同时将　Builder　内联到　Product　中去，通过链式调用构建　Product 对象。
具体实现如下：

```java
public final class Graph {
    private Shape shape;
    private Color color;
    private Size size;

    private BuilderGraph(Builder builder) {
        this.shape = builder.shape;
        this.color = builder.color;
        this.size = builder.size;
    }

    public Shape getShape() {
        return shape;
    }

    public Color getColor() {
        return color;
    }

    public Size getSize() {
        return size;
    }

    @Override
    public String toString() {
        return size + " " + color + " " + shape;
    }

    public static class Builder {
        private Shape shape;
        private Color color;
        private final Size size;

        public Builder(Size size) {
            this.size = size;
        }

        // return this　实现链式调用
        public Builder shape(Shape shape) {
            this.shape = shape;
            return this;
        }

        public Builder color(Color color) {
            this.color = color;
            return this;
        }

        // 可以在这里添加约束规则
        public BuilderGraph build() {
            return new BuilderGraph(this);
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Graph graph = new BuilderGraph.Builder(Size.LARGE)
                .shape(Shape.RECTANGLE)
                .color(Color.BLACK)
                .build();
        System.out.println(graph);
    }
}
```

# 总结

建造者模式的主要功能是细化的、分步骤的构建复杂的对象，其本质在于分离构建算法和具体的构建实现，从而使构建算法可以复用，使具体的构造实现可以方便地扩展和切换。建造者模式强调的是对整体构建算法的固定和对具体构建实现的灵活切换。

建造者模式的优点：

* Director 可复用
* Builder 易扩展，实现　Builder 接口即可
* 良好的封装性，Product 内部组成的细节对 Client 透明

建造者模式的缺点：

* 会产生多余的　Director 对象和　Builder 对象，消耗内存

# Ref

* [Builder Pattern](http://www.oodesign.com/builder-pattern.html)
* [java-design-patterns-builder](https://github.com/iluwatar/java-design-patterns/blob/master/builder/README.md)