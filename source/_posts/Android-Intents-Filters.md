---
title: Android Intent
date: 2017-04-08 11:07:04
tags: [Intent, Intent Filter]
categories: Android
---

Intent 相关介绍和 intent-filter 匹配规则

<!-- more -->

# Intent

Intent 用于启动[应用组件](https://developer.android.com/guide/components/fundamentals.html#Components)并传递消息，有两种类型：

* 显式 Intent(explicit intent)：明确指定被启动对象的组件信息，包括包名和类名，主要用于启动自己 app 内的组件
* 隐式 Intent(implicit intent)：不需要明确指定被启动对象的组件信息，只要匹配了 intent-filter 设定的规则就可启动对应的组件，主要用于启动其它 app 内的组件，当 Intent 与多个 app 的组件的 intent-filter 都匹配的话，会弹窗让用户选择

> * 原则上一个 Intent 不应该既是显示的又是隐式的，如果二者共存的话以显式调用为主，且不再匹配 intent-filter
> * 一个组件如果未设置 intent-filter，则只能被显式启动
> * 为了安全，通常应该只使用显式 Intent 启动 Service，不给清单文件中的 service 标签设置 intent-filter，因为 Service 没有用户界面，使用隐式启动时用户并不知道哪个应用的哪个 Service 会响应该 Intent。Android 5.0(API level 21)及其之后，如果使用隐式 Intent bindService()，系统将会抛出异常

清单文件中每个 activity 标签可以有多个 intent-filter，一个 Intent 只要能匹配任何一组 intent-filter 就可以成功的隐式启动对应的 Activity。intent-filter 针对的是 Intent 的隐式调用，对显式调用没有意义。一个 intent-filter 可以嵌套三种元素：

* action
* data
* category

而构建 Intent 的时候可以包含多个信息，包括:

* Component name
* Action
* Category
* Data
* Extras
* Flags

下面分别介绍

## component name

用于指定被启动组件的类名，该属性是可选的，如果不指定，Intent 会根据其它信息(如 action、data、category)隐式调用相应的组件。设定该属性有下面几种方法:

* Intent 构造器
* Intent.setClasss()
* Intent.setClassNanme
* Intent.setCompoent()

## action

用于指定 Intent 的行为，通常隐式启动 Activity 时会使用该属性。系统提供了一些通用的 action，详细参看：[Intent](https://developer.android.com/reference/android/content/Intent.html)。设定该属性的方法有：

* Intent 构造器
* Intent.setAction()

Intent 只能设定一个 action，多次设定会被最后一个覆盖。如果不使用系统的预设值，可以自定义字符串(区分大小写)，当自定义时注意使用包名作为前缀，以示区分:

```java
static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";
```

相应的，intent-filter 可以声明零个或者多个 action 元素。action 的匹配规则是：

* Intent 指定 action 且和 intent-filter 其中一个 action 相同，则可匹配成功
* intent-filter 没有指定 action，则所有的 Intent 匹配失败
* Intent 未指定 action ，无论 intent-filter 是否指定 action，都匹配失败(官方文档指出若 Intent 未指定 action，如果 intent-filter 指定至少一个 action，仍会匹配成功，但经过测试这是不正确的)

## category

该属性主要用于隐式启动 Activity，如果不使用系统的预设值，可以自定义字符串(区分大小写)。一个 Intnet 可以添加多个 category，可以通过下列方法添加：

* Intent.addCategory()

相应的，intent-filter 可以声明零个或者多个 category 元素。category 的匹配规则是：

* 如果 activity 想要可以被隐式启动，必须在 intent-filter 中声明 `<category android:name="android.intent.category.DEFAULT" />`，如果未声明，则匹配失败。因为通过 startActivity() 或者 startActivityForResult() 启动 Activity 时，Intent 会被默认添加 Intent.CATEGORY_DEFAULT 的 category，intent-filter 必须有与之对应的 category 才能匹配成功，也就是如果 intent-filter 声明了 `<category android:name="android.intent.category.DEFAULT" />`，Intent 不添加 category 也可以匹配成功
* Intent 可以添加多个 catetory，每个 category 都与 intent-filter 中的 category 匹配时才能匹配成功(但不要求 intent-filter 中的每个 category 都与 Intent 的 category 匹配，可以有多余的 category)

## Data

用于传递的数据，由 MIME Type(Multipurpose Internet Mail Extensions Type) 和 URI(Uniform Resource Identifier) 两部分组成，MIME Type 用于指定媒体类型，URI 用于标识对应的资源，URI 的结构如下：

```uri
<scheme>://<authority>/[<path>|<pathPrefix>|<pathPattern>]
// 或者
// authority 由 host:port 组成
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
```

具体的示例如下：

```uri
content://com.example.project:200/folder/subfolder/etc
```

URI 结构参数说明:

* scheme：URI 的模式，如 http、file、content 等
* host: URI 的主机名，如 www.google.com
* port: URI 主机端口号
* path|pathPattern|pathPrefix：这三个参数表示路径信息。其中 path 表示完整的路径信息；pathPattern 也表示完整的路径信息，但可以包含通配符 “*”，“*” 表示零个或者多个任意字符，需要注意的是，由于正则表达式的规范，如果想表示真实的字符串，那么 “\*” 要写成 “\\\*”，“\” 要写成 “\\\\\\\\”；pathPrefix 表示路径的前缀信息

URI 结构参数之间有下列的依赖关系：

* 未指定 scheme，host 会被忽略，URI 无效
* 未指定 host，port 会被忽略，URI 无效
* 未指定 scheme 和 host，path 会被忽略，URI 无效

通过下面的方法可以设置这两部分：

* Intent.setData() // 设置 URI
* Intent.setType() // 设置 MIME Type
* Intent.setDataAndType()

> Intent.setData() 和 Intent.setType() 会清除对方的值，所以如果需要同时设定 MIME Type 和 URI 需要使用 Intent.setDataAndType()

相应的，intent-filter 中的 data 元素也由 MIME Type 和 URI  两部分组成，但与 action 不同的是，data 有如下两种特殊的写法，其作用一样：

```xml
<!-- 写法二 -->
<intent-filter>
    <data android:scheme="file" />
    <data android:host="www.google.com" />
</intent-filter>

<!-- 写法二 -->
<intent-filter>
    <data android:scheme="file" android:host="www.google.com">
</intent-filter>

```

> intent-filter 可以指定多个 data 元素，也就是 mimeType、scheme、host 等都可以指定多个

data 的匹配规则如下：

* Intent 和 intent-filter 都不指定 MIME Type 和 URI，则匹配成功
* Intent 设定了 URI 未设定 MIME Type，intent-filter 只有设定匹配的 URI 同时不设定 mimeType，才能匹配成功
* Intent 设定了 MIME Type 未设定 URI，intent-filter 只有设定相同的 mimeType 并且不设定 URI，才能匹配成功
* Intent 同时设定了 URI 和 MIME Type，intent-filter 设定了匹配的 URI 和 相同的 mimeType，则匹配成功
* intent-filter 只设定了 mimeType 未设定 URI，Intent 只用设定相同的 mimeType，且 URI 的 scheme 必须是 content 或者 file，才能匹配成功(因为 intent-filter 默认添加了 scheme 为 contnet 和 file 的 data 元素))

> Intent 设定的 MIME Type 和 intent-filter 设定的 mimeType 相同即可匹配

上面的匹配规则用到的 URI 的匹配规则，具体如下：

* intent-filter 只设定了 scheme，Intent 设定含有相同 scheme 的 URI 即可匹配
* intent-filter 未设定 path，而只设定了 scheme 和 authority，Intent 只需要设定含有相同的 scheme 和 authority 即可匹配成功，而忽略 path 的匹配
* intent-filter 设定了 scheme、authority 和 path，Intent 需要设定含有相同 scheme、authority 和 path 的 URI 才能匹配成功
* intent-filter 未设定 URI，Intent 只有设定 scheme 为 content 或 file 的 URI，才能匹配成功

## Extras

用来在组件间传递键值对数据，可以传递多个键值对，有如下方法:

* Intent.putExtra()
* Intnet.putExtras() // 需要先将一个或多个键值对放入 Bundle 对象中

键值对的键值可以自定义，通常会带上包名以示区分：

```java
static final String EXTRA_GIGAWATTS = "com.example.EXTRA_GIGAWATTS";
```

> 使用 Intent 隐式调用时，不能向其它 app 传递序列化的数据(Parcelable or Serializable data)，因为如果其它 app 没有权限从 Bundle 中获取数据时，会抛出 RuntimeException

# Tips & Tricks

## MainActivity

对于程序的入口 Activity(即 MainActivity)，其必须声明下面的 intent-filter：

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

## ResolveActivity

隐式启动组件时，如果没有匹配的组件，应用会直接 crash，可以通过下面方法先判断是否有匹配的组件：

```java
// 使用隐式 intent 调用该方法，如果返回 null，表示没有匹配组件
Intent.resolveActivity(PackageManager pm)

// 或

// 如果返回 null，表示没有匹配组件
// 第二个参数一般设置为 PackageManager.MATCH_DEFAULT_ONLY
// 返回最佳匹配的组件
PackageManager.resolveActivity(Intent intent, @ResolveInfoFlags int flags)
// 返回所有可以成功匹配的组件
PackageManager.queryIntentActivities(Intent intent, @ResolveInfoFlags int flags);
```

> 对于 Service 和 BroadcastReceiver, PackageManager 也提供了类似的方法去获取成功匹配的组件的信息

## Forcing an app chooser

当有多个组件可以响应隐式 Intent 时，会弹窗让用户选择，可以设置默认选项，但是通过下列方法可以强制每次都弹窗让用户选择：

```java
Intent sendIntent = new Intent(Intent.ACTION_SEND);
...

String title = getResources().getString(R.string.chooser_title);
// Create intent to show the chooser dialog
Intent chooser = Intent.createChooser(sendIntent, title);

// Verify the original intent will resolve to at least one activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```

# Ref

* [Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html)