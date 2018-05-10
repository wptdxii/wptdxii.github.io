---
title: Android 状态栏&导航栏
date: 2018-04-25 13:55:47
tags: [System Bar, Status Bar, Navigation Bar]
categories: Android
---

记录状态栏和导航栏的 tips & tricks

<!-- more -->

Android System Bar 有两种:

* Status Bar
* Navigation Bar

# 淡化 system bars

Android 4.0 (API level 14) 及其以后的版本，可以通过下列代码片段淡化 System Bar，而对于之前的版本，系统并没有内建淡化 Status Bar 的方式：

```java
// 不需要进行版本判断，对于低版本无效
// 这里使用了 DecorView，但其实可以使用任何一个可见的 View
View decorView = getActivity().getWindow().getDecorView();
int uiOptions = View.SYSTEM_UI_FLAG_LOW_PROFILE;
decorView.setSystemUiVisibility(uiOptions);
```

> 点击或者触摸 System Bar 时，flag 会被清除，淡化效果会消失

可以通过下面的代码片段清除 flag：

```java
View decorView = getActivity().getWindow().getDecorView();
decorView.setSystemUiVisibility(0);
```

# 隐藏 System bar

当 Activity 主题被设置为全屏模式时，Status Bar 会被隐藏：

```xml
  <style name="AppTheme.FullScreen" parent="AppTheme">
        <item name="android:windowFullscreen">true</item>
  </style>
```

使用 activity theme 的优势如下:

* 容易维护，相较于使用 flag 更不容易出错
* 系统在 Activity 初始化之前就会获得隐藏状态栏的请求，UI 过度更平滑

也可以通过代码实现 System Bar 的隐藏，不同 Android 版的实现方式不同

Android 4.0 (API level 14) 及其之前的版本，通过给 WindowManager 设置 Flags 实现隐藏 Status Bar：

```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // api 14/15 都是 Android 4.0
        if (Build.VERSION.SDK_INT < 16) {
            getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                    WindowManager.LayoutParams.FLAG_FULLSCREEN);
        }
        setContentView(R.layout.activity_main);
    }
    ...
```

* WindowManager flags 如果不手动清除，会一直生效
* 可以使用 FLAG_LAYOUT_IN_SCREEN，其作用与 FLAG_FULLSCREEN 相同，但其可以阻止状态栏在显示和隐藏两种状态之间切换时重新分配 UI 大小

Android 4.1 (API level 15) 及其之后的版本，不再使用 WindowManager，而是将 UI Flag 作用于具体的 View 层来隐藏 Status Bar，这些 Flags 最终会聚合到 Window，通过这种方式可以更细粒度的控制 Status Bar，但是对于 Navigation Bar，Android 4.0 (API level 14) 及其之后的版本都可以通过具体的 View 实现隐藏：

```java
// 不用判断版本，对于 api < 16 的情况，View.SYSTEM_UI_FLAG_FULLSCREEN 不生效
// 对于 api >= 14，View.SYSTEM_UI_FLAG_HIDE_NAVIGATION 都生效
View decorView = getWindow().getDecorView();
// 两个 flag 分别隐藏 Status Bar 和 Navigation Bar
// 根据设计规范，隐藏 Navigation Bar 时也要隐藏 Status Bar
int uiOptions = View.SYSTEM_UI_FLAG_FULLSCREEN|View.SYSTEM_UI_FLAG_HIDE_NAVIGATION;
decorView.setSystemUiVisibility(uiOptions);
// 根据设计规范，如果隐藏了 Status Bar，ActionBar 不应该单独显示，如果有也需要隐藏
ActionBar actionBar = getActionBar();
actionBar.hide();
```

使用这种方式隐藏 System Bar 时需要注意以下几点：

* 导航到其它 Activity 或者 Home 键回到桌面，再返回 Activity，Flags 会被清空
* UI Flags 被清除后需要重设，所以相较于 onCreate()，在 onResume() or onWindowFocusChanged() 中设置 Flags 更好
* setSystemUiVisibility() 只对可见的 View 生效

# 全屏

全屏分为三种模式，全屏模式下，Activity 任然可以接收到触摸事件，这三种模式区别在于唤起被隐藏的 System Bar 的方式：

* Lean back
  * Lean back 模式适用于与用户交互较少的全屏模式，例如视频播放，该模式下，轻触屏幕的任何一个地方，都会唤起 System Bar。全屏模式时不使用任何 Flag，则进入 Lean back 模式
* Immersive
  * Immersive 模式适用于与用户交互较多的全屏模式，例如游戏、相册等，该模式下，从隐藏了 System Bar 的边缘向内侧滑动，就会唤起 System Bar，唤起的 System Bar 会屏蔽用户在该位置的操作。全屏模式时使用 View.SYSTEM_UI_FLAG_IMMERSIVE，则进入 Immersive 模式
* Sticky immersive
  * Sticky immersive 模式与 Immersive 模式类似，但唤起的 System Bar 不会屏蔽用户在该位置的操作，例如画图应用，边缘滑动唤起 System Bar 时并不会屏蔽用户操作，依然可以在 System Bar 的位置作画。全屏模式时使用 View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY，则进入 Sticky immersive 模式

上面三种模式中，Lean back 和 Immersive 模式可以监听到 System Bar 的变动：

```java
// 可以在 onCreate() 中调用
View decorView = getWindow().getDecorView();
decorView.setOnSystemUiVisibilityChangeListener
        (new View.OnSystemUiVisibilityChangeListener() {
    @Override
    public void onSystemUiVisibilityChange(int visibility) {
        // Note that system bars will only be "visible" if none of the
        // LOW_PROFILE, HIDE_NAVIGATION, or FULLSCREEN flags are set.
        if ((visibility & View.SYSTEM_UI_FLAG_FULLSCREEN) == 0) {
            // 显示 System Bar 的回调
        } else {
            // 隐藏 System Bar 的回调
        }
    }
});
```

而对于 Sticky immersive 模式，无法接收到上面的回调，所以无法监听到 Sysytem Bar 显示隐藏的变动。

当回到桌面或者开启其它 Activity 再回到全屏 Activity 时，隐藏 System Bar 的 Flags 会被清除，所以实现全屏的代码最好放在 onWindowFocusChanged() 中：

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        hideSystemUI();
    }
}

private void hideSystemUI() {
    // 下面代码使用的是 immersive 模式
    // 如果需要使用 lean back 模式, 移除 SYSTEM_UI_FLAG_IMMERSIVE.
    // 如果需要使用 sticky immersive 模式, 将 SYSTEM_UI_FLAG_IMMERSIVE 替换为 SYSTEM_UI_FLAG_IMMERSIVE_STICKY
    View decorView = getWindow().getDecorView();
    decorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_IMMERSIVE
            // Set the content to appear under the system bars so that the
            // content doesn't resize when the system bars hide and show.
            | View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            // Hide the nav bar and status bar
            | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_FULLSCREEN);
}

// Shows the system bars by removing all the flags
// except for the ones that make the content appear under the system bars.
private void showSystemUI() {
    View decorView = getWindow().getDecorView();
    decorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
}
```

# 沉浸 System Bar

Android 4.1 (API level 16) 及其之后的版本，布局可以沉浸到状态栏后面，

```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.sample_activity_system_bar);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            View decorView = getWindow().getDecorView();
            // 该 flags 使布局可以沉浸到 Status Bar
            int uiOptions = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION |
                                View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
            decorView.setSystemUiVisibility(uiOptions);
        }
    }
```

当布局沉浸到 Status Bar 之后，状态栏会部分地挡住控件，可以给相应地控件添加下列属性避免其沉浸到 Status Bar 后面：

```xml
android:fitsSystemWindows="true"
```

该属性会调整该控件对应 Parent ViewGroup 的 Padding，

# System Bar 着色

Android 4.4 (API level 19) 可以使用透明状态栏，在主题文件中定义：

```xml
 // /values-v19/styles.xml
 <style name="AppTheme.Translucent" parent="AppTheme">
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
 </style>
```

使用了该主题后，布局会沉浸到 System Bar 下面

详情参看：

* [Translucent system bars](https://developer.android.com/about/versions/android-4.4#TranslucentBars)
* [windowTranslucentStatus](https://developer.android.com/reference/android/R.attr.html#windowTranslucentStatus)
* [windowTranslucentNavigation](https://developer.android.com/reference/android/R.attr#windowTranslucentNavigation)

对于 Android 5.0 (API level 21) 及其之后版本，还可以给 System Bar 着色，详情参看：

* [setStatusBarColor](https://developer.android.com/reference/android/view/Window#setstatusbarcolor)
* [setNavigationBarColor](https://developer.android.com/reference/android/view/Window#setnavigationbarcolor)

# Ref

* [Control the system UI visibility](https://developer.android.com/training/system-ui/)
* [Android沉浸式状态栏完全解析](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650236820&idx=1&sn=78cc47bc3448d59b391faab8ca3c5123&mpshare=1&scene=1&srcid=0425qkfSwk7aepPuA8ji8cuW#rd)
* [Android 5.0 如何实现将布局的内容延伸到状态栏?](https://www.zhihu.com/question/31468556)
* [Android状态栏着色实践](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650237284&idx=1&sn=6e3b30cc63d675bae8cb3e7c9563384e&chksm=8863980bbf14111d1e3ac9522ae56433a3486d58e7ad5b58c1894270a126823983fc3a6017ac&mpshare=1&scene=1&srcid=042589CR7xQ05W3lmABxI5sY#rd)