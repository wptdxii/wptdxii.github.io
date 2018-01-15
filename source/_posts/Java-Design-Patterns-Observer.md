---
title: 观察者模式(Observer Pattern)
date: 2017-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

观察者模式又被叫做发布-订阅模式，属于行为型模式

<!-- more -->

# 意图

* 定义对象间一种一对多的依赖关系，当一个对象的状态发生改变时，所以依赖于它的对象都会得到通知并被自动更新

# 场景

当遇到下面场景时可以考虑使用观察者模式：

* 当一个对象状态改变时，其他对象也需要得到通知，且两者之间松散耦合，即关联行为场景
* 事件多级触发场景
* 跨系统的消息交换场景，如消息队列、事件总线的处理机制

# UML 类图

观察者模式类图如下：

![Java-Design-Pattern-Observer.png](http://otg3f8t90.bkt.clouddn.com/2018/1/4/Java-Design-Pattern-Observer.png)

类图说明：

* Subject：主题对象，又叫做被观察者(Observable)，内部维护观察者对象的集合，提供添加和移除观察者、通知观察者的操作，可以是抽象接口，如无必要也可以直接声明为具体的对象
* ConcreteSubject：主题对象的实现，如果 Subject 被声明为具体的对象，则无需再声明 ConcreteSubject
* Observer：抽象观察者，定义了更新接口，当收到被观察者通知时更新自己
* ConcreteObserver：具体的观察者，根据业务实现相应的更新逻辑

# 实现

观察者模式的实现有多种方式，有最基本的实现方式，也有使用泛型的实现方式，也有使用 Java 提供的工具类实现的，下面依次示例。

## 基本实现

定义枚举 WeatherType，用于标识天气类型：

```java
public enum WeatherType {
    SUNNY, RAINY, WINDY, COLD;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

下面几种不同的实现方式中都使用了 WeatherType，不再重复定义

定义 Weather，对应类图中的 Subject：

```java
public class Weather {
    private WeatherType currentWeather;
    private List<WeatherObserver> observers;

    public Weather() {
        currentWeather = WeatherType.SUNNY;
        observers = new ArrayList<>();
    }

    public void addObserver(WeatherObserver observer) {
        observers.add(observer);
    }

    public boolean removeObserver(WeatherObserver observer) {
        return observers.remove(observer);
    }

    public WeatherType getCurrentWeather() {
        return currentWeather;
    }

    public void timePasses() {
        WeatherType[] enumValues = WeatherType.values();
        currentWeather = enumValues[(currentWeather.ordinal() + 1) % enumValues.length];
        notifyObservers();
    }

    private void notifyObservers() {
        for (WeatherObserver observer : observers) {
            observer.update(this);
        }
    }
}
```

因为 Weather 被定义为具体的对象，就不再定义类图中 ConcreteSubject 的对应实现

定义 WeatherObserver，对应类图中的 Observer：

```java
public interface WeatherObserver {
    void update(Weather weather);
}
```

定义 Hobbits、Orcs，对应类图中的 ConcreteObserver：

```java
public class Hobbits implements WeatherObserver {
    @Override
    public void update(Weather weather) {
        System.out.println("The Hobbits watch the weather forecast : " + weather.getCurrentWeather());
    }
}
```

```java
public class Orcs implements WeatherObserver {
    @Override
    public void update(Weather weather) {
        System.out.println("The Orcs watch the weather forecast : " + weather.getCurrentWeather());
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Weather weather = new Weather();
        // 添加观察者
        weather.addObserver(new Hobbits());
        weather.addObserver(new Orcs());

        // 通知观察者
        weather.timePasses();
        weather.timePasses();
        weather.timePasses();
        weather.timePasses();
    }
}
```

## 泛型实现

为了增强代码的复用性，可以将主题对象抽象出来，为了不与具体的主题对象耦合，就利用到了泛型的特性。

定义 Observable，对应类图中的 Subject：

```java
// 使用泛型与具体的主题对象解耦
public abstract class Observable<S extends Observable<S, O>, O extends Observer<S, O>> {
    protected List<O> observers;

    public Observable() {
        this.observers = new CopyOnWriteArrayList<>();
    }

    public void add(O observer) {
        this.observers.add(observer);
    }

    public boolean remove(O observer) {
        return this.observers.remove(observer);
    }

    public void notifyObservers() {
        for (O observer : observers) {
            observer.update((S) this);
        }
    }
}
```

定义 Observer：

```java
public interface Observer<S extends Observable<S, O>, O extends Observer<S, O>> {
    void update(S subject);
}
```

定义 Weather，其为 Observable 的实现，对应类图中的 ConcreteSubject：

```java
public class Weather extends Observable<Weather, Race> {
    private WeatherType currentWeather;

    public Weather() {
        currentWeather = WeatherType.SUNNY;
    }

    public WeatherType getCurrentWeather() {
        return currentWeather;
    }

    public void timePasses() {
        WeatherType[] enumValues = WeatherType.values();
        currentWeather = enumValues[(currentWeather.ordinal() + 1) % enumValues.length];
        notifyObservers();
    }
}
```

上面代码中的 Race 是观察者的抽象，继承自 Observer，其作用是为具体观察者指定泛型并提供统一接口，是一个空实现，具体如下：

```java
public interface Race extends Observer<Weather, Race> {}
```

定义 Hobbits、Orcs，对应类图中的 ConcreteObserver：

```java
public class Hobbits implements Race{
    @Override
    public void update(Weather subject) {
        System.out.println("The Hobbits watch the weather forecast : " + subject.getCurrentWeather());
    }
}
```

```java
public class Orcs implements Race {
    @Override
    public void update(Weather subject) {
        System.out.println("The Orcs wathc the weather forecast : " + subject.getCurrentWeather());
    }
}
```

客户端调用与上面的示例相同，不再重复

## 使用工具类实现

JDK 中提供了实现观察者模式的工具类，所以类图中的 Subject 和 Observer 不再需要自己定义，直接使用相应的工具类即可。

定义 Weather，对应类图中的 ConcreteSubject：

```java
// Observable 是 java.util 包下的类，对应类图中的 Subject
public class Weather extends Observable {
    private WeatherType currentWeather;

    public Weather() {
        currentWeather = WeatherType.SUNNY;
    }

    public WeatherType getCurrentWeather() {
        return currentWeather;
    }

    public void timePasses() {
        WeatherType[] enumValues = WeatherType.values();
        currentWeather = enumValues[(currentWeather.ordinal() + 1) % enumValues.length];
        // 通知观察者前，需要先调用该方法将被观察者的状态标记为已改变，否则通知无效
        super.setChanged();
        super.notifyObservers();
    }
}
```

定义 Hobbits、Orcs，对应类图中的 ConcreteObserver：

```java
// Observer 是 java.util 包下的类，对应类图中的 Observer
public class Hobbits implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        if (o instanceof Weather) {
            WeatherType currentWeather = ((Weather) o).getCurrentWeather();
            System.out.println("The Hobbits watch the weather forecast : " + currentWeather);
        }
    }
}
```

```java
// Observer 是 java.util 包下的类，对应类图中的 Observer
public class Orcs implements Observer{
    @Override
    public void update(Observable o, Object arg) {
        if (o instanceof Weather) {
            WeatherType currentWeather = ((Weather) o).getCurrentWeather();
            System.out.println("The Orcs watch the weather forecast : " + currentWeather);
        }
    }
}
```

客户端调用与上面的示例相同，不再重复

# 总结

观察者模式的本质是触发联动，是典型的一对多的关系

观察者模式优点：

* 观察者模式实现了主题对象和观察者对象之间的抽象耦合，实现了具体目标对象和具体观察者对象之间的解耦
* 观察者模式实现了动态联动，可以在运行时控制注册和取消注册观察者对象

观察者模式缺点：

* 有些观察者并不需要一些通知，可能会引起无谓的更新或误更新
* Java 中主题对象发出通知时，观察者对象默认是顺序更新的，一个观察者阻塞，可能会影响整体的执行效率，这种情况下可以考虑使用异步实现

# Ref

* [Observer Pattern](http://www.oodesign.com/observer-pattern.html)
* [java-design-patterns-observer](https://github.com/iluwatar/java-design-patterns/blob/master/observer/README.md)