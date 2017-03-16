title: Android 相关重难点知识整理
date: 2016-12-15 22:00:00
categories: Android
tags: [Java, Android]
---


<!--more-->

# 集合
1. 对 `HashMap` 进行排序:  `HashMap` 本身无序，但其子类 `LinkedHashMap` 使用链表结构，实现了有序。通过 `HashMap#entrySet()` 方法可以将 `Map` 转为 `Set<Entry>` ，再在 `ArrayList` 的构造函数中可以传入 `Collections` ，正好 `Set` 和 `List` 的父类就是 `Collections` ，这样就可以调用 `Collections.sort(list, comparator)` 进行排序了。排序好后，使用 `for` 遍历加入 `LinkedHashMap` 。

# 引用类型
1. 引用类型

 - 强引用
 ```
 String str = "abc";
 ```

 - 软引用
 ```
SoftReference<String> soft = new SoftReference<String>(str);
 ```

 - 弱引用
 ```
WeakReference<String> wek = new WeakReference<String>(str);
 ```
 - 虚引用
 ```
PhantomReference
 ```


 ```java
// 注意
String str = "abc"; // 常量池中
String str = new String("abc"); // 堆内存中
 ```

2. 对象可及性
强可及对象: 除非虚拟机 `OOM` ，否则永远不会被回收
软可及对象: 系统内存不足时，被回收
弱可及对象: 当系统 `GC` 发现发现该对象，就被回收

# 线程池
1. 控制一个方法的并发数量限制
 - 方法一，使用信号量
 `Semaphore` 信号量，构造函数传入允许个数。
 `Semaphore#acquire()` 取得锁， `Semaphore#release()` 释放锁。

 - 方法二，使用线程池
 ```
Excutors.newFixedThreadPool(num);
 ```

2. 手动实现线程池
```java
new ThreadPoolExecutor(
        int corePoolSize, // 核心池大小: 建议 CPU 个数+1
        int maximumPoolSize, // 线程池最大容量
        long keepAliveTime, // 任务执行完毕后释放延时
        TimeUnit unit, // 时间的单位
        BlockingQueue<Runnable> workQueue, // 工作队列
        ThreadFactory threadFactory, // 将 Runnable 包装成线程的工厂
        RejectedExecutionHandler handler)
```

 - 线程数总大小为 最大池大小(Thread)+队列(Runnable)
 - 接口 `BlockingQueue` 是单端队列， `BlockingDueue` 是双端队列。对于单端队列，其子类有 `Array` 和 `Linked` 等，对于此处频繁增删的需求，使用 `LinkedBlockingQueue` 更佳。
 - 对于 `i++` ，要使用线程安全的 `AtomInteger#getAndIncrement()` 方法

# IOC (DI)

1. IOC(DI) 概念
Inverse Of Controller，控制反转; Dependency Inject 依赖注入。
2. ViewUtils框架，XUtils中的四大部分之一，使用到就是 IOC
3. 自定义注解
 - Target，注解类作用的对象，如FILED、METHOD等
 - Retention，生命周期，SOURCE(源码中存在，编译成字节码被清除)、CLASS(字节码中存在，运行时被清除)、RUNTIME(运行期运行期有效，会被加载到虚拟机中)
 - 定义体中， `value` 作为默认变量名，如 `@XXX("abc") `
4. 反射
 -  `getFiled()` 只能获取 `public` 修饰的字段，通常使用 `getDeclaredFileld()` 获取申明的字段
 -  设置字段的值先通过 `DeclaredFiled#setAccessible(true)` 暴力取得权限，再通过 `DeclaredField#set(user, name)` 设置


# Handler 机制
1.  `Handler` 通常用于子线程给主线程发送消息
2.  `Looper.prepare()`，创建 `Looper` ，创建 `MessageQueue` ，通过 `ThreadLocal` 将 `Looper` 与主线程绑定
3.   `new Handler()`，从 `ThreadLocal` 中取得 `Looper`，从 `Looper` 中取得 `MessageQueue` 的引用
4. `handler#sendMeessage()`，消息中添加 `msg.target=this`，然后放入 `MessageQueue` 
5.  `Looper#loop()`，循环取消息池，调用 `dispatchMessage()` 
6.  `Looper` 需要调用 `Looper#quit()` 终止

# Fragment
1.  `Fragment` 切换使用 `Fragment#hide()` 和 `Fragment#show()` 效率最高
2. 手动实现回退栈
- 每次替换 `Fragment` 的时候添加到 `List` 

 ```java
 if(list.contains(fragment)){
    list.remove(fragment);
    list.add(fragment);
 }else {
    list.add(fragment);
 }
 ```
 - 监听返回按钮
 ```java
if(list.size() > 1){
        list.remove(list.size()-1);
        transcation
            .hide(...)
            .show(list.get(list.size()-1))
            .commit();
}else {
        finish();
}
 ```

# 图片处理
 - ImageLoader
 年限久，用户量大
 - Glide
官方推荐使用，功能强大
 - Picasso
 热门，受欢迎
 - Fresco
 三级缓存，不能使用原生的控件

# 序列化
1.  `Serializable` 是 `Java SE` 实现的用于对象序列化的接口， `Parcelable` 是 `Android` 推出的用于序列化的接口。 `Serializable` 实现更简单，但性能不如 `Parcelable` 
2.  `transient` 关键字用于保留字段不被序列化

# Activity 保存
1.  `onSaveInstanceState()` 是会在 `onStop()` 前调用，用于保存 `Activity` 的状态
2. 调用时机: 横竖屏切换、HOME键后台等，但按返回键将不会调用

# 自定义权限
1. 在清单文件中 `Activity` 可设置属性 `permission` 来自定义启用该活动所需要的权限，可以任意命名
2. 自定义权限在使用的时候要先声明再使用:
```xml
 // 声明权限
 <permission android:name="com.xxx.xxx.AAA" />
 // 使用权限
 <use-permission android:name="com.xxx.xxx.AAA" />
```
3. 四大组件都是可以声明自定义权限的
4. 在广播接收者中，调用 `sendBroadcast(intent, String permission)` 指定权限，但仍然需要在清单文件中申明与使用，广播的自定义权限主要用于友方相互唤醒

# Service
1.  `startService()` 方式启动，`Service` 就一直在后台运行，与其他组件的生命周期无关，`bindService()` 方式启动，`Activity` 销毁时该 `Service` 也同时销毁
2. `IntentService` 用于执行较耗时的后台任务，执行完成后自动销毁

# 7.0 新特性
1. `JIT` 编译器: 安装速度提升 `75%`，并减少 `50%` 的应用程序编译代码，并在同等 `CPU` 性能从 `30%` 提高到 `600%`，使用 `JIT` 可以让用户安装程序、运行应用更快。
2. `Vulkan API`: `Open GL` 的下一个版本，`Android 7.0` 将支持相关 `API`
3. 多窗口模式: 可以分屏开多个多窗口
4. 可回复通知: 通过 `Notifiction.builder` 中 `addAction()` 可以设置回复
5. 目录访问权限: 使用 `StorageManager` 访问目录，，，它将动态申请权限，而不需要在清单文件中申明权限
6. 流量节省程序: 系统增加了一个全局的流量节省工具
7. `ICU4J API` 支持: 系统内置了该免费开源 `Unicode` 工具库，不需要再在应用中集成
8. `Direct Boot`: 新的系统中应用可以申请在开机未解锁的情况下直接启动，比如微信、第三方的闹钟等
9. `VR` 使用: 支持 `VR` 应用程序编写，使用 `com.google.vr.sdk.widgets.pano.VrPanoramaView` 控件和 `com.google.vr.sdk.widgets.pano.VrVideView` 分别显示图片和视频
