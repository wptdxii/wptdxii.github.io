---
title: 解释器模式(Interpreter Pattern)
date: 2017-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

解释器模式属于行为型模式

<!-- more -->

# 意图

* 给定一个语言，定义它的文法表示，并定义一个解释器，该解释器使用该文法表示来解释语言中的语句

> 文法通俗点讲就是语法规则

# 场景

当遇到下面场景可以考虑使用解释器模式：

* 当一个语言需要解释执行，并且可以将语言中的句子表现为抽象语法树的时候。但这要求语言的文法足够简单，当文法比较的复杂时，比不适合使用解释器模式
* 在某些特定领域出现不断重复的问题时，可以将该领域的问题转化为一种有着某种文法并且可以抽象出语法树的语句，然后构建解释器来解释该语句

> 抽象语法树指的是包含终结符元素和非终结符元素的树状结构，其中终结符元素指的是不能推导或包含其它元素的元素，非终结符元素指的是可以推导或包含其它元素的元素

# UML 类图

解释器模式类图如下：

![Java-Design-Patters-Interpreter.png](http://otg3f8t90.bkt.clouddn.com/2017/12/20/Java-Design-Patters-Interpreter.png)

类图说明：

* Context： 上下文，通常包含各个解释器需要的全局信息或公共的功能
* Expression：解释器接口，约定解释器的解释操作，用于解释抽象语法树，并执行每个节点对应的功能
* TerminalExpression：终结符解释器，实现文法中与终结符有关的解释操作
* NonterminalExpression：非终结符解释器，实现文法中与非终结符有关的解释操作
* Client：客户端，把客户端调用要求的表达式，经过解析，形成抽象语法树

# 实现

Context 根据具体的使用场景定义，因为下面的示例比较简单用不到 Context，就不再定义

定义 ArithmeticExpression 接口，对应类图中的 Expression:

```java
public interface ArithmeticExpression {
    int interpret();
}
```

定义 NumberExpression，对应类图中的 TerminalExpression：

```java
public class NumberExpression implements ArithmeticExpression {
    private int number;

    public NumberExpression(int number) {
        this.number = number;
    }

    public NumberExpression(String number) {
        this.number = Integer.parseInt(number);
    }

    @Override
    public int interpret() {
        return number;
    }
}
```

定义抽象的 OperatorExpression，对应类图中的 NonterminalExpression：

```java
public abstract class OperatorExpression implements ArithmeticExpression {
    protected ArithmeticExpression leftExpression;
    protected ArithmeticExpression rightExpression;

    public OperatorExpression(ArithmeticExpression leftExpression, ArithmeticExpression rightExpression) {
        this.leftExpression = leftExpression;
        this.rightExpression = rightExpression;
    }

    @Override
    public abstract int interpret();
}
```

实现 OperatorExpression：

```java
public class PlusExpression extends OperatorExpression {

    public PlusExpression(ArithmeticExpression leftExpression, ArithmeticExpression rightExpression) {
        super(leftExpression, rightExpression);
    }

    @Override
    public int interpret() {
        return leftExpression.interpret() + rightExpression.interpret();
    }
}
```

```java
public class MinusExpression extends OperatorExpression {

    public MinusExpression(ArithmeticExpression leftExpression, ArithmeticExpression rightExpression) {
        super(leftExpression, rightExpression);
    }

    @Override
    public int interpret() {
        return leftExpression.interpret() - rightExpression.interpret();
    }
}
```

定义解析器，用以生成抽象语法树，是对类图中 Client 部分功能的封装：

```java
public class Calculator {
    private Stack<ArithmeticExpression> expressionStack = new Stack<>();

    public Calculator(String expression) {
        parseExpression(expression);
    }

    private void parseExpression(String expression) {
        String[] tokenList = expression.split(" ");
        for (int i = 0; i < tokenList.length; i++) {
            String s = tokenList[i];
            if (isOperator(s)) {
                ArithmeticExpression leftExpression = expressionStack.pop();
                ArithmeticExpression rightExpression = new NumberExpression(tokenList[++i]);
                ArithmeticExpression operatorExpression = getOperatorExpression(s, leftExpression, rightExpression);
                expressionStack.push(new NumberExpression(operatorExpression.interpret()));

            } else {
                expressionStack.push(new NumberExpression(s));
            }

        }
    }

    private ArithmeticExpression getOperatorExpression(String operator, ArithmeticExpression leftExpression,
                                                       ArithmeticExpression rightExpression) {

        switch (operator) {
            case "+":
                return new PlusExpression(leftExpression, rightExpression);
            case "-":
                return new MinusExpression(leftExpression, rightExpression);
            default:
                return null;
        }
    }

    private boolean isOperator(String s) {
        return "+".equals(s) || "-".equals(s);
    }

    public int calculate() {
        return expressionStack.pop().interpret();
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        // 由于是简单示例，约定表达式运算符号只能包含 "+"、"-"，且数字符号之间以空格隔开
        String expression = "1 + 2 + 5 - 4 + 6";
        Calculator calculator = new Calculator(expression);
        System.out.println(calculator.calculate()); // 10
    }
}
```

# 总结

解释器模式的本质是分离实现，解释执行。

解释器模式优点：

* 易于实现语法，因为一个解释器只解释一个语法规则
* 易于扩展新的语法，扩展新语法时扩展相应的解释器即可

解释器模式缺点：

* 不适合复杂的语法
* 每一条文法至少对应一个解释器，容易造成大量的衍生类
* 效率不高

# Ref

* [Interpreter](http://www.oodesign.com/interpreter-pattern.html)
* [java-design-patterns-interpreter](https://github.com/iluwatar/java-design-patterns/blob/master/interpreter/README.md)