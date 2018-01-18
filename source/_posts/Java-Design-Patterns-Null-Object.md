---
title: 空对象模式(Null Object Pattern)
date: 2017-11-29 15:06:00
tags: Behavioral Pattern
categories: Java Design Patterns
---

访问者模式属于行为型模式，不是一个标准的设计模式，不属于　GoF 23 种设计模式之一

<!-- more -->

# 意图

* 提供一个给定类型的空代理对象，该对象不执行任何操作，对外部隐藏实现细节

# 场景

* 当需要频繁地判空检查时可以考虑使用空对象模式

# UML 类图

空对象模式类图如下：

![Java-Design-Patterns-Null-Object.png](http://otg3f8t90.bkt.clouddn.com/2018/1/18/Java-Design-Patterns-Null-Object.png)

类图说明：

* AbstractObject：抽象对象，定义子类需要实现的操作
* RealObject：抽象对象的真正实现
* NullObject：抽象对象的空实现，可以定制输出信息，根据需要可以实现为单例
* Client：客户端

# 实现

定义 Node，对应类图中的 AbstractObject：

```java
public interface Node {
    void walk();
}
```

定义 NodeImpl，对应类图中的 RealObject：

```java
public class NodeImpl implements Node {
    private String name;

    public NodeImpl(String name) {
        this.name = name;
    }

    @Override
    public void walk() {
        System.out.println("Node name:" + name);
    }
}
```

定义 NullNode，对应类图中的 NullObject：

```java
// 如果频繁地创建空对象，可以将其实现为单例
public class NullNode implements Node {
    private NullNode() {
        if (SingletonHolder.INSTANCE != null) {
            throw new UnsupportedOperationException("Already instantiated.");
        }
    }

    public static NullNode getInstance() {
        return SingletonHolder.INSTANCE;
    }

    @Override
    public void walk() {
        // Do nothing
        // 可以定制空对象情况下的输出
    }

    private static class SingletonHolder {
        private static final NullNode INSTANCE = new NullNode();
    }
}
```

定义 NodeFactory，用于选择实现:

```java
// 这里使用了工厂模式
public class NodeFactory {
    private NodeFactory() {
        throw new UnsupportedOperationException("Can't instantiated.");
    }

    public static Node createNode(String name) {
        if (name == null || name.trim().isEmpty()) {
            // 使用空对象模式，如果不满足创建条件，返回的是空对象而不必是 null，避免了调用方的判空
            return NullNode.getInstance();
        }
        return new NodeImpl(name);
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        // 客户端调用时不会担心 node 对象为 null
        Node node = NodeFactory.createNode("1");
        node.walk();
        node = NodeFactory.createNode(null);
        node.walk();
    }
}
```

# 总结

空对象模式优点：

* 可以优雅有效地防止空指针
* 可以方便地定制空对象情况的输出

空对象模式缺点：

* 需要提取抽象并增加空对象类

# Ref

* [Null Object Pattern](http://www.oodesign.com/null-object-pattern.html)
* [java-design-patterns-null-object](https://github.com/iluwatar/java-design-patterns/blob/master/null-object/README.md)