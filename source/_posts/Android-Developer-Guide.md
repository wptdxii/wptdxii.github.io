---
title: Android Developer Guide
date: 2016-03-27 22:27:39
tags: Guide
categories: Android
---

Android 官方开发指南摘要

<!-- more -->

# App basics

## Application Fundamentals

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

# Core topics

## App data&files

Android 数据存储方式：

### Internal file storage

Internal file storage 有如下特点：

* 存储空间始终可用
* 存储的数据 app 私有
* app 卸载时该存储空间的数据会被清空
* 该存储空间主要用来存储用户和其他 app 都无需访问的数据
* 操作 Internal file storage 的 api 不需要申请权限
* Internal file storeage 还有一个 cache files 路径，该路径有以下特点
  * 当系统存储空间不够时，该路径下的缓存文件可能被系统删除
  * 该缓存文件应该由开发者维护，设定最大缓存空间，不应该依赖系统做清理，可采用 LRU 算法

#### Write a file

```java
File file = new File(context.getFilesDir(), filename);
```

* External file storage
  * 存储空间不一定可用，如挂载到电脑上时，所以访问之前要先判断存储是否可用
  * 可以从外部读取
  * app 卸载时 getExternalFilesDir() 路径下的数据会被清除，其他路径下的数据不会被清除
  * 该存储空间主要用来存储非隐私的数据
* Shared preferences
  * 以键值对的方式存储基本数据类型和字符串
  * 底层以 XML 格式保存
  * 
* Database

# Ref

* [Developer Guides](https://developer.android.com/guide/index.html)
