---
title: 建造者模式(Builder Pattern)
date: 2017-11-29 15:06:00
tags: [Creational Pattern, GoF]
categories: Java Design Patterns
---

建造者模式也叫做生成器模式，属于创建型模式

<!-- more -->

# 意图

* 将一个复杂对象的构建和它的表示分离，使得同样的构建过程可以创建不同的表示

# 场景

当遇到下列场景时可以考虑使用建造者模式：

* 创建对象的过程(算法)，应该独立于该对象的组成部分和它们的装配方式
* 相同的构建过程有着不同的表示
* 对象的构造比较复杂，有着众多的参数和配置

# UML 类图

建造者模式的类图如下：

![Java-Design-Patterns-Builder.png](http://otg3f8t90.bkt.clouddn.com/2017/12/8/Java-Design-Patterns-Builder.png)

类图说明：

* Product：被构建的对象，一般为具体类，不必要定义抽象接口，其由 PartA、PartB 组成。
* Builder：建造者接口，定义创建一个 Product 对象所需的各个部件的接口和返回 Product 对象的接口。如果有需要，可以在这两个地方添加约束规则
* ConcreteBuilder：Builder 接口的具体实现，负责 Product 对象所需的各个部件的创建和组装，同时提供一个获取组装完成的 Product 对象的方法。切换不同的实现，即可通过相同的构建过程构建不同的 Product 对象
* Director：主要用来使用 Builder 接口，以统一的过程构建所需的 Product 对象，代表可重用的构建过程。在构建的过程中在需要创建和组装具体部件的时候，将这些功能通过委托，交给　Builder 去完成。Ｄirector 通过　Builder 方法的参数和返回值与　Builder 进行交互
* Client：外部调用，通过 Director 和具体的 Builder 实现来构建复杂的 Product 对象

# 实现

## 标准实现

定义 Profession、Weapon、Armor、HairColor、HairStyle，对应类图中的 Part：

```java
public enum Profession {
    WARRIOR, THIEF, MAGE, PRIEST;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

```java
public enum Weapon {
    DAGGER, SWORD, AXE, WARHAMMER, BOW;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

```java
public enum Armor {
    CLOTHES, LEATHER, CHAIN_MAIL, PLATE_MAIL;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

```java
public enum HairColor {
    WHITE, BLOND, RED, BROWN, BLACK;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

```java
public enum HairStyle {
    BALD, SHORT, CURLY, LONG_STRAIGHT, LONG_CURLY;

    @Override
    public String toString() {
        return name().toLowerCase();
    }
}
```

定义 Hero，对应类图中的 Product：

```java
// 为了防止在包外创建对象，setter 方法修饰符都设为 default
public final class Hero {
    private String name;
    private Profession profession;
    private Weapon weapon;
    private Armor armor;
    private HairStyle hairStyle;
    private HairColor hairColor;

    public String getName() {
        return name;
    }

    void setName(String name) {
        this.name = name;
    }

    public Profession getProfession() {
        return profession;
    }

    void setProfession(Profession profession) {
        this.profession = profession;
    }

    public Weapon getWeapon() {
        return weapon;
    }

    void setWeapon(Weapon weapon) {
        this.weapon = weapon;
    }

    public Armor getArmor() {
        return armor;
    }

    void setArmor(Armor armor) {
        this.armor = armor;
    }

    public HairStyle getHairStyle() {
        return hairStyle;
    }

    void setHairStyle(HairStyle hairStyle) {
        this.hairStyle = hairStyle;
    }

    public HairColor getHairColor() {
        return hairColor;
    }

    void setHairColor(HairColor hairColor) {
        this.hairColor = hairColor;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("This is a ")
                .append(profession)
                .append(" named ")
                .append(name);

        if (hairStyle != null || hairColor != null) {
            sb.append(" with ");

            if (hairColor != null) {
                sb.append(hairColor).append(" ");
            }

            if (hairStyle != null) {
                sb.append(hairStyle).append(" ");
            }

            sb.append(hairStyle != HairStyle.BALD ? "hair" : "head");
        }

        if (armor != null) {
            sb.append(" wearing ").append(armor);
        }
        if (weapon != null) {
            sb.append(" and wielding a ").append(weapon);
        }
        sb.append('.');

        return sb.toString();
    }
}
```

定义 Builder：

```java
public interface Builder {
    void buildProfession(Profession profession);

    void buildWeapon(Weapon weapon);

    void buildArmor(Armor armor);

    void buildHairStyle(HairStyle hairStyle);

    void buildHairColor(HairColor hairColor);

    Hero build();
}
```

实现 Builder 接口：

```java
public class ConcreteBuilder implements Builder {
    private Hero hero;

    // 用构造器约束必备参数
    public ConcreteBuilder(String name, Profession profession) {
        if (name == null || name.trim().isEmpty() || profession == null) {
            throw new IllegalArgumentException("Hero name can't be null");
        }

        hero = new Hero();
        hero.setName(name);
        hero.setProfession(profession);
    }

    @Override
    public void buildProfession(Profession profession) {
        hero.setProfession(profession);
    }

    @Override
    public void buildWeapon(Weapon weapon) {
        hero.setWeapon(weapon);
    }

    @Override
    public void buildArmor(Armor armor) {
        hero.setArmor(armor);
    }

    @Override
    public void buildHairStyle(HairStyle hairStyle) {
        hero.setHairStyle(hairStyle);
    }

    @Override
    public void buildHairColor(HairColor hairColor) {
        hero.setHairColor(hairColor);
    }

    @Override
    public Hero build() {
        // 可以在这里添加约束规则
        return hero;
    }
}
```

定义 Director：

```java
// construct 方法中只是简单的顺序调用，实际应用中通常都是定义的比较复杂的构建算法
public class Director {
    private Builder builder;

    public void setBuilder(Builder builder) {
        this.builder = builder;
    }

    public void constructMage() {
        builder.buildHairColor(HairColor.BLACK);
        builder.buildWeapon(Weapon.DAGGER);
    }

    public void constructWarrior() {
        builder.buildHairColor(HairColor.BLOND);
        builder.buildHairStyle(HairStyle.LONG_CURLY);
        builder.buildArmor(Armor.CHAIN_MAIL);
        builder.buildWeapon(Weapon.SWORD);
    }

    public void constructThief() {
        builder.buildHairStyle(HairStyle.BALD);
        builder.buildWeapon(Weapon.BOW);
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Director director = new Director();
        constructMage(director);
        constructWarrior(director);
        constructThief(director);
    }

    private static void constructMage(Director director) {
        Builder mageBuilder = new ConcreteBuilder("Mage", Profession.MAGE);
        director.setBuilder(mageBuilder);
        director.constructMage();
        System.out.println(mageBuilder.build());
    }

    private static void constructWarrior(Director director) {
        Builder warriorBuilder = new ConcreteBuilder("Warrior", Profession.WARRIOR);
        director.setBuilder(warriorBuilder);
        director.constructWarrior();
        System.out.println(warriorBuilder.build());
    }

    private static void constructThief(Director director) {
        Builder thiefBuilder = new ConcreteBuilder("Thief", Profession.THIEF);
        director.setBuilder(thiefBuilder);
        director.constructThief();
        System.out.println(thiefBuilder.build());
    }
}
```

## 简化实现

建造者模式更常用的是其简化形式，就是将　Client 和　Director 融合，同时将　Builder　内联到　Product　中去，通过链式调用构建　Product 对象。
只需要定义一个具有内联 Builder 的 Product 类即可，具体实现如下：

```java
public final class Hero {
    private final String name;
    private final Profession profession;
    private final Weapon weapon;
    private final Armor armor;
    private final HairStyle hairStyle;
    private final HairColor hairColor;

    private Hero(Builder builder) {
        this.name = builder.name;
        this.profession = builder.profession;
        this.weapon = builder.weapon;
        this.armor = builder.armor;
        this.hairStyle = builder.hairStyle;
        this.hairColor = builder.hairColor;
    }

    public String getName() {
        return name;
    }

    public Profession getProfession() {
        return profession;
    }


    public Weapon getWeapon() {
        return weapon;
    }


    public Armor getArmor() {
        return armor;
    }


    public HairStyle getHairStyle() {
        return hairStyle;
    }


    public HairColor getHairColor() {
        return hairColor;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("This is a ")
                .append(profession)
                .append(" named ")
                .append(name);

        if (hairStyle != null || hairColor != null) {
            sb.append(" with ");

            if (hairColor != null) {
                sb.append(hairColor).append(" ");
            }

            if (hairStyle != null) {
                sb.append(hairStyle).append(" ");
            }

            sb.append(hairStyle != HairStyle.BALD ? "hair" : "head");
        }

        if (armor != null) {
            sb.append(" wearing ").append(armor);
        }
        if (weapon != null) {
            sb.append(" and wielding a ").append(weapon);
        }
        sb.append('.');

        return sb.toString();
    }

    public static final class Builder {
        private final String name;
        private final Profession profession;
        private Weapon weapon;
        private Armor armor;
        private HairStyle hairStyle;
        private HairColor hairColor;

        // 用构造器约束必备参数
        public Builder(String name, Profession profession) {
            if (name == null || name.trim().isEmpty() || profession == null) {
                throw new IllegalArgumentException("Hero name can't be null");
            }
            this.name = name;
            this.profession = profession;
        }

        public Builder weapon(Weapon weapon) {
            this.weapon = weapon;
            return this;
        }

        public Builder armor(Armor armor) {
            this.armor = armor;
            return this;
        }

        public Builder hairStyle(HairStyle hairStyle) {
            this.hairStyle = hairStyle;
            return this;
        }

        public Builder hairColor(HairColor hairColor) {
            this.hairColor = hairColor;
            return this;
        }

        public Hero build() {
            // 可以在这里添加校验规则
            return new Hero(this);
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Hero mage = new Hero.Builder("Mage", Profession.MAGE)
                .hairColor(HairColor.BLACK)
                .weapon(Weapon.DAGGER).build();
        System.out.println(mage);

        Hero warrior = new Hero.Builder("Warrior", Profession.WARRIOR)
                .hairColor(HairColor.BLOND)
                .hairStyle(HairStyle.LONG_CURLY)
                .weapon(Weapon.SWORD)
                .armor(Armor.CHAIN_MAIL).build();
        System.out.println(warrior);

        Hero thief = new Hero.Builder("Thief", Profession.THIEF)
                .hairStyle(HairStyle.BALD)
                .weapon(Weapon.BOW).build();
        System.out.println(thief);
    }
}
```

# 总结

建造者模式的主要功能是细化的、分步骤的构建复杂的对象，其本质在于分离构建算法和具体的构建实现，从而使构建算法可以复用，使具体的构造实现可以方便地扩展和切换。建造者模式强调的是对整体构建算法的固定和对具体构建实现的灵活切换。

建造者模式的优点：

* Director 可复用
* Builder 易扩展，实现　Builder 接口即可
* 良好的封装性，Product 内部组成的细节对 Client 透明

建造者模式的缺点：

* 会产生多余的　Director 对象和　Builder 对象，消耗内存

# Ref

* [Builder Pattern](http://www.oodesign.com/builder-pattern.html)
* [java-design-patterns-builder](https://github.com/iluwatar/java-design-patterns/blob/master/builder/README.md)