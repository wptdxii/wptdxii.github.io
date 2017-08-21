---
title: Dagger 2.x Tutorial
date: 2017-08-13 20:50:43
tags: Dagger
categories: Android
---

<!-- more -->

# Dependency Injection

控制反转(Inversion of Control，简称 IoC) 是面向对象编程中的一个设计原则，用来降低对象之间的耦合。

当一个对象需要依赖另外一个对象实现功能时，最简单的方式就是静态的在构造器中创建被依赖的对象，
如下面代码所示：

```java
public class Foo {
    private Log log;
    public Foo() {
        this.log = new LogImpl();
    }

    public void test() {
        log.debug("test...");
    }
}
```

这时调用对象控制着依赖对象的创建，但这样做耦合性太大，当依赖对象改动时或者需要换一个实现的时候，调用对象也要做出相应的改动，因为紧密耦合，也不方便单独对调用对象进行测试。
依赖反转就是就是为了解决这种强耦合的硬依赖，将调用对象创建依赖对象的的控制权反转，交给外部的依赖容器来进行创建，然后注入到调用对象中，实现松散耦合的目的。

依赖注入(Dependency Injection，简称 DI)和依赖查找(Dependency Lookup)是实现控制反转的两种常见方式，下面主要说一下依赖注入，依赖注入是指通过提供注入依赖的方法，让依赖容器决定依赖关系，从而实现控制反转，将依赖对象注入到调用对象中,增加代码的可维护性和可测试性。依赖注入主要有三种注入方式：构造器注入、方法注入和接口注入。

## Consturctor Injection

创建一个使用依赖对象的抽象接口作为参数的构造器，使依赖对象从构造器注入，如下面代码所示：

```java
public class Foo {
    private Log log;

    public Foo(Log log) {
        this.log = log;
    }

    ...
}
```

## Setter Injection

创建一个使用依赖对象的抽象接口作为参数的方法，使依赖对象从方法注入，如下面代码所示：

```java
public class Foo {
    private Log log;

    public void setLog(Log log) {
        this.log = log;
    }

    ...
}
```

## Interface Injection

创建一个接口，给该接口定义一个使用依赖对象的抽象接口作为参数的抽象方法，使调用对象实现该接口，通过接口方法实现依赖对象的注入，如下面代码所示：

```java
public interface InjectLog {
    void injectLog(Log log);
}

public class Foo implements InjectLog {
    private Log log;

    @Override
    public void injectLog(Log log) {
        this.log = log;
    }

    ...
}
```

详细的内容可以参看：

* [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
* [Using dependency injection in Java - Introduction - Tutorial](http://www.vogella.com/tutorials/DependencyInjection/article.html#dependencyinjection_annotations)

# Dagger 2.x

* 项目地址： [dagger 2](https://github.com/google/dagger)
* 文档地址： [User documentation](https://google.github.io/dagger/)
* stackoverflow tag [dagger-2](https://stackoverflow.com/questions/tagged/dagger-2)

Dagger 2 是一个全静态的，编译时的依赖注入框架，通过注解处理器(Annotation Processor)在编译时生成高度可读的类似手写的依赖图，消除了运行时使用反射带来的性能问题。Dagger 2 描述依赖的注解基于 Java Specification Request(简称JSR) 330，注入的顺序是：

1. 构造器注入(Constructor Injection)：注入构造方法参数
1. 字段注入(Field Injection)：注入非私有的成员变量
1. 方法注入(Method Injection)：注入方法参数

被 @Inject 注解标注的字段或者方法被调用的顺序是由 JSR 330 定义的，而不是在类中被声明的顺序。因字段注入和方法参数注入是在构造参数注入之后进行的，所以在构造器中不能使用被被注入的成员变量和方法参数。

Dagger 2 依赖注入框架主要由三部分组成：

* 依赖提供者(Dependency Provider): 使用 @Module 注解
* 依赖消费者(Dependency Consumer): 使用 @Inject 注解
* 依赖连接器(Dependency Connector): 使用 @Component 注解

## @Inject



## @Compoennt

## @Module & @Provides

## @Qualifier

## Scope

# Dagger 2.x on Android

# Ref

* [Introduction to Dagger 2, Using Dependency Injection in Android: Part 1](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-1-223289c2a01b)
* [Introduction to Dagger 2, Using Dependency Injection in Android: Part 2](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-2-b55857911bcd)
* [Dependency Injection with Dagger 2](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2)
* [Tasting Dagger 2 on Android](https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)