---
title: 迭代器模式(Iterator Pattern)
date: 2017-11-29 15:06:00
tags: 行为型模式(Behavioral Pattern) 
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

定义类型标识：

```java
// 用于在迭代器中过滤
public enum ShapeType {
    CIRCLE, RECTANGLE, ALL
}
```

定义实体类，其为聚合对象的组成元素:

```java
public class Shape {
    private static int id = 0;
    private ShapeType shapeType;
    private int shapeId;

    public Shape(ShapeType shapeType) {
        this.shapeType = shapeType;
        this.shapeId = id++;
    }

    public ShapeType getShapeType() {
        return shapeType;
    }

    @Override
    public String toString() {
        return "Shape:" + shapeType.name().toLowerCase() + " id:" + shapeId;
    }
}
```

定义 Iterator：

```java
// 为了保证类型不丢失，加上泛型
public interface Iterator<T> {
    boolean hasNext();

    T next();
}
```

实现 Iterator：

```java
public class ArrayIterator implements Iterator<Shape> {
    // 该类是聚合对象的具体实现，其代码在后面
    private ArrayAggregate aggregate;
    private ShapeType type;
    private int cursor = -1;

    public ArrayIterator(ArrayAggregate aggregate, ShapeType type) {
        this.aggregate = aggregate;
        this.type = type;
    }

    @Override
    public boolean hasNext() {
        return findNextIndex() != -1;
    }

    @Override
    public Shape next() {
        cursor = findNextIndex();
        if (cursor != -1) {
            return aggregate.getArray()[cursor];
        }
        return null;
    }

    private int findNextIndex() {
        Shape[] shapes = aggregate.getArray();
        int index = cursor;
        while (true) {
            index++;
            if (index >= shapes.length) {
                index = -1;
                break;
            }

            // 用于过滤
            if (ShapeType.ALL.equals(type) || type.equals(shapes[index].getShapeType())) {
                break;
            }
        }
        return index;
    }
}
```

```java
public class ListIterator implements Iterator<Shape> {
    // 聚合对象，在后面定义
    private ListAggregate aggregate;
    private ShapeType type;
    private int cursor = -1;

    public ListIterator(ListAggregate aggregate, ShapeType type) {
        this.aggregate = aggregate;
        this.type = type;
    }

    @Override
    public boolean hasNext() {
        return findNextIndex() != -1;
    }

    @Override
    public Shape next() {
        cursor = findNextIndex();
        if (cursor != -1) {
            return aggregate.getList().get(cursor);
        }
        return null;
    }

    private int findNextIndex() {
        List<Shape> list = aggregate.getList();
        int index = cursor;
        while (true) {
            index++;
            if (index >= list.size()) {
                index = -1;
                break;
            }

            // 用于过滤
            if (ShapeType.ALL.equals(type) || type.equals(list.get(index).getShapeType())) {
                break;
            }
        }
        return index;
    }
}
```

定义 Aggregate：

```java
public interface Aggregate {
    Iterator iterator();
}
```

实现 Aggregate：

```java
public class ArrayAggregate implements Aggregate {
    private ShapeType filter;
    private Shape[] shapeArray = new Shape[6];

    public ArrayAggregate(ShapeType filter) {
        this.filter = filter;
        shapeArray[0] = new Shape(ShapeType.CIRCLE);
        shapeArray[1] = new Shape(ShapeType.RECTANGLE);
        shapeArray[2] = new Shape(ShapeType.CIRCLE);
        shapeArray[3] = new Shape(ShapeType.RECTANGLE);
        shapeArray[4] = new Shape(ShapeType.CIRCLE);
        shapeArray[5] = new Shape(ShapeType.RECTANGLE);
    }

    public Shape[] getArray() {
        return shapeArray;
    }

    public void setFilter(ShapeType filter) {
        this.filter = filter;
    }

    @Override
    public Iterator iterator() {
        // 创建对应的迭代器
        return new ArrayIterator(this, filter);
    }
}
```

```java
public class ListAggregate implements Aggregate {
    private ShapeType filter;
    private List<Shape> list = new ArrayList<>();

    public ListAggregate(ShapeType filter) {
        this.filter = filter;
        list.add(new Shape(ShapeType.CIRCLE));
        list.add(new Shape(ShapeType.RECTANGLE));
        list.add(new Shape(ShapeType.CIRCLE));
        list.add(new Shape(ShapeType.RECTANGLE));
        list.add(new Shape(ShapeType.CIRCLE));
        list.add(new Shape(ShapeType.RECTANGLE));
        list.add(new Shape(ShapeType.CIRCLE));
        list.add(new Shape(ShapeType.RECTANGLE));
    }

    public List<Shape> getList() {
        return list;
    }

    public void setFilter(ShapeType filter) {
        this.filter = filter;
    }

    @Override
    public Iterator iterator() {
        // 创建对应的迭代器
        return new ListIterator(this, filter);
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        ArrayAggregate arrayAggregate = new ArrayAggregate(ShapeType.ALL);
        iterate(arrayAggregate.iterator());

        // 设置过滤器
        arrayAggregate.setFilter(ShapeType.CIRCLE);
        iterate(arrayAggregate.iterator());

        ListAggregate listAggregate = new ListAggregate(ShapeType.ALL);
        iterate(listAggregate.iterator());

        listAggregate.setFilter(ShapeType.RECTANGLE);
        iterate(listAggregate.iterator());
    }

    // 遍历
    private static void iterate(Iterator iterator) {
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

# 总结

迭代器模式的本质是控制访问聚合对象中的元素。在无需暴露聚合对象的内部表示的情况下，可以透明的访问聚合对象的元素，并可以根据需要在迭代器中做过滤操作。同时也解决了在聚合对象中直接提供遍历操作时无法对同一个对象同时多个遍历的问题(迭代是需要维护当前位置，故无法同时遍历)

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