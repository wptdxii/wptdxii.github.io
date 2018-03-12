---
title: Android Widget Tricks&Tips
date: 2018-01-19 11:05:59
tags: Android Widget
categories: Android
---

Android Widget 使用中遇到的坑和一些技巧

<!-- more -->

# ConstraintLayout

**Ref**:

* [Build a Responsive UI with ConstraintLayout](https://developer.android.com/training/constraint-layout/index.html)
* [Constraint library class index](https://developer.android.com/reference/android/support/constraint/classes.html)
* [ConstraintLayout](https://constraintlayout.com/)
* [Use ConstraintLayout to design your Android views](https://codelabs.developers.google.com/codelabs/constraint-layout/#0)
* [ConstraintLayout](https://blog.stylingandroid.com/category/layouts/constraintlayout/)
* [What's new in Constraint Layout 1.1.x](https://medium.com/@rafael_toledo/whats-new-in-constraint-layout-1-1-x-f0bdd4dbdfb3)
* [New features in ConstraintLayout 1.1.x](http://androidkt.com/constraintlayout/)
* [Constraint Layout 1.1.x带来了哪些新东西？](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2017/1019/8618.html)

# SwipeRefreshLayout

# CoordinatorLayout

CoordinatorLayout 是加强版的 FrameLayout，可以监听子控件的各种事件，自动做出最合理的响应。例如使用 FloatingActionButton 和 Snackbar 时，为了防止 Snackbar 遮盖 FloatingActionButton，可以使用 CoordinatorLayout 作为根部局。

**Ref**:

* [CoordinatorLayout](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html)
* [Intercepting everything with CoordinatorLayout Behaviors](https://medium.com/google-developers/intercepting-everything-with-coordinatorlayout-behaviors-8c6adc140c26)
* [Using CoordinatorLayout in Android apps](https://www.androidauthority.com/using-coordinatorlayout-android-apps-703720/)
* [Mastering the Coordinator Layout](http://saulmm.github.io/mastering-coordinator)

# AppBarLayout

AppBarLayout 是应用了 Material Design 设计理念并封装了许多滚动事件的垂直线性布局，其必须作为 CoordinatorLayout 的直接子控件使用，否则其功能将无效。可以套在 Toolbar 外边防止
CoordinatorLayout 子控件之间覆盖，以及实现滑动隐藏 Toolbar 的效果

**Ref**:

* [AppBarLayout](https://developer.android.com/reference/android/support/design/widget/AppBarLayout.html)

# CollapsingToolbarLayout

CollapsingToolbarLayout 用于包裹 Toolbar 以实现折叠标题栏的效果，其必须是 AppBarLayout 的直接子控件

**Ref**:

* [CollapsingToolbarLayout](https://developer.android.com/reference/android/support/design/widget/CollapsingToolbarLayout.html)