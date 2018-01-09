---
title: 状态模式(State Pattern)
date: 2017-11-29 15:06:00
tags: 行为型模式(Behavioral Pattern) 
categories: Java Design Patterns
---

状态模式属于行为型模式

<!-- more -->

# 意图

* 允许一个对象在其内部状态改变时改变其行为。这个对象看起来就好像修改了它的类

# 场景

当遇到下面场景时可以考虑使用状态模式：

* 当一个对象的行为取决于它的状态，并且它必须可以在运行时根据状态来改变它的行为时
* 当代码中包含大量与对象状态有关的条件语句时

# UML 类图

状态模式类图如下：

![Java-Design-Pattern-State.png](http://otg3f8t90.bkt.clouddn.com/2018/1/9/Java-Design-Pattern-State.png)

类图说明：

* State：状态接口，定义上下文状态对应的行为
* ConcreteState：具体的状态实现类，每个类实现一个与上下文状态对应的具体行为，需要的外部数据可以通过上下文环境传递
* Context：上下文环境，用于维护当前状态，定义客户端感兴趣的接口，同时也可以用来封装状态实现类需要的数据

# 实现

定义 User，用于封装数据：

```java
public class User {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

定义 UserState，对应类图中的 State：

```java
public interface UserState {
    // UserManager 是上下文环境类，可以用来封装数据，传递给状态类
    void comment(UserManager context);
}
```

定义 LoginState、LogoutState，对应类图中的 ConcreteState：

```java
public class LoginState implements UserState {

    @Override
    public void comment(UserManager context) {
        System.out.println(context.getUser().getName() + " comment the message!");
    }
}
```

```java
public class LogoutState implements UserState{

    @Override
    public void comment(UserManager context) {
        System.out.println("Unable to comment, please login first!");
    }
}
```

定义 UserManager，对应类图中的 Context：

```java
public class UserManager {
    private UserState state;
    private User user;

    public UserManager() {
        // 默认是未登陆状态
        this.state = new LogoutState();
    }


    // 可以在运行时改变状态
    public void setState(UserState state) {
        this.state = state;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public void comment() {
        this.state.comment(this);
    }

    // 改变状态，可以放到客户端
    public void login() {
        System.out.println("Login...");

        setState(new LoginState());
    }

    // 改变状态，可以放到客户端
    public void logout() {
        System.out.println("Logout...");

        setState(new LogoutState());
    }
}
```

Context 可以根据具体业务实现为单例模式，如上面的例子中，维护用户全局的登陆状态就可以实现为单例

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        User user = new User();
        user.setName("Someone");

        UserManager context = new UserManager();
        context.setUser(user);
        context.comment();

        context.login();
        context.comment();

        context.logout();
        context.comment();
    }
}
```

# 总结

# Ref