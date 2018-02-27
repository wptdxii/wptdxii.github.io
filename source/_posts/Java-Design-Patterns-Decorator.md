---
title: 装饰模式(Decorator Pattern)
date: 2016-11-29 15:06:00
tags: [Structural Pattern, GoF]
categories: Java Design Patterns
---

装饰模式属于结构型模式

<!-- more -->

# 意图

* 动态地给一个对象添加一些额外地职责

# 场景

当遇到下面场景时可以考虑使用装饰模式：

* 需要透明、动态地扩展类地功能时
* 当不适合通过子类扩展功能时。如当需要扩展多个独立功能时，使用继承会造成子类数量爆炸性增加。或者继承会造成子类有大量属性或方法冗余

# UML 类图

装饰模式类图如下：

![Java-Design-Patterns-Decorator.png](http://otg3f8t90.bkt.clouddn.com/2018/2/6/Java-Design-Patterns-Decorator.png)

类图说明：

* Component：抽象组件，可以是接口或抽象类
* ConcreteComponent：组件具体实现类，充当的是被装饰的原始对象，可以动态的给这些对象添加职责
* Decorator：抽象装饰器，持有被装饰对象的引用。当装饰逻辑单一，即只有一个具体的装饰器对象时，该抽象层可以省略，直接实现为具体的装饰器
* ConcreteDecorator：装饰器对象的具体实现，用于给被装饰对象添加功能

# 实现

定义 Troll，对应类图中的 Component：

```java
public interface Troll {
    void attack();

    int getAttackPower();

    void fleeBattle();
}
```

定义 SimpleTroll，对应类图中的 ConcreteCompoent：

```java
public class SimpleTroll implements Troll {
    @Override
    public void attack() {
        System.out.println("The troll tries to grab you!");
    }

    @Override
    public int getAttackPower() {
        return 0;
    }

    @Override
    public void fleeBattle() {
        System.out.println("The troll shrieks in horror and runs away!");
    }
}
```

定义 TrollDecorator，对应类图中的 Decorator：

```java
// 该抽象层不是必须的，如果装饰器只有一个，则可直接实现
public abstract class TrollDecorator implements Troll {
    private Troll troll;

    // 持有被装饰对象，该对象不一定是原始的被装饰对象，也可以是被装饰过的对象
    public TrollDecorator(Troll troll) {
        this.troll = troll;
    }

    @Override
    public void attack() {
        troll.attack();
    }

    @Override
    public int getAttackPower() {
        return troll.getAttackPower();
    }

    @Override
    public void fleeBattle() {
        troll.fleeBattle();
    }
}
```

定义 ClubbedTroll、FlyingTroll，对应类图中的 ConcreteDecorator：

```java
public class ClubbedTroll extends TrollDecorator {
    public ClubbedTroll(Troll troll) {
        super(troll);
    }

    @Override
    public void attack() {
        super.attack();
        // 增强被装饰对象的功能
        System.out.println("The troll swings at you with a club!");
    }

    @Override
    public int getAttackPower() {
        return super.getAttackPower() + 10;
    }
}
```

```java
public class FlyingTroll extends TrollDecorator {
    public FlyingTroll(Troll troll) {
        super(troll);
    }

    @Override
    public void attack() {
        super.attack();
        System.out.println("The troll is flying!");
    }

    @Override
    public int getAttackPower() {
        return super.getAttackPower() + 20;
    }

    @Override
    public void fleeBattle() {
        // 重写被装饰对象的功能
        System.out.println("The troll win the battle!");
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Troll troll = new SimpleTroll();
        troll.attack();
        troll.fleeBattle();
        System.out.println("The troll power：" + troll.getAttackPower());

        troll = new ClubbedTroll(troll);
        troll.attack();
        troll.fleeBattle();
        System.out.println("The Troll power：" + troll.getAttackPower());

        troll = new FlyingTroll(troll);
        troll.attack();
        troll.fleeBattle();
        System.out.println("The Troll power：" + troll.getAttackPower());
    }
}
```

# 总结

装饰模式的本质是动态组合。动态是手段，组合才是目的。通过动态组合装饰器对象，为被装饰对象透明地增加功能。装饰模式是继承关系的一种替代方案。

装饰模式优点：

* 比继承灵活。继承是静态的，一旦继承子类会获取父类所有非私有的属性和方法，会造成冗余。且当需要扩展多个功能时，会造成子类爆炸性的增加；而装饰模式是通过动态组合来实现透明的给对象添加功能，更加灵活且不会冗余
* 容易复用装饰器对象，被装饰对象可以实现被任意多的装饰器对象递归装饰
* 可以简化原始被装饰对象的定义，只定义必要的功能，其他的功能通过装饰器对象扩展

装饰模式缺点：

* 运行时动态的为对象添加功能可能会造成调试困难
* 会产生很多细粒度对象

# Ref

* [Decorator Pattern](http://www.oodesign.com/decorator-pattern.html)
* [java-design-patterns-decorator](https://github.com/iluwatar/java-design-patterns/blob/master/decorator/README.md)