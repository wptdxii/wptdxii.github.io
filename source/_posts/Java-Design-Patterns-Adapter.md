---
title: 适配器模式(Adapter Pattern)
date: 2017-11-29 15:06:00
tags: [Structural Pattern, GoF]
categories: Java Design Patterns
---

适配器模式又叫包装器模式，属于结构型模式，有对象适配器和类适配器两种实现方式

<!-- more -->

# 意图

* 将一个类的接口转换成客户端需要的另一种接口，从而使原本因为接口不兼容而无法一起工作的类可以一起工作

# 场景

当遇到下面场景时可以考虑使用适配器模式：

* 想要使用一个已存在的类，但其接口不符合系统要求
* 需要创建一个可复用的类，该类可能和一些不兼容的类工作，该可复用类指的是被适配的对象
* 使用第三方类库时，使用适配器作为中间层将系统与类库解耦，方便替换第三方类库

# UML 类图

适配器模式有两种实现方式，一种是对象适配器，一种是类适配器。对象适配器依赖对象组合实现，而类适配器依赖多继承实现

## 对象适配器

对象适配器模式类图如下：

![Java-Design-Patterns-Adapter-Object.png](http://otg3f8t90.bkt.clouddn.com/2018/1/23/Java-Design-Patterns-Adapter-Object.png)

类图说明：

* Target：目标接口，即客户端需要的接口
* Adaptee：已存在的类，其功能满足客户端的需求，但接口不匹配，是要适配
* Adapter：对象适配器，将 Adaptee 适配为目标接口，该类是具体的实现类，持有 Adaptee 对象并实现 Target 接口
* Client：客户端，Target 接口的需求方，不是指通常意义上的客户端

## 类适配器

类适配器模式类图如下：

![Java-Design-Patterns-Adapter-Class.png](http://otg3f8t90.bkt.clouddn.com/2018/1/23/Java-Design-Patterns-Adapter-Class.png)

类图说明：

* Target、Adaptee、Client：与对象适配器模式一致
* Adapter：类适配器，作用与对象适配器一致，但其不再持有 Adaptee 对象，而是继承 Adaptee，将自身实现为 Adaptee，同时继承 Target 实现接口适配
* 因 Java 语言不支持多继承，所以无法实现标准的类适配器模式，而是通过实现 Target 接口并继承 Adapteee 实现接口适配

# 实现

定义 FishingBoat，对应类图中的 Adaptee：

```java
public class FishingBoat {
    public void sail() {
        System.out.println("The fishing boat is sailing.");
    }
}
```

定义 RowingBoast，对应类图中的 Target：

```java
public interface RowingBoat {
    void row();
}
```

定义 Captain，对应类图中的 Client，其需要的目标接口是 RowingBoat：

```java
public class Captain {
    private RowingBoat rowingBoat;

    public void setRowingBoat(RowingBoat rowingBoat) {
        this.rowingBoat = rowingBoat;
    }

    public void row() {
        rowingBoat.row();
    }
}
```

## 对象适配器实现

首先实现对象适配器，定义 FishingBoatAdapter，对应类图中的 Adapter，依赖对象组合实现：

```java
public class FishingBoatAdapter implements RowingBoat {
    private FishingBoat fishingBoat;

    public FishingBoatAdapter(FishingBoat fishingBoat) {
        this.fishingBoat = fishingBoat;
    }

    @Override
    public void row() {
        fishingBoat.sail();
    }
}
```

在客户端调用(这里的 Client 不是类图中的 Client，这里仅是演示调用)：

```java
public class Client {
    public static void main(String[] args) {
        Captain captain = new Captain();
        FishingBoatAdapter adapter = new FishingBoatAdapter(new FishingBoat());
        captain.setRowingBoat(adapter);
        captain.row();
    }
}
```

## 类适配器实现

实现类适配器，定义 FishingBoatAdapter，对应类图中的 Adapter，利用继承实现：

```java
// 不再组合对象，而是将自身实现为 Adaptee
public class FishingBoatAdapter extends FishingBoat implements RowingBoat {

    @Override
    public void row() {
        sail();
    }
}
```

在客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Captain captain = new Captain();
        captain.setRowingBoat(new FishingBoatAdapter());
        captain.row();
    }
}
```

# 总结

适配器模式的本质是转换匹配，复用功能。转换匹配是手段，复用功能才是目的。这里的复用指的是复用被适配对象，即类图中的 Adaptee

对象适配器和类对象适配器的区别：

* 对象适配器使用的对象组合的方式，是动态的定义方式；类适配器使用的是对象继承的方式，是静态的定义方式
* 对于类适配器，由于适配器直接继承了 Adaptee，自身就相当于 Adaptee，所以就不能再适配 Adaptee 的子类，同时由于 Java 是单继承，限制了对其他类的继承；对于对象适配器，使用了组合 Adaptee 对象实现适配，所以也可以适配 Adapteee 的子类
* 类适配器可以重新定义 Adaptee 的行为，而对象适配器则不行，其需要先扩展 Adaptee 的子类
* 类适配器本身就是 Adaptee，不再需要通过额外的引用得到 Adaptee 对象；而对象适配器需要从外部注入 Adaptee 对象

适配器模式优点：

* 复用已存在的功能，即复用被适配对象
* 使用适配器作为中间层，解耦系统和第三方类库，方便动态替换 

适配器模式缺点：

* 过多地使用适配器，会让系统非常凌乱，不易整体把握

# Ref

* [Adapter Pattern](http://www.oodesign.com/adapter-pattern.html)
* [java-design-patterns-adapter](https://github.com/iluwatar/java-design-patterns/blob/master/adapter/README.md)