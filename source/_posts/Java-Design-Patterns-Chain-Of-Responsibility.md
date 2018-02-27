---
title: 责任链模式(Chain of Responsibility Pattern)
date: 2016-11-29 15:06:00
tags: [Behavioral Pattern, GoF]
categories: Java Design Patterns
---

责任链模式也叫职责链模式，属于行为型模式

<!-- more -->

# 意图

* 使多个对象都有机会处理处理请求，从而避免请求的发送者和接收者之间的耦合
* 使这些对象连成一条链，并沿着这条链传递请求，直到有对象处理它为止

# 场景

当遇到下面场景可以考虑使用责任链模式：

* 多个对象可以处理同一请求，但具体由哪个对象处理是未知的，需要在运行时决定时
* 在不明确指定请求处理者，需要向多个处理对象中的一个提交请求时
* 需要动态指定可以处理一个请求的对象合集时

# UML 类图

责任链模式类图如下：

![Java-Design-Patterns-Chain.png](http://otg3f8t90.bkt.clouddn.com/2017/12/27/Java-Design-Patterns-Chain.png)

类图说明：

* RequestType：标识请求类型，可以定义为枚举类型
* Request：请求的封装类，通常会定义请求的类型和内容供子类实现，可根据具体业务定义，组合了 RequestType
* Handler：处理请求的接口类，在其中实现后继链，持有下一个处理节点对象的引用
* ConcreteHandler:处理请求的实现类，在这个类中实现对职责范围内请求的处理，如果不处理，就继续转发请求至后继的处理节点
* Client：责任链的客户端，组装责任链并向链上提交请求

# 实现

在标准的责任链模式中，只要有对象处理了请求，这个请求就到此为止，不再被传递和处理，当然也可以链上所有对象都不处理该请求，这样的责任链叫做标准链；对标准链进行变形，当请求在责任链中传递时，每个处理对象都对请求进行处理，处理完成后请求并不停止而是继续向下一个处理节点传递，直到被处理完，这样的责任链叫做功能链，通常用于实现过滤的功能

定义　RequestType：

```java
public enum RequestType {
    DEFEND_CASTLE, TORTURE_PRISONER, COLLECT_TAX, DEFAULT
}
```

定义　Request：

```java
public final class Request {
    private boolean handled; // 标识请求是否被处理
    private boolean checked; // 标识请求是否通过校验
    private RequestType requestType;
    private String requestDescription;

    public Request(RequestType requestType, String requestDescription) {
        this.requestType = requestType;
        this.requestDescription = requestDescription;
    }

    public void markHandled() {
        handled = true;
    }

    public void setChecked(boolean checked) {
        this.checked = checked;
    }

    public boolean isHandled() {
        return handled;
    }

    public boolean isChecked() {
        return checked;
    }

    public RequestType getRequestType() {
        return requestType;
    }

    public String getRequestDescription() {
        return requestDescription;
    }
}
```

定义 Handler：

```java
public interface Handler {

    void setSuccessor(Handler successor);

    // 子类实现请求的处理逻辑
    boolean propagateRequest(Request request);

    // 处理请求的分发，由抽象子类实现为标准链或功能链
    boolean handleRequest(Request request);

    void printRequest(Request request);
}
```

定义　Handler 的抽象实现：

```java
// 标准链抽象
public abstract class RequestHandler implements Handler {
    private Handler successor;

    @Override
    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }

    @Override
    public abstract boolean propagateRequest(Request request);

    // 标准链实现，请求被处理后不再向后面的节点传递
    @Override
    public final boolean handleRequest(Request request) {
        return propagateRequest(request) || (successor != null && successor.handleRequest(request));
    }

    @Override
    public void printRequest(Request request) {
        System.out.println(this + " handle the request:" + request.getRequestDescription());
    }

    @Override
    public String toString() {
        return getClass().getSimpleName();
    }
}
```

```java
// 功能链抽象
public abstract class CheckerHandler implements Handler {
    private Handler successor;

    @Override
    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }

    @Override
    public abstract boolean propagateRequest(Request request);

    // 功能链实现，请求被处理后继续向后面的节点传递
    @Override
    public final boolean handleRequest(Request request) {
        request.setChecked(propagateRequest(request));
        return request.isChecked() && (successor == null || successor.handleRequest(request));
    }

    @Override
    public void printRequest(Request request) {
        System.out.println(this + " check the request.");
    }

    @Override
    public String toString() {
        return getClass().getSimpleName();
    }
}
```

上面的 RequesHandler 和 CheckerHandler 对应类图中的 Handler，不过是为了实现标准链和功能链，所以抽象出了公共接口 Handler

定义抽象类 RequestHandler 的实现:

```java
public final class Commander extends RequestHandler {

    @Override
    public boolean propagateRequest(Request request) {
        if (RequestType.DEFEND_CASTLE.equals(request.getRequestType())) {
            printRequest(request);
            request.markHandled();
            return true;
        }
        return false;
    }
}
```

```java
public final class Officer extends RequestHandler{

    @Override
    public boolean propagateRequest(Request request) {
        if (RequestType.TORTURE_PRISONER.equals(request.getRequestType())) {
            printRequest(request);
            request.markHandled();
            return true;
        }
        return false;
    }
}
```

```java
public final class Soldier extends RequestHandler {

    @Override
    public boolean propagateRequest(Request request) {
        if (RequestType.COLLECT_TAX.equals(request.getRequestType())) {
            printRequest(request);
            request.markHandled();
            return true;
        }
        return false;
    }
}
```

定义抽象类 CheckerHandler 的实现：

```java
public final class TypeChecker extends CheckerHandler {

    @Override
    public boolean propagateRequest(Request request) {
        if (request.getRequestType() != null) {
            printRequest(request);
            return true;
        }
        System.out.println("Please check the request type");
        return false;
    }
}
```

```java
public final class DescriptionChecker extends CheckerHandler {

    @Override
    public boolean propagateRequest(Request request) {
        String description = request.getRequestDescription();
        if (description != null && !description.trim().isEmpty()) {
            printRequest(request);
            return true;
        }
        System.out.println("Pleas check the request description");
        return false;
    }
}
```

上面 Commander、Officer、Soldier、TypeChecker、DescriptionChecker 对应类图中的 ConcreteHandler。

定义 King，用于组装责任链，在类图中未标出：

```java
public final class King {
    private Handler chain;

    public King() {
        chain = buildChain();
    }

    private Handler buildChain() {
        Handler typeChecker = new TypeChecker();
        Handler descriptionChecker = new DescriptionChecker();
        Handler commander = new Commander();
        Handler officer = new Officer();
        Handler soldier = new Soldier();

        // TypeChecker 和 DescriptionChecker 是功能链，用于校验请求，放在责任链的首端
        typeChecker.setSuccessor(descriptionChecker);
        descriptionChecker.setSuccessor(commander);
        commander.setSuccessor(officer);
        officer.setSuccessor(soldier);

        return typeChecker;
    }

    public void makeRequest(Request request) {
        if (!chain.handleRequest(request)) {
            if (request.isChecked() && !request.isHandled()) {
                System.out.println("Nobody handle the request");
            }
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        King king = new King();
        king.makeRequest(new Request(RequestType.COLLECT_TAX, "collect tax"));
        king.makeRequest(new Request(RequestType.TORTURE_PRISONER, "torture prisoner"));
        king.makeRequest(new Request(RequestType.DEFEND_CASTLE, "defend castle"));
        king.makeRequest(new Request(RequestType.DEFAULT, "nothing"));
        king.makeRequest(new Request(RequestType.DEFAULT,""));
    }
}
```

# 总结

责任链模式的本质是分离职责，动态组合。分离职责是前提，动态组合是精华

责任链模式优点：

* 请求发送者和处理者松散耦合，请求者并不知道最终的处理者，实现了两者之间的解耦
* 动态组合责任链，可以根据需求动态指定责任的处理顺序，提高代码的灵活性

责任链模式缺点：

* 责任链每个处理节点只负责处理一方面的功能，那么完成整个功能就需要组合多个职责对象，这样就会产生很多细粒度对象
* 请求在整个责任链传递完了，也不一定被处理，需要提供默认的处理，注意构建链的有效性
* 责任链模式需要对链中处理对象进行遍历，特别是在一些递归调用中，会影响性能

# Ref

* [Chain of Responsibility](http://www.oodesign.com/chain-of-responsibility-pattern.html)
* [java-design-patterns-chain](https://github.com/iluwatar/java-design-patterns/blob/master/chain/README.md)