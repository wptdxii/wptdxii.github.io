---
title: Android-Platforms-Migrating
date: 2018-04-03 16:49:50
tags: Migrating
categories: Android
---

Android 新版本适配

<!-- more -->

# Android P

# Android O

[Android Oreo](https://developer.android.com/about/versions/oreo/index.html) 需要适配的新特性

## Adaptive Icons

Android 8.0 (API level 26) 提出了 Adaptive Icons 概念，在不同的设备上可以显示不同形状的启动图标，使用该特性时需要将 targetSdk 指定为 26 及其之后。使用 Adaptive Icons 时需要提供两个图层(支持多种格式)：

* 前景层
* 背景层

不同的 OEM 厂商只需要指定 Mask 即可定制自己的 ROM 启动器风格。

Ref:

* [Adaptive Icons](https://developer.android.com/guide/practices/ui_guidelines/icon_design_adaptive.html)
* [Create App Icons with Image Asset Studio](https://developer.android.com/studio/write/image-asset-studio.html#delete)
* [Android应用图标微技巧，8.0系统中应用图标的适配](https://blog.csdn.net/guolin_blog/article/details/79417483)

# Android N

Android 7.1 (API level 25) 可以指定圆形的启动器图标，当 targetSdk 指定为 25 及其之后，运行该版本的设备会根据配置选择 android:icon 或者 android:roundIcon，同时也为了兼容之前的老版本，所有应该同时指定这两个属性，如下：

```xml
<application
    android:name="ApplicationTitle"
    android:label="@string/app_label"
    android:icon="@mipmap/ic_launcher"
    android:roundIcon="@mipmap/ic_launcher_round"
    ...>
```

> 因为在 Android 8.0 (API level 26) 提出了 Adaptive Icons 的概念，所以 android:roundIcon 主要对 Android 7.1 生效

Ref:

* [Round Icon Resources](https://developer.android.com/about/versions/nougat/android-7.1.html#circular-icons)