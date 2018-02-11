---
title: 享元模式(Flyweight Pattern)
date: 2018-02-08 16:20:34
tags: [Structural Pattern, GoF]
categories: Java Design Patterns
---

享元模式属于结构型模式

<!-- more -->

# 意图

* 通过共享有效地支持对大量细粒度对象的需求(这里细粒度对象指的是具有相同内部状态和可变外部状态的对象)

# 场景

当遇到下面场景时可以考虑使用享元模式：

* 系统中使用了大量的细粒度对象，可以使用享元模式减少对象数量
* 需要将对象状态分离为内部状态和外部状态
* 需要缓冲池的场景

# UML 类图

享元模式类图如下：

![Java-Design-Patterns-Flyweight.png](http://otg3f8t90.bkt.clouddn.com/2018/2/8/Java-Design-Patterns-Flyweight.png)

类图说明：

* Flyweight：享元接口。通过这个接口享元对象可以接受并处理外部状态
* ConcreteFlyweight：具体的享元对象。封装了对象的内部状态，一般由构造器传入
* FlyweightFactory：享元工厂。主要用来创建和管理共享的享元对象，并向外提供获取享元对象的接口
* Client：享元客户端。主要用来维持对享元对象的引用，并处理外部状态

# 实现

定义 Potion，对应类图中的 Flyweight：

```java
public interface Potion {
    // 该方法参数指的就是外部状态
    void drink(String potion);
}
```

定义 HealingPotion、PoisonPotion，对应类图中的 ConcreteFlyweight：

```java
public class HealingPotion implements Potion {
    private PotionType type;

    // 构造参数指的就是内部状态
    public HealingPotion(PotionType type) {
        this.type = type;
    }

    @Override
    public void drink(String potion) {
        System.out.println(potion + " will heal you! IntrinsicState : " + type.name().toLowerCase() + " HashCode = " + hashCode());
    }
}
```

```java
public class PoisonPotion implements Potion {
    private PotionType type;

    public PoisonPotion(PotionType type) {
        this.type = type;
    }

    @Override
    public void drink(String potion) {
        System.out.println(potion + " will hurt you! IntrinsicState : " + type.name().toLowerCase() + " HashCode = " + hashCode());
    }
}
```

定义 PotionType，用于标识享元类型，对应类图中 FlyweightFactory 中的 Key：

```java
public enum PotionType {
    HEALING, POISON
}
```

定义 PotionFactory，对应类图中的 FlyweightFactory：

```java
public class PotionFactory {
    private static final Map<PotionType, Potion> potions = new EnumMap<>(PotionType.class);

    private PotionFactory() {
        throw new UnsupportedOperationException("Cant't be instantiated!");
    }

    static Potion createPotion(PotionType type) {
        Potion potion = potions.get(type);
        if (potion == null) {
            switch (type) {
                case HEALING:
                    potion = new HealingPotion(type);
                    potions.put(type, potion);
                    break;
                case POISON:
                    potion = new PoisonPotion(type);
                    potions.put(type, potion);
                    break;
                default:
                    break;
            }
        }
        return potion;
    }
}
```

定义 AlchemistShop，对应类图中的 Client：

```java
public class AlchemistShop {
    private List<Potion> topShelf;
    private List<Potion> bottomShelf;

    public AlchemistShop() {
        topShelf = new ArrayList<>();
        bottomShelf = new ArrayList<>();

        fillShelves();
    }

    private void fillShelves() {
        topShelf.add(PotionFactory.createPotion(PotionType.HEALING));
        topShelf.add(PotionFactory.createPotion(PotionType.HEALING));

        bottomShelf.add(PotionFactory.createPotion(PotionType.POISON));
        bottomShelf.add(PotionFactory.createPotion(PotionType.POISON));
    }

    public void enumerate(String potion) {
        System.out.println("Enumerating top shelf potions:");
        for (Potion p : topShelf) {
            p.drink(potion);
        }

        System.out.println("Enumerating bottom shelf potions:");

        for (Potion p : bottomShelf) {
            p.drink(potion);
        }
    }
}
```

在客户端中演示调用：

```java
public class Client {
    public static void main(String[] args) {
        AlchemistShop shop = new AlchemistShop();
        // 这里传入的参数最终会作为外部状态传递给享元对象
        shop.enumerate("HolyWater");
        shop.enumerate("MagicWater");
    }
}
```

# 总结

享元模式的本质是分离和共享。其重点在于分离变与不变，把一个对象的状态分成内部状态和外部状态，内部状态是不变的，外部状态是可变的，通过共享不变的部分，达到减少对象数量节约内存的目的。享元对象的内部状态通常指的是包含在享元对象内部的、对象本身的状态，是独立于使用享元的场景的信息，创建后就不再变化的状态，因此可以共享，通常通过构造器传入；享元对象的外部状态则取决于享元的使用场景，会随着使用场景的不同而变化，因此不可共享，通常通过方法参数传入。

享元模式优点：

* 减少对象数量，节约内存空间

享元模式缺点：

* 维护共享对象需要额外的开销
* 需要分离对象内部状态和外部状态，使程序的逻辑复杂化
* 享元对象的外部状态是由外部读取的，可能稍慢

# Ref

* [Flyweight Pattern]http://www.oodesign.com/flyweight-pattern.html()
* [java-design-patterns-flyweight](https://github.com/iluwatar/java-design-patterns/blob/master/flyweight/README.md)