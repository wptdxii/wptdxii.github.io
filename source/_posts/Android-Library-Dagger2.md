---
title: 依赖注入框架 Dagger 2.x
date: 2017-08-13 20:50:43
tags: [Library, Dagger 2.x]
categories: Android
---

依赖注入框架 Dagger 2 的使用总结。

<!-- more -->

# 依赖注入

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

- 构造器注入(Constructor Injection)
- 方法注入(Setter Injection)
- 接口注入(Interface Injection)

详细的内容可以参看：

- [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
- [Using dependency injection in Java - Introduction - Tutorial](http://www.vogella.com/tutorials/DependencyInjection/article.html#dependencyinjection_annotations)

# Dagger

- 项目地址： [dagger 2](https://github.com/google/dagger)
- stackoverflow tag [dagger-2](https://stackoverflow.com/questions/tagged/dagger-2)

> 下面的示例代码来自 dagger2 官方项目的 examples

Dagger 2 是一个全静态的，编译时的依赖注入框架，通过注解处理器(Annotation Processor)在编译时生成高度可读的类似手写的依赖图，消除了运行时使用反射带来的性能问题。Dagger 2 描述依赖的注解基于 Java Specification Request(简称 JSR) 330，注入的顺序是：

1. 构造器注入(Constructor Injection)：注入构造方法参数
1. 字段注入(Field Injection)：注入非私有的成员变量
1. 方法注入(Method Injection)：注入方法参数

被 @Inject 注解标注的字段或者方法被调用的顺序是由 JSR 330 定义的，而不是在类中被声明的顺序。因字段注入和方法参数注入是在构造参数注入之后进行的，所以在构造器中不能使用未被注入的成员变量和方法参数。

Dagger 2 依赖注入框架主要由三部分组成：

- 依赖提供者(Dependency Provider)
- 依赖消费者(Dependency Consumer)
- 依赖连接器(Dependency Connector)

## 依赖提供者

Dagger2 依赖提供者可以通过 @Inject 和 Module 两种方式提供依赖。

## 使用 @Inject

对于自定义的类，可以使用 @Inject 注解被依赖类的构造器，如果构造器需要参数，参数也需要有对应的依赖提供者。示例如下：

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

然后在需要实例的地方也用 @Inject 注入即可:

```java
class CoffeeMaker {
  @Inject Thermosiphon thermosiphon;
  ...
}
```

## 使用 Module

@Inject 注解构造器可以提供依赖，但在下面几种情形下不能实现:

- 不能构造接口类型的实例
- 第三方类库中的类无法使用 @Inject 注解
- 可配置的对象必须可配置

在这些情况下需要使用 Module 提供被依赖类的实例。使用这种方式是首先需要使用 @Moduel 注解类，该类约定使用 Module 作为后缀，然后使用 @Provides 注解提供依赖的方法，该方法约定使用 provide 作为前缀，根据情况可以声明为静态的。当该方法有参数且需要将参数直接返回时，可以将该方法声明为 abstract，并使用 @Binds 注解。示例如下:

```java
// 以 Module 为后缀
// 如果使用了 @Binds 需要声明为 abstractt
@Module
public abstract class PumpModule {
  // 以 provide 为前缀
  @Binds
  abstract Pump providePump(Thermosiphon pump);

  // 当需要直接返回方法参数时，可以使用上面的用 @Binds 注解的抽象方法方式简写
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


  @Provides static Heater provideHeater() {
    return new ElectricHeater();
  }
}
```

上面两种提供依赖的方式的优先级是不同的，对于相同类型的依赖，@Module 要比 @Inject 的优先级高，依赖容器会优先从 Module 中查找依赖，如果没有找到，再去查找 @Inject 注解的依赖

## 依赖连接器

依赖连接器(Dependency Connector)也叫做依赖注入器(Dependency Injector)，用于提供依赖提供者提供的依赖，或者关联依赖提供者和依赖消费者，将依赖注入到依赖消费者中。依赖注入器定义为接口，约定使用 Component 为后缀，Dagger 会实现该接口，实现类以 Dagger 为前缀，然后拼接上接口的名字。通过注入器就可以使用依赖提供者提供的依赖，示例如下:

```java
// 实现类为 DaggerCoffeeShop
// 如果依赖是由 Module 提供，需要指定对应的 Module
// 可以指定多个: module= {AModulce.class, BModule.class}
@Component(modules = PumpModule.class)
interface CoffeeShopComponent {
  // 使用生成的实现类可以直接获取依赖
  Pump pump();
}
```

生成的实现类使用如下:

```java
public class CoffeeApp {
  public static void main(String[] args) {
    // CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
        // .dripCoffeeModule(new DripCoffeeModule())
        // .build();

    // 当 Module 不需要外部传参时，不用通过 Builder 创建 Module，可以直接简写如下:
    CoffeeShopComponent coffeeShopComponent = DaggerCoffeeShopComponent.create();
    Pump pump = coffeeShopComponent.pump();
  }
}
```

## 依赖消费者

依赖消费者(Dependency Consumer) 指的是消费依赖提供者提供的依赖的类，Android 开发中通常指的是 Activity 或 Fragment 等，当 Component 将两者关联之后，可直接通过 @Inject 获取注入实例。使用时需要在 Component 接口中定义一个  inject() 方法，用于将两者关联，示例代码如下：

```java
@Component(modules = CoffeeModule.class)
public interface CoffeeShopComponent {

    // 方法参数必须是具体的类，而不能是其父类，例如这里不能写 BaseActivity
    void inject(DaggerActivity daggerActivity);
}
```

然后就可以使用

```java
// DaggerActivity 就是依赖消费者
public class DaggerActivity extends BaseActivity {

    // 该 Field 被注入
    // 不能是 Private 和 Final
    @Inject
    CoffeeMaker mCoffeeMaker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.sample_activity_dagger);

        // 关联依赖消费者
        DaggerCoffeeShopComponent.create().inject(this);
        // 可以直接使用 mCoffeeMaker
        mCoffeeMaker.brew();
    }
}
```

## @Scope

@Scope 用于限定依赖提供者提供的依赖的作用域，该注解用于自定义注解，不能直接使用，Dagger 中提供了两种默认实现:

- @Singleton
- @Reusable

### @Singleton

当需要确保依赖提供者提供的依赖为单例时，可以使用 @Singleton，对于 Module 需要将 @Singleton 注解到 provide 方法上:

```java
@Provides
@Singleton
public static Heater provideHeater() {
  return new ElectricHeater();
}
```

对于使用 @Inject 提供依赖的依赖提供者，将 @Sinlge 注解到类上:

```java
@Singleton
class CoffeeMaker {
  ...
}
```

对于使用了 @Singleton 依赖提供者，对应的 Component 也必须加上对应的注解，否则编译不通过，如下：

```java
@Singleton
@Component(modules = CoffeeModule.class)
public interface CoffeeShopComponent {

    void inject(DaggerActivity daggerActivity);

}
```

使用 @Singleton 注解的依赖，Dagger 在生成的代码中使用 DoubleCheck\<T\> 获取示例，DoubleCheck\<T\> 的实现如下：

```java
// 省略了部分代码
public final class DoubleCheck<T> implements Provider<T>, Lazy<T> {

...

  private volatile Object instance = UNINITIALIZED;

 @SuppressWarnings("unchecked") // cast only happens when result comes from the provider
  @Override
  public T get() {
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          result = provider.get();
          instance = reentrantCheck(instance, result);
          /* Null out the reference to the provider. We are never going to need it again, so we
           * can make it eligible for GC. */
          provider = null;
        }
      }
    }
    return (T) result;
  }
}
```

所以 Dagger 实现的单例使用了双重校验锁的实现方式，并且解决了 DCL 失效的问题，实现了懒加载，并且时线程安全的。

@Singleton 实现单例的条件有两个：

- 依赖提供者和对应的依赖注入器使用 @Singleton 注解
- 使用同一个依赖注入器，即同一个 Component

> 当在不同的页面使用 Component.create() 或者 Component.Builder.build() 时会创建不同的 Component，会获取不同的单例

### @Reusable

当需要限制依赖提供者提供的依赖的数量但不需要保证单例时，可以使用 @Reusable 注解，该注解的使用与 @Singleton 类似，不同的是其不能注解 Component，即 @Reusable 不会与单个的 Component 绑定，使用绑定的每个 Component 都会缓存实例对象。如果使用 @Reusable 注解的依赖提供者与一个 Component 绑定，但其 SubComponent 使用了绑定，那么只有 SubComponent 会缓存实例。当 Component 缓存了实例后，SubComponent 可以直接复用实例

使用 @Singleton 注解的依赖，Dagger 在生成的代码中使用 SingleCheck\<T\> 获取示例，SingleCheck\<T\> 的实现如下：

```java
// 省略部分代码
public final class SingleCheck<T> implements Provider<T>, Lazy<T> {
  private static final Object UNINITIALIZED = new Object();
  private volatile Provider<T> provider;
  private volatile Object instance = UNINITIALIZED;

  private SingleCheck(Provider<T> provider) {
    assert provider != null;
    this.provider = provider;
  }

  @SuppressWarnings("unchecked") // cast only happens when result comes from the delegate provider
  @Override
  public T get() {
    Object local = instance;
    if (local == UNINITIALIZED) {
      // provider is volatile and might become null after the check, so retrieve the provider first
      Provider<T> providerReference = provider;
      if (providerReference == null) {
        // The provider was null, so the instance must already be set
        local = instance;
      } else {
        local = providerReference.get();
        instance = local;

        // Null out the reference to the provider. We are never going to need it again, so we can
        // make it eligible for GC.
        provider = null;
      }
    }
    return (T) local;
  }
  ...
}
```

### 自定义 scope

使用 Scope 时需要注意以下几点:

- 依赖提供者使用了 scope ，那么 Component 就必须使用同一个注解
- 依赖提供者没有使用 scope ，那么 Component 是否加注解都无关紧要，可以通过编译，但无效
- Component 的 dependencies 与 Component 自身的 scope 不能相同，应该为不用的 Component 定义不同的 Scope
- @Singleton 注解的 Component 不能依赖其他 scope 的 Component，只能其他 scope 的组件依赖 @Singleton 注解的 Component
- 没有 scope 的 Component 不能依赖有 scope 的 Component
- 一个 component 不能同时有多个 scope(Subcomponent 除外)
- Component 注入的 Activity 在其他 Component 中不能再去注入

## Lazy 注入

当注入的依赖需要实现懒加载时，可以使用 Lazy\<T\> 实现，实例如下:

```java
public class GrindingCoffeeMaker {
  @Inject Lazy<Grinder> lazyGrinder;

  public void brew() {
    while (needsGrinding()) {
      // Grinder created once on first call to .get() and cached.
      // 对于同一个 Lazy<T>，无论 T 是否是单例，Lazy.get() 返回的都是同一个实例
      lazyGrinder.get().grind();
    }
  }
}
```

## Provider 注入

与 Lazy\<T\> 作用相似，但是对于非单例的依赖，使用 Provider\<T\> 每次都可以获取不同的实例，使用如下:

```java
class BigCoffeeMaker {
  @Inject Provider<Filter> filterProvider;

  public void brew(int numberOfPots) {
  ...
    for (int p = 0; p < numberOfPots; p++) {
      maker.addFilter(filterProvider.get()); //new filter every time.
      maker.addCoffee(...);
      maker.percolate();
      ...
    }
  }
}
```

## 标识符(Qualifiers)

对于依赖提供者而言，当提供多个同类型依赖时，为了区分彼此，可以使用标识符加以区分，Dagger 提供了默认的实现 @Named，可以自定义标识符：

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface AppContext {}

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityContext {}
```

标识符需要注解于依赖提供者和依赖消费者

# Android 最佳实践

# Dagger.Android

[Keeping the Daggers Sharp](https://medium.com/square-corner-blog/keeping-the-daggers-sharp-%EF%B8%8F-230b3191c3f)

# Ref

- [dagger-user-guide](https://google.github.io/dagger/users-guide)
- [Introduction to Dagger 2, Using Dependency Injection in Android: Part 1](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-1-223289c2a01b)
- [Tasting Dagger 2 on Android](https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
- [That Missing Guide: How to use Dagger2](https://medium.com/@Zhuinden/that-missing-guide-how-to-use-dagger2-ef116fbea97)
- [Using Dagger 2 for dependency injection in Android - Tutorial](http://www.vogella.com/tutorials/Dagger/article.html)
- [Dagger 2 for Android Beginners — Introduction](https://medium.com/@harivigneshjayapalan/dagger-2-for-android-beginners-introduction-be6580cb3edb)
- [Dependency Injection with Dagger 2 (Devoxx 2014)](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)
- [Dependency Injection with Dagger 2](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2)
- [New Android Injector with Dagger 2 — part 1](https://medium.com/@iammert/new-android-injector-with-dagger-2-part-1-8baa60152abe)
- [[HOW-TO] Android Dagger (2.11–2.16) Butterknife (8.7-8.8) MVP (Part 1)](https://proandroiddev.com/how-to-android-dagger-2-10-2-11-butterknife-mvp-part-1-eb0f6b970fd)
- [【安卓】rxjava2的disposable](https://blog.csdn.net/c_j33/article/details/78774546)