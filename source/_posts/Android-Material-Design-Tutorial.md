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

* 应用栏(ActionBar)
* 独立控件(Standalone Widget)

当作为应用栏使用时，需要通过setSupportActionBar() 将 Toolbar 设置为应用栏，此时可以使用 ActionBar 提供的一些诸如 ActionBar.show()/hide() 之类的 API，但以这种方式使用 Toolbar 时具有以下缺点：

* 屏幕上只能显示一个应用栏，如果在多个 Fragment 中都使用 Toolbar 作为应用栏，当几个 Fragment 需要并列同时显示，例如在对平板进行适配时，其各自的应用栏无法同时显示
* 在 Fragment 中创建溢出菜单时会遇到回调地狱，只有调用 setHasOptionsMenu(true) 后，onCreateOptionsMenu() 回调才会被触发

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
        setSupportActionBar(toolbar);
    }
    // ...
}
```

> * Activity 需要继承 AppCompatActivity
> * 调用 setSupportActionBar() 可以将 Toolbar 设置为应用栏，否则作为独立控件使用
> * Toolbar 设置为应用栏后，可以通过 getSupportActionBar() 获取 ActionBar 对象的引用

## 配置 Toolbar

Toolbar 主要由以下几部分组成：

![toolbar_widget.png](http://otg3f8t90.bkt.clouddn.com/2017/11/13/toolbar_widget.png)

### 设置 Navigation Button

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

> * android:parentActivityName 属性支持 API 16 及其以上的版本
> * < meta-data > 标签为了兼容 API 16 以下的版本
> * 当未声明 Parent Activity 时返回上个 Activity

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

> * Up button 的 id 固定为：android.R.id.home

当 Toolbar 作为独立控件使用时，不再触发系统回调，即 onOptionsItemSelected() 方法不会被触发，可以通过 Toolbar 的方法模拟出 Up Button 的效果：

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

> 如果 Toolbar 作为应用栏使用，Toolbar.setNavigationOnClickListener() 需要在 setSupportActionBar() 之后调用才有效，且会覆盖 onOptionsItemSelected() 方法的触发，否则触发的还是onOptionsItemSelected() 方法

### 设置 Logo/Title/Subtitle

Toolbar 可以通过以下方法设置 Logo/Title/Subtitle:

```java
        Toolbar toolbar = findView(R.id.toolbar);
        toolbar.setLogo();
        toolbar.setTitle();
        toolbar.setSubtitle();
        setSupportActionBar(toolbar);
```

当 Toolbar 被设置为应用栏时，Title 会被默认设置为应用的名称，Toolbar.setTitle() 必须在 setSupportActionBar() 之前调用才生效，此时可以通过下面的方式隐藏掉 Title:

```java
        ActionBar actionBar = getSupportActionBar();
        actionBar.setDisplayShowTitleEnabled(false);
```

> 当 Toolbar 作为独立控件使用时，Title 不会默认显示为应用名称

### 设置 Overflow Menu

在 res/menu 路径下创建 menu 文件：

```xml
res/menu/sample.xml

<menu xmlns:android="http://schemas.android.com/apk/res/android" >

    <!-- "Mark Favorite", should appear as action button if possible -->
    <item
        android:id="@+id/action_favorite"
        android:icon="@drawable/ic_favorite_black_48dp"
        android:title="@string/action_favorite"
        app:showAsAction="ifRoom"/>

    <!-- Settings, should always be in the overflow -->
    <item android:id="@+id/action_settings"
          android:title="@string/action_settings
          app:showAsAction="never"/>

</menu>
```

> 属性 app:show AsAction 用于指定 Action 类型，如果 app:showAsAction="ifRoom"，当应用栏有空间时 Action 会以 Button 的方式会显示在应用栏上，如果没有空间则会以 Menu Item 的方式显示。如果 app:showAsAction="never"，则会一直以 Menu Item 显示

如果使用应用栏，通过下面这种方式加载 Overflow Menu:

```java
  @SuppressLint("RestrictedApi")
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        if (menu instanceof MenuBuilder) {
            ((MenuBuilder) menu).setOptionalIconsVisible(true);
        }
        getMenuInflater().inflate(R.menu.sample, menu);
        // toolbar.inflateMenu(R.menu.sample);
        return true;
    }
```

此时下面的回调会被触发：

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.action_settings:
            // ...
            return true;

        case R.id.action_favorite:
            // ...
            return true;
        default:
            // ...
            return super.onOptionsItemSelected(item);
    }
}
```

> * Overflow Menu 中默认不显示 Menu Item 的图标，可以通过 MenuBuilder.setOptionalIconsVisible() 控制其显示
> * 如果调用了 setSupportActionBar()，则在 onCreate() 方法中再调用 Toolbar.inflateMenu() 不会生效，只有在 onCreateOptionsMenu() 回调中才能加载 Overflow Menu，使用 MenuInflater 或者 Toolbar 加载都可以。点击触发的还是 onOPtionsItemSelected() 方法
> * 在其他地方可以通过 Toolbar.inflateMenu() 重新加载 Menu(需要先通过 Toolbar.getMenu().clear() 清除之前的 Menu)，通过 Toolbar.setOnsetOnMenuItemClickListener() 重新设置回调

如果 Toolbar 作为独立控件使用，通过下面这种方式加载 Overflow Menu 并设置点击回调:

```java
        Toolbar toolbar = findView(R.id.toolbar);
        Menu menu = toolbar.getMenu();
        if (menu instanceof MenuBuilder) {
            ((MenuBuilder) menu).setOptionalIconsVisible(true);
        }
        toolbar.inflateMenu(R.menu.sample);
        toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                 switch (item.getItemId()) {
                    case R.id.action_settings:
                        // ...
                        return true;
                    case R.id.action_favorite:
                        // ...
                        return true;
                        default:
                        // ...
                        return false;
            }
        });
```

需要在运行时改变 Overflow Menu 时需要先清除之前的 Menu，否则两个 Menu 都会被加载:

```java
    Toolbar.getMenu().clear();
    Toolbar.inflateMenu(R.menu.sample);
```

> 无论 Toolbar 是作为应用栏还是独立控件使用， 都可以通过 Toolbar 动态改变 Overflow Menu

### 设置 Action View

Action View 用于在无需启动新页面的情况下扩展应用栏的功能，例如使用 Search Action View 可以在当前应用栏实现搜索功能，而不必新打开 Activity 或 Fragment，如下图所示：

![action_view.png](http://otg3f8t90.bkt.clouddn.com/2017/11/1/action_view.png)

为了使用 Action View，需要给菜单文件的 \<item\> 添加下面两个属性之一：

* app:actionViewClass：实现功能扩展
* app:actionLayout：在应用栏显示自定义布局

 下面代码示例了 actionViewClass 的使用：

```xml
<item android:id="@+id/action_search"
     android:title="@string/action_search"
     android:icon="@drawable/ic_search"
     app:showAsAction="ifRoom|collapseActionView"
     app:actionViewClass="android.support.v7.widget.SearchView" />
```

> * 当 Action Button/Menu Item 不与用户交互时显示 android:icon，交互时扩展开 app:actionViewClass
> * collapseActionView 可以控制应用栏是否展开

当使用应用栏时，通过下面代码可以获取 SearchView  实例：

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main_activity_actions, menu);
    MenuItem searchItem = menu.findItem(R.id.action_search);
    SearchView searchView = (SearchView) menuItem.getActionView();
    return super.onCreateOptionsMenu(menu);
}
```

如果 Toolbar 作为独立控件使用，通过下面代码可以获取 SearchView 实例：

```java
        toolbar.inflateMenu(R.menu.activity_main);
        Menu menu = toolbar.getMenu();
        MenuItem menuItem = menu.findItem(R.id.menu_search);
        SearchView searchView = (SearchView) menuItem.getActionView();
```

可以通过下面代码可以给 Menu Item 设置收缩展开监听：

```java
    getMenuInflater().inflate(R.menu.activity_main, menu);
    MenuItem menuItem = menu.findItem(R.id.menu_search);
    menuItem.setOnActionExpandListener(new MenuItem.OnActionExpandListener() {
        @Override
        public boolean onMenuItemActionExpand(MenuItem item) {
            Toast.makeText(MainActivity.this, "Expand", Toast.LENGTH_SHORT).show();
            return true; // Return true to collapse action view
        }

        @Override
        public boolean onMenuItemActionCollapse(MenuItem item) {
            Toast.makeText(MainActivity.this, "Collapse", Toast.LENGTH_SHORT).show();
            return true; // Return true to expand action view
        }
    });
```

> 当使用应用栏时，如果 \<item\> 元素添加了 collapseActionView 属性，并且 onOptionsItemSelected() 方法被重写，被重写的子类必须调用 super.onCreateOptionsMenu(menu)，Action View 才能正常伸缩

actionLayout 用于在应用栏显示自定义布局，如下图效果：

![toolbar_action_layout.jpg](http://otg3f8t90.bkt.clouddn.com/2017/11/14/toolbar_action_layout.jpg)

实现该效果需要先定义布局文件：

```xml
res/layout/message_action.xml

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="@dimen/menu_item_size"
    android:layout_height="@dimen/menu_item_size">

    <FrameLayout
        android:layout_width="@dimen/menu_item_container_size"
        android:layout_height="@dimen/menu_item_container_size"
        android:layout_gravity="center">

        <ImageView
            android:id="@+id/iv_message"
            android:layout_width="@dimen/menu_item_icon_size"
            android:layout_height="@dimen/menu_item_icon_size"
            android:layout_gravity="center"
            android:scaleType="fitXY"
            android:src="@drawable/ic_message_white_24dp" />

        <TextView
            android:id="@+id/tv_badge"
            android:layout_width="@dimen/menu_item_badge_size"
            android:layout_height="@dimen/menu_item_badge_size"
            android:layout_gravity="top|end"
            android:background="@drawable/shape_badge_bg_red"
            android:gravity="center"
            android:textColor="@color/color_white_ffffffff"
            android:textSize="10sp"
            android:visibility="visible"
            android:text="5"
            tools:ignore="SmallSp"
            tools:visibility="visible" />
    </FrameLayout>
</FrameLayout>
```

使用 actionLayou 属性：

```xml
...
<item
        android:id="@+id/menu_message_modified"
        android:icon="@drawable/ic_message_white_24dp"
        android:title="消息"
        app:actionLayout="@layout/message_action"
        app:showAsAction="always" />
```

与 actionViewClass 的使用类似，通过下面代码可以获取对应的 View 实例，继而给子 View 添加对应的逻辑：

```java
     // ...
    MenuItem menuItem = menu.findItem(R.id.menu_message);
    FrameLayout flAction = (FrameLayout)menuIte.getActionView();
    // ...
```

### 设置 Action Provider

Action Provider 用于对应用栏上的 Action 进行自定义操作，可以自定义布局，其需要继承 android.support.v4.view.ActionProvider。app:actionLayout 属性主要用来比较方便的展示布局，当需要一些自定义操作比如点击时展示菜单等，可以使用 Action Provider 进行封装。下面用 Action Provider 实现与上边 actionLayout 相同的效果。

首先定义 ActionProvider 子类：

```java
// MessageActionProvider.java

public class MessageActionProvider extends ActionProvider {
    private View actionView;
    private TextView tvBadge;

    public MessageActionProvider(Context context) {
        super(context);
        actionView = View.inflate(context, R.layout.provider_message_action, null);
        tvBadge = actionView.findViewById(R.id.tv_badge);
    }

    @Override
    public View onCreateActionView() {
        return actionView;
    }

    public void setOnClickListener(View.OnClickListener onClickListener) {
        actionView.setOnClickListener(onClickListener);
    }

    public void setMessageCount(int count) {
        tvBadge.setVisibility(count > 0 ? View.VISIBLE : View.GONE);
        tvBadge.setText(String.valueOf(count));
    }

    public void setMessageCount(String count) {
        if (TextUtils.isDigitsOnly(count)) {
            setMessageCount(Integer.parseInt(count));
        }
    }
}
```

使用 app:actionProviderClass 属性：

```xml
...
<item
        android:id="@+id/menu_message_modified"
        android:icon="@drawable/ic_message_white_24dp"
        android:title="消息"
        app:actionProviderClass="com.sample.MessageActionProvider"
        app:showAsAction="always" />
```

使用应用栏时，通过下面代码获取 ActionProvider 实例：

```java
    @SuppressLint("RestrictedApi")
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        if (menu instanceof MenuBuilder) {
            ((MenuBuilder) menu).setOptionalIconsVisible(true);
        }

        getMenuInflater().inflate(R.menu.activity_toolbar_sample, menu);
        MenuItem menuItem = menu.findItem(R.id.menu_message);
        MessageActionProver actionProvider = (MessageActionProvider) MenuItemCompat.getActionProvider(menuItem);
        actionProvider.setOnClickListener(view -> onOptionsItemSelected(menuItem));
        actionProvider.setMessageCount("9");
        return super.onCreateOptionsMenu(menu);
    }

```

> 在 onCreateOptionsMenu() 方法中获取 ActionProvider 时， ActionProvider 的 onCreateActionView() 并未被触发，如果在 onCreateActionView() 获取其子 View，并在 onCreateOptionsMenu() 中调用 ActionProvider 的子 View 会报空指针异常，所以应该在 ActionProvider 的构造器中初始化子 View

或者使用 Toolbar 获取其实例：

```java
        Menu menu = toolbar.getMenu();
        MenuItem menuItem = menu.findItem(R.id.menu_share);
        MessageActionProvider mActionProvider = (MessageActionProvider) MenuItemCompat.getActionProvider(menuItem);
        actionProvider.setMessageCount(9);
```


### 设置 Overflow Menu Button

## Ref

* [Setting Up the App Bar](https://developer.android.com/training/appbar/setting-up.html)
* [Adding and Handling Actions](https://developer.android.com/training/appbar/actions.html#handle-actions)
* [AppCompat v21 — Material Design for Pre-Lollipop Devices!](https://android-developers.googleblog.com/2014/10/appcompat-v21-material-design-for-pre.html)
* [Using the Android Toolbar (ActionBar) - Tutorial](http://www.vogella.com/tutorials/AndroidActionBar/article.html)
* [Goodbye ActionBar APIs, hello Toolbar](https://medium.com/@ZakTaccardi/goodbye-actionbar-apis-hello-toolbar-af6ae7b31e5d)
* [Have you been calling 'setSupportActionBar()'? You don't have to!](https://www.reddit.com/r/androiddev/comments/3m3pd0/have_you_been_calling_setsupportactionbartoolbar/?st=j6yhe4s9&sh=6528a88d)
* [Using Custom Views As Menu Items](https://stablekernel.com/using-custom-views-as-menu-items/#top)