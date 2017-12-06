---
title: 单例模式(Singleton Pattern)
date: 2017-08-07 23:45:03
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

单例模式(Singleton Pattern)是创建型模式(Creational Pattern)

<!-- more -->

# 场景

在如下场景中可以考虑使用单例模式：

* 避免产生多个对象消耗过多的资源
* 某些对象应该有且只有一个

# 意图

* 单例模式可以确保一个类只有一个实例，并向全局提供该实例

# UML 类图

单例模式的类图如下：

![Java-Design-Patterns-Singleton.png](http://otg3f8t90.bkt.clouddn.com/2017/11/28/Java-Design-Patterns-Singleton.png)

实现单例模式时要注意以下几点：

* 构造函数不对外开放，一般为 Private，同时为了防止利用反射创建新的对象，可以在构造器中通过判断并抛出异常处理
* 通过一个静态方法或者枚举返回单例类对象
* 确保单例类有且仅有一个对象，尤其是在多线程环境下
* 确保单例类对象在反序列化时不会重新创建对象

# 实现

## 饿汉式

饿汉式单例模式的示例代码如下：

```java
public class EagerSingleton {
    private static EagerSingleton sInstance = new EagerSingleton();

    private EagerSingleton() {
        // 防止利用反射创建对象
        if (INSTANCE != null) {
            throw new UnsupportedOperationException("Already initialized.");
        }
    }

    public static EagerSingleton getInstance() {
        return sInstance;
    }
}
```

优点：

* 线程安全

缺点：

* 单例对象不是懒加载，有可能造成资源的浪费
* 存在反序列化时对象重新生成的问题

## 懒汉式

懒汉式单例模式的示例代码如下：

```java
public class LazySingleton {
    private static LazySingleton sInstance;

    private LazySingleton() {
        // 防止利用反射创建对象
        if (sInstance != null) {
            throw new UnsupportedOperationException("Already initialized.");
        }
    }

    public static synchronized LazySingleton getInstance() {
        if (sInstance == null) {
            sInstance = new LazySingleton();
        }
        return sInstance;
    }
}
```

优点：

* 单例对象懒加载
* 线程安全

缺点：

* 单例对象懒加载，所以第一次调用时反应可能会稍慢
* 每次获取实例时都会进行方法同步，造成不必要的同步开销
* 存在反序列化时对象重新生成的问题

## 双重校验锁单例

使用双重校验锁(Double Check Lock，简称 DCL) 机制实现的单例模式在获取实例的方法中需要进行两层判空，第一层判空主要是为了避免不必要的同步；第二层判空是为了避免重复的实例化。示例代码如下：

```java
public class DCLSingleton {
    private volatile static DCLSingleton sInstance;

    private DCLSingleton() {
        // 防止利用反射创建对象
        if (sInstance != null) {
            throw new UnsupportedOperationException("Already initialized.");
        }
    }

    public static DCLSingleton getInstance() {
        // 使用局部变量可以提高性能
        DCLSingleton result = sInstance;
        if (result == null) {
            synchronized (DCLSingleton.class) {
                // 在多线程环境下的每次调用，局部变量会被重新创建，重新赋值是为了防止在当前线程阻塞的时候其他线程完成了单例对象初始化
                result = sInstance;
                if (result == null) {
                    sInstance = result = new DCLSingleton();
                }
            }
        }
        return result;
    }
}
```

优点：

* 单例对象懒加载
* 线程安全

缺点：

* 单例对象懒加载，所以第一次调用时反应可能会稍慢
* 存在反序列化时对象重新生成的问题
* JDK1.5 之前可能会出现 DCL 失效问题

单例实例的初始化语句即 sInstance = new DCLSingleton() 不是一个原子性操作，该语句最终会被编译成多条汇编指令：

1. 给 Singleton 的实例分配内存
1. 调用 Singleton 类的构造器，初始化成员字段
1. 将 sInstance 对象指向分配的内存空间(此时 sInstance 就不为 null 了)

为了提高编译效率，Java 编译器会进行指令重排序，允许处理器乱序执行，以及 JDK1.5  之前 Java 内存模型(Java Memory Model，简称 JMM)中对 Cache、寄存器到主内存回写顺序的规定，上面 2 和 3 的执行顺序是无法保证的，也就是说，整个的执行的顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，在多线程环境下，当线程 A 执行 sInstance = new DCLInstance() 且执行到上边第 3 步时，sInstance 已经不为 null，此时切换到了线程 B，线程 B 会直接将 sIntance 取走，在使用时就会因为 sInstance 尚未被完全初始化而出错，这就是 DCL 失效问题。

volatile 关键字主要有两个作用：

* 保证共享变量在线程间的可见性：当共享变量被 volatile 修饰后，会保证变量在线程间的可见性，当一个线程修改了变量的值时，变量的新值对其他线程来说是立即可见的
* 保证指令的有序性，禁止指令重排序：当程序执行到 volatile 修饰的变量时，在其前面的操作一定已经全部执行完毕，在其后面的操作还没有进行，前面操作的结果对后面的的操作是可见的。在进行指令优化时，volatile 变量前的语句不能在 volatile 变量后执行，volatile 变量后的语句也不能放到 volatile 变量前执行。

利用 volatile 可以保证指令有序性，禁止指令重排序的特性，可以解决 DCL 失效 的问题。 当用 volatile 关键字修饰共享变量 sIntance 时，会强制上面的 2 和 3 顺序执行，线程 B 在获取的 sInstance 要么为 null  要么一定是初始化完全的对象。

> * 在 JDK1.5 之前 volatile 语义较弱，只能保证共享变量在线程间的可见性，不能禁止指令重排序，所以只有在 JDK1.5 及其以后的版本使用 volatile 才能解决 DCL 失效的问题
> * volatile 会少许的影响性能，但是对于程序的正确性而言，这点儿性能牺牲是值得的

## 静态内部类单例

内部类是延迟加载的，只会在第一次使用时加载，而静态变量随着类的加载而加载且加载一次，利用这个特性同样可以实现单例模式，实例代码如下：

```java
public class HolderSingleton {
    private HolderSingleton() {
        // 防止利用反射创建对象
        if (SingletonHolder.INSTANCE != null) {
            throw new UnsupportedOperationException("Already initialized.");
        }
    }

    public static HolderSingleton getInstance(){
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final HolderSingleton INSTANCE = new HolderSingleton();
    }
}
```

优点：

* 单例对象懒加载
* 线程安全，对不同版本 JDK 兼容性好

缺点：

* 单例对象懒加载，所以第一次调用时反应可能会稍慢
* 存在反序列化时对象重新生成的问题

## 枚举单例

枚举对象数量是固定的，当设定只有一个对象时，当然就实现了所谓的单例，示例代码如下：

```java
public enum EnumSingleton {
    INSTANCE;
}
```

优点：

* 写法简单
* 线程安全
* 不存在反序列化时对象重新生成的问题
* 不会被利用反射重新创建对象

缺点：

* 思路清奇
* 枚举在 Android 开发中有较大的性能开销，官方不建议在不必要的时候使用

## 容器单例

程序初始化时，可以创建一个容器管理所有的单例类型，示例代码如下：

```java
public class SingletonContainer {
    private static Map<String, Object> containerMap = new HashMap<>();

    private SingletonContainer() {
    }

    public static void putInstance(String key, Object instance) {
        if (!containerMap.containsKey(key)) {
            containerMap.put(key, instance);
        }
    }

    public static Object getInstance(String key) {
        return containerMap.get(key);
    }
}
```

优点：

* 通过统一的接口提供获取操作，实现单例对象的统一管理

缺点：

* 不算真正意义上的单例模式，而且还需要记住对应对象的键值
* 会造成单例对象类型的丢失，需要强转
* 容器类本身不是单例的，其管理的类型也不是单例的，在反序列化时不能阻止重新创建对象，存在反序列化的问题

# 反序列化

上面几种单例模式的实现方式中，除了枚举单例外和容器单例外，单例类在实现了 Serializable 接口对单例对象序列化后，在反序列化时会造成单例对象的重新创建，通过加入下面方法可以避免这个问题：

```java
  private Object readResolve() throws ObjectStreamException {
        return getInstance();
    }
```

> 该方法可以在 Serializable 接口的 javadoc 中查看，其可以使用任意修饰符；getInstance() 方法返回的是单例对象

# 总结

单例模式是使用频率很高的模式，推荐多使用 DCL 单例和静态内部类单例。

单例模式的优点：

* 由于单例模式在内存中只有一个实例，减少了内存开销和性能开销
* 单例模式可以在系统设置全局的访问点，优化和共享资源访问，避免对资源的多重占用

单例模式的缺点：

* 单例模式一般没有接口，扩展性较差，若要扩展，只能修改源码
* 单例模式的生命周期较长，对短生命周期的对象的引用容易造成内存泄漏。如在 Android 开发中，单例类应该尽量持有 Application Context

# Ref

* [Singleton Pattern](http://www.oodesign.com/singleton-pattern.html)
* [java design patterns-singleton](https://github.com/iluwatar/java-design-patterns/blob/master/singleton/README.md)
* [Java Volatile Keyword](http://tutorials.jenkov.com/java-concurrency/volatile.html)
* [Java并发编程：volatile关键字解析](http://www.cnblogs.com/dolphin0520/p/3920373.html)