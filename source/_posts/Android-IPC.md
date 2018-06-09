---
title: Android-IPC
date: 2018-06-04 15:16:03
tags: [IPC, Binder]
categories: Android
---

IPC(Inner-process Communitatoinl) 含义是进程间通信，指的是两个进程间数据交换的过程。

<!-- more -->

# 进程和线程

进程(Process)和线程(Thread)都是 CPU 一个时间段的描述，但是粒度不一样。

进程是系统资源分配的基本单元，有自己的地址空间。进程有三种状态：就绪、运行和阻塞。而线程是 CPU 调度的基本单元。线程可以利用进程所拥有的资源，由于线程比进程更小，基本上不拥有系统资源，故对它的调度所付出的开销就会小得多，能更高效的提高系统多个程序间并发执行的程度。多线程是为了同步完成多项任务，不是为了提高运行效率，而是为了提高资源使用效率来提高系统的效率。线程是在同一时间需要完成多项任务的时候实现的。

具体可以参看：[进程、线程、多线程相关总结](https://www.cnblogs.com/fuchongjundream/p/3829508.html)

Android 中只有在 UI 线程才能操作页面元素。但也有极端情况，详情参看 [Android中子线程真的不能更新UI吗？](https://blog.csdn.net/xyh269/article/details/52728861)

# Android 多进程模式

# Binder

Android 中最有特色的 IPC 方式就是 Binder。

# Interview

* 进程和线程的区别
* 如何开启多进程?应用是否可以开启N个进程？