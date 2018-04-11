---
title: Activity
date: 2016-11-29 22:44:41
tags: [Activities, Activity, Lifecycle, LaunchMode, Intent Flag]
categories: Android
---

Android 四大组件之 Activity

<!-- more -->

# 生命周期

## Activity 正常生命周期

正常情况下，Activity 有如下生命周期方法：

![android_activity_lifecycle.png](http://otg3f8t90.bkt.clouddn.com/2018/3/13/android_activity_lifecycle.png)

生命周期方法的说明：

* onCreate()：该回调必须实现，用于进行初始化(成员变量也要在该回调中进行初始化)，初始化 Activity 生命周期中只发生一次的操作。此时 Activity 不可见
* onRestart()：Activity 从不可见状态回到可见状态时会回调该方法如切换到桌面再返回，或者打开新的非透明主题的 Activity 再返回。此时 Activity 不可见
* onStart()：Activity 进入启动状态时会回调该方法，此时 Activity 处于可见状态，但还未出现在前台，未获取焦点，无法与用户交互。这个回调可以用来初始化在 Activity 整个生命周期可能发生多次的操作，
    初始化在 onStop() 中释放的资源。例如注册用于监听 UI 状态变化的 BroadcastReceiver
* onResume()：Activity 已可见，且处于前台，可以与用户进行交互。这个回调可以用来初始化在 Activity 获取焦点时才会使用的组件，初始化会在 onPause() 中释放的资源，例如启动动画、播放视频等
* onPause()：Activity 可见，但失去焦点，不能与用户进行交互。这个回调中可以用来释放在失去焦点时不需要的资源，如暂停动画、视频播放，注销 BroadcastReceiver、传感器等，也可以用来做一些轻量级的数据存储，
    但是注意不能太耗时，例如保存用户数据、进项网络请求、开启数据库事务等，因为这会影响到新的 Activity 的显示，Parent Activity 的 onPause() 必须先执行完， Child Activity 才会启动
* onStop()：Activity 进入不可见状态，该方法中可以做稍微重量级的回收工作，释放在不可见状态下不需要的资源，或者会造成内存泄漏的资源(因为有些情况下系统杀死应用进程时不会回调 onDestory())
* onDestory()：Activity 生命周期的最后一个回调，所有未释放的资源可以在这里释放，在调用 finish() 或者内存不足系统临时杀死应用进程时会回调该方法(有些情况下系统杀死应用进程不会回调该方法，
    所以 onDestory() 不是一定会被调用)，可以使用 isFinishing() 来区分这两种场景。

> 在 onPause() 和 onStop() 中可以通过 isFinishing() 来判断应用是暂处这两种状态还是会回调 onDestory()

## Activity 异常生命周期

资源相关的系统配置发生改变和资源内存不足时，会导致 Activity 异常终止并重建，在回调正常的生命周期方法外，还会回调下面两个方法：

* onSaveInstanceState()：Activity 在异常终止的情况下会回调这个方法，该方法的调用时机是在 onStop() 之前，与 onPause() 没有既定的时序关系，可以通过 Bundle 保存状态数据，
    然后该 Bundle 会作为参数传递给重建时的 onCreate() 和 onRestorInstanceState()
* onRestorInstanceState()：Activity 在异常终止后重建时会调用这个方法，通过 Bundle 参数可以取出在 onSaveInstanceState() 中保存的数据，该方法中的 Bundle 的参数与 onCreate() 方法中的 Bundle 参数的
    区别是前者一定不为空，而后者使用时需要先判空。从时序上来说，该方法会在 onStart() 之后回调

Activity 在异常终止并重建的过程中，会自动保存并恢复当前的视图结构，因为每个 View 也都有 onSaveInstanceState() 和 onRestoreInstanceState() 方法，当 Activity 回调 onSaveInstanceState() 时，
Activity 会委托 Window 保存数据，然后 Window 会委托其上面的顶层容器保存数据，顶层容器是一个 ViewGroup，一般来说很可能是 DecorView，会通知所有的子 View 调用各自的 onSaveInstanceState() 保存数据，
这就实现了当前 Activity 视图结构和控件数据的自动保存。恢复过程与保存过程正好相反。文本框的输入数据和 ListView 的滚动位置等都可以实现自动保存和恢复

当系统内存不足时，系统会根据目标 Activity 的优先级杀死其所在的进程，没有四大组件运行的进程会很快被系统杀死，所以后台任务最好放到 Service 中保证进程有一定的优先级，才不容易被系统杀死。Activity 的优先级如下：

1. 前台 Activity  也就是处于 Resumed 状态的 Activity 优先级最高
1. 可见非前台 Activity 也就是处于 Paused 状态的 Activity 优先级次之
1. 后台 Activity 也就是处于 Stopped 状态的 Activity 优先级最低

当给 Activity 的清单文件中添加下列属性，可阻止 Activity 因下列资源相关的系统配置变动造成的重建：

```xml
android:configChanges="orientation|keyboardHidden|screenSize|locale"
```

参数说明：

* orientation：屏幕的方向发生改变，如横竖屏的切换
* keyboardHidden：键盘的可访问性发生改变，如调出键盘
* screenSize：屏幕的尺寸发生变动时，如旋转屏幕。该参数比较特殊，生效与编译环境有关而与运行环境无关，当编译环境的 minSdkVersion 和 targetSdkVersion 有一个大于等于 13 时，需要配置该参数才不会导致 Activity 重启
* locale：设备的本地位置发生变动，一般指切换了系统语言

当配置了上面的参数后，相应的配置发生变动时 Activity 不会异常重启，即生命周期方法不会被调用，onSaveInstanceState() 和 onRestoreInstanceState() 也不会被回调，取而代之的是 onConfigurationChanged() 会被回调，
可以在这个方法中做相应的处理

## 常见回调过程

Activity 在操作过程中有如下常见的生命周期过程:

* 启动 Activity 然后 finish()：

    onCreate() -> onStart() -> onResume() -> onPause() -> onStop() -> onDestory()

* 打开新的非透明主题的 Activity、Home 键返回桌面、来电、锁屏、查看最近任务，然后返回该 Activity：

    onResume() -> onPause() -> onStop() -> onRestart() -> onStart() -> onResume()

* 打开透明主题的 Activity、弹窗、Android 7.0(API 24)及其以后版本的多窗口模式下失去焦点，然后返回该 Activity：

    onResume() -> onPause() -> onResume()

* 打开最近任务滑动强杀：onDestory() 不一定被调用，不同的机型实现不一样
* 下拉通知栏不会触发生命周期方法
* 未配置 android:configChanges 属性系统配置发生变动或内存不足，Activity 异常终止并重建：

    onResume() -> onPause() -> onStop() -> onDestory -> onCreate() -> onStart() -> onResume，其中 onSaveInstanceState() 在 onStop() 之前回调，与 onPause() 时序不一定，onRestorInstoreState() 会在 onStart() 与 onResume() 之间回调

* 配置了 android:configChanges 属性系统配置发生变动时：

    onConfigurationChanged()

* 启动使用栈内复用模式或栈顶复用模式的处于栈顶的 Activity：

    onPause() -> onNewIntent() -> onResume()

* 启动使用栈内复用模式且处于非栈顶的 Activity：

    onNewIntent -> onRestart() -> onStart() -> onResume()

# 启动模式

Activity 的启动模式定义了新 Activity 在任务栈中创建的方式，可以通过下面两种方式指定：

* Using the manifest file
* Using Intent flags

这两种方式都能指定 Activity 的启动模式，但这两者之间还是有区别的。首先，第二种方式的优先级高于第一种，两者同时存在时，以第二种方式为准；其次，两种方式在限定范围上不同，比如，第一种方式无法为 Activity 指定
FLAG_ACTIVITY_CLEAR_TOP Flag，第二种无法为 Activity 指定 singleInstance 模式

> 每个 Task 都有对应的 Back Stack，下面将 Task 称之为任务栈

清单文件中可以使用 Activity 标签的 android:launchMode 属性指定启动模式，共有四种模式：

## standard

标准模式，默认启动模式。这种模式下，每启动一次 Activity 都会重新创建一个新的实例，而不管任务栈中这个实例是否存在，实例会被放入启动了它的 Activity 所在的任务栈。每个 Activity 实例可以属于不同的任务栈，每个任务栈也可以有多个 Activity 实例。

## singleTop

栈顶复用模式。在这种模式下，若新启动的 Activity 未处于栈顶，则会重新创建实例；若新启动的 Activity 如果已经处于栈顶，则不会被重新创建，而是会按下列顺序回调栈顶 Activity 的方法：

    onPause() -> onNewIntent() -> onStop()

## singleTask

栈内复用模式。在这种模式下，只有当新启动 Activity 在想要启动的栈内不存在时才会创建实例，如果存在那么无论启动多少次都不会重新创建实例。该模式默认具有 Clear Top 的效果，当 Activity 已经在栈内存在且处于栈顶时，
启动该 Activity 时会按下列顺序回调其方法：

    onPause() -> onNewIntent() -> onStop()

当不在栈顶时启动该 Activity，系统会将位于该 Activity 之上的所有 Activity 出栈，并按下列顺序回调已存在 Activity 的方法：

    onNewIntent() -> onRestart() -> onStart() -> onResume()

## singleInstance

单实例模式。这是一种加强的栈内复用模式，除了具有栈内模式所有的特性外，使用该模式的 Activity 只能独立地存在于一个任务栈中，该任务栈中不能有其他的 Activity 存在。当使用该模式的 Activity 启动其他 Activity 时，
如果其他 Activity 未指定 android:taskAffinity 属性，则会被放入默认的任务栈，如果其他 Activity(包括默认模式) 指定了 android:taskAffinity 属性，则会被放入对应的任务栈。单实例模式的 Activity 启动其他 Activity 时相当于使用了 FLAG_ACTIVITY_NEW_TASK  flag，所以其他 Activity 的 android:taskAffinity 属性也会生效

当 Activity 使用了 android:launchMode="singleTask" 属性，且已经存在于后台任务栈内，此时处于前台任务栈的 Activity 去启动该 Activity，那么后台任务栈会被切换到前台(而不是加入到之前的前台任务栈)并置于当前任务栈的顶部，此时点击回退键，栈中的 Activity 会一一出栈，如下图所示：

![activity_backstack_singletask_multiactivity.png](http://otg3f8t90.bkt.clouddn.com/2018/3/16/activity_backstack_singletask_multiactivity.png)

上图中如果 Activity X 也使用了 android:launchMode="singleTask"，Activity 2 去启动 X，那么 X 所在的任务栈会被切换到前台且只剩下 X，整个列表会变为 1 -> 2 -> X(1 和 2 在同一个任务栈, X 在一个任务栈)，Activity Y 被出栈，此时点击回退键，栈中的 Activity 会一一出栈。

任务栈分为前台任务栈和后台任务栈，前台任务栈和后台任务栈可以同属于一个应用，也可以分属于不用的应用，上图示例的一种具体场景是，同一个应用内，Activity 1 启动了 2，2 又启动了 X，X 设置了 android:launchMode="singleTask" 和 android:taskAffinity 两个属性，X 又启动了 Y，此时点击 Home 键返回桌面，从桌面再次进入应用，此时前台任务栈是 Activity 1、2 所在的任务栈，此时再启动 Y 就复现了图中的过程，如果不是启动 Y 而是启动 X，则复现了第二种情况。

# Intent flags

startActivity() 启动 Activity 时，可以使用 Intent.addFlags() 来改变新 Activity 的默认行为，有下面几个常见的 flags：

## FLAG_ACTIVITY_NEW_TASK

[官方文档](https://developer.android.com/guide/components/activities/tasks-and-back-stack.html)指出，使用标记位 FLAG_ACTIVITY_NEW_TASK 的作用与使用 android:launchMode="singleTask" 的作用是相同的，
但其实是不正确的，FLAG_ACTIVITY_NEW_TASK 标记位用于给新启动的 Activity 指定所需的任务栈，通常与 android:taskAffinity 属性配合使用，如果不指定该属性，默认使用应用包名对应的任务栈。
当只使用了该 flag 启动 standard 模式的并指定了 android:taskAffinity 属性的 Activity  时，有下面几种不同的情况:

* Activity 所需的任务栈不存在，那么会创建对应的任务栈，并将 Activity 入栈(此时为 root Activity)，再次启动该 Activity 时不会有任何相应，也不会回调任何方法

* Activity 所需的任务栈存在且处于前台，且 Activity 没有处于栈顶(不是 root Activity)，如果再次启动该 Activity，会创建新的实例

* Activity 所需任务栈存在且处于后台，那么从前台再次启动该 Activity 时，如果 Activity 处于后台任务栈的栈底(root Activity)，那么会将后台任务栈按照原有顺序切换到前台，但并不会创建新的实例，也不会回调已存在实例的任何方法；如果 Activity 处于后台任务栈的非栈底(不是 root Activity)，那么会将后台任务栈切换到前台，并创建新的实例

可以参看下面的资料：

> * [Why Behaviours are different ?- android:launchMode=“singleTask” , android:taskAffinity=“” And Intent.FLAG_ACTIVITY_NEW_TASK](https://stackoverflow.com/questions/46427805/why-behaviours-are-different-androidlaunchmode-singletask-androidtaskaf)
> * [FLAG_ACTIVITY_NEW_TASK clarification needed](https://stackoverflow.com/questions/9772927/flag-activity-new-task-clarification-needed)

当使用 ApplicationContext 去启动标准模式的 Activity 时会出现运行时异常，这是因为非 Activity 的 Context 并没有任务栈，需要给 Intent 指定 FLAG_ACTIVITY_NEW_TASK flag，这样启动时就会为 Activity 创建一个新的任务栈

> root Activity 指的是任务栈内第一个 Activity(包括 finish() 掉的 Activity)，例如 SplashActivity 是主 Activity，SplashActivity -> finish() 掉 -> MainActivity，那么该任务栈中的 root Activity 指的是 SplashActivity

## FLAG_ACTIVITY_SINGLE_TOP

Intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP) 的作用与清单文件中设置 android:launchMode="singleTop" 属性的作用一致

## FLAG_ACTIVITY_CLEAR_TOP

当 Activity 的实例已经存在于相应的任务栈时，使用 Intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP) 启动 Activity，如果该 Activity 使用了 android:launchMode="singleTop" 或 android:launchMode="singleTask" 模式(singleTask 模式其实已经具备 Clear Top 的效果，再使用该 flag 其实是多余的)，那么不会重新创建该 Activity 的实例，而是会将该 Activity 之上的其他 Activity 出栈，并回调自身的  onNewIntent() 方法；如果该 Activity 使用了 android:launchMode="standard" 模式，那么该 Activity 自身及其之上的 Activity 都会被出栈，同时创建该 Activity 新的实例入栈。所以该 flag 通常不单独使用，而是与其他 flag 组合使用：

* Intent.addFlags(FLAG_ACTIVITY_NEW_TASK|FLAG_ACTIVITY_SINGLE_TOP|FLAG_ACTIVITY_CLEAR_TOP) 的作用与使用属性 android:launchMode="singleTask" 一致
* Intent.addFalgs(FLAG_ACTIVITY_SINGLE_TOP|FLAG_ACTIVITY_CLEAR_TOP) 与 android:launchMode="singleTask" 的作用相相似，但不完全相同，这种实现方式指定的 android:taskAffinity 属性无效。

# TaskAffinity

android:taskAffinity 属性用于给 Activity 指定任务栈。默认情况下，一个应用只有一个任务栈，该任务栈的名字为应用的包名，可以为每个 Activity 都指定 android:taskAffinity 属性，该属性值不能与包名相同，否则相当于没有指定，且该属性值为字符串但是对格式有要求，该字符串必须包含包名分隔符 “.”。同一个应用可以有不用的任务栈，不用的应用也可以使用同一个任务栈，该属性有使用场景的限制，并不是在任何情况下都能生效，具体情况如下：

* 对于使用 standard 模式的 Activity，设置 android:taskAffinity 属性无效，启动时 Activity 会被放入启动它的 Activity 所在的任务栈
* 对于使用 singleTask 或 singleInstance 模式的 Activity，或者使用 Intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK) 启动的 Activity，设置 android:taskAffinity 属性都会生效(如果未指定，默认是应用包名)，Activity 会被放入特定的任务栈，然后按照具体的模式或者 flag 表现相应的特性。
* 对于使用 android:allowTaskReparenting="true" 属性的 Activity，设置 android:taskAffinity 属性也会生效。当设置 android:allowTaskReparenting="true" 时，如果 Activty 被放入启动了它的 Activity 所在的栈中(不是 taskAffinity 指定的任务栈)，那么当该 Activity 所需的任务栈(也就是 taskAffinity 指定的任务栈)被切换到前台是，该 Activity 会被移动到它所需的任务栈中。举个例子说明一下更清楚，如果应用 A 启动了应用 B 的某个 Activity(该 Activity 设置了属性 android:allowTaskReparenting="true")，且被放入应用 A 的任务栈中，点击 Home 键回到桌面，然后从桌面进入应用 B,这时并不是启动了 B 的主 Activity，而是重新显示了被应用 A 启动了的 Activity，也就是说该 Activity 从应用 A 的任务栈转移到了应用 B 的任务栈

> 当同一个应用有多个任务栈时，可以在最近任务(Recent Screen)切换不同的任务栈

# Clearing the back stack

默认情况下，当任务栈在后台时间过久，系统会将任务栈内除了栈底(root Activity)之外的所有 Activity 都出栈，当用户再次将该任务栈切换到前台时，只有栈底的 Activity 会恢复。在清单文件中给 Activity 标签设置下列属性可以更改默认行为：

## android:alwaysRetainTaskState

该属性值只对任务栈的 root Activity 生效，如果任务栈的 root Activity 将该属性设为 "true"，任务栈会保存栈内所有 Activity，无论处于后台多长时间，将任务栈切换到前台时所有的 Activity 都会被恢复

## android:clearTaskOnLaunch="true"

该属性值也是只对任务栈的 root Activity 生效，作用于 android:alwaysRetainTaskState 正好相反，如果任务栈的 root Activity 将该属性设为 "true"，任务栈从后台切换到前台时只会恢复任务栈栈底的 Activity，其他 Activity 都会被出栈。例如应用的启动过程为：SplashActivity -> finish() -> MainActivity，其中 SplashActivity 为主 Activity(需要为 SplashActivity 设置该属性而不是 MainActivity)，那么再启动其他 Activity 然后切换到后台，再次返回应用时，任务栈中只剩下 MainActivity

## android:finishOnTaskLaunch="true"

该属性值可以作用于任意的 Activity(但是对任务栈栈底的 Activity 不生效)，与上边两个属性不同的是这个属性只作用于单个的 Activity 而不是整个任务栈，当该属性值设为 "true" 时，将任务栈从后台切换到前台，对应的 Activity 已经被出栈

# 常见面试题

# Ref

* [Activities](https://developer.android.com/guide/components/activities/index.html)
* [Activity](https://developer.android.com/reference/android/app/Activity.html)
