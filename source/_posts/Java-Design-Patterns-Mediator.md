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

* Mediator：中介者的抽象接口，定义与 Colleague 对象交互的接口。如果不需要切换具体的中介者对象时，可以不抽象出该接口
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
        member.joinParty(this);
    }

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

        hobbit.act(Action.ENEMY);
        wizard.act(Action.GOLD);
        rogue.act(Action.TALE);
        hobbit.act(Action.HUNT);
    }
}
```

# 总结

# Ref