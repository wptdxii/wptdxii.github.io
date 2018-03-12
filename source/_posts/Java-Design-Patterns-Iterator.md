---
title: 迭代器模式(Iterator Pattern)
date: 2016-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

迭代器模式属于行为型模式

<!-- more -->

# 意图

* 提供一种方法顺序访问聚合对象中的各个元素，而又不需要暴露该对象的内部表示
> 聚合指的是一组对象的组合结构，如 Java 集合、数组等

# 场景

当遇到下面场景可以考虑使用迭代器模式：

* 需要访问一个聚合对象的内容，又不想暴露其内部表示
* 需要使用多种遍历方式访问聚合对象时
* 需要在遍历不同的聚合对象时提供统一接口
* 需要同时遍历同一个聚合对象时

# UML 类图

迭代器模式类图如下：

![Java-Design-Patterns-Iterator.png](http://otg3f8t90.bkt.clouddn.com/2017/12/21/Java-Design-Patterns-Iterator.png)

类图说明：

* Iterator：迭代器接口，负责定义访问、遍历元素的接口
* ConcreteIterator：迭代器接口的实现类，实现对聚合对象的遍历，并跟踪遍历的当前位置
* Aggregate：聚合对象接口，定义创建相应迭代器对象的接口
* ConcreteAggregate：聚合对象接口的实现类，创建对应的迭代器对象
* Client：客户端，主要使用迭代器接口

# 实现

定义 TreasureType，用于标识聚合对象元素类型：

```java
// 用于在迭代器中过滤
public enum TreasureType {
    ANY, WEAPON, RING, POTION
}
```

定义 Treasure，其为聚合对象的组成元素:

```java
public class Treasure {
    private TreasureType type;
    private String treasure;

    public Treasure(TreasureType type, String treasure) {
        this.type = type;
        this.treasure = treasure;
    }

    public TreasureType getTreasureType() {
        return type;
    }

    public String getTreasure() {
        return treasure;
    }

    @Override
    public String toString() {
        return treasure;
    }
}
```

定义 TreasureChest　和　Iterator，其中　TreasureChest 对应类图中的 Aggregate：

```java
public interface TreasureChest {
    Iterator iterator(TreasureType treasureType);
}
```

```java
// 为了保证类型不丢失，加上泛型
public interface Iterator<T> {
    boolean hasNext();

    T next();
}
```

实现 TreasureChest 和 Iterator：

```java
public class ListTreasureChest implements TreasureChest {
    private List<Treasure> treasures;

    public ListTreasureChest() {
        treasures = new ArrayList<>();
        treasures.add(new Treasure(TreasureType.POTION, "Potion of courage"));
        treasures.add(new Treasure(TreasureType.RING, "Ring of shadows"));
        treasures.add(new Treasure(TreasureType.POTION, "Potion of wisdom"));
        treasures.add(new Treasure(TreasureType.POTION, "Potion of blood"));
        treasures.add(new Treasure(TreasureType.WEAPON, "Sword of silver +1"));
        treasures.add(new Treasure(TreasureType.POTION, "Potion of rust"));
        treasures.add(new Treasure(TreasureType.RING, "Ring of armor"));
        treasures.add(new Treasure(TreasureType.WEAPON, "Steel halberd"));
        treasures.add(new Treasure(TreasureType.WEAPON, "Dagger of poison"));
        treasures.add(new Treasure(TreasureType.POTION, "Potion of healing"));
    }

    @Override
    public Iterator iterator(TreasureType treasureType) {
        // 创建对应的　Iterator，ListTreasureChestIterator 的实现见下面
        return new ListTreasureChestIterator(this, treasureType);
    }

    public List<Treasure> getTreasures() {
        return treasures;
    }
}
```

```java
public class ArrayTreasureChest implements TreasureChest {
    private Treasure[] treasures;

    public ArrayTreasureChest() {
        treasures = new Treasure[10];
        treasures[0] = new Treasure(TreasureType.POTION, "Potion of courage");
        treasures[1] = new Treasure(TreasureType.RING, "Ring of shadows");
        treasures[2] = new Treasure(TreasureType.POTION, "Potion of wisdom");
        treasures[3] = new Treasure(TreasureType.POTION, "Potion of blood");
        treasures[4] = new Treasure(TreasureType.WEAPON, "Sword of silver +1");
        treasures[5] = new Treasure(TreasureType.POTION, "Potion of rust");
        treasures[6] = new Treasure(TreasureType.RING, "Ring of armor");
        treasures[7] = new Treasure(TreasureType.WEAPON, "Steel halberd");
        treasures[8] = new Treasure(TreasureType.WEAPON, "Dagger of poison");
        treasures[9] = new Treasure(TreasureType.POTION, "Potion of healing");
    }

    @Override
    public Iterator iterator(TreasureType treasureType) {
        // 创建对应的　Iterator，ArrayTreasureChestIterator 的实现见下面
        return new ArrayTreasureChestIterator(this, treasureType);
    }

    public Treasure[] getTreasures() {
        return treasures;
    }
}
```

```java
public class ListTreasureChestIterator implements Iterator<Treasure> {
    private ListTreasureChest treasureChest;
    private TreasureType treasureType;
    private int cursor = -1;

    public ListTreasureChestIterator(ListTreasureChest treasureChest, TreasureType treasureType) {
        this.treasureChest = treasureChest;
        this.treasureType = treasureType;
    }

    @Override
    public boolean hasNext() {
        return findNextIndex() != -1;
    }

    @Override
    public Treasure next() {
        cursor = findNextIndex();
        if (cursor != -1) {
            return treasureChest.getTreasures().get(cursor);
        }
        return null;
    }

    private int findNextIndex() {
        List<Treasure> treasures = treasureChest.getTreasures();
        int index = cursor;
        while (true) {
            index++;
            if (index >= treasures.size()) {
                index = -1;
                break;
            }

            // 过滤
            if (TreasureType.ANY.equals(treasureType) || treasureType.equals(treasures.get(index).getTreasureType())) {
                break;
            }
        }
        return index;
    }
}
```

```java
public class ArrayTreasureChestIterator implements Iterator<Treasure> {
    private ArrayTreasureChest treasureChest;
    private TreasureType treasureType;
    private int cursor = -1;

    public ArrayTreasureChestIterator(ArrayTreasureChest treasureChest, TreasureType treasureType) {
        this.treasureChest = treasureChest;
        this.treasureType = treasureType;
    }

    @Override
    public boolean hasNext() {
        return findNextIndex() != -1;
    }

    @Override
    public Treasure next() {
        cursor = findNextIndex();
        if (cursor != -1) {
            return treasureChest.getTreasures()[cursor];
        }
        return null;
    }

    private int findNextIndex() {
        Treasure[] treasures = treasureChest.getTreasures();
        int index = cursor;
        while (true) {
            index++;
            if (index >= treasures.length) {
                index = -1;
                break;
            }

            // 过滤
            if (TreasureType.ANY.equals(treasureType) || treasureType.equals(treasures[index].getTreasureType())) {
                break;
            }
        }
        return index;
    }
}
```

上面的　ListTreasureChest 和　ArrayTreasureChest 对应类图中的　ConcreteAggregate，ListTreasureChestIterator 和　ArrayTreasureChestIterator 对应类图中的　ConcreteIterator

定义　TreasureChestType，用于标识　TreasureChest 的实现：

```java
public enum TreasureChestType {
    LIST, ARRAY
}
```

定义　TreasureChestFactory，用于选择　TreasureChest 的实现，这里使用了简单工厂模式：

```java
public class TreasureChestFactory {
    private TreasureChestFactory() {
        throw new UnsupportedOperationException("Cant't be instantiated");
    }

    public static TreasureChest create(TreasureChestType treasureChestType) {
        switch (treasureChestType) {
            case LIST:
                return new ListTreasureChest();
            case ARRAY:
                return new ArrayTreasureChest();
            default:
                throw new IllegalArgumentException("TreasureChestType not supported");
        }
    }
}
```

客户端调用：

```java
public class Client {

    public static void main(String[] args) {
        filter(TreasureChestFactory.create(TreasureChestType.LIST));

        filter(TreasureChestFactory.create(TreasureChestType.ARRAY));
    }

    // 过滤
    private static void filter(TreasureChest treasureChest) {
        iterate(treasureChest, TreasureType.ANY);
        iterate(treasureChest, TreasureType.POTION);
        iterate(treasureChest, TreasureType.RING);
        iterate(treasureChest, TreasureType.WEAPON);
    }

    // 遍历
    private static void iterate(TreasureChest treasureChest, TreasureType treasureType) {
        Iterator iterator = treasureChest.iterator(treasureType);
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        System.out.println("-------------------------");
    }
}
```

# 总结

迭代器模式的本质是控制访问聚合对象中的元素。在无需暴露聚合对象的内部表示的情况下，可以透明的访问聚合对象的元素，并可以根据需要在迭代器中做过滤操作。同时也解决了在聚合对象中直接提供遍历操作时无法对同一个对象同时多个遍历的问题(遍历是需要维护当前位置，故无法同时遍历)

现在的高级语言基本上都为其容器提供了迭代器，极少会需要自己实现了

迭代器模式优点：

* 可以使用不同的遍历方式遍历聚合对象中的元素，弱化了聚合对象和遍历算法之间的关系
* 可以对同一个聚合对象同时多个遍历
* 统一了的遍历接口，方便客户端调用

迭代器模式缺点：

* 类文件的增加

# Ref

* [Iterator](http://www.oodesign.com/iterator-pattern.html)
* [java-design-patterns-iterator](https://github.com/iluwatar/java-design-patterns/blob/master/iterator/README.md)