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

# Dagger

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

## 依赖提供者

Dagger2 依赖提供者(Dependency Provider)有两种方式可以提供依赖：

* 使用 @Inject 注解被依赖类的构造器，如果构造器需要参数，参数也需要有对应的依赖提供者。这种方式使用简单，可以避免过多的 @Module 和 @Provide
* 使用 Module 提供被依赖类的实例。这种方式主要针对第三方类库或者需要提供接口类型的实例，此时可以使用 @Provides 注解提供依赖的方法，该方法约定使用 provide 作为前缀，根据情况可以声明为静态的，当该方法有参数且需要将参数直接返回时，可以将该方法声明为 abstract，并使用 @Binds 注解。

> 对于相同类型的依赖，@Module 要比 @Inject 的优先级高，依赖容器会优先从 Module 中查找依赖，如果没有找到，再去查找 @Inject 注解的依赖

@Inject 提供依赖的示例如下：

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

Module 提供依赖的示例如下：

```java
// Module 类通常以 Module 为后缀
// 如果使用了 @Binds 需要声明为 abstract
@Module
public abstract class PumpModule {
  @Binds
  abstract Pump providePump(Thermosiphon pump);

  // 当需要直接返回方法参数时，可以使用上面的抽象方法这种方式简写
  // 根据实际情况可以声明为静态的
  //@Provides
  //public static Pump provideAnothierPump(Thermosiphon pump) {
    //  return pump;
  //}

//   @Provides
  // 如果需要提供单例，在这里使用 @Singleton 注解
  // @Singleton
 // public static Pump provideNewPump() {
   //   return new Thermosiphon();
  //}

  // 上面三个方法的返回值一样，如果需要同时存在的话，需要使用限定符区分
}
```

## 依赖连接器

依赖连接器(Dependency Connector)也叫做依赖注入器(Dependency Injector)，用于关联依赖提供者和依赖消费者。依赖注入器定义为接口，Dagger 会实现该接口，实现类以 Dagger 为前缀，然后加上接口名字，接口中需要定义一个用于注入依赖消费者的方法，如下:

```java
// 实现类为 DaggerCoffeeShopComponent
// 需要指定关联的 Module，可以指定多个: module= {AModulce.class, BModule.class}
@Component(modules = CoffeeModule.class)
public interface CoffeeShopComponent {

    // 方法参数必须是具体的类，而不能是其父类，例如这里不能写 BaseActivity
    void inject(DaggerActivity daggerActivity);
}
```

生成的实现类使用如下:

```java
    CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
        .dripCoffeeModule(new DripCoffeeModule())
        .build();
    // 当 Module 不需要外部传参时，不用通过 Builder 创建 Module，可以直接简写如下:
    CoffeeShop coffeeShop = DaggerCoffeeShop.create();
```

## 依赖消费者

依赖消费者(Dependency Consumer) 在使用 Dagger 生成的 Component 关联当前对象后，就可以通过 @Inject 获取注入实例，如下：

```java
// DaggerActivity 就是依赖消费者
public class DaggerActivity extends BaseActivity {

    // 该 Field 被注入
    @Inject
    CoffeeMaker mCoffeeMaker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.sample_activity_dagger);

        DaggerCoffeeShopComponent.create().inject(this);
        // 可以直接使用 mCoffeeMaker
        mCoffeeMaker.brew();
    }
}
```

## @Scope

@Scope 注解用于组织约束依赖提供者和依赖注入器，需要自定义注解，其中 @Singleton 是其默认实现，如下：

```java
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
````

module 的 provide 方法使用了 scope ，那么 component 就必须使用同一个注解
module 的 provide 方法没有使用 scope ，那么 component 和 module 是否加注解都无关紧要，可以通过编译，但无效
component的dependencies与component自身的scope不能相同，即组件之间的scope不同
Singleton的组件不能依赖其他scope的组件，只能其他scope的组件依赖Singleton的组件
没有scope的component不能依赖有scope的component
一个component不能同时有多个scope(Subcomponent除外)
@Singleton 的生命周期依附于component，同一个module provide singleton ,不同component 也是不一样
Component注入的Activity 在其他Component中不能再去注入
component 的 inject 函数不要声明基类参数；
Scope 注解必须用在 module 的 provide 方法上，否则并不能达到局部单例的效果；
如果 module 的 provide 方法使用了 scope 注解，那么 component 就必须使用同一个注解，否则编译会失败；
如果 module 的 provide 方法没有使用 scope 注解，那么 component 和 module 是否加注解都无关紧要，可以通过编译，但是没有局部单例效果；
对于直接使用 @Inject 构造函数的依赖，如果把 scope 注解放到它的类上，而不是构造函数上，就能达到局部单例的效果了；
@Module提供依赖的优先级高于@Inject
@Singleton并不是真的能创建单例，但我们依然可以保证在App的生命周期内一个类只存在一个对象。@Singleton更重要的作用是通过标记提醒我们自己来达到更好的管理实例的目的。
Component的作用域必须与对应的Module作用域一致，如果@Module没有标记作用域，就不影响。
Component和依赖的Component作用域范围不能一样，否则会报错。一般来讲，我们应该对每个Component都定义不同的作用域。
由于@Inject，@Module和@Provides注解是分别验证的，所有绑定关系的有效性是在@Component层级验证。

## 标识符

## Lazy 注入

## Provider 注入

## 组织 Compoennt

# Android最佳实践

# Dagger.Android

[Keeping the Daggers Sharp](https://medium.com/square-corner-blog/keeping-the-daggers-sharp-%EF%B8%8F-230b3191c3f)

# Ref

* [dagger-user-guide](https://google.github.io/dagger/users-guide)
* [Introduction to Dagger 2, Using Dependency Injection in Android: Part 1](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-1-223289c2a01b)
* [Tasting Dagger 2 on Android](https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
* [That Missing Guide: How to use Dagger2](https://medium.com/@Zhuinden/that-missing-guide-how-to-use-dagger2-ef116fbea97)
* [Using Dagger 2 for dependency injection in Android - Tutorial](http://www.vogella.com/tutorials/Dagger/article.html)
* [Dagger 2 for Android Beginners — Introduction](https://medium.com/@harivigneshjayapalan/dagger-2-for-android-beginners-introduction-be6580cb3edb)
* [Tasting Dagger 2 on Android](https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)