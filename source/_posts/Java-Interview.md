---
title: Java-Interview
date: 2017-03-10 16:16:19
tags: Interview
categories: Java
---

<!-- Java 常见面试题收集 -->

# Java中 ==、equals()、hashCode() 的区别

* == 用于比较基本数据类型和引用数据类型的引用(即对象地址)
* Object 的 equals() 默认比较的是对象地址，自定义对象根据需求需要重写 equals() 和 hashCode()，实现逻辑相等
* 如果两个对象 equals() 返回 true，则要求 两个对象的 hashCode() 也相等；反之，如果返回 false，则 hashCode() 可以相等也可以不等

# Java 类的生命周期

* [详解java类的生命周期](https://blog.csdn.net/zhengzhb/article/details/7517213)

# Java 类加载机制

* [类加载机制](https://blog.csdn.net/ns_code/article/details/17881581)