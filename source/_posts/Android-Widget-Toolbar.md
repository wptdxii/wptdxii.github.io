---
title: Android-Widget-Toolbar
date: 2017-08-21 16:47:06
tags: [Android Widget, Material Design]
categories: Android
---
Material Design 是 Google 推出的一套视觉设计语言，主要面向 UI 设计人员，详细资料可以参看：

* [Material Design](https://material.io/)
* [Material Design 中文版](http://design.1sters.com/)

为了方便实现 Materrial Design 规范的设计效果，Google 同时针对 Android 开发提供了 Design Support 库，将一些代表性的控件和效果进行了封装，可以很方便的将应用 Material 化。

<!-- more -->

# Toolbar

ActionBar 由于其设计的原因，被限定只能位于 Activity 的顶部，不能自定义布局，而且不能实现 Material Design 的效果，所以不再推荐使用，官方推出了灵活性更高的 Toolbar 作为替代。
Toolbar 有两种使用方式：

* 应用栏(AS ActionBar)
* 普通控件(As Standalone Widget)

当使用应用栏时，需要通过 setSupportActionBar() 将 Toolbar 设置为应用栏，但这种方式时具有以下缺点：

* 屏幕上只能显示一个应用栏，如果在多个 Fragment 中都使用 Toolbar 作为应用栏，当几个 Fragment 需要并列同时显示，例如在对平板进行适配时，其各自的应用栏无法同时显示
* 在 Fragment 中创建溢出菜单时会遇到回调地狱，只有调用 setHasOptionsMenu(true) 后，onCreateOptionsMenu() 回调才会被触发

从功能上讲，Toolbar 继承并扩展了 ActionBar 的所有功能，所以 Toolbar 可以作为普通控件使用而完全代替 ActionBar，而不必将其设置为应用栏，但是对于使用了 ActionBar 的老项目，为了复用系统回调的实现代码，减少迁移成本，将 Toolbar 设置为应用栏最简便。所以对于有历史包袱的老项目，可以将 Toolbar 设置为系统应用栏使用；对于新项目，建议将 Toolbar 作为普通控件使用。

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

> * 主题配置文件中的 android:colorPrimary 指定的是 ActionBar 的背景颜色，需要将其应用到 android:background 属性才可正常显示
> * 根据  [Material-Design-Shadows](https://material.io/guidelines/material-design/elevation-shadows.html#elevation-shadows-shadows) 规范，应该给 Toolbar 设置 4dp 的高度阴影

然后就可以在 Activity 中使用 Toolbar：

```java
public class MainActivity extends AppCompatActivity {
       @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
    }
    // ...
}
```

> * Activity 需要继承 AppCompatActivity
> * 调用 setSupportActionBar() 可以将 Toolbar 设置为应用栏，否则作为普通控件使用
> * Toolbar 设置为应用栏后，可以通过 getSupportActionBar() 获取 ActionBar 对象的引用

## 设置 Navigation

Toolbar 左上角提供了 Navigation Button，通常用于返回主页或者侧边抽屉触发，图标和点击触发的逻辑都是可以自定义的。当使用应用栏时，该 Button 默认作为 Up Button，用于返回 Parent Activity，为了实现该功能，首先要在清单文件中声明 Parent Activity：

```xml
<application ... >
    ...

    <activity
        android:name="com.example.ChildActivity"
        android:label="@string/title_activity_child"
        android:parentActivityName="com.example.MainActivity" >

        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
```

> * android:parentActivityName 属性支持 API 16 及其以上的版本 < meta-data > 标签为了兼容 API 16 以下的版本
> * 当未声明 Parent Activity 时，点击 Up Button 无响应

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
> * Up button 的默认图标是一个返回箭头，可以通过 ActionBar.setHomeAsUpIndicator() 自定义。当使用默认图标时，图标颜色可由主题中的 colorAccent 属性设定

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

> Up button 的 id 固定为：android.R.id.home

当 Toolbar 作为普通控件使用时，不再触发系统的 onOptionsItemSelected() 回调，但可以通过下面代码模拟出 Up Button 的效果：

```java
        Toolbar toolbar = findViewById(R.id.toolbar);
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

> 如果使用应用栏，Toolbar.setNavigationOnClickListener() 需要在 setSupportActionBar() 之后调用才有效，且会覆盖 onOptionsItemSelected() 方法的触发，否则触发的还是onOptionsItemSelected() 方法

## 设置 Logo/Title/Subtitle

使用应用栏时，应用栏上的 Title 会默认显示为应用的名称，可通过下面代码隐藏：

```java
        ActionBar actionBar = getSupportActionBar();
        actionBar.setDisplayShowTitleEnabled(false);
```

> * 当 Toolbar 作为普通控件使用时，不会显示默认 Title
> * 在 onCReate() 方法中初始化时，如果调用上面代码，调用 Toolbar.setTitle() 和 Toolbar.setSubtitle() 都是无效的；如果不调用上面代码，Toolbar.setTitle() 必须在 setSupportActionBar() 方法前调用才有效， Toolbar.setSubtitle() 则无影响

## 设置 Menu

### 加载 Menu

Menu 文件中 \<item\> 标签的 app:showAsAction 属性用于指定 Menu Item 的显示方式，其有以下几种值：

* app:showAsAction="always"：Menu Item 一直显示在应用栏
* app:showAsAction="ifRoom"：当应用栏有空间时，Menu  Item 显示在应用栏，如果没有足够空间则显示在溢出菜单
* app:showAsAction="never"：Menu Item 只显示在溢出菜单

当使用应用栏时，在初始化时下面两个方法会依次被回调：

* onCreateOptionsMenu()：系统回调该方法后，会保留被填充的 Menu 实例，除非因某些原因该实例失效，否则该方法不会被再次调用。所以在只应该在此方法中加载菜单的初始状态，不应该在 Activity 的生命周期内执行任何更改
* onPrepareOptionsMenu()：该方法持有已经被加载的 Menu 实例，主要用于在运行时更改菜单，如添加、移除或隐藏 Menu Item

> Activity 初始化时，这两个方法会依次被调用；当调用 invalidateOptionsMenu() 时，这两个方法也都会被调用；当点击溢出菜单按钮时，只有 onPrepareOptionsMenu() 被调用

点击对应的菜单项时，会触发下面的回调：

* onOptionsItemSelected()

> 在 setSupportActionBar() 之前调用 Toolbar.setOnMenuItemClickListener() 是无效的；在其之后调用则会覆盖系统的 onOptionsItemSelected() 回调

溢出菜单 Menu Item 默认不显示图标，可在加载菜单时通过下面代码显示：

```java
@SuppressLint("RestrictedApi")
public void showOptionalIcons(Menu menu) {
    if(menu instanceof MenuBuilder) {
        ((MenuBuilder)menu).setOptionalIconsVisible(true);
    }
}
```

> Activity 初始化完成后，可以通过 Toolbar.inflateMenu() 重新加载 Menu(需要先通过 Toolbar.getMenu().clear() 清除之前的 Menu)，通过 Toolbar.setOnsetOnMenuItemClickListener() 重置回调

### 设置 Action View

Action View 可以实现在无需启动新页面的情况下扩展应用栏的功能，为了使用 Action View，需要给菜单文件的 \<item\> 标签添加下面两个属性之一：

* app:actionViewClass：View 类的全类名
* app:actionLayout：自定义布局

当 Menu Item 不与用户交互时显示 android:icon，交互时扩展开 app:actionViewClass。通过 MenuItem.getAction() 强转后即可获取对应的 ActionView 对象。

可以通过下面代码可以给 Menu Item 设置收缩展开监听：

```java
    MenuItem.setOnActionExpandListener(new MenuItem.OnActionExpandListener() {
        @Override
        public boolean onMenuItemActionExpand(MenuItem item) {
            // todo
            return true; // Return true to collapse action view
        }

        @Override
        public boolean onMenuItemActionCollapse(MenuItem item) {
            // todo
            return true; // Return true to expand action view
        }
    });
```

### 设置 Action Provider

MenuItem 可以通过代码或者在布局文件中指定 ActionProvider，ActionProvider 可以为需要显示的 MenuItem 提供布局，并为被点击的 MenuItem 提供默认的响应。自定义 ActionProvider 需要注意下面几点：

* 为了兼容性需要继承 android.support.v4.view.ActionProvider
* 在布局文件中指定 ActionProvider 时需要使用  app:actionProviderClass 属性，并指定全类名
* 重写 onCreateActionView() 方法，返回需要显示的布局，最好在构造器中加载布局及其子控件，因为如果 Activity 使用应用栏，在 Activity 的 onCreateOptionsMenu() 方法中获取对应的 ActionProvider 时， ActionProvider 的 onCreateOptionsMenu() 还未被回调，在 onCreateActionView() 中使用 ActionProvider 布局的子控件容易报空指针异常
* onCreateActionView() 返回 null 时 onPrepareSubMenu() 才会被回调
* 如果 ActionProvider 指定了子菜单，系统不会回调 onPerformDefaultAction()

## 设置 Overflow Menu Button

可以通过代码设置溢出菜单按钮的样式：

```java
Toolbar.setOverflowIcon();
```

也可以在主题文件中设定，首先在主题文件中添加如下样式：

```xml
  <style name="ToolbarTheme.OverflowButton" parent="Widget.AppCompat.ActionButton.Overflow">
        <item name="android:src">@drawable/ic_add_white_24dp</item>
  </style>
```

然后将其应用到 Toolbar 样式的属性：

```xml
 <style name="ToolbarStyle" parent="ThemeOverlay.AppCompat.Dark.ActionBar">
        ...
        <item name="actionOverflowButtonStyle">@style/ToolbarTheme.OverflowButton</item>
        ...
  </style>
```

## 设置 Overflow Menu

溢出菜单默认是覆盖在应用栏上的，可以通过主题将其设置在应用栏的下方。首先在 res/values/styles.xml 文件中添加样式：

```xml

 <style name="ToolbarOverflowStyle" parent="Widget.AppCompat.PopupMenu.Overflow"">
        <item name="overlapAnchor">false</item>
        <item name="android:dropDownVerticalOffset">-4dp</item>
 </style>

```

为了保证兼容性，同时在 res/values-v21/style.xml 文件中添加样式：

```xml
 <style name="ToolbarOverflowStyle" parent="Widget.AppCompat.PopupMenu.Overflow"">
        <item name="overlapAnchor">false</item>
        <item name="android:dropDownVerticalOffset">4dp</item>
 </style>
```

然后将上边的样式应用到 Toolbar 样式的下列属性：

```xml
 <style name="ToolbarStyle" parent="ThemeOverlay.AppCompat.Dark.ActionBar">
        ...
        <item name="actionOverflowMenuStyle">@style/ToolbarTheme.OverflowMenu</item>
        ...
  </style>
```

## Ref

* [Adding the App Bar](https://developer.android.com/training/appbar/index.html)
* [AppCompat v21 — Material Design for Pre-Lollipop Devices!](https://android-developers.googleblog.com/2014/10/appcompat-v21-material-design-for-pre.html)
* [Using the Android Toolbar (ActionBar) - Tutorial](http://www.vogella.com/tutorials/AndroidActionBar/article.html)
* [Goodbye ActionBar APIs, hello Toolbar](https://medium.com/@ZakTaccardi/goodbye-actionbar-apis-hello-toolbar-af6ae7b31e5d)
* [Have you been calling 'setSupportActionBar()'? You don't have to!](https://www.reddit.com/r/androiddev/comments/3m3pd0/have_you_been_calling_setsupportactionbartoolbar/?st=j6yhe4s9&sh=6528a88d)
* [Menu](https://developer.android.com/guide/topics/ui/menus.html)
* [Using Custom Views As Menu Items](https://stablekernel.com/using-custom-views-as-menu-items/#top)
* [ActionProvider](https://developer.android.com/reference/android/support/v4/view/ActionProvider.html)
* [Support v7 ActionBarActivity, OnCreateOptionsMenu() called after each call to SupportInvalidateOptionsMenu()](https://stackoverflow.com/questions/28803908/support-v7-actionbaractivity-oncreateoptionsmenu-called-after-each-call-to-su)
* [How to set Toolbar text and back arrow color](https://stackoverflow.com/questions/26969424/how-to-set-toolbar-text-and-back-arrow-color)
* [How to implement DrawerArrowToggle from Android appcompat v7 21 library](https://stackoverflow.com/questions/26434504/how-to-implement-drawerarrowtoggle-from-android-appcompat-v7-21-library)