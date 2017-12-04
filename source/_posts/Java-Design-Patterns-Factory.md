---
title: 工厂模式(Factory Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

工厂模式(Factory Pattern)又叫做简单工厂模式或者静态工厂模式，是创建型模式(Creational Pattern)，可以看做是工厂方法模式(Factory Method Patter)的弱化版本。

<!-- more -->

# 场景

面向接口编程是面向对象编程的一个重要原则。接口的思想是“封装隔离”，“封装”指的是对被隔离体的行为和职责进行封装，“隔离”指的是对外部调用和内部实现进行隔离。只要接口不变，内部实现的变化就不会影响到外部调用，从而使系统具有更好的扩展性和可维护性。接口保证了系统的可插拔性。但在面向接口编程时会出现外部调用只知道接口而不知道具体实现的问题，工厂模式就是为了解决此类问题。

> 这里的接口在 Java 中通常指的是 接口(Interface)和抽象类(Abstract Class)

# 意图

* 在不暴露内部实现的情况下为外部调用创建接口实例
* 为外部调用提供创建接口实例的统一接口

# UML 类图

# 实现

# 总结

# Ref