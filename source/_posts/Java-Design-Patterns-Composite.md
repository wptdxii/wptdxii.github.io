---
title: 组合模式(Composite Pattern)
date: 2017-11-29 15:06:00
tags: [Structural Pattern, GoF]
categories: Java Design Patterns
---

桥接模式属于结构型模式

<!-- more -->

# 意图

* 将对象组合成树状结构以表示“部分-整体”的层次结构，使客户端对单个对象和组合对象的使用具有一致性

# 场景

当遇到下面场景时可以考虑使用组合模式：

* 需要表示对象部分-整体的树形层次结构
* 当需要忽略单个对象和组合对象之间的不同，统一地使用组合结构中的所有对象

# UML 类图

组合模式有两种实现方式，一种是透明的组合模式，一种是安全的组合模式。

透明的组合模式类图如下：

![Java-Design-Patterns-Composite-Uniform.png](http://otg3f8t90.bkt.clouddn.com/2018/2/5/Java-Design-Patterns-Composite-Uniform.png)

安全的组合模式类图如下：

![Java-Design-Patterns-Composite-Safe.png](http://otg3f8t90.bkt.clouddn.com/2018/2/5/Java-Design-Patterns-Composite-Safe.png)

类图说明：

* Component：抽象的组件节点，为组合中的对象声明接口
  * 透明组合模式的实现中，该类中除了会定义待子类实现的抽象的方法外，还会定义组合节点所需要的增删查子节点的方法，这些方法通常会提供抛出异常的缺省实现，组合节点需要重写这些方法，而叶子对象不用处理
  * 安全组合模式的实现中，该类中只定义了待子类实现的抽象方法
* Composite：组合节点对象，用于存储子节点，包括组合节点和叶子节点
  * 透明组合模式的实现中，需要重写父类的缺省实现
  * 安全组合模式的实现中，需要定义组合节点所需要的增删查子节点的方法
* Leaf：叶子节点对象，树形结构中不可以包含子节点的节点
* Client：客户端

透明的组合模式使组合节点对象和叶子节点对象的使用具有一致性，即客户端可以使用统一的 Component 接口，而不必区分组合节点对象和叶子节点对象，但因为叶子节点不具备包含子节点的能力，所以客户端在使用叶子节点对象增删查子节点时可能会出现异常，这就是其不安全的地方；安全的组合模式将增删查子节点的方法定义在组合节点对象中而非公共接口中，这在根本上避免了叶子节点对象操作增删查子对节点，因而客户端可以安全地使用叶子节点对象，但这种实现的弊端是组合节点对象和叶子节点对象在使用上不再具有一致性，使用时必须区分，不能再使用统一的 Component 接口。以上两种实现方法可以根据需求自行选择，但透明的实现方法更符合组合模式的本意。

# 实现

## 透明组合模式实现

定义 LetterComponent，对应类图中的 Component：

```java
public abstract class LetterComponent {

    // 添加子节点，只有组合节点才应该有这个方法，所以组合节点需要重写该方法，叶子节点被调用时直接抛异常
    public void add(LetterComponent child) {
        throw new UnsupportedOperationException("cant't add child nodes!");
    }

    // 待子类实现的抽象方法
    public abstract void print();
}
```

定义 LetterComposite、Word、Sentence，其中 Word、Sentence 对应类图中的 Composite，LetterComposite 是公共部分抽取出来的父类：

```java
public abstract class LetterComposite extends LetterComponent {
    private List<LetterComponent> children = new ArrayList<>();

    public LetterComposite(List<LetterComponent> children) {
        for (LetterComponent child : children) {
            add(child);
        }
    }

    @Override
    public void add(LetterComponent child) {
        this.children.add(child);
    }

    @Override
    public void print() {
        for (LetterComponent child : children) {
            child.print();
        }
    }
}
```

```java
public class Word extends LetterComposite {

    public Word(List<LetterComponent> children) {
        super(children);
    }

    @Override
    public void print() {
        System.out.print(" ");
        super.print();
    }
}
```

```java
public class Sentence extends LetterComposite {

    public Sentence(List<LetterComponent> children) {
        super(children);
    }

    @Override
    public void print() {
        super.print();
        System.out.println(".");
    }
}
```

定义 Letter，对应类图中的 Leaf：

```java
// 叶子节点调用 add() 方法则会抛异常，这是其不安全的地方
public class Letter extends LetterComponent {
    private char letter;

    public Letter(char letter) {
        this.letter = letter;
    }

    @Override
    public void print() {
        System.out.print(letter);
    }
}
```

定义 Messenger，用于构建树形结构，方便客户端调用，未在类图中标出：

```java
// 可以使用统一的接口，但如果是叶子对象，调用 add() 会抛异常
public class Messenger {

    public LetterComponent messageForOrcs() {
        List<LetterComponent> words = new ArrayList<>();

        words.add(new Word(Arrays.asList(new Letter('W'), new Letter('h'), new Letter('e'), new Letter(
                'r'), new Letter('e'))));
        words.add(new Word(Arrays.asList(new Letter('t'), new Letter('h'), new Letter('e'), new Letter(
                'r'), new Letter('e'))));
        words.add(new Word(Arrays.asList(new Letter('i'), new Letter('s'))));
        words.add(new Word(Arrays.asList(new Letter('a'))));
        words.add(new Word(Arrays.asList(new Letter('w'), new Letter('h'), new Letter('i'), new Letter(
                'p'))));
        words.add(new Word(Arrays.asList(new Letter('t'), new Letter('h'), new Letter('e'), new Letter(
                'r'), new Letter('e'))));
        words.add(new Word(Arrays.asList(new Letter('i'), new Letter('s'))));
        words.add(new Word(Arrays.asList(new Letter('a'))));
        words.add(new Word(Arrays.asList(new Letter('w'), new Letter('a'), new Letter('y'))));

        return new Sentence(words);
    }

    public LetterComponent messageFromElves() {
        List<LetterComponent> words = new ArrayList<>();

        words.add(new Word(Arrays.asList(new Letter('M'), new Letter('u'), new Letter('c'), new Letter(
                'h'))));
        words.add(new Word(Arrays.asList(new Letter('w'), new Letter('i'), new Letter('n'), new Letter(
                'd'))));
        words.add(new Word(Arrays.asList(new Letter('p'), new Letter('o'), new Letter('u'), new Letter(
                'r'), new Letter('s'))));
        words.add(new Word(Arrays.asList(new Letter('f'), new Letter('r'), new Letter('o'), new Letter(
                'm'))));
        words.add(new Word(Arrays.asList(new Letter('y'), new Letter('o'), new Letter('u'), new Letter(
                'r'))));
        words.add(new Word(Arrays.asList(new Letter('m'), new Letter('o'), new Letter('u'), new Letter(
                't'), new Letter('h'))));

        return new Sentence(words);
    }
}
```

客户端调用：

```java
public class UniformClient {
    public static void main(String[] args) {
        Messenger messenger = new Messenger();
        messenger.messageForOrcs().print();
        messenger.messageFromElves().print();
    }
}
```

## 安全组合模式实现

安全组合模式的实现与透明组合模式的结构一致，只是具体实现不同

定义 LetterComponent

```java
public interface LetterComponent {
    void print();
}
```

定义 LetterComposite、Word、Sentence：

```java
// 只有组合节点可以调用 add() 方法
public abstract class LetterComposite implements LetterComponent {
    private List<LetterComponent> children = new ArrayList<>();

    public void add(LetterComponent child) {
        this.children.add(child);
    }

    @Override
    public void print() {
        for (LetterComponent child : children) {
            child.print();
        }
    }
}
```

```java
public class Word extends LetterComposite {

    public Word(List<Letter> letters) {
        for (Letter letter : letters) {
            add(letter);
        }
    }

    @Override
    public void print() {
        System.out.print(" ");
        super.print();
    }
}
```

```java
public class Sentence extends LetterComposite {

    public Sentence(List<Word> words) {
        for (Word word : words) {
            this.add(word);
        }
    }

    @Override
    public void print() {
        super.print();
        System.out.println(".");
    }
}
```

定义 Letter：

```java
public class Letter implements LetterComponent {
    private char letter;

    public Letter(char letter) {
        this.letter = letter;
    }

    @Override
    public void print() {
        System.out.print(letter);
    }
}
```

定义 Messenger：

```java
// 不能使用统一的接口，如果想调用 add()，必须强转为组合对象
public class Messenger {

    public LetterComponent messageForOrcs() {
        List<Word> words = new ArrayList<>();

        words.add(new Word(Arrays.asList(new Letter('W'), new Letter('h'), new Letter('e'), new Letter(
                'r'), new Letter('e'))));
        words.add(new Word(Arrays.asList(new Letter('t'), new Letter('h'), new Letter('e'), new Letter(
                'r'), new Letter('e'))));
        words.add(new Word(Arrays.asList(new Letter('i'), new Letter('s'))));
        words.add(new Word(Arrays.asList(new Letter('a'))));
        words.add(new Word(Arrays.asList(new Letter('w'), new Letter('h'), new Letter('i'), new Letter(
                'p'))));
        words.add(new Word(Arrays.asList(new Letter('t'), new Letter('h'), new Letter('e'), new Letter(
                'r'), new Letter('e'))));
        words.add(new Word(Arrays.asList(new Letter('i'), new Letter('s'))));
        words.add(new Word(Arrays.asList(new Letter('a'))));
        words.add(new Word(Arrays.asList(new Letter('w'), new Letter('a'), new Letter('y'))));

        return new Sentence(words);
    }

    public LetterComponent messageFromElves() {
        List<Word> words = new ArrayList<>();

        words.add(new Word(Arrays.asList(new Letter('M'), new Letter('u'), new Letter('c'), new Letter(
                'h'))));
        words.add(new Word(Arrays.asList(new Letter('w'), new Letter('i'), new Letter('n'), new Letter(
                'd'))));
        words.add(new Word(Arrays.asList(new Letter('p'), new Letter('o'), new Letter('u'), new Letter(
                'r'), new Letter('s'))));
        words.add(new Word(Arrays.asList(new Letter('f'), new Letter('r'), new Letter('o'), new Letter(
                'm'))));
        words.add(new Word(Arrays.asList(new Letter('y'), new Letter('o'), new Letter('u'), new Letter(
                'r'))));
        words.add(new Word(Arrays.asList(new Letter('m'), new Letter('o'), new Letter('u'), new Letter(
                't'), new Letter('h'))));

        return new Sentence(words);
    }
}
```

客户端调用与上面一样，不再重复定义

# 总结

组合模式的本质是统一组合节点对象和叶子节点对象。

组合模式优点：

* 可以方便地扩展组合对象和叶子对象
* 清楚地定义叶子对象和组合对象地类层次结构
* 高层模块使用叶子对象和组合对象时具有一致性，简化了高层模块的代码(需要使用透明的实现方式)

组合模式缺点：

* 很难限制组合中的组件类型，因为它们具有相同的抽象层，需要在运行时动态检测
* 透明实现方式的安全性和安全实现方式的透明性

# Ref

* [Composite Pattern](http://www.oodesign.com/composite-pattern.html)
* [java-design-patterns-composite](https://github.com/iluwatar/java-design-patterns/blob/master/composite/README.md)
