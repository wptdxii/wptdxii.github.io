---
title: Android-Material-Design-Tutorial
date: 2017-08-21 16:47:06
tags: Android UI Widget
categories: Android
---

<!-- more -->

Material Design 是 Google 推出的一套视觉设计语言，主要面向 UI 设计人员，详细资料可以参看：

* [Material Design](https://material.io/)
* [Material Design 中文版](http://design.1sters.com/)

为了方便实现 Materrial Design 规范的设计效果，Google 同时针对 Android 开发提供了 Design Support 库，将一些代表性的控件和效果进行了封装，可以很方便的将应用 Material 化。

# Toolbar

Actionbar 由于其设计的原因，被限定只能位于 Activity 的顶部，不能自定义布局，而且不能实现 Material Design 的效果，所以不再推荐使用，官方推出了灵活性更高的 Toolbar 作为替代。
Toolbar 有两种使用方式：

* 应用栏(Action Bar)
* 独立控件(Standalone Widget)

Toolbar 既可以作为 app 的应用栏使用，也可以作为一个独立的普通控件使用。当作为应用栏使用时，需要通过 setSupportActionBar() 将 Toolbar 设置为应用栏；当作为一个独立的控件使用时，
用法同普通的 ViewGroup 用法一样。从功能上讲，Toolbar 继承并扩展了 ActionBar 的所有功能，所以 Toolbar 可以作为独立的普通控件完全代替 ActionBar，而不必将其设置为应用栏，但是对于使用了 ActionBar 的老项目，为了复用其代码，减少迁移成本，将 Toolbar 设置为应用栏最简便。所以对于有历史包袱的老项目，应该将 Toolbar 设置为系统应用栏使用；对于新项目，可以将 Toolbar 作为普通控件使用。

可以参看：

* [Goodbye ActionBar APIs, hello Toolbar](https://medium.com/@ZakTaccardi/goodbye-actionbar-apis-hello-toolbar-af6ae7b31e5d)
* [Have you been calling setSupportActionBar()? You don't have to!](https://www.reddit.com/r/androiddev/comments/3m3pd0/have_you_been_calling_setsupportactionbartoolbar/?st=j6yhe4s9&sh=6528a88d)

## 引入 Toolbar

为了兼容性，需要使用 support 库中的 Toolbar，添加依赖：

```gradle
compile 'com.android.support:appcompat-v7:$supportVersion'
```

设置隐藏 ActionBar 的主题，根据应用风格，可以选择深色主题和浅色主题：

* Theme.AppCompat.NoActionBar
* Theme.AppCompat.Light.NoActionBar

这里选择浅色主题：

```xml
res/values/themes.xml:

<resources>
  <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar"">
    <item name="android:colorPrimary">@color/primary</item>
    <item name="android:colorPrimaryDark">@color/primary_dark</item>
    <item name="android:colorAccent">@color/accent</item>
  </style>
</resources>
```

在布局文件中引入 support V7 包下的 Toolbar：

```xml
res/layout/toolbar.xml:

<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?attr/colorPrimary"
    android:minHeight="?attr/actionBarSize">
</android.support.v7.widget.Toolbar>
```

> 主题配置文件中的 android:colorPrimary 指定的是 ActionBar 的背景颜色，若要应用到 Toolbar，需要为 Toolbar 指定 android:background 属性。

然后就可以在 Activity 中使用 Toolbar：

```java
public class MainActivity extends AppCompatActivity {
       @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        if (toolbar != null) {
            setSupportActionBar(toolbar);
        }
    }
    // ...
}
```

> * Activity 需要继承 AppCompatActivity
> * 调用 setSupportActionBar(), 可以将 Toolbar 设置为应用栏，否则作为独立控件使用。

## 使用 Toolbar



## 配置 Toolbar

其中各个属性如下图所示：

![ThemeColor](http://otg3f8t90.bkt.clouddn.com/2017/8/29/ThemeColors.png)


Activity 需要继承 AppCompatActivity：


这样就实现了用 Toolbar 替换 ActionBar

## 溢出菜单(Overflow Menu)

### 定制溢出菜单按钮(Overflow Menu Button)

### 显示溢出菜单 Icon

溢出菜单(Overflow Menu) 的条目图标默认是不显示的，需要通过 MenuBuilder 设置，当 Tool 替换 ActionBar 使用时：

```java
  @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // 显示 Menu Item Icon
        if (menu instanceof MenuBuilder) {
            ((MenuBuilder) menu).setOptionalIconsVisible(true);
        }
        getMenuInflater().inflate(R.menu.activity_main, menu);
        return true;
    }
```

当 Toolbar 作为独立控件使用时：

```java
public class MainActivity extends AppCompatActivity {
       @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        if (toolbar != null) {
            setSupportActionBar(toolbar);
            // 显示 Menu Item Icon
            Menu menu = toolbar.getMenu();
            if (menu instanceof MenuBuilder) {
                ((MenuBuilder) menu).setOptionalIconsVisible(true);
              }
            toolbar.inflateMenu(R.menu.activity_main);
        }
    }
    // ...
}
```

## 配置 Toolbar


## Action Bar

## Standalone

## Ref

* [Setting Up the App Bar](https://developer.android.com/training/appbar/setting-up.html)
* [Adding and Handling Actions](https://developer.android.com/training/appbar/actions.html#handle-actions)
* [AppCompat v21 — Material Design for Pre-Lollipop Devices!](https://android-developers.googleblog.com/2014/10/appcompat-v21-material-design-for-pre.html)