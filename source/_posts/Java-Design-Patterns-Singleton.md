---
title: 单例模式(Singleton Pattern)
date: 2017-08-07 23:45:03
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

单例模式是创建型模式(Creational Pattern)
<!-- more -->

# 意图

单例模式可以确保一个类只有一个实例，并向全局提供该实例

# 应用场景

在如下场景中可以考虑使用单例模式：

* 避免产生多个对象消耗过多的资源
* 某些对象应该有且只有一个

# UML 类图

单例模式的 UML 类图如下所示：

![Java-Design-Patterns-Singleton.png](http://otg3f8t90.bkt.clouddn.com/2017/11/28/Java-Design-Patterns-Singleton.png)

实现单例模式时要注意以下几点：

* 构造函数不对外开放，一般为 Private
* 通过一个静态方法或者枚举返回单例类对象
* 确保单例类有且仅有一个对象，尤其是在多线程环境下
* 确保单例类对象在反序列化时不会重新构建对象

# 实现方式

## 饿汉式

饿汉式单例模式的示例代码如下：

```java
public class EagerSingleton {
    private static EagerSingleton sInstance = new EagerSingleton();

    private EagerSingleton() {}

    public static EagerSingleton getInstance() {
        return sInstance;
    }
}
```

饿汉式的优点是：

* 线程安全

饿汉式的缺点是：

* 在类加载的时候就进行了实例的初始化，有可能造成资源的浪费

## 懒汉式

懒汉式单例模式的示例代码如下：

```java
public class LazySingleton {
    private static LazySingleton sInstance;

    private LazySingleton() {}

    public static synchronized LazySingleton getInstance() {
        if (sInstance == null) {
            sInstance = new LazySingleton();
        }
        return sInstance;
    }
}
```

懒汉式的优点是：

* 单例类的实例在第一次使用时才被初始化，在一定程度上节约了资源

懒汉式的缺点是：

* 单例类的实例在第一次使用需要进行初始化，所以第一次使用反应会稍慢
* 每次获取实例时都会进行方法同步，造成不必要的同步开销

## 双重校验锁单例

使用双重校验锁(Double Check Lock，简称 DCL) 机制实现的单例模式在获取实例的方法中需要进行两层判空，第一层判空主要是为了避免不必要的同步；第二层判空是为了避免重复的实例化。示例代码如下：

```java
public class DCLSingleton {
    private volatile static DCLSingleton sInstance;

    private DCLSingleton() {}

    public static DCLSingleton getInstance() {
        if (sInstance == null) {
            synchronized (DCLSingleton.class) {
                if (sInstance == null) {
                    sInstance = new DCLSingleton();
                }
            }
        }
        return sInstance;
    }
}
```

DCL 单例的优点是：

* 单例类的实例在第一次使用时才被初始化，在一定程度上节约了资源
* 线程安全

DCL 单例的缺点是：

* 单例类的实例在第一次使用需要进行初始化，所以第一次使用反应会稍慢
* JDK1.5 之前可能会出现 DCL 失效问题

单例实例的初始化语句即 sInstance = new DCLSingleton() 不是一个原子性操作，该语句最终会被编译成多条汇编指令：

1. 给 Singleton 的实例分配内存
1. 调用 Singleton 类的构造器，初始化成员字段
1. 将 sInstance 对象指向分配的内存空间(此时 sInstance 就不为 null 了)

为了提高编译效率，Java 编译器会进行指令重排序，允许处理器乱序执行，以及 JDK1.5  之前 Java 内存模型(Java Memory Model，简称 JMM)中对 Cache、寄存器到主内存回写顺序的规定，上面 2 和 3 的执行顺序是无法保证的，也就是说，整个的执行的顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，在多线程环境下，当线程 A 执行 sInstance = new DCLInstance() 且执行到上边第 3 步时，sInstance 已经不为 null，此时切换到了线程 B，线程 B 会直接将 sIntance 取走，在使用时就会因为 sInstance 尚未被完全初始化而出错，这就是 DCL 失效问题。

volatile 关键字主要有两个作用：

* 保证共享变量在线程间的可见性：当共享变量被 volatile 修饰后，会保证变量在线程间的可见性，当一个线程修改了变量的值时，变量的新值对其他线程来说是立即可见的
* 保证指令的有序性，禁止指令重排序：当程序执行到 volatile 修饰的变量时，在其前面的操作一定已经全部执行完毕，在其后面的操作还没有进行，前面操作的结果对后面的的操作是可见的。在进行指令优化时，volatile 变量前的语句不能在 volatile 变量后执行，volatile 变量后的语句也不能放到 volatile 变量前执行。

利用 volatile 可以保证指令有序性，禁止指令重排序的特性，可以解决 DCL 失效 的问题。 当用 volatile 关键字修饰共享变量 sIntance 时，new DCLSingleton() 一定会执行完毕，sInstance 一定被完全的初始化。

## 静态内部类单例

内部类是延迟加载的，只会在第一次使用时加载，而静态变量随着类的加载而加载且加载一次，利用这个特性同样可以实现单例模式，实例代码如下：

```java
public class HolderSingleton {
    private HolderSingleton() {}

    public static HolderSingleton getInstance(){
        return SingletonHolder.sInstance;
    }

    private static class SingletonHolder {
        private static final HolderSingleton sInstance = new HolderSingleton();
    }
}
```

使用静态内部类实现单例模式有如下优点：

* 单例类的实例在第一次使用时才被初始化，在一定程度上节约了资源
* 线程安全

## 枚举单例

## 容器单例

# Ref

* [Singleton Pattern](http://www.oodesign.com/singleton-pattern.html)
* [Java并发编程：volatile关键字解析](http://www.cnblogs.com/dolphin0520/p/3920373.html)
