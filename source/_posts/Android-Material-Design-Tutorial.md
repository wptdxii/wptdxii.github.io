---
title: Android-Material-Design-Tutorial
date: 2017-08-21 16:47:06
tags: Android UI
categories: Android
---

<!-- more -->

Material Design 是 Google 推出的一套视觉设计语言，主要面向 UI 设计人员，详细资料可以参看：

* [Material Design](https://material.io/)
* [Material Design 中文版](http://design.1sters.com/)

为了方便实现 Materrial Design 规范的设计效果，Google 同时针对 Android 开发提供了 Design Support 库，将一些代表性的控件和效果进行了封装，可以很方便的将应用 Material 化。

# Toolbar

ActionBar 由于其设计的原因，被限定只能位于 Activity 的顶部，不能自定义布局，而且不能实现 Material Design 的效果，所以不再推荐使用，官方推出了灵活性更高的 Toolbar 作为替代。
Toolbar 有两种使用方式：

* 应用栏(Action Bar)
* 独立控件(Standalone Widget)

当作为应用栏使用时，需要通过setSupportActionBar() 将 Toolbar 设置为应用栏，此时可以使用 ActionBar 提供的一些诸如 ActionBar.show()/hide() 之类的 API，但以这种方式使用 Toolbar 时具有以下缺点：

* 屏幕上只能显示一个应用栏，如果在多个 Fragment 中都使用 Toolbar 作为应用栏，当几个 Fragment 需要并列同时显示，例如在对平板进行适配时，其各自的应用栏无法同时显示
* 在 Fragment 中创建溢出菜单时会遇到回调地狱，只用调用 setHasOptionsMenu(true) 后，onCreateOptionsMenu() 回调才会被触发

当作为独立的控件使用时，用法同普通的 ViewGroup 用法一样。从功能上讲，Toolbar 继承并扩展了 ActionBar 的所有功能，所以 Toolbar 可以作为独立的普通控件完全代替 ActionBar，而不必将其设置为应用栏，但是对于使用了 ActionBar 的老项目，为了复用系统回调的实现代码，减少迁移成本，将 Toolbar 设置为应用栏最简便。所以对于有历史包袱的老项目，可以将 Toolbar 设置为系统应用栏使用；对于新项目，建议将 Toolbar 作为普通控件使用。

## 引入 Toolbar

为了兼容性，需要使用 support 库中的 Toolbar，添加依赖：

```gradle
compile 'com.android.support:appcompat-v7:$supportVersion'
```

然后通过设置主题隐藏原生 ActionBar 类提供的应用栏，根据应用风格可以选择深色主题和浅色主题：

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

主题文件中各个属性对应的区域如下图所示：

![ThemeColor](http://otg3f8t90.bkt.clouddn.com/2017/8/29/ThemeColors.png)

接着将主题应用在 AndroidManifest.xml 中的 application 标签即可。

在布局文件中相应位置引入 Toolbar：

```xml
res/layout/toolbar.xml:

<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:elevation="4dp"
    android:background="?attr/colorPrimary"
    android:minHeight="?attr/actionBarSize">
</android.support.v7.widget.Toolbar>
```

> 主题配置文件中的 android:colorPrimary 指定的是 ActionBar 的背景颜色，需要将其应用到 android:background 属性才可正常显示

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
> * 调用 setSupportActionBar() 可以将 Toolbar 设置为应用栏，否则作为独立控件使用
> * Toolbar 设置为应用栏后，可以通过 getSupportActionBar() 获取 ActionBar 对象的引用，继而可以调用 ActionBar 之类的 API

## 配置 Toolbar

Toolbar 主要由以下几部分组成：

![toolbar_widget.jpg](http://otg3f8t90.bkt.clouddn.com/2017/10/31/toolbar_widget.jpg)

### 设置 Navigation Button

Toolbar 左上角提供了 Navigation Button,通常用于返回主页或者侧边抽屉触发，图标和点击触发的逻辑都是可以自定义的。当 Toolbar 作为应用栏时，
该 Button 又叫做 Up Button，默认用于返回 Parent Activity，为了实现该功能，首先要在清单文件中声明 Parent Activity：

```xml

<application ... >
    ...

    <activity
        android:name="com.example.MainActivity" ...>
        ...
    </activity>

    <activity
        android:name="com.example.MyChildActivity"
        android:label="@string/title_activity_child"
        android:parentActivityName="com.example.MainActivity" >

        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>

```

> * android:parentActivityName 属性支持 API 16 及其以上的版本
> * < meta-data > 标签为了兼容 API 16 以下的版本
> * 当未声明 Parent Activity 时点击无响应

将 Toolbar 设置为应用栏，然后调用 ActionBar 的方法即可：

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        ActionBar actionBar = getSupportActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);
        actionBar.setHomeAsUpIndicator(R.drawable.ic_near_me_white_24dp);
    }
```

> * 调用 ActionBar.setDisplayHomeAsUpEnabled(true) 启用 Up Button
> * Up button 的默认图标是一个返回箭头，可以通过 ActionBar.setHomeAsUpIndicator() 自定义。当使用默认图标时，图标颜色可由主题中的 colorAccent 设定

Up Button 默认触发的回调是返回父 Activity，但可以自定义：

```java
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
                Toast.makeText(this, "Up Button", Toast.LENGTH_SHORT).show();
                break;
            default:
                return super.onOptionsItemSelected(item);
        }
        return true;
    }
```

> * Up button 的 id 固定为：android.R.id.home
> * Up button 的触发逻辑可以根据需求自定义

当 Toolbar 作为独立控件使用时，不再接收系统回调，即 onOptionsItemSelected() 方法不会被触发，可以通过 Toolbar 的方法模拟出 Up Button 的效果：

```java
        Toolbar toolbar = findView(R.id.toolbar);
        toolbar.setNavigationIcon(R.drawable.ic_arrow_back_white_24dp);
        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent upIntent = NavUtils.getParentActivityIntent(MainActivity.this);
                if (upIntent == null) {
                    finish();
                    return;
                }
                NavUtils.navigateUpTo(MainActivity.this, upIntent);
            }
        });
```

在给 Toolbar 设置监听时需要注意：

> 如果 Toolbar 作为应用栏使用，Toolbar.setNavigationOnClickListener() 需要在 setSupportActionBar() 之后调用才有效，且会覆盖 onOptionsItemSelected() 方法的触发，否则触发的还是onOptionsItemSelected() 方法

### 设置 Logo/Title/Subtitle

Toolbar 可以通过以下方法设置 Logo/Title/Subtitle:

```java
        Toolbar.setLogo();
        Toolbar.setTitle();
        Toolbar.setSubtitle();
```

> 当 Toolbar 被设置为应用栏时，Title 会被默认设置为应用的名称，Toolbar.setTitle() 必须 在 setSupportActionBar() 之后调用才生效

### 设置 Action View

### 设置 Action Provider

### 设置 Overflow Menu

#### 定制溢出菜单按钮(Overflow Menu Button)

#### 显示溢出菜单 Icon

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

## Ref

* [Setting Up the App Bar](https://developer.android.com/training/appbar/setting-up.html)
* [Adding and Handling Actions](https://developer.android.com/training/appbar/actions.html#handle-actions)
* [AppCompat v21 — Material Design for Pre-Lollipop Devices!](https://android-developers.googleblog.com/2014/10/appcompat-v21-material-design-for-pre.html)
* [Using the Android Toolbar (ActionBar) - Tutorial](http://www.vogella.com/tutorials/AndroidActionBar/article.html)
* [Goodbye ActionBar APIs, hello Toolbar](https://medium.com/@ZakTaccardi/goodbye-actionbar-apis-hello-toolbar-af6ae7b31e5d)
* [Have you been calling 'setSupportActionBar()'? You don't have to!](https://www.reddit.com/r/androiddev/comments/3m3pd0/have_you_been_calling_setsupportactionbartoolbar/?st=j6yhe4s9&sh=6528a88d)