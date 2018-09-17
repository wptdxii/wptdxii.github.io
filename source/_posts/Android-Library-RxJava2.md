---
title: Android-Library-RxJava2
date: 2018-07-03 14:49:54
tags: [Library, RxJava, RxJava2]
categories: Android
---

RxJava 使用总结

<!-- more -->

# 发送和接收事件

上游被观察者通过 Emitter 发送事件，下游观察者在订阅时会接收到上游发送的事件，发送和接收事件时需要注意以下几点:

- 上游可以发送无限个 onNext()，下游可以接收无限个 onNext()
- 上游发送一个 onComplete() 或 onError() 之后，仍然可以继续发送 onNext() 和 onCompete()，但下游在接收到第一个对应的 onComplete() 或 onError() 后不再接收其他事件(发送多个 onError() 会抛异常)
- 上游可以不发送 onComplete() 和 onError()
- 上游发送 onComplete() 和 onError() 是互斥的，只能发送一个，且应该只发送一次。如果同时调用两者，如果先调用 onComplete() 再调用 onError() 会抛异常；如果先调用 onError() 再调用 onComplete()，则不会报异常，但只有观察者的 onError() 会被回调，onComplete() 方法不被回调
- dispose() 之后上游仍然继续发送事件，但下游不能接收到事件

# 线程切换

# 操作符

## flatMap()

flatMap() 不保证发送事件的顺序，如果需要保证顺序应该使用 concatMap()

## doOnSubscribe()

# 背压

# 使用场景

## 嵌套网络请求

# Ref

- [RxJava2 学习资料推荐](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650823932&idx=1&sn=198b18f2f9359e2eee1ffc8703d31905&chksm=80b78862b7c001741916c681d070ca3c1a58eef5632ea394797029d0335f312816afecf87e7d&mpshare=1&scene=1&srcid=0703CyPIKTZBIERu921rmlJU#rd)
- [给初学者的RxJava2.0教程(一)](https://www.jianshu.com/p/464fa025229e)
- [RxJava2 实战知识梳理(1) - 后台执行耗时操作，实时通知 UI 更新](https://www.jianshu.com/p/c935d0860186)
- [RxJava（一） create操作符的用法和源码分析](https://blog.csdn.net/johnny901114/article/details/51524470)
- [这可能是最好的RxJava 2.x 教程（完结版）](https://www.jianshu.com/p/0cd258eecf60)
- [RxJava - Better crash logs or how to always know which of your observables crashed](https://rongi.github.io/kotlin-blog/rxjava/2017/09/25/breadcrumbs-rxjava-error-handling.html)
- [Retrofit+RxJava 优雅的处理服务器返回异常、错误](https://blog.csdn.net/jdsjlzx/article/details/51882661)
- [RxJava简洁封装之道](https://www.jianshu.com/p/f3f0eccbcd6f)
- [Retrofit+RxJava网络请求异常处理](https://www.jianshu.com/p/9c3f0af1180d)
- [Rxjava +Retrofit 你需要掌握的几个技巧，Retrofit缓存，统一对有无网络处理, 异常处理，返回结果问题](https://www.jianshu.com/p/b1979c25634f)
- [RxJava 沉思录（一）：你认为 RxJava 真的好用吗？](https://juejin.im/post/5b8f536c5188255c352d3528)