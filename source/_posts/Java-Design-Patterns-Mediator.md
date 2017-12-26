---
title: 中介者模式(Mediator Pattern)
date: 2017-11-29 15:06:00
tags: 行为型模式(Behavioral Pattern) 
categories: Java Design Patterns
---

中介者模式属于行为型模式

<!-- more -->

# 意图

* 定义一个中介对象来封装一系列对象的交互，中介者对象使各个对象不需要显式地相互引用，从而使其松散耦合，可以独立地改变它们之间的交互

# 场景

当遇到下面场景时可以考虑使用中介者模式:

* 当对象之间的交互操作很多，修改一个对象的行为同时要涉及到其它对象时
* 如果一个对象引用很多对象，并直接与这些对象交互，导致该对象难以复用时

# UML 类图

中介者模式类图如下:

![Java-Design-Patterns-Mediator.png](http://otg3f8t90.bkt.clouddn.com/2017/12/25/Java-Design-Patterns-Mediator.png)

类图说明：

* Mediator：中介者的抽象接口，定义与 Colleague 对象交互的接口。如果不需要多个中介者对象或不需要扩展中介者对象时，可以不抽象出该接口
* ConcreteMediator：中介者的具体实现，持有所有的 Colleague 对象，并维护各个 Colleague 对象之间的交互
* Colleague：同事对象的抽象，定义同事对象的行为接口和公共实现，定义获取中介者对象的构造器或者方法
* ConcreteColleague：同事接口的具体实现
* Client：客户端

# 实现

定义枚举，约束同事对象的行为:

```java
public enum Action {
    HUNT("hunted a rabbit", "arrives for dinner"),
    TALE("tells a tale", "comes to listen"),
    GOLD("found gold", "takes his share of the gold"),
    ENEMY("spotted enemies", "runs for cover");

    private String title;
    private String description;

    Action(String title, String description) {
        this.title = title;
        this.description = description;
    }

    public String getTitle() {
        return title;
    }

    public String getDescription() {
        return description;
    }
}
```

定义 Party 接口，对应类图中的 Mediator:

```java
public interface Party {

    void addMember(PartyMember member);

    void act(PartyMember actor, Action action);
}
```

实现 Party 接口，对应类图中的 ConcreteMediator：

```java
public class PartyImpl implements Party {
    private List<PartyMember> members;

    public PartyImpl() {
        this.members = new ArrayList<>();
    }

    @Override
    public void addMember(PartyMember member) {
        members.add(member);
        // 同事对象同时会持有中介者对象
        member.joinParty(this);
    }

    // 当一个具体的同事对象通知中介者时，其它同事对象会接到通知
    @Override
    public void act(PartyMember actor, Action action) {
        for (PartyMember member : members) {
            if (!member.equals(actor)) {
                member.partyAction(action);
            }
        }
    }
}
```

定义 PartyMember，对应类图中的 Colleague：

```java
public abstract class PartyMember {
    private Party party;

    public void joinParty(Party party) {
        this.party = party;
    }

    public void partyAction(Action action) {
        System.out.println(toString() + ":" + action.getDescription());
    }

    public void act(Action action) {
        if (party != null) {
            // 通过中介者与其它同事对象交互
            party.act(this, action);
        }
    }

    @Override
    public abstract String toString();
}
```

实现具体的 PartyMember，对应类图中的 ConcreteColleague：

```java
public class Hobbit extends PartyMember {
    @Override
    public String toString() {
        return "Hobbit";
    }
}
```

```java
public class Hunter extends PartyMember {
    @Override
    public String toString() {
        return "Hunter";
    }
}
```

```java
public class Rogue extends PartyMember {
    @Override
    public String toString() {
        return "Rogue";
    }
}
```

```java
public class Wizard extends PartyMember {
    @Override
    public String toString() {
        return "Wizard";
    }
}
```

客户端调用:

```java
public class Client {
    public static void main(String[] args) {
        Party party = new PartyImpl();

        PartyMember hobbit = new Hobbit();
        PartyMember wizard = new Wizard();
        PartyMember rogue = new Rogue();
        PartyMember hunter = new Hunter();

        party.addMember(hobbit);
        party.addMember(wizard);
        party.addMember(rogue);
        party.addMember(hunter);

        // 每个同事对象调用 act() 方法都会通过中介者通知其它同事对象
        hobbit.act(Action.ENEMY);
        wizard.act(Action.GOLD);
        rogue.act(Action.TALE);
        hobbit.act(Action.HUNT);
    }
}
```

# 广义中介者模式

上面类图表示的是中介者模式的标准实现，在实际开发中，通常会简化中介者模式，具体简化的地方如下：

* 通常会去掉同事对象的父类，这样可以让任意对象，只要需要相互交互，就可以成为同事
* 通常不定义中介者接口，把具体的中介者对象实现为单例
* 同事对象不再持有中介者对象，而是在需要的时候直接获取中介者对象并调用
* 中介者对象也不再持有同事对象，而是在具体的处理方法中创建或获取或通过参数传入

上面经过简化、变形的中介者模式叫做广义中介者模式

# 总结

中介者模式的本质是封装交互，将对象之间多对多的关系变成一对多的关系，中介者对象将系统从网状结构变成以中介者为中心的星型结构。

中介者模式优点：

* 同事对象之间松散耦合
* 中介者对象集中控制交互，使同事对象之间的关系由多对多变为一对多

中介者模式缺点：

* 如果同事对象之间的交互非常多而且比较复杂，容易使中介者对象过于集中化，不易管理和维护
* 如果同事对象之间的交互逻辑原本就比较简单，使用中介者模式易造成过度设计

# Ref

* [Mediator Pattern](http://www.oodesign.com/mediator-pattern.html)
* [java-design-patterns-mediator](https://github.com/iluwatar/java-design-patterns/blob/master/mediator/README.md)