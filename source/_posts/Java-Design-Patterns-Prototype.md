---
title: 原型模式(Prototype Pattern)
date: 2016-11-29 15:06:00
tags: [Creational Pattern, GoF]
categories: Java Design Patterns
---

原型模式属于创建型模式

<!-- more -->

# 意图

* 用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象

# 场景

* 类初始化需要消耗非常多的资源，可以通过原型拷贝避免这些消耗
* 通过 new 创建一个对象需要非常繁琐的数据准备或访问权限，这时可以使用原型模式直接拷贝对象
* 一个对象需要提供给其他多个对象访问，而各个调用者可能都需要修改其值的时，可以考虑使用原型模式拷贝多个对象供各个调用者使用，即保护性拷贝

> 上边的第一个和第二个场景只针对使用 Cloneable 接口和　Object.clone() 实现的原型模式有效

# UML 类图

原型模式的类图如下：

![Prototype.png](http://otg3f8t90.bkt.clouddn.com/2017/12/12/Prototype.png)

类图说明：

* Prototype：声明克隆自身的接口，用来约束想要用来克隆自己的类
* ConcreteType：具体的原型类，实现了拷贝自身的功能
* Client：客户端，首先获取原型实例，然后通过原型实例的拷贝方法获取新的对象

# 拷贝深度

原型模式主要使用来拷贝自身对象，在使用的时候会出现浅拷贝(shallow clone)和深拷贝(deep　clone)的问题：

* 浅拷贝

    也叫影子拷贝，只拷贝值传递的数据，如基本数据类型和 String 类型等

* 深拷贝

    既拷贝值传递的数据也拷贝引用传递的数据，实现被拷贝实例所有属性的拷贝。深拷贝要求涉及到的对象都正确实现拷贝方法并递归地进行拷贝

在开发过程中，如无特殊需要，为避免出错，都要使用深拷贝

# 实现

原型模式主要是实现原型接口定义的拷贝方法，但其具体实现没有统一的要求和算法，所以可以自定义实现，也可以利用 Java 语言平台特性实现。

## 自定义实现

定义 Prototype 接口：

```java
// 使用泛型可以避免强转
public interface Prototype<T> {
    T clone();
}
```

定义　WeaponType 用于标识　Weapon　类型：

```java
public enum WeaponType {
    SWORD, AXE, BOW
}
```

定义 Weapon，对应类图中的 ConcretePrototype：

```java
public class Weapon implements Prototype<Weapon> {
    private WeaponType weaponType;

    public Weapon(WeaponType weaponType) {
        this.weaponType = weaponType;
    }

    public void setWeaponType(WeaponType weaponType) {
        this.weaponType = weaponType;
    }

    public WeaponType getWeaponType() {
        return weaponType;
    }

    @Override
    public Weapon clone() {
        return new Weapon(weaponType);
    }

    @Override
    public String toString() {
        return weaponType.name().toLowerCase();
    }
}
```

定义 Hero，也对应类图中的 ConcretePrototype：

```java
public class Hero implements Prototype<Hero> {
    private String name;
    private Weapon weapon;

    public Hero(String name, Weapon weapon) {
        this.name = name;
        this.weapon = weapon;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public WeaponType getWeapon() {
        return weapon.getWeaponType();
    }

    public void setWeapon(WeaponType weaponType) {
        this.weapon.setWeaponType(weaponType);
    }

    @Override
    public Hero clone() {
        // 深拷贝
        return new Hero(name, weapon.clone());

        // 浅拷贝
        // return new Hero(name, weapon);
    }

    @Override
    public String toString() {
        return "This is a hero named " + name + " with " + weapon;
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Hero hero = new Hero("Thief", new Weapon(WeaponType.BOW));
        System.out.println(hero);

        Hero newHero = hero.clone();
        System.out.println(newHero);

        // 浅拷贝的拷贝对象的属性改变会影响到原型对象，深拷贝则不会，两个对象完全独立
        newHero.setName("Warrior");
        newHero.setWeapon(WeaponType.SWORD);
        System.out.println(newHero);

        System.out.println(hero);
    }
}
```

这种实现方式可以完整的体会原型模式，但具体的克隆实现还是通过 new 操作符创建对象然后给属性赋值，并不能实现减少对象创建的资源消耗，只是对拷贝对象的过程进行了封装。

## 使用 Cloneable 接口和 Object.Clone() 方法实现

实现原型模式也可以利用 Java 的语言特性实现，不需要定义 Prototype 接口，直接使用 Cloneable 接口，该接口是个标识接口，没有需要实现的方法，使用的时候需要重写 Object 类中的 clone() 方法，调用 super.Clone() 实现拷贝，这种拷贝是在内存中的二进制流拷贝，所以比直接使用　new 创建对象的性能好很多，但这种拷贝方式不会调用原型类的构造方法，使用的时候需要注意。下面是实现方式:

定义　Prototype：

```java
public abstract class Prototype<T> implements Cloneable {
    @Override
    protected T clone() throws CloneNotSupportedException {
        return (T) super.clone();
    }
}
```

> 利用　Java　语言特性实现原型模式时不必定义该接口，但为了清晰的演示原型模式，这里依旧定义了该抽象

定义 Weapon，对应类图中的 ConcretePrototype：

```java
public class Weapon extends Prototype<Weapon> {
    private WeaponType weaponType;

    public Weapon(WeaponType weaponType) {
        this.weaponType = weaponType;
    }

    public void setWeaponType(WeaponType weaponType) {
        this.weaponType = weaponType;
    }

    public WeaponType getWeaponType() {
        return weaponType;
    }

    @Override
    protected Weapon clone() throws CloneNotSupportedException {
        // 这里可以直接调用　super.clone() 而不必手动创建新对象
        return super.clone();
    }

    @Override
    public String toString() {
        return weaponType.name().toLowerCase();
    }
}
```

> Weapon 中的　WeaponType 与上面自定义实现原型模式的示例中定义相同，这里不再重复定义

定义 Hero，也对应类图中的 ConcretePrototype：

```java
public class Hero extends Prototype<Hero> {
    private String name;
    private Weapon weapon;

    public Hero(String name, Weapon weapon) {
        this.name = name;
        this.weapon = weapon;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public WeaponType getWeapon() {
        return weapon.getWeaponType();
    }

    public void setWeapon(WeaponType weaponType) {
        this.weapon.setWeaponType(weaponType);
    }

    @Override
    public Hero clone() throws CloneNotSupportedException {
        // 深拷贝
        Hero hero = super.clone();
        hero.name = name;
        hero.weapon = weapon.clone();
        return hero;

        // 浅拷贝
        // return super.clone();
    }

    @Override
    public String toString() {
        return "This is a hero named " + name + " with " + weapon;
    }
}
```

然后就是客户端调用，代码也与上面的相同，不再重复定义

# 总结

原型模式的本质是拷贝生成对象，拷贝是手段，目的是生成对象。

原型模式的优点：

* 通过 Cloneable 接口和　Object.clone() 方法实现的原型模式是在内存中二进制流的拷贝，比 new 对象的性能好很多，特别是对于嵌套很多引用对象的原型类，效果更明显
* 原型类有统一的接口，可以通过接口获取拷贝对象，可以在运行时动态改变具体的实现类型

原型模式的缺点：

* 若要实现深拷贝，需要使所有涉及的引用类型对象都实现拷贝方法并递归调用
* 通过 Cloneable 接口和　Object.clone() 方法实现的原型模式拷贝对象时是直接在内存中拷贝，原型类的构造方法不会被调用，这是个潜在的问题

# Ref

* [Prototype Pattern](http://www.oodesign.com/prototype-pattern.html)
* [java-design-patterns-prototype](https://github.com/iluwatar/java-design-patterns/blob/master/prototype/README.md)