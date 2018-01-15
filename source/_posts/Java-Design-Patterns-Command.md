---
title: 命令模式(Command Pattern)
date: 2017-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

命令模式也叫行为模式(Action Patter)或事务模式(Transaction Patter)，属于行为型模式

<!-- more -->

# 意图

* 将一个请求封装成一个对象，从而让用户使用不同的请求将客户端参数化
* 对请求排队或记录请求日志
* 支持撤销、恢复操作

# 场景

当遇到下面场景可以考虑使用命令模式：

* 需要抽象出待执行的行为，然后以参数的形式提供出来。命令模式是回调机制的一个面向对象的的替代品
* 需要在不同的时间指定、排列和执行请求。将这些请求封装成命令对象，然后实现请求队列化
* 需要支持撤销、恢复操作
* 需要支持日志功能，当系统崩溃时，可以根据日志获取命令列表，再重新执行一遍功能
* 需要支持事务操作

# UML 类图

命令模式类图如下：

![Java-Design-Patterns-Command.png](http://otg3f8t90.bkt.clouddn.com/2017/12/19/Java-Design-Patterns-Command.png)

类图说明：

* Command：命令的抽象接口
* ConcreteCommand：具体的命令对象，调用 Receiver 相应的方法实现要执行的操作
* Receiver：命令对象的接收者，真正执行命令的对象，可以是任意的对象，只有能实现相应的功能即可
* Invoker：请求者，通过命令发出请求
* Client：创建具体的命令对象并设置命令对象的接收者，并非常规意义上的客户端，而是在组装命令对象和接收者

# 实现

命令模式有其标准实现，比较简单，下面代码演示了利用命令模式实现撤销、恢复操作。可撤销操作的实现方式又分为两种，一种是补偿式的，又称反操作式的，一种是存储恢复式的，下面的示例代码使用了第一张方式，而第二种方式可以结合备忘录模式使用。

定义 Operation，对应类图中的 Receiver：

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
}
```

定义 Command 接口:

```java
public interface Command {
    void execute(int num);

    // 撤销
    void undo();

    // 恢复
    void redo();
}
```

定义 Command 的实现：

```java
public class AddCommand implements Command {
    private Operation operation;
    private int num;

    public AddCommand(Operation operation) {
        this.operation = operation;
    }

    @Override
    public void execute(int num) {
        this.num = num;
        operation.add(num);
    }

    @Override
    public void undo() {
        // 反操作
        operation.subtract(num);
    }

    @Override
    public void redo() {
        // 反操作
        operation.add(num);
    }
}
```

```java
public class SubtractCommand implements Command {
    private Operation operation;
    private int num;

    public SubtractCommand(Operation operation) {
        this.operation = operation;
    }

    @Override
    public void execute(int num) {
        this.num = num;
        operation.subtract(num);
    }

    @Override
    public void undo() {
        // 反操作
        this.operation.add(num);
    }

    @Override
    public void redo() {
        // 反操作
        this.operation.subtract(num);
    }
}
```

定义 Calculator，对应类图中的 Invoker：

```java
public class Calculator {
    // 利用两个队列实现撤销和恢复的功能
    private Deque<Command> undoCommands = new LinkedList<>();
    private Deque<Command> redoCommands = new LinkedList<>();

    public void calculate(Command command, int num) {
        command.execute(num);
        undoCommands.offerLast(command);
    }

    public void undo() {
        if (!undoCommands.isEmpty()) {
            Command previousCmd = undoCommands.pollLast();
            redoCommands.offerLast(previousCmd);
            previousCmd.undo();
        }
    }

    public void redo() {
        if (!redoCommands.isEmpty()) {
            Command previousCmd = redoCommands.pollLast();
            undoCommands.offerLast(previousCmd);
            previousCmd.redo();
        }
    }
}
```

定义 Client：

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

命令模式的本质是封装请求。将请求封装成命令对象，然后就可以对这个命令对象进行一系列处理，实现客户端的参数化配置、撤销/恢复操作、宏命令、队列请求、日志请求等功能处理。

命令模式优点：

* 命令的发起者和命令的执行者完全解耦
* 命令模式将请求封装起来后，可以动态的将其参数化、队列化和日志化，从而使系统更具灵活性
* 命令对象可以组合成复合对象，实现宏命令
* 可以方便的扩展命令对象

命令模式缺点：

* 容易造成类膨胀，有大量的衍生类被创建

# Ref

* [Command Pattern](http://www.oodesign.com/command-pattern.html)
* [java-design-patterns-command](https://github.com/iluwatar/java-design-patterns/blob/master/command/README.md)