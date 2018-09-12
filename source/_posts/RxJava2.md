---
title: RxJava2
date: 2016-08-21 23:40:36
tags: 
---
## 1.RxJava简介
RxJava项目地址：[RxJava](https://github.com/ReactiveX/RxJava)
RxAndroid项目地址：[RxAndroid](https://github.com/ReactiveX/RxAndroid)

RxJava 在 GitHub 主页上的自我介绍是 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库），RxJava的本质是异步。RxJava 的优势是简洁，但它的简洁的与众不同之处在于，随着程序逻辑变得越来越复杂，依然能够保持简洁，可以将复杂逻辑都能穿成一条线。
## 2.RxJava原理
### 1.Rx的观察者模式
RxJava 的异步实现，是通过一种扩展的观察者模式来实现的。

RxJava有四个基本的概念：
    
- Observable -- 被观察者
- Observer -- 观察者
- subcribe -- 订阅
- event -- 事件

Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 onNext() 之外，还定义了两个特殊的事件：

- onCompleted()
    事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志
- onError()
事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。

在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个

## 3.基本实现
### 1.创建Observer
实现Observer接口
```java
Observer<String> observer = new Observer<String>() {
    @Override
    public viod onNext(String s) {
        Log.d(tag, "Item:" + s);
    }

    @Override
    public void onCompleted() {
        Log.d(tag, "Completed");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error");
    }
}
```
RxJava 还内置了一个实现了 Observer 的抽象类：Subscriber。 Subscriber 对 Observer 接口进行了一些扩展，但基本使用方式是完全一样的
```java
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Item:" + s);
    }

    @Override
    public void onCompleted() {
        Log.d(tag, "Completed");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error");
    }
}
```
不仅基本使用方式一样，实质上，在 RxJava 的 subscribe 过程中，Observer 也总是会先被转换成一个 Subscriber 再使用。所以如果你只想使用基本功能，选择 Observer 和 Subscriber 是完全一样的。两者之间的区别主要是Subscriber增加了两个方法：

- onStart()
该方法会在 subscribe刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe()
- unsubscribe()
该方法是Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 unsubscribe() 这个方法很重要，因为在 subscribe() 之后， Observable(被观察者) 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（如 onPause() onStop() 等方法中）调用 unsubscribe()来解除引用关系，以避免内存泄露的发生
### 2.创建Observerable
Observable 即被观察者，它决定什么时候触发事件以及触发怎样的事件。 RxJava 使用 create() 方法来创建一个 Observable ，并为它定义事件触发规则：
```java
Observerable observerable = new Observerable.create(new Observerable.OnSubscribe() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        //当订阅时，下面一系列方法会被调用
        subscriber.onNext("Hello");
        subscriber.onNext("World");
        subscriber.onNext("RxJava");
        subscriber.onCompleted();
    }
});
```
onCreate()方法传入了一个 OnSubscribe 对象作为参数。OnSubscribe 会被存储在返回的 Observable 对象中，它的作用相当于一个计划表，当 Observable 被订阅的时候，OnSubscribe 的 call() 方法会自动被调用，事件序列就会依照设定依次触发（对于上面的代码，就是观察者Subscriber 将会被调用三次 onNext() 和一次 onCompleted()）。这样，由被观察者调用了观察者的回调方法，就实现了由被观察者向观察者的事件传递，即观察者模式。

create() 方法是 RxJava 最基本的创造事件序列的方法。基于这个方法， RxJava 还提供了一些方法用来快捷创建事件队列：

- just(T...)
将传入的参数依次发送出来
```java
Observerable observerable = new Observerable.just("Hello", "World", "RxJava");

// 将会依次调用：
// onNext("Hello");
// onNext("World");
// onNext("RxJava");
// onCompleted();
```
- from(T[])/from(Iterable<? extends T>)
将传入的数组或 Iterable 拆分成具体对象后，依次发送出来
```java
String[] event = new String[]{"Hello"， "World", "RxJava"};
Observerable observerable = new Observerable.from(event);

// 将会依次调用：
// onNext("Hello");
// onNext("World");
// onNext("RxJava");
// onCompleted();
```
### 3.Subscribe(订阅)
创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了：
```java
Observerable.subscribe(Observer);
//或者
Observerable.subscribe(Subscriber);
```
Observable.subscribe(Subscriber) 的内部实现是这样的（仅核心代码）：
```java
// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
public Subscription subscribe(Subscriber subscriber) {
    subscriber.onStart();
    onSubscribe.call(subscriber);
    return subscriber;
}
```
subscribe() 做了3件事：

- 调用 Subscriber.onStart()
onStart()是一个可选的准备方法,该方法会在 subscribe刚开始，而事件还未发送之前被调用，可以用于做一些准备工作
- 调用 Observable 中的 OnSubscribe.call(Subscriber)
在这里，事件发送的逻辑开始运行。从这也可以看出，在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。
- 将传入的 Subscriber 作为 Subscription 返回。这是为了方便 unsubscribe()

除了 subscribe(Observer) 和 subscribe(Subscriber) ，subscribe() 还支持不完整定义的回调，RxJava 会自动根据定义创建出 Subscriber 。形式如下：
```java
Action1<String> onNextAction = new Action1<String>() {
    //onNext()
    @Override
    public void call(String s) {
        Log.d(tag, s);
    }
}

Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    //onError()
    @Override
    public void call(Throwable e) {
        // Error Handling
    }
}

Action0 onCompletedAction = new Action0<>() {
    //onCompleted()
    @Override
    public void call() {
        Log.d(tag, "Completed");
    }
}


// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
observable.subscribe(onNextAction);
// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
observable.subscribe(onNextAction, onErrorAction);
// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```
Action0 是 RxJava 的一个接口，它只有一个方法 call()，这个方法是无参无返回值的；由于 onCompleted() 方法也是无参无返回值的，因此 Action0 可以被当成一个包装对象，将 onCompleted() 的内容打包起来将自己作为一个参数传入 subscribe() 以实现不完整定义的回调。这样其实也可以看做将 onCompleted() 方法作为参数传进了 subscribe()，相当于其他某些语言中的『闭包』。 Action1 也是一个接口，它同样只有一个方法 call(T param)，这个方法也无返回值，但有一个参数；与 Action0 同理，由于 onNext(T obj) 和 onError(Throwable error) 也是单参数无返回值的，因此 Action1 可以将 onNext(obj) 和 onError(error) 打包起来传入 subscribe() 以实现不完整定义的回调。事实上，虽然 Action0 和 Action1 在 API 中使用最广泛，但 RxJava 是提供了多个 ActionX 形式的接口 (例如 Action2, Action3) 的，它们可以被用以包装不同的无返回值的方法。
### 4.场景示例
**打印字符串数据**
```java
String[] names = ...;
Observerable.from(names)
            .subscribe(new Action1<string>(){
                @Override
                public void call(String s) {
                    Log.d(tag, s);
                }
            });
```
**由id取得图片并显示**
由指定的一个 drawable 文件 id drawableRes 取得图片，并显示在 ImageView 中，并在出现异常的时候打印 Toast 报错：
```java
int drawableRes = ...;
ImageView ImageView = ...;
Observerable.create(new Observerable.OnSubscribe<Drawable>(){
    @Override
    public void call(Subscriber<? super Drawable> subscriber) {
        Drawable drawable = getTheme().getDrawable(drawableRes);
        subscriber.onNext(drawable);
    }

}).subscribe(new SubScriber<Drawable>(){
    @Override
    public void onNext(Drawable drawable) {
        ImageView.setImageDrawable(drawable);
    }

    @Override
    public void onCompleted() {

    }

    @Override
    public void onError(Throwable e) {
        Toast.makeText().(context, "Error",Toast.LENGTH_SHOR).show();
    }
});
```
在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，如果只用上面的方法，实现出来的只是一个同步的观察者模式。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。而要实现异步，则需要用到 RxJava 的另一个概念： Scheduler 。
## 3.变换
### 1.map()
首先看map()使用的实例
```java
filePath = "image/logo/png"
// 输入类型 String
Observerable.just(filePaht)
            .map(new Func1<String Bitmap>(){// 参数类型 String
                @Override
                public Bitmap call(String filePath){
                    return getBitmapFromFile(filePath); // 返回类型 Bitmap
                }
            })
            .subsciebe(new Action1<Bitmap>(){
                @Override
                public void call(Bitmap Bitmap){// 参数类型 Bitmap
                    showBitmap(bitmap);
                }
            })
```
Func 类和 Action 类非常相似，也是 RxJava 的一个接口，用于包装含有一个参数的方法。 Func1 和 Action 的区别在于， Func1 包装的是有返回值的方法。另外，和 ActionX 一样， FuncX 也有多个，用于不同参数个数的方法。FuncX 和 ActionX 的区别在 FuncX 包装的是有返回值的方法。

map() 方法将参数中的 String 对象转换成一个 Bitmap 对象后返回，而在经过 map() 方法后，事件的参数类型也由 String 转为了 Bitmap。RxJava 不仅可以针对事件对象，还可以针对整个事件队列变换，这使得 RxJava 变得非常灵活

### 2.flatMap()
首先用示例来说明，假设有一个数据结构『学生』，现在需要打印出一组学生的名字：
```java
Student[] students = ...;
Observerable.from(students)
    .map(new Func1<Student,String>(){
        @Override
        public String call(Student student){
            return student.getName();
        }
    })
    .subscibe(new Action1<String>(){
        @Override
        public void call(String name) {
            Log.d(tag, name);
        }
    })
```
接着打印出每个学生所需要修的所有课程，首先可以这样实现
```java
Student[] students = ...;
Observerable.create(new Observerable.OnSubscribe(){
    @Override
    public void call(Subscirber<? extend Student> subscirber) {
        for(Student student : students) {
            subscriber.onNext(student);
        }
    }
})
.subscibe(new Subscirber<Student>(){
    @Override
    public void onNext(Student student) {
        List<Course> couses = student.getCources();
        for(int i = 0; i < course.size(); i++) {
            Cource cource = couses.get(i);
            Log.d(tag, cource.getName());
        }
    }
});
```
可是如果不想在 Subscriber 中使用 for 循环，而是希望 Subscriber 中直接传入单个的 Course 对象（这对于代码复用很重要），用 map() 显然是不行的，因为 map() 是一对一的转化，而现在的要求是一对多的转化。这时可以使用flatMap()来实现
```java
Student[] students = ...;
Observerable.from(students)
        .flatMap(new Func1<Student,Observerable<Cource>(){
            @Override
            public Observerable<Cource> call(Student student){
                return Observerable.from(student.getCources());
            }
        })
        .subscribe(new Action1<Cource>(){
            @Override
            public void call(Cource cource) {
                Log.d(tag, cource.getName());
            }
        })
```

由实例可以看出，flatMap() 和 map() 有一个相同点：它也是把传入的参数转化之后返回另一个对象。但需要注意，和 map() 不同的是， flatMap() 中返回的是个 Observable 对象，并且这个 Observable 对象并不是被直接发送到了 Subscriber 的回调方法中。
flatMap() 的原理是这样的：
1. 使用传入的事件对象创建一个 Observable 对象；
2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；
3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable ，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。

这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 flatMap() 所谓的 flat。

由于可以在嵌套的 Observable 中添加异步代码， flatMap() 也常用于嵌套的异步操作，例如嵌套的网络请求。示例代码（Retrofit + RxJava）：
```java
ApiFactory.getToken()// 返回 Observable<String>，在订阅时请求 token，并在响应后发送 token
    .flaMap(new Func1<String, Observerable<Messages>(){
        @Override
        public Observerable<Messages> call(String token) {
            return ApiFactory.getMessages(token);
        }
    })
    .subscribe(new Action1<Messages>(){
        @Override
        public void call(Messages messages){
            showMessages(messages);
        }
    });
```
RxJava 提供了对事件序列进行变换的支持，所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。
### 3.throttleFirst()
在每次事件触发后的一定时间间隔内丢弃新的事件。常用作去抖动过滤
### 4.变换的原理
#### 1.lift()
变换虽然功能各有不同，但实质上都是针对事件序列的处理和再发送。而在 RxJava 的内部，它们是基于同一个基础的变换方法： lift(Operator)。首先看一下 lift() 的内部实现（仅核心代码）：
```java
// 注意：这不是 lift() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
public <R> Observable<R> lift(Operator<? extends R, ? super T> operator) {
    return Observable.create(new OnSubscribe<R>() { 
        @Override
        public void call(Subscriber subscriber) {
            //observeOn()在这里控制线程  在operator 的call()方法内向下一级Subscriber发送通知
            Subscriber newSubscriber = operator.call(subscriber);
            newSubscriber.onStart();
            //这个onSubscribe对象指的是最外部Observable对象持有的OnSubscribed对象
            //subscribeOn（）控制了这里的线程切换，onSubscribe指的是上一层Observable对象持有的onSubscribe对象
            onSubscribe.call(newSubscriber);//注意这个语句
        }
    });
}
```
lift()方法生成了一个新的 Observable 并返回，而且创建新 Observable 所用的参数 OnSubscribe 的回调方法 call() 中的实现与之前的Observable.subscribe()方法的实现很相似，再看一下Observable.subscribe()方法：
```java
// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
public Subscription subscribe(Subscriber subscriber) {
    subscriber.onStart();
    onSubscribe.call(subscriber);//两个方法都有该句
    return subscriber;
}
```
这两个方法内部都有该语句：
```java
 onSubscribe.call(subscriber)
```
onSubscribe 是 Observable 内部持有一个对象，当 Observable 调用 create()方法进行对象创建时被初始化。在上面那相同条语句中，onSubscribe所代指的对象是不相同的：

- subscribe() 方法中的 onSubscribe 指的是 Observable 中的 onSubscribe 对象
- 当含有 lift() 时：
    - lift()方法创建了一个新的Observable 对象并返回，在lift()方法被原始的Observable 对象调用之后，加上之前的原始 Observable，已经有两个 Observable 对象
    - 而同样地，新 Observable 里的新 OnSubscribe 加上之前的原始 Observable 中的原始 OnSubscribe，也就有了两个 OnSubscribe
    - 链式调用 Observable.lift().suscribe(Subscriber)的过程是，当调用经过 lift() 后的 Observable 的 subscribe() 的时候，使用的是 lift() 所返回的新的 Observable ，于是它所触发的 onSubscribe.call(subscriber)，也是用的新 Observable 中的新 OnSubscribe，即在 lift() 中生成的那个 OnSubscribe
    - 而这个新 OnSubscribe 的 call() 方法中的 onSubscribe ，就是指的原始 Observable 中的原始 OnSubscribe ，在这个 call() 方法里，新 OnSubscribe 利用 operator.call(subscriber) 生成了一个新的 Subscriber（Operator 就是在这里，通过自己的 call() 方法将新 Subscriber 和原始 Subscriber 进行关联，并插入自己的『变换』代码以实现变换），然后利用这个新 Subscriber 向原始 Observable 进行订阅，即原始的 OnSubscribe 的 call() 方法被调用

这样就实现了 lift() 的过程，有点像一种代理机制，通过事件拦截和处理实现事件序列的变换。精简掉细节的话，也可以这么说： 在 Observable 执行了 lift(Operator) 方法之后，会返回一个新的 Observable，这个新的 Observable 会像一个代理一样，负责接收原始的 Observable 发出的事件，并在处理后发送给 Subscriber。
为了便于理解，举一个具体的 Operator 的实现。下面这是一个将事件中的 Integer 对象转换成 String 的例子，仅供参考：
```java
//这里的泛型Operator<R,T>正好与泛型map(T,R)相反
observable.lift(new Observable.Operator<String, Integer>() {
    @Override
    public SubScriber<? super Integer> call(final Subscriber<? super String> subscriber{
        return new Subscriber<Integer>(){
            @Override
            public void onNext(Integer integer) {
                subscriber.onNext(integer+"");
            }
             @Override
            public void onCompleted() {
                subscriber.onCompleted();
            }

            @Override
            public void onError(Throwable e) {
                subscriber.onError(e);
            }
        };
    })
});
```
RxJava 都不建议开发者自定义 Operator 来直接使用 lift()，而是建议尽量使用已有的 lift() 包装方法（如 map() flatMap() 等）进行组合来实现需求，因为直接使用 lift() 非常容易发生一些难以发现的错误

#### 2.compose()
除了 lift() 之外， Observable 还有一个变换方法叫做 compose(Transformer)。它和 lift() 的区别在于， lift() 是针对事件项和事件序列的，而 compose() 是针对 Observable 自身进行变换。举个例子，假设在程序中有多个 Observable ，并且他们都需要应用一组相同的 lift() 变换。可以这么写：
```java
observable1
    .lift1()
    .lift2()
    .lift3()
    .lift4()
    .subscribe(subscriber1);
observable2
    .lift1()
    .lift2()
    .lift3()
    .lift4()
    .subscribe(subscriber2);
```
冗余代码太多，可以改为这样：
```java
private Observable liftAll(Observable observable) {
    return observable
        .lift1()
        .lift2()
        .lift3()
        .lift4();
}
...
liftAll(observable1).subscribe(subscriber1);
liftAll(observable2).subscribe(subscriber2);
```
这种方式对于 Observale 的灵活性增添了限制，破坏了流式的风格，这个时候，可以用 compose() 来解决：
```java
public class LiftAllTransformer implements Observable.Transformer<Integer, String> {
    @Override
    public Observable<String> call(Observable<Integer> observable) {
        return observable
            .lift1()
            .lift2()
            .lift3()
            .lift4();
    }
}
...
Transformer liftAll = new LiftAllTransformer();
observable1.compose(liftAll).subscribe(subscriber1);
observable2.compose(liftAll).subscribe(subscriber2);
```

## 4.线程控制
在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。
### 1.Scheduler使用
在RxJava 中Scheduler(调度器)，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：

- Schedulers.immediate()
直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler
-  Schedulers.trampoline()
在当前线程执行，与immediate不同的是，它并不会立即执行，而是将其存入队列，等待当前Scheduler中其它任务执行完毕后执行，这个在我们时常使用的并不多，它主要服务于repeat ，retry这类特殊的变换操作。
- Schedulers.newThread()
总是启用新线程，并在新线程执行操作
- Schedulers.io()
I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程
- Schedulers.computation()
计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU
- AndroidSchedulers.mainThread()
Android中专用的Scheduler，指定的操作将在 Android 主线程运行，由RxAndroid提供

有了Scheduler，使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制：

- subscribeOn()
指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程
- observeOn()
指定 Subscriber 所运行在的线程。或者叫做事件消费的线程

```java
Oberverable.just(1,2,3,4)
    .subscribeOn(Schedulers.io())// 指定 subscribe() 发生在 IO 线程
    .observeOn(AndroidSchedulers.mainThread())// 指定 Subscriber 的回调发生在主线程
    .subscribe(new Action1<Integer>(){
        @Override
        public void call(Integer number) {
            Log.d(tag, number);
        }
    });
```
改写由id取得图片并显示的示例
```java
int drawableRes = ...;
ImageView ImageView = ...;
Observerable.create(new Observerable.OnSubscribe<Drawable>(){
    @Override
    public void call(Subscriber<? super Drawable> subscriber) {
       //加载图片将会发生在 IO 线程
        Drawable drawable = getTheme().getDrawable(drawableRes);
        subscriber.onNext(drawable);
    }

})
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(new SubScriber<Drawable>(){
    @Override
    public void onNext(Drawable drawable) {
        //设置图片则被设定在了主线程
        ImageView.setImageDrawable(drawable);
    }

    @Override
    public void onCompleted() {

    }

    @Override
    public void onError(Throwable e) {
        Toast.makeText().(context, "Error",Toast.LENGTH_SHOR).show();
    }
});

Oberverable.just(1,2,3,4)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Integer>(){
        @Override
        public void call(Integer number) {
            Log.d(tag, number);
        }
    });
```
observeOn() 指定的是 Subscriber 的线程，而这个 Subscriber 并不是（严格说应该为『不一定是』，但这里不妨理解为『不是』）subscribe() 参数中的 Subscriber ，而是 observeOn() 执行时的当前 Observable 所对应的 Subscriber ，即它的直接下级 Subscriber 。换句话说，observeOn() 指定的是它之后的操作所在的线程。因此如果有多次切换线程的需求，只要在每个想要切换线程的位置调用一次 observeOn() 即可。示例代码：
```java
Observerable.just(1,2,3)// IO 线程，由 subscribeOn() 指定
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.newThread())
    .map(mapOperator)//// 新线程，由 observeOn() 指定
    .observerOn(Schedulers.io())
    .map(mapOperator2)// IO 线程，由 observeOn() 指定
    .oberveOn(AndroidSchedulers.mainThread())
    .subscribe(SubScriber);// Android 主线程，由 observeOn() 指定
```
通过 observeOn() 的多次调用，程序实现了线程的多次切换。不同于 observeOn() ， subscribeOn() 的位置放在哪里都可以，但它是只能调用一次的。当使用了多个 subscribeOn() 的时候，只有第一个 subscribeOn() 起作用。

### 2.doOnSubscribe()
虽然超过一个的 subscribeOn() 对事件处理的流程没有影响，但在流程之前却是可以利用的。Subscriber 的 onStart() 可以用作流程开始前的初始化。然而 onStart() 由于在 subscribe() 发生时就被调用了，因此不能指定线程，而是只能执行在 subscribe() 被调用时的线程。这就导致如果 onStart() 中含有对线程有要求的代码（例如在界面上显示一个 ProgressBar，这必须在主线程执行），将会有线程非法的风险，因为有时你无法预测 subscribe() 将会在什么线程执行。

与 Subscriber.onStart() 相对应的，有一个方法 Observable.doOnSubscribe() 。它和 Subscriber.onStart() 同样是在 subscribe() 调用后而且在事件发送前执行，但区别在于它可以指定线程。默认情况下， doOnSubscribe() 执行在 subscribe() 发生的线程；而如果在 doOnSubscribe() 之后有 subscribeOn() 的话，它将执行在离它最近的 subscribeOn() 所指定的线程。

示例代码：
```java
Observerable.just(1,2,3)//IO 线程，由 subscribeOn() 指定
            .subscribeOn(Shcedulers.io())
            .doOnSubscribe(new Action0(){Android 主线程，由后边最近的 subscribeOn() 指定
                @Override
                public void call(){
                    progressBar.setVisibility(View.Visible);
                }
            })
            .subscribeOn(AndroidSchedulers.mainThread())//指定doOnSubscribe()线程
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(Subscriber);// Android 主线程，由 observeOn() 指定
```
### 3.Scheduler原理
subscribeOn() 和 observeOn() 都做了线程切换的工作。
subscribeOn() 和 observeOn() 的内部实现，也是用的 lift()。subscribeOn() 和 observeOn() 方法都会创建一个新的 Observable 对象，不同的是， subscribeOn() 的线程切换发生在 新的Observable 对象的 OnSubscribe 中即在它通知上一级 OnSubscribe 时，这时事件还没有开始发送，因此 subscribeOn() 的线程控制可以从事件发出的开端就造成影响；而 observeOn() 的线程切换则发生在它内建的 Subscriber 中，即发生在它即将给下一级 Subscriber 发送事件时，因此 observeOn() 控制的是它后面的线程。
#### 1.subscribOn()
#### 2.observOn()
#### 3.线程切换规律
线程切换时有如下规律：

- 当使用了多个 subscribeOn() 的时候，只有第一个 subscribeOn() 起作用，这个 subscribeOn() 控制从流程开始的第一个操作，直到遇到第一个 observeOn()
- 可以通过 observeOn() 的多次调用，实现线程的多次切换。每个 observeOn() 将导致一次线程切换()，这次切换开始于这次 observeOn() 的下一个操作
- 不论是 subscribeOn() 还是 observeOn()，每次线程切换如果不受到下一个 observeOn() 的干预，线程将不再改变，不会自动切换到其他线程

## 5.RxJava使用场景
### 1.与Retrofit结合
Retrofit 除了提供了传统的 Callback 形式的 API，还有 RxJava 版本的 Observable 形式 API。下面用对比的方式来介绍 Retrofit 的 RxJava 版 API 和传统版本的区别。
以获取一个 User 对象的接口作为例子。使用Retrofit 的传统 API，可以这样实现：
```java
public interface Api {
    @GET("user")
    Call<User> getUser(@Query("userId") String userId);
}
```
使用Retrofit实现请求
```java
Call<User>> call  = ApiFactory.getUser();
call.enquue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {

        userView.setUser(response.body());
    }

    @Override
    public void onFailure(Call<User> call, Throwable t) {
        // Error Handling
    }
});
```
而使用 RxJava 形式的 API，定义同样的请求是这样的：
```java
public interafce Api {
    @GET("user")
    Observerable<User> getUser(@Query("userId") String userId);
}
```
使用Retrofit实现请求
```java
ApiFactory.getUser(userId)//返回Observable<User>
          .observeOn(AndroidSchedulers.mainThread())
          .subscribe(new Subscriber<User>(){
            @Override
            public void onNext(User user) {
                userView.setUser(user);
            }
            @Override
            public void onCompleted() {
                // Completed Handling
            }
            @Override
            public void onError(Throwable t) {
                // Error Handling
            }
          });
```
对比来看， Callback 形式和 Observable 形式长得不太一样，但本质都差不多，而且在细节上 Observable 形式似乎还比 Callback 形式要差点。但当情况变得复杂一些，假设这么一种情况：取到的 User 并不应该直接显示，而是需要先与数据库中的数据进行比对和修正后再显示。使用 Callback 方式大概可以这么写：
```java
Call<User>> call  = ApiFactory.getUser(userId);
call.enquue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {
        new Thread(){
            @Override
            pub void run() {
                processUser(user);//在分线程修正数据
                runOnUiThread(new Runnable(){
                    @Override
                    public void run() {
                        userView.setUser(user);//在主线程更新界面
                    }
                });
            }
        }.start();
    }

    @Override
    public void onFailure(Call<User> call, Throwable t) {
        // Error Handling
    }
});
```
这时的代码可读性太差，如果使用RxJava 形式的代码来实现：
```java
ApiFactory.getUser(userId)
          .doOnNext(new Action1<User>(){
            @Override
            public void call(User User){
                processUser(User);
            }
          })
          .subscribeOn(Schedulers.io())
          .observeOn(AndroidSchedulers.mainThread())
          .subscribe(new SUscriber<User>(){
            @Override
            public void onNext(User user) {
                userView.setUser(user);
            }
            @Override
            public void onCompleted() {
                // Completed Handling
            }
            @Override
            public void onError(Throwable t) {
                // Error Handling
            }
          });
```
### 2.RxBinding
RxBing项目地址: [RxBinding](https://github.com/JakeWharton/RxBinding)

RxBinding 提供了一套在 Android 平台上的基于 RxJava 的 Binding API。所谓 Binding，就是类似设置 OnClickListener 、设置 TextWatcher 这样的注册绑定对象的 API。
### 3.异步操作
### 4.RxBus
