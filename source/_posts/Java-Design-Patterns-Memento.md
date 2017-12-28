---
title: 备忘录模式(Memento Pattern)
date: 2017-11-29 15:06:00
tags: 行为型模式(Behavioral Pattern) 
categories: Java Design Patterns
---

备忘录模式属于行为型模式

<!-- more -->

# 意图

* 在不破坏封闭的原则下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态

# 场景

当遇到下面场景时可以考虑使用备忘录模式：

* 需要保存一个对象在某一时刻全部或部分的状态，方便以后在需要的时候可以把这个对象恢复到先前的状态时
* 需要保存一个对象的内部状态，但又不希望暴露对象的实现细节和破坏对象的封装性时

# UML 类图

备忘录模式类图如下：

![Java-Design-Patterns-Memento.png](http://otg3f8t90.bkt.clouddn.com/2017/12/28/Java-Design-Patterns-Memento.png)

类图说明：

* Memento：备忘录接口，只起到标识的作用，为了防止原发器之外的对象操作备忘录对象，该接口一般不提供任何方法
* ConcreteMemento：Memento 接口的具体实现，主要用来封装原发器对象的内部状态，其数据只能被原发器对象访问和操作。所以通常实现为原发器对象的私有内部类
* Originator：原发器，需要通过备忘录对象存储和恢复自身状态的对象，提供创建备忘录对象的方法和通过备忘录对象恢复自身状态的方法
* Caretaker：负责管理备忘录对象，不能对备忘录对象的内容进行操作和访问，只能保存备忘录对象和向原发器提供备忘录对象
* Client：客户端

# 实现

下面的示例代码演示了简单的加减运算和撤销、恢复操作，而撤销、恢复操作的实现方式又分为两种，一种是补偿式的，又称反操作式的，一种是存储恢复式。示例结合了命令模式和备忘录模式实现了第二种撤销、恢复的操作

定义　Ｍemento：

```java
public interface Memento {}
```

定义　Operation，备忘录模式主要在这个类中体现，对应类图中的　Originator：

```java
public class Operation {
    private int result;

    public int getResult() {
        return result;
    }

    public void add(int num) {
        result += num;
    }

    public void subtract(int num) {
        result -= num;
    }

    // 提供创建备忘录对象的方法
    public Memento createMemento() {
        MementoImpl mementoImpl = new MementoImpl();
        mementoImpl.setResult(result);
        return mementoImpl;
    }

    // 提供通过备忘录对象恢复状态的方法
    public void setMemento(Memento memento) {
        MementoImpl mementoImpl = (MementoImpl) memento;
        this.result = mementoImpl.getResult();
    }

    // 对应类图中的　ConcreteMemento
    private static class MementoImpl implements Memento {
        private int result;

        int getResult() {
            return result;
        }

        void setResult(int result) {
            this.result = result;
        }
    }
}
```

定义　Command　接口，这里使用了命令模式：

```java
public interface Command {

    void execute(int num);

    void undo(Memento memento);

    void redo(Memento memento);

    Memento createMemento();
}
```

定义　Command 接口的抽象实现：

```java
public abstract class AbstractCommand implements Command {
    protected Operation operation;

    public AbstractCommand(Operation operation) {
        this.operation = operation;
    }

    @Override
    public abstract void execute(int num);

    @Override
    public void undo(Memento memento) {
        operation.setMemento(memento);
    }

    @Override
    public void redo(Memento memento) {
        operation.setMemento(memento);
    }

    @Override
    public Memento createMemento() {
        return operation.createMemento();
    }
}
```

定义加法操作的命令:

```java
public class AddCommand extends AbstractCommand{

    public AddCommand(Operation operation) {
        super(operation);
    }

    @Override
    public void execute(int num) {
        this.operation.add(num);
    }
}
```

定义减法操作的命令：

```java
public class SubtractCommand extends AbstractCommand {

    public SubtractCommand(Operation operation) {
        super(operation);
    }

    @Override
    public void execute(int num) {
        this.operation.subtract(num);
    }
}
```

定义　Calculator，对应命令模式中的　Invoker，同时也对应备忘录模式中的　Caretaker：

```java
public class Calculator {
    private Deque<Memento> undoMementos = new LinkedList<>();
    private Deque<Memento> redoMementos = new LinkedList<>();
    private Deque<Command> undoCommands = new LinkedList<>();
    private Deque<Command> redoCommands = new LinkedList<>();

    public void calculate(Command command, int num) {
        undoMementos.offerLast(command.createMemento());
        undoCommands.offerLast(command);
        command.execute(num);
    }

    public void undo() {
        if (!undoMementos.isEmpty()) {
            Command previousCmd = undoCommands.pollLast();
            redoCommands.offerLast(previousCmd);

            Memento previousMemento = undoMementos.pollLast();
            redoMementos.offerLast(previousCmd.createMemento());
            previousCmd.undo(previousMemento);
        }
    }

    public void redo() {
        if (!redoMementos.isEmpty()) {
            Command previousCmd = redoCommands.pollLast();
            undoCommands.offerLast(previousCmd);

            Memento previousMemento = redoMementos.pollLast();
            undoMementos.offerLast(previousCmd.createMemento());
            previousCmd.redo(previousMemento);
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Operation operation = new Operation();
        System.out.println(operation.getResult());

        Calculator calculator = new Calculator();
        calculator.calculate(new AddCommand(operation), 5);
        System.out.println(operation.getResult());

        calculator.calculate(new SubtractCommand(operation), 3);
        System.out.println(operation.getResult());

        calculator.undo();
        System.out.println(operation.getResult());

        calculator.redo();
        System.out.println(operation.getResult());
    }
}
```

# 总结

备忘录模式的本质是保存和恢复内部状态。保存是手段，恢复是目的。

备忘录模式优点：

* 使用备忘录对象存储原发器的内部状态，并保存在原发器外部，但其只能被原发器访问和操作，具有很好的封装性
* 提供了一种恢复状态的机制，可以方便的回到某个历史状态
* 备忘录对象存储在原发器外部，简化了原发器

备忘录模式缺点：

* 原发器内部状态过多或者频繁创建备忘录对象时，因为需要缓存备忘录对象，所以内存开销可能比较大

# Ref

* [Memento Pattern](http://www.oodesign.com/memento-pattern.html)
* [java-design-patterns-memento](https://github.com/iluwatar/java-design-patterns/blob/master/memento/README.md)