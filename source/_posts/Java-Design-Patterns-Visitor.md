---
title: 访问者模式(Visitor Pattern)
date: 2017-11-29 15:06:00
tags: 行为型模式(Behavioral Pattern) 
categories: Java Design Patterns
---

访问者模式属于行为型模式

<!-- more -->

# 意图

* 封装一些作用于某种对象结构中的元素的操作，使其可以在不改变对象结构的前提下定义作用于这些元素的新的操作

# 场景

当遇到下面场景时可以考虑使用访问者模式：

* 当对象结构比较稳定，但经常需要给对象结构中的元素对象定义新的操作时
* 当需要给对象结构中的元素对象添加许多不同且不相关的操作时，为了避免这些操作污染元素对象，同时也为了避免新增操作需要修改元素对象类

# UML 类图

访问者模式类图如下：

![Java-Design-Patterns-Visitor.png](http://otg3f8t90.bkt.clouddn.com/2018/1/15/Java-Design-Patterns-Visitor.png)

类图说明：

* Visitor：访问者结构，定义了访问每一个 ConcreteElement 的行为，所以要求对象结构稳定
* ConcreteVisitor：具体的访问者实现，定义要作用于 ConcreteElement 的新的操作
* Element：对象结构的抽象，定义可被访问者访问的行为
* ConcreteElement：具体的元素对象，使用访问者模式时要求对象结构稳定，其实指的就是要求 ConcreteElement 稳定，不能频繁的创建或移除，因为每次变动都要相应的调整访问者及其实现
* ObjectSturcture：对象结构，对象结构是一个抽象描述，内部管理所有的元素对象，并提供迭代这些元素供访问者访问的方法
* Client：客户端

# 实现

定义 Unit，对应类图中的 Element：

```java
public abstract class Unit {

    public abstract void accept(UnitVisitor visitor);

    @Override
    public String toString() {
        return getClass().getSimpleName().toLowerCase();
    }
}
```

定义 Commander、Sergeant、Soldier，对应类图中的 ConcreteElement：

```java
public class Commander extends Unit {

    // UnitVisitor 的定义见后面
    @Override
    public void accept(UnitVisitor visitor) {
        visitor.visit(this);
    }
}
```

```java
public class Sergeant extends Unit {

    @Override
    public void accept(UnitVisitor visitor) {
        visitor.visit(this);
    }
}
```

```java
public class Soldier extends Unit{

    @Override
    public void accept(UnitVisitor visitor) {
        visitor.visit(this);
    }
}
```

定义 UnitVisitor，对应类图中的 Visitor：

```java
public interface UnitVisitor {
    void visit(Commander commander);

    void visit(Sergeant sergeant);

    void visit(Soldier soldier);
}
```

定义 CommanderVisitor、SergeantVisitor、SoldierVisitor，对应类图中的 ConcreteVisitor：

```java
public class CommanderVisitor implements UnitVisitor {
    @Override
    public void visit(Commander commander) {
        System.out.println("Good to see you " + commander);
    }

    @Override
    public void visit(Sergeant sergeant) {
        // Do nothing
    }

    @Override
    public void visit(Soldier soldier) {
        // Do nothing
    }
}
```

```java
public class SergeantVisitor implements UnitVisitor {
    @Override
    public void visit(Commander commander) {
        // Do nothing
    }

    @Override
    public void visit(Sergeant sergeant) {
        System.out.println("Hello " + sergeant);
    }

    @Override
    public void visit(Soldier soldier) {
        // Do nothing
    }
}
```

```java
public class SoldierVisitor implements UnitVisitor{
    @Override
    public void visit(Commander commander) {
        // Do nothing
    }

    @Override
    public void visit(Sergeant sergeant) {
        // Do nothing
    }

    @Override
    public void visit(Soldier soldier) {
        System.out.println("Greetings " + soldier);
    }
}
```

定义 UnitStructure，对应类图中的 ObjectStructure：

```java
public class UnitStructure {
    private List<Unit> units = new ArrayList<>();

    public UnitStructure() {
        units.add(new Commander());
        units.add(new Sergeant());
        units.add(new Sergeant());
        units.add(new Soldier());
        units.add(new Soldier());
        units.add(new Soldier());
        units.add(new Soldier());
        units.add(new Soldier());
        units.add(new Soldier());
    }

    public void addUnit(Unit unit) {
        units.add(unit);
    }

    public void accept(UnitVisitor visitor) {
        for (Unit unit : units) {
            unit.accept(visitor);
        }
    }
}
```

客户端调用:

```java
public class Client {
    public static void main(String[] args) {
        UnitStructure structure = new UnitStructure();
        structure.accept(new CommanderVisitor());
        structure.accept(new SergeantVisitor());
        structure.accept(new SoldierVisitor());
    }
}
```

# 总结

访问者模式的本质是预留通路，回调实现。通过预先定义好调用的通路，在被访问的对象上定义 accept() 方法，在访问者对象上定义 visit() 方法，在调用真正发生时，通过两次分发，利用预先定义好的通路，回调到访问者的具体实现上。

访问者模式优点：

* 当需要为元素对象扩展新的操作时，直接实现访问者接口即可，因而具有良好的扩展性
* 使对象结构和作用于元素对象上的操作解耦
* 具体访问者之间职责分离，符合单一职责原则

访问者模式缺点：

* 要求对象结构稳定，当改动元素对象时，需要重构访问者接口及其实现，成本太大且不符合开闭原则
* 元素对象需要暴露内部数据给访问者对象，破坏了封装性，违反了迪米特原则
* 为了区别元素对象，访问者接口依赖的是具体而不是抽象，违背了依赖倒置原则

# Ref

* [Visitor Pattern](http://www.oodesign.com/visitor-pattern.html)
* [java-design-patterns-visitor](https://github.com/iluwatar/java-design-patterns/blob/master/visitor/README.md)
