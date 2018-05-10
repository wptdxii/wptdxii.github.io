---
title: Android Developer Guide
date: 2016-03-27 22:27:39
tags: Guide
categories: Android
---

Android 开发指南

<!-- more -->

# Application Fundamentals

* Android 操作系统属于多用户的 Linux 操作系统，每个 app 是不同的用户
* 默认情况下，系统会分配给每个 app 一个唯一的 Linux User ID，该 ID 只供系统使用，对 app 是透明的。系统会给 app 内所有的文件添加权限，只有系统给访问这些文件的 app 的 分配对应的User ID 才能正常访问
* 每个进程有单独的虚拟机(VM)
* 默认情况下，每个 app 运行在独自的 Linux 进程
* 为了在不同的 app 直接共享数据和文件，可以给它们分配相同的 Linux User ID，具有相同 Linux User ID 的 app 可以运行在同一个进程，共享同一个虚拟机(VM)，前提是 app 需要有相同的签名
* 隐式 Intent 启动其它 app 时，被启动的 app 属于其自身进程

Android 四大组件：

* Activity
* Service
* BroadcastReceiver
* ContentProvider

# Ref

* [Developer Guides](https://developer.android.com/guide/index.html)
