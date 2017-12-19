---
title: 责任链模式(Chain of Responsibility Pattern)
date: 2017-11-29 15:06:00
tags: 行为型模式(Behavioral Pattern) 
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

![Java-Design-Patterns-Chain-Of-Responsibility.png](http://otg3f8t90.bkt.clouddn.com/2017/12/18/Java-Design-Patterns-Chain-Of-Responsibility.png)

类图说明：

* RequestType：表示请求类型，可以定义为枚举类型
* Request：请求的接口类,通常会定义请求的类型和内容供子类实现，可根据具体业务定义
* ConcreteRequest：请求的实现类
* Handler：处理请求的接口类，在其中实现后继链，持有下一个处理节点对象的引用
* ConcreteHandler:处理请求的实现类，在这个类中实现对职责范围内请求的处理，如果不处理，就继续转发请求至后继的处理节点
* Client：责任链的客户端，组装责任链并向链上提交请求

# 实现

在标准的责任链模式中，只要有对象处理了请求，这个请求就到此为止，不再被传递和处理，当然也可以链上所有对象都不处理该请求，这样的责任链叫做标准链；对标准链进行变形，当请求在责任链中传递时，每个处理对象都对请求进行处理，处理完成后请求并不停止而是继续向下一个处理节点传递，直到被处理完，这样的责任链叫做功能链，通常用于实现过滤的功能

定义　RequestType：

```java
public enum RequestType {
    CIRCLE, RECTANGLE, TRIANGLE, CHECK
}
```

定义　Request：

```java
public abstract class Request {
    // 该标识主要在功能链中使用
    private boolean handled;

    public void markHandled() {
        handled = true;
    }

    public boolean isHandled() {
        return handled;
    }

    public abstract RequestType getRequestType();

    public abstract String getRequestDescription();
}
```

## 标准链

定义　Request　实现，对应类图中的　ConcreteRequest：

```java
public class CircleRequest extends Request {
    @Override
    public RequestType getRequestType() {
        return RequestType.CIRCLE;
    }

    @Override
    public String getRequestDescription() {
        return "Draw circle";
    }
}
```

```java
public class RectangleRequest extends Request {
    @Override
    public RequestType getRequestType() {
        return RequestType.RECTANGLE;
    }

    @Override
    public String getRequestDescription() {
        return "Draw rectangle";
    }
}
```

```java
public class TriangleRequest extends Request {
    @Override
    public RequestType getRequestType() {
        return RequestType.TRIANGLE;
    }

    @Override
    public String getRequestDescription() {
        return "Draw triangle";
    }
}
```

定义 Handler：

```java
public abstract class Handler {
    private Handler successor;

    public Handler(Handler successor) {
        this.successor = successor;
    }

    // 该方法要求子类实现对请求的处理逻辑，而不是被客户端调用来处理请求
    protected abstract boolean propagateRequest(Request request);

    // 标准链的实现，声明该方法是为了防止在未能处理请求的情况下忘记将请求传递给下一个节点，可以端需要调用这个方法处理请求
    public final boolean handleRequest(Request request) {
        return propagateRequest(request) || (successor != null && successor.handleRequest(request));
    }
}
```

定义　Handler 实现，对应类图中的　ConcreteHandler：

```java
public class CircleHandler extends Handler {

    public CircleHandler(Handler successor) {
        super(successor);
    }

    @Override
    protected boolean propagateRequest(Request request) {
        if (RequestType.CIRCLE.equals(request.getRequestType())) {
            System.out.println("CircleHandler:" + request.getRequestDescription());
            return true;
        }
        return false;
    }
}
```

```java
public class RectangleHandler extends Handler {

    public RectangleHandler(Handler successor) {
        super(successor);
    }

    @Override
    protected boolean propagateRequest(Request request) {
        if (RequestType.RECTANGLE.equals(request.getRequestType())) {
            System.out.println("RectangleHandler:" + request.getRequestDescription());
            return true;
        }
        return false;
    }
}
```

```java
public class TriangleHandler extends Handler {

    public TriangleHandler(Handler successor) {
        super(successor);
    }

    @Override
    protected boolean propagateRequest(Request request) {
        if (RequestType.TRIANGLE.equals(request.getRequestType())) {
            System.out.println("TriangleHandler:" + request.getRequestDescription());
            return true;
        }
        return false;
    }
}
```

定义　Drawer，用来对组装责任链进行封装，其属于　Client 的一部分：

```java
public class Drawer {
    private Handler chain;

    public Drawer() {
        buildChain();
    }

    private void buildChain() {
        chain = new CircleHandler(new RectangleHandler(new TriangleHandler(null)));
    }

    public void makeRequest(Request request) {
        if (!chain.handleRequest(request)) {
            System.out.println("Can't handle the request");
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        // 组装责任链
        Drawer drawer = new Drawer();
        // 定义请求
        Request circleRequest = new CircleRequest();
        Request rectangleRequest = new RectangleRequest();
        Request triangleRequest = new TriangleRequest();
        // 发送请求，每个请求都是从责任链的首端开始发送
        drawer.makeRequest(circleRequest);
        drawer.makeRequest(rectangleRequest);
        drawer.makeRequest(triangleRequest);
    }
}
```

以上是标准链的示例实现

## 功能链

功能链的　Handler 定义与标准链不同：

```java
public abstract class Handler {
    private Handler successor;

    public Handler(Handler successor) {
        this.successor = successor;
    }

    protected abstract boolean propagateRequest(Request request);

    // 功能链实现，与标准链的区别主要在这里，这里实现了过滤的功能，当满足过滤条件时，请求继续沿链传递，但不满足过滤条件时，请求终止传递
    public boolean handleRequest(Request request) {
        return propagateRequest(request) && (successor == null || successor.handleRequest(request));
    }
}
```

定义　Handler 实现：

```java
public class TypeHandler extends Handler {

    public TypeHandler(Handler successor) {
        super(successor);
    }

    @Override
    protected boolean propagateRequest(Request request) {
        if (request.getRequestType() != null) {
            System.out.println("Request type checked");
            return true;
        }
        return false;
    }
}
```

```java
public class DescriptionHandler extends Handler {

    public DescriptionHandler(Handler successor) {
        super(successor);
    }

    @Override
    protected boolean propagateRequest(Request request) {
        String description = request.getRequestDescription();
        if (description != null && !description.isEmpty()) {
            System.out.println("Request description checked");
            return true;
        }
        return false;
    }
}
```

> 因上面　TypeHandler 和　DescriptionHandler　的实现比较简单，并没有如类图所示的那样关联　RequestType

```java
public class CheckerHandler extends Handler {

    public CheckerHandler(Handler successor) {
        super(successor);
    }

    @Override
    protected boolean propagateRequest(Request request) {
        if (request.getRequestType().equals(RequestType.CHECK)) {
            System.out.println(request.getRequestDescription());
            request.markHandled();
            return true;
        }
        return false;
    }
}
```

定义　Checker，用来对组装责任链进行封装：

```java
public class Checker {
    private Handler chain;

    public Checker() {
        buildChain();
    }

    private void buildChain() {
        chain = new TypeHandler(new DescriptionHandler(new CheckerHandler(null)));
    }

    public void makeRequest(Request request) {
        if (chain.handleRequest(request)) {
            if (!request.isHandled()) {
                System.out.println("Can't handle the request");
            }
        } else {
            System.out.println("Failed to pass the check");
        }
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        Checker checker = new Checker();
        Request checkerRequest = new CheckerRequest();
        checker.makeRequest(checkerRequest);
    }
}
```

以上是功能链的示例实现

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