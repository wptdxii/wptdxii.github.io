---
title: 外观模式(Facade Pattern)
date: 2017-02-17 08:59:15
tags: [Structural Pattern, GoF]
categories: Java Design Patterns
---

外观模式属于结构型模式

<!-- more -->

# 意图

* 为子系统中的一组接口提供一个对外的统一的高层接口，该接口使子系统更易使用

# 场景

当遇到下面场景时可以考虑使用外观模式：

* 当需要为复杂的子系统提供一个简单的接口时，可以通过外观对象提供对子系统的缺省调用，简化客户端的使用，满足大多数客户端的需求
* 当需要使客户端和子系统松散耦合时，可以通过外观对象分离客户端和子系统，从而提高子系统的独立性和可移植性
* 当需要构建多层结构的系统时，可以使用外观对象作为每层的入口，可以简化层间调用，也可以松散层间的耦合关系

# UML 类图

外观模式类图如下：

![Java-Design-Patterns-Facade.png](http://otg3f8t90.bkt.clouddn.com/2018/2/27/Java-Design-Patterns-Facade.png)

类图说明：

* Facade：外观对象，定义了子系统对外的高层接口(这里的接口指的是客户端和子系统之间的通道，而不是 Java 中的 interface)，为了方便多个模块调用，通常实现为单例或者静态的工具类
* Subsystem：子系统接口或模块
* Client：客户端，通过外观对象调用子系统功能

> 外观模式中，外观类知道子系统，但子系统不知道外观对象

# 实现

定义枚举 Action 标识类型：

```java
public enum Action {
    GO_TO_SLEEP, WAKE_UP, GO_HOME, GO_TO_MINE, WORK
}
```

定义 DwarvenMineWorker，对应类图中 Subsystem 的抽象：

```java
public abstract class DwarvenMineWorker {

    public void goToSleep() {
        System.out.println(name() + " goes to sleep.");
    }

    public void wakeUp() {
        System.out.println(name() + " wake up.");
    }

    public void goHome() {
        System.out.println(name() + " goes home.");
    }

    public void goToMine() {
        System.out.println(name() + " goes to the mine.");
    }

    public abstract String name();

    public abstract void work();

    private void action(Action action) {
        switch (action) {
            case GO_TO_SLEEP:
                goToSleep();
                break;
            case WAKE_UP:
                wakeUp();
                break;
            case GO_TO_MINE:
                goToMine();
                break;
            case WORK:
                work();
                break;
            case GO_HOME:
                goHome();
                break;
            default:
                System.out.println("Undefined work!");
                break;
        }
    }

    public void action(Action... actions) {
        for (Action action : actions) {
            action(action);
        }
    }
}
```

实现上面的抽象，对应类图中的 Subsystem：

```java
public class DwarvenCartOperator extends DwarvenMineWorker{
    @Override
    public String name() {
        return "Dwarf cart operator";
    }

    @Override
    public void work() {
        System.out.println(name() + " moves the gold chunks out of the mine.");
    }
}
```

```java
public class DwarvenGoldDigger extends DwarvenMineWorker{
    @Override
    public String name() {
        return "Dwarf gold digger";
    }

    @Override
    public void work() {
        System.out.println(name() + " digs for gold.");
    }
}
```

```java
public class DwarvenTunnelDigger extends DwarvenMineWorker {
    @Override
    public String name() {
        return "Dwarf tunnel digger";
    }

    @Override
    public void work() {
        System.out.println(name() + " creates another promising tunnel.");
    }
}
```

定义 DwarvenMineFacade，对应类图中的 Facade：

```java
// 实现为单例
public class DwarvenMineFacade {
    private final List<DwarvenMineWorker> workers;

    private DwarvenMineFacade() {
        if (SingletonHolder.INSTANCE != null) {
            throw new UnsupportedOperationException("Already instantiated!");
        }

        workers = new ArrayList<>();
        workers.add(new DwarvenGoldDigger());
        workers.add(new DwarvenCartOperator());
        workers.add(new DwarvenTunnelDigger());
    }

    public static DwarvenMineFacade getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private void makeActions(List<DwarvenMineWorker> workers, Action... actions) {
        for (DwarvenMineWorker worker : workers) {
            worker.action(actions);
        }
    }

    public void startNewDay() {
        makeActions(workers, Action.WAKE_UP, Action.GO_TO_MINE);
    }

    public void digOutGold() {
        makeActions(workers, Action.WORK);
    }

    public void endDay() {
        makeActions(workers, Action.GO_HOME, Action.GO_TO_SLEEP);
    }

    private static class SingletonHolder {
        private static final DwarvenMineFacade INSTANCE = new DwarvenMineFacade();
    }

}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        DwarvenMineFacade facade = DwarvenMineFacade.getInstance();
        facade.startNewDay();
        facade.digOutGold();
        facade.endDay();
    }
}
```

# 总结

外观模式的本质是封装交互、简化调用。

外观模式很好的体现了“最少知道原则”，通过外观类使客户端和子系统解耦，使系统更有弹性，当多个子系统发生变动时，只需要变动外观类即可，并不影响客户端，也就是在不影响客户端的情况下，实现系统内部的维护或扩展。

外观模式优点：

* 松散客户端和子系统之间的耦合关系，对客户端隐藏子系统的内部细节
* 外观类是对子系统的接口封装，简化了客户端的调用
* 通过外观类可以将子系统划分层次，不用层次的子系统定义不同的外观

外观模式缺点：

* 不合理的外观类容易造成迷惑：客户端是应该使用外观类还是直接与子系统交互
* 外观对象只能封装大部分的客户端对子系统的调用，但不能涵盖所有，在某些情况下客户端还是需要与子系统直接交互

# Ref

* [java-design-patterns-facade](https://github.com/iluwatar/java-design-patterns/blob/master/facade/README.md)