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

* Memento：备忘录接口，该接口是一个窄接口，起到标识的作用
* ConcreteMemento：
* Originator：原发器
* Client：客户端

# 实现

# 总结

# Ref