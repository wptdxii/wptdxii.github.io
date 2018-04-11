---
title: Android Studio User Guide
date: 2017-04-02 22:37:16
tags: Guide
categories: Android Studio
---

Android Studio 用户指南

<!-- more -->

# Configure Your Build

## Set the Application ID

Android app 在开发过程中有三个容易误解的概念:

* Java 项目结构(Project Structure)对应的包名
* 清单文件根标签中的 package 属性
* 每个 Module 下的 build.gradle 文件中的 applicationId 属性

其中前两个标识值相互对应，需要保持一致；而最后一个属性值是 app 的唯一标识，一旦发布到应用市场就不能再更改

详情见：

* [Set the Application ID](https://developer.android.com/studio/build/application-id.html)
* [Android Package Name Vs Application ID](https://blog.mindorks.com/android-package-name-vs-application-id-ad95b08815a6)
