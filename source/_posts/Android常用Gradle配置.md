---
title: Android常用Gradle配置
date: 2016-08-14 00:13:15
tags:
---
## 1.gradle简介
Gradle是以Groovy为基础，面向java应用，基于DSL语法的自动化构建工具。是google引入，替换ant和maven的新工具，其依赖兼容maven和ivy。
Gradle 里的任何东西都是基于这两个基础概念:

- projects (项目)
    - 每一个构建都是由一个或多个 projects 构成
    - 一个 project 是由一个或多个 tasks 构成
- tasks (任务)
    - 一个 task 代表一些更加细化的构建
    
Gradle中每一个待编译的工程都是一个Project，一个具体的编译过程是由一个一个的Task来定义和执行的。在Android Studio工程中，每一个Library和每一个App都是单独的Project。根据Gradle的要求，每一个Project在其根目录下都需要有一个build.gradle。build.gradle文件就是该Project的编译脚本，类似于Makefile。当需要同时编译多个Project时，需要在AS工程的根目录创建 settings.gradle文件，添加如下内容：
```gradle
//表示当前工程有三个Project，即在AS工程中有三个Module
include ':app', ':data' , ':domain'
```

## 2.项目结构
使用Android Studio创建的项目通常具有如下结构：
```
App
├── build.gradle
├── settings.gradle
├── gradle/wrapper
    ├── wrapper
        ├── gradle-wrapper.jar
        ├── gradle-wrapper.properties
├── gradle
├── gradle.bat
└── app
    ├── build.gradle
    ├── build
    ├── libs
    └── src
        └── main
            ├── assets
            ├── java
            │   └── com.package.myapp
            └── res
                ├── drawable
                ├── layout
                └── etc.
```
## 3.gradle配置
执行gradle的时候，gradle首先是按顺序解析各个gradle文件，所以在添加配置的时候，要注意顺序，被引用的要定义在前边。
### 1.加载插件
每一个build.gradle文件都会转换成一个Project对象。在Gradle术语中，Project对象对应的是Build Script。Project包含若干Tasks。由于Project对应具体的工程，所以需要为Project加载所需要的插件，比如为Java工程加载Java插件。其实，一个Project包含多少Task往往是插件决定的。在AS工程中，每个Module对应一个Project，在project的builde.gradle文件的首行要加载对应的插件，在Android工程中常用的插件如下：
```gradle
//如果是编译Android APP，则加载此插件，一个AS工程只有一个该类型的project
apply plugin: 'com.android.application'
//如果是编译Android Library，则加载此插件
apply plugin: 'com.android.library'
//如果编译纯Java Library，则加载此插件
apply plugin: 'java'
```
apply是一个函数，plugin表示参数类型，后边的内容表示参数值。
### 2.配置敏感参数
gradle.properties文件适合配置IDE的属性，当然也适合配置项目的关键/敏感参数，因为它将运行在Incubating parallel mode，也是属于默认的gitignore，这样敏感信息（key账号密码，appkey等）就不会被push到git了，需要注意的是，属性中有中文的话，记得转成unicode来显示，不然可能引发一些莫名的错误
可以把签名keystore的信息，服务端的endpoint，第三方服务的appkey，ide的配置信息放在gradle.properties ：
```properties
KEY_ALIAS=
KEYSTORE_PASSWORD=
KEY_PASSWORD=
umeng_appkey_product= 
umeng_appkey_dev=
deepshare_appid=
bugly_appid_product=  
bugly_appid_dev=
rong_appkey=
endpoint_product=
endpoint_dev=
app_name=
```
这些属性通常会在App/app/builde.gradle中被引用，在signingConfigs{}中直接通过键名引用，
### 3.配置签名信息
在App/app/build.gradle中添加下面代码来实现签名信息的配置：
```gradle
//定义局部变量，定义密匙存放位置
def keystore = file('../../keystore/wptdxii.jks')
signingConfigs {
    debug {
        //KEY_ALIAS，KEY_PASSWORD，STORE_PASSWORD引用自上边gradle.proerties中的定义
        keyAlias KEY_ALIAS
        keyPassword KEY_PASSWORD
        storePassword STORE_PASSWORD
        //引用局部变量
        storeFile keystore
    }

    release {
        keyAlias KEY_ALIAS
        keyPassword KEY_PASSWORD
        storePassword STORE_PASSWORD
        storeFile keystore
    }
}
```
### 4.自定义BuildConfig字段
在Gradle脚本中默认的debug和release两种模式BuildCondig.DEBUG字段分别为true和false，而且不可更改。该字段编译后自动生成，在Studio中生成的目录在 app/build/source/BuildConfig/Build Varients/package name/BuildConfig 文件下。
当需要自定义BuildConfig字段时，在buildType{}中定义：
```gradle
buildTypes {
        debug {
          
            buildConfigField "boolean", "API_ENV", "true"
        }
        release {
            buildConfigField "boolean", "ENDPOINT", "false"
        }
    }
```
### 5.配置构建类型
在App/app/build.gradle中（注意signingConfigs {}要声明在buildTypes{}前边）添加：
```gradle
buildTypes {
        debug {
            versionNameSuffix "-debug"
            minifyEnabled false
            shrinkResources false
            zipAlignEnabled false
            signingConfig signingConfigs.debug
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            buildConfigField "String", "ENDPOINT", "\"${endpoint_product}\""
            resValue "string", "umeng_appkey", "${umeng_appkey_dev}"
            resValue "string", "deepshare_appid", "${deepshare_appid}"
            resValue "string", "bugly_appid", "${bugly_appid_dev}"
            resValue "string", "rong_appkey", "${rong_appkey}"
            resValue "string", "channel", "dev"
            resValue "string", "op_app_name", "dev_${app_name}"
        }
        release {
            //版本后缀加上-debug
            versionNameSuffix "-release"
            //是否开启混淆
            minifyEnabled false
            //是否删除无效资源
            shrinkResources true
            //是否zip对齐
            zipAlignEnabled true
            //签名所用配置文件，在上边signingConfigs {}中定义
            signingConfig signingConfigs.release
            //混淆所用文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //$后边的值从gradle.properties中读取
            //在Java中通过BuildConfig.ENDPOINT调用
            buildConfigField "String", "ENDPOINT", "\"${endpoint_product}\""
            //在xml文件中通过R.string.key调用
            resValue "string", "umeng_appkey", "${umeng_appkey_product}"
            resValue "string", "deepshare_appid", "${deepshare_appid}"
            resValue "string", "bugly_appid", "${bugly_appid_product}"
            resValue "string", "rong_appkey", "${rong_appkey}"
            resValue "string", "channel", "product"
            resValue "string", "op_app_name", "${app_name}"
        }
    }
```

- 使用 \${key}引用gradle.properties中定义的属性值
- 通过  buildConfigField "String", "ENDPOINT", "\"\${endpoint_product}\"" 定义的字段，在Java代码中通过BuildConfig.ENDPOINT引用
- 通过 resValue "string", "umeng_appkey", "${umeng_appkey_dev}" 定义的字段，可以在xml文件中通过 R.string.umeng_appkey引用

### 6.统一依赖管理
在gradle中有三种依赖方式：
```gradle
dependencies {

     //依赖libs目录下的jar包
     compile fileTree(include: ['*.jar'], dir: 'libs')
     //远程依赖
     compile 'com.android.support:support-v4:24.1.1'
     //依赖module
     compile project(':domain')
}
```
当多个module有相同的依赖时，如果依赖的远程库升级后，需要更改不同module下的配置，比较麻烦，这是可以统一依赖管理。
在App/build.gradle中添加：
```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

def supportVersion = "24.1.1"
def retrofitVersion = "2.0.2"
def okHttpVersion = '3.4.1'
//def rxBindingVersion = '0.4.0'
//def daggerVersion = "2.4"
project.ext {
    applicationId = "com.github.moduth.petlover"
    buildToolsVersion = "24.0.1"
    compileSdkVersion = 23
    minSdkVersion = 16
    targetSdkVersion = 22
    versionCode = 1
    versionName = "1.0.0"
    abortOnLintError = false
    checkLintRelease = false
    useJack = false

    javaVersion = JavaVersion.VERSION_1_8

    libSupportV4 = "com.android.support:support-v4:${supportVersion}"
    libSupportAppcompatV7 = "com.android.support:appcompat-v7:${supportVersion}"
    libSupportDesign = "com.android.support:design:${supportVersion}"
    libRecyclerViewV7 = "com.android.support:recyclerview-v7:${supportVersion}"
    
    libRetrofit = "com.squareup.retrofit2:retrofit:${retrofitVersion}"
    libRetrofitConverterGson = "com.squareup.retrofit2:converter-gson:${retrofitVersion}"
//    libRetrofitConverterJackson = "com.squareup.retrofit2:converter-jackson:${retrofitVersion}"
    libRetrofitAdapterRxJava = "com.squareup.retrofit2:adapter-rxjava:${retrofitVersion}"
    
    libOkHttpLoggingInterceptor = "com.squareup.okhttp3:logging-interceptor:${okHttpVersion}"
    
//    libGson = "com.google.code.gson:gson:2.6.2"
    libFastJson = "com.alibaba:fastjson:1.1.52.android"
    
    libRxJava = "io.reactivex:rxjava:1.1.9"
    libRxAndroid = "io.reactivex:rxandroid:1.2.1"
    
    libGlide = "com.github.bumptech.glide:glide:3.7.0"
    
//    libEventBus = "org.greenrobot:eventbus:3.0.0"
    
//    libJavaxAnnotation = "javax.annotation:jsr250-api:1.0"
//    libJavaxInject = "javax.inject:javax.inject:1"
}
```
原则上是有多处引用时，将版本号提出去，否则还是定义在一起。

配置过后有的子项目或者所有的modules都可以从这个配置文件里读取内容，
比如在App/app/build.gradle可以这么引用依赖：
```java
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile rootProject.ext.libSupportV4
    compile rootProject.ext.libSupportAppcompatV7
    compile rootProject.ext.libSupportDesign
    compile rootProject.ext.libRecyclerViewV7
}
```
这样当依赖的版本升级时，只需要在根目录的配置文件中统一更改即可。
### 7.三方库文件冲突
当第三方库在导入时候的一些声明文件会产生冲突冲突，可以添加如下配置：
```gradle
packagingOptions {
  exclude 'META-INF/LICENSE'
  exclude('META-INF/LICENSE.txt')
  exclude('META-INF/NOTICE.txt')
  exclude 'META-INF/NOTICE'
  exclude 'META-INF/DEPENDENCIES'
  // umeng推送的jar包含有的okio库跟okhttp的okio库冲突
  exclude 'META-INF/maven/com.squareup.okio/okio/pom.xml'
  exclude 'META-INF/maven/com.squareup.okio/okio/pom.properties'
}
```
### 8.指定JDK版本
当需要指定编译使用的jdk版本时，添加如下配置：
```gradle
compileOptions {
  sourceCompatibility JavaVersion.VERSION_1_7
  targetCompatibility JavaVersion.VERSION_1_7
}
```
### 9.打包管理
在android{}标签外部添加如下定义：
```gradle
//打包时使用
def getDate() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC")) // 注意时区
}

// 获取当前git的Revision，打包时使用
def getRevision() {
    if (!System.getenv('CI_BUILD')) {
        return 0
    }
    return ext.hash = 'git rev-parse --short HEAD'.execute().text.trim()
}
```
定义的方法只能在根目录定义，且要定义在被引用之前，然后在android{}中添加如下配置：
```gradle
 //打包管理，渠道名
    productFlavors {
        rc_open {}
        rc_360 {}
        rc_yingyongbao {}
        rc_baidu {}
        rc_91 {}
        rc_wandoujia {}
        rc_anzhuo {}
        rc_xiaomi {}
        rc_meizu {}
        rc_oppo {}
        rc_huawei {}
        rc_weibo {}
        rc_staging {}
        rc_product {}
    }
      productFlavors.all { flavor ->
//        // 这里只是方便友盟统计每个渠道的数据
//        flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
//    }
    // 修改打包后APK的文件名
    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def oldFile = output.outputFile
            if (variant.buildType.name.equals('release')) {
                // 输出apk名称为yat3s_v1.0_2016-04-12_yingyongbao_a23f2e1.apk
                def releaseApkName = 'ap-v' + defaultConfig.versionName + '_' + getDate() + '_' + variant.productFlavors[0].name + "_" + getRevision() + '.apk'
                output.outputFile = new File(oldFile.parent, releaseApkName)
            }
            if (variant.buildType.name.equals('debug')) {
                // Do nothing
            }
        }
    }
```
- 多渠道打包用的是Gradle的productFlavors
- git的Revision 方便你管理你的release和tag
- 打包的时候可以用Jenkins来自动Build包

### 10.gradle加速
在gradle.propertie中添加如下配置，开启守护线程：
```properties
org.gradle.jvmargs=-Xmx2048m -XX\:MaxPermSize\=512m -XX\:+HeapDumpOnOutOfMemoryError -Dfile.encoding\=UTF-8
org.gradle.daemon=true
org.gradle.configureondemand=true
org.gradle.parallel=true
android.useDeprecatedNdk=true
```
### 11.Eclipse导入AS
在Eclipse项目路径下创建build.gradle文件：
```gradle
apply plugin: 'com.android.application'
buildscript {
  repositories {
      jcenter() 
  }
  dependencies {
      classpath 'com.android.tools.build:gradle:x.x.x'
  }
}
android {
  compileSdkVersion x
  buildToolsVersion "x.x.x"
  sourceSets {
      main {
          manifest.srcFile 'AndroidManifest.xml'
          java.srcDirs = ['src']
          resources.srcDirs = ['src']
          aidl.srcDirs = ['src']
          renderscript.srcDirs = ['src']
          res.srcDirs = ['res']
          assets.srcDirs = ['assets']
          jniLibs.srcDirs = ['libs']
      }
      androidTest.setRoot('tests')
  } 
}
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
### 12.arr引用
aar是android中特有的归档文件，既包含字节码文件也包含android的资源文件等。
**导出arr**
首先Android Library项目的gradle脚本只需要在开头声明：
```gradle
apply plugin: 'com.android.library'
```
**执行命令**
```gradle
//win
gradle assembleRelease
```
然后在build/outputs/aar 文件夹里生成aar文件
**引用arr**
aar文件放在一个文件目录内，如放在libs目录内，在需要引用该arr文件的module中，如在app的build.gradle文件添加如下内容：
```gradle
repositories {
    flatDir {
        dirs 'libs' //this way we can find the .aar file in libs folder
    }
}
```
在build.gradle文件中添加依赖：
```gradle
compile(name:'test', ext:'aar') //arr文件放到libs目录下，'test'为arr文件名
```
### 13.Module arr依赖
## 4.gradle命令
在Win下使用 gradle xxx
在Mac下使用 ./gradlew xxx
### 1.查看工程信息
查看当前项目project数量，也就是settings.gradle文件中包含的project
```gradle
//win
gradle projects

//mac
./gradlew projects
```
### 2.查看任务信息
查看某个project的task信息
```gradle
//win
gradle <project-name>:tasks

//mac
./gradle <project-name>:tasks
```
如果进入了project的目录，则不需要加project-name
```gradle
//win
gradle tasks

//mac
./gradle tasks
```
### 3.执行任务
```gradle
//win
gradle task-name

//mac
./gradlew task-name
```
### 4.gradle版本号
```gradle
//win
gradle -v

//mac
./gradlew -v
```
### 5.清除build文件夹
```gradle
//win
gradle clean

//mac
./gradlew clean
```
### 6.检查依赖编译打包
打包所有环境的包：
```gradle
//win
gradle build

//mac
./gradlew build
```
根据build.gradle配置打对应的包：
```gradle
//win
gradle assembleDebug/assembleRelease

//mac
./gradlew assembleDebug/assembleRelease
```
Apk路径: App/app/build/outputs/apk
