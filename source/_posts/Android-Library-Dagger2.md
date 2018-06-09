---
title: 依赖注入框架 Dagger 2.x
date: 2017-08-13 20:50:43
tags: [Library, Dagger 2.x]
categories: Android
---

依赖注入(DI)框架 Dagger 2 的使用总结。

<!-- more -->

# 依赖注入(Dependency Injection)

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

依赖注入(Dependency Injection，简称 DI)和依赖查找(Dependency Lookup)是实现控制反转的两种常见方式，下面主要说一下依赖注入，依赖注入是指通过提供注入依赖的方法，让依赖容器决定依赖关系，从而实现控制反转，将依赖对象注入到调用对象中,增加代码的可维护性和可测试性。依赖注入主要有三种注入方式：

* 构造器注入(Constructor Injection)
* 方法注入(Setter Injection)
* 接口注入(Interface Injection)

详细的内容可以参看：

* [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
* [Using dependency injection in Java - Introduction - Tutorial](http://www.vogella.com/tutorials/DependencyInjection/article.html#dependencyinjection_annotations)

# Dagger 2.x

* 项目地址： [dagger 2](https://github.com/google/dagger)
* stackoverflow tag [dagger-2](https://stackoverflow.com/questions/tagged/dagger-2)

Dagger 2 是一个全静态的，编译时的依赖注入框架，通过注解处理器(Annotation Processor)在编译时生成高度可读的类似手写的依赖图，消除了运行时使用反射带来的性能问题。Dagger 2 描述依赖的注解基于 Java Specification Request(简称JSR) 330，注入的顺序是：

1. 构造器注入(Constructor Injection)：注入构造方法参数
1. 字段注入(Field Injection)：注入非私有的成员变量
1. 方法注入(Method Injection)：注入方法参数

被 @Inject 注解标注的字段或者方法被调用的顺序是由 JSR 330 定义的，而不是在类中被声明的顺序。因字段注入和方法参数注入是在构造参数注入之后进行的，所以在构造器中不能使用未被注入的成员变量和方法参数。

Dagger 2 依赖注入框架主要由三部分组成：

* 依赖提供者(Dependency Provider)
* 依赖消费者(Dependency Consumer)
* 依赖连接器(Dependency Connector)

下面分别对这些组成部分进行介绍

## 依赖提供者(Dependency Provider)

Dagger2 中有两种方式提供依赖：

* 使用 @Inject 注解被依赖类的构造器，如果构造器需要参数，参数也需要有对应的依赖提供者。这种方式使用简单，可以避免过多的 @Module 和 @Provide
* 使用 @Module 和 @Provide 提供被依赖类的实例。这种方式主要针对第三方类库，因为对于第三方类库无法用 @Inject 注解其构造器

@Inject 提供依赖的示例代码如下：

```java

// 如果需要提供单例，在这里使用 @Singleton 注解
// @Singleton
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) {// 构造方法的参数也需要有相应的依赖提供者提供
    this.heater = heater;
  }

  @Override public void pump() {
    if (heater.isHot()) {
        // todo
    }
  }
}
```

@Module 和 @Provide 提供依赖的示例代码如下：

```java

```

## 依赖消费者(Dependency Consumer)

## 依赖连接器(Dependency Connector)

## @Compoennt

## @Module & @Provides

## @Qualifier

## Scope

# Dagger 2.x on Android

# Ref

* [Introduction to Dagger 2, Using Dependency Injection in Android: Part 1](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-1-223289c2a01b)
* [Tasting Dagger 2 on Android](https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
* [That Missing Guide: How to use Dagger2](https://medium.com/@Zhuinden/that-missing-guide-how-to-use-dagger2-ef116fbea97)
* [Using Dagger 2 for dependency injection in Android - Tutorial](http://www.vogella.com/tutorials/Dagger/article.html)
* [Dagger 2 for Android Beginners — Introduction](https://medium.com/@harivigneshjayapalan/dagger-2-for-android-beginners-introduction-be6580cb3edb)
* [Android：dagger2让你爱不释手-终结篇](https://www.jianshu.com/p/65737ac39c44)
* [Dagger2 彻底了解如何构建依赖关系](https://blog.csdn.net/u012943767/article/details/51954939)
* [深入浅出，一篇文章让你学会Dagger2使用](https://blog.csdn.net/gg199402/article/details/78990900)
* [Dagger2 Scope 注解能保证依赖在 component 生命周期内的单例性吗？](https://blog.piasy.com/2016/04/11/Dagger2-Scope-Instance/)