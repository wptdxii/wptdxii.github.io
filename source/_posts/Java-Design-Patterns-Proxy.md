---
title: 代理模式(Proxy Pattern)
date: 2018-02-09 10:07:49
tags: [Structural Pattern, GoF]
categories: Java Design Patterns
---

代理模式属于结构型模式

<!-- more -->

# 意图

* 为其他对象提供一种代理以控制对这个对象的访问

# 场景

当遇到下面场景时可以考虑使用代理模式：

* 需要控制对原始对象的访问时
* 需要创建大量开销比较大的对象时，可以使用虚代理对象按需加载部分数据，减小开销，在必要时再加载完整的原始对象

# UML 类图

代理模式类图如下：

![Java-Design-Patterns-Proxy.png](http://otg3f8t90.bkt.clouddn.com/2018/2/15/Java-Design-Patterns-Proxy.png)

类图说明：

* Subject：抽象主题。定义具体主题对象和代理对象的共同接口方法，可以是接口也可以是抽象类
* RealSubject：具体的主题对象。实现抽象主题定义的接口
* Proxy：代理对象。实现抽象主题定义的接口，持有具体主题对象的引用，在实现的接口方法中转调具体主题对象的方法，实现代理
* Client：客户端

# 实现

Java 语言中代理模式的实现分为静态代理和动态代理，静态代理指的是代理对象需要自定义实现，而动态代理则是利用 Java 内建的对代理模式的支持来实现代理。

## 静态代理

定义 Wizard，用于封装数据：

```java
public class Wizard {
    private final String name;

    public Wizard(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

定义 WizardTower，对应类图中的 Subject：

```java
public interface WizardTower {
    void enter(Wizard wizard);
}
```

定义 IvoryTower，对应类图中的 RealSubject：

```java
public class IvoryTower implements WizardTower {
    @Override
    public void enter(Wizard wizard) {
        System.out.println(wizard + " enters the tower.");
    }
}
```

定义 WizardTowerProxy，对应类图中的 Proxy：

```java
public class WizardTowerProxy implements WizardTower {
    private static final int NUM_WIZARDS_ALLOWED = 3;
    private final WizardTower wizardTower;
    private int numWizards;

    public WizardTowerProxy(WizardTower wizardTower) {
        this.wizardTower = wizardTower;
    }

    @Override
    public void enter(Wizard wizard) {
        if (numWizards < NUM_WIZARDS_ALLOWED) {
            wizardTower.enter(wizard);
            numWizards++;
        } else {
            System.out.println(wizard + " is not allowed to enter!");
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        WizardTowerProxy proxy = new WizardTowerProxy(new IvoryTower());
        proxy.enter(new Wizard("Red wizard"));
        proxy.enter(new Wizard("White wizard"));
        proxy.enter(new Wizard("Black wizard"));
        proxy.enter(new Wizard("Green wizard"));
        proxy.enter(new Wizard("Brown wizard"));
    }
}
```

## 动态代理

动态代理的代理对象需要利用 Java 内建的 java.lang.reflect 包下的 InvocatioHander 接口和 Porxy 实现：

```java
// 实现 InvocationHander 接口
public class WizardTowerProxy implements InvocationHandler {
    private static final int NUM_WIZARDS_ALLOWED = 3;
    private WizardTower wizardTower;
    private int numWizards;

    // 方法参数指的是被代理对象，即类图中的 RealSubject
    public WizardTower getProxy(WizardTower wizardTower) {
        this.wizardTower = wizardTower;
        return (WizardTower) Proxy.newProxyInstance(wizardTower.getClass().getClassLoader(),
                wizardTower.getClass().getInterfaces(), this);
    }

    // 实现 invoke() 接口方法
    // 参数 args 指的是被代理方法的所有参数，可以使用索引取
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (numWizards < NUM_WIZARDS_ALLOWED) {
            numWizards++;
            return method.invoke(wizardTower, args[0]);
        } else {
            System.out.println(args[0] + "is not allowed to enter!");
            return null;
        }
    }
}
```

上面使用的的类和客户端都与静态代理的实现一致，不再重复定义。

# 总结

代理模式的本质是控制对象访问。通过代理具体的主题对象，把代理对象插入到客户端和主题对象之间，从而为客户端和主题对象引入一定的间接性，给代理对象提供了操作空间，在转调具体的主题对象前后，可以附加很多操作。

# Ref

* [Proxy Pattern](http://www.oodesign.com/proxy-pattern.html)
* [java-design-patterns-proxy](https://github.com/iluwatar/java-design-patterns/blob/master/proxy/README.md)
* [Java动态代理原理分析](http://objcoding.com/2017/08/16/Java-Dynamic-proxy/)