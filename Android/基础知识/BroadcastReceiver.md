## Q1：广播的实现原理

广播使用了设计模式中的观察者模式。

![image](https://github.com/Citrus-maxima/Android-interview/assets/46516051/83027a9d-67f9-4f84-8384-cf779217cf4c)

## Q2：广播的注册方式

![image](https://github.com/Citrus-maxima/Android-interview/assets/46516051/c90e0fd5-3f51-40ae-b389-ed0c9c503956)

## Q3：广播的类型

- 普通广播

  开发者自身定义Intent的广播（最常用）。通过sendBroadcast()方法发送。

- 系统广播

  Android内置广播，涉及手机的基本操作（如开机、网络状态变化、拍照等）。

- 有序广播

  通过sendOrderedBroadcast(intent)发送。

  （1）广播接收者按顺序接收广播，按照Priority属性值从大到小排序，Priority属性相同者，动态注册的广播优先。

  （2）先接收的广播接收者可以对广播进行截断，即后接收的广播接收者不再接收到此广播。

  （3）先接收的广播接收者可以对广播进行修改，那么后接收的广播接收者将接收到被修改后的广播。

  备注：优先级可以声明在<intent-filter android:priority="n".../>，也可以调用IntentFilter对象的setPriority设置。

- 粘性广播

  通过sendStickyBroadcast()方法发送，需要android.Manifest.permission.BROADCAST_STICKY权限。

  粘性广播是指在广播发送之后，即使没有注册接收器，也可以在注册接收器后接收到该广播。也就是说，粘性广播可以在发送之后被缓存，并在注册接收器后立即发送给接收器。粘性广播一般用来确保重要的状态改变后的信息被持久保存，并且能随时广播给新的广播接收器，比如电源的改变。

  使用场景：

  （1）应用程序在后台运行时，用户可能会接收不到广播消息，因为没有相应的接收器注册。使用粘性广播，可以确保在应用程序重新进入前台时，能够立即接收到之前发送的广播消息。

  （2）应用程序启动后，获取之前发送的广播消息。这对于需要在应用程序启动后恢复状态或数据的情况非常有用。

- 本地广播（App应用内广播）

  App应用内广播可理解为一种局部广播，广播的发送者和接收者都同属于一个App。相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 & 效率高。

  使用方式1：将全局广播设置成局部广播

  （1）注册广播时将exported属性设置为false，使得非本App内部发出的此广播不被接收。

  （2）在广播发送和接收时，增设相应权限permission，用于权限验证。

  （3）发送广播时指定该广播接收器所在的包名，此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

  使用方式2：使用封装好的LocalBroadcastManager类

## Q4：常见的系统广播

android.intent.action.DATE_CHANGED：日期变化广播

android.intent.action.BATTERY_CHANGED：电量变化广播

android.intent.action.SCREEN_ON：屏幕亮起广播

android.intent.action.SCREEN_OFF：屏幕关闭广播

android.intent.action.PACKAGE_ADDED：应用安装广播

android.intent.action.PACKAGE_REMOVED：应用卸载广播  

android.intent.action.BOOT_COMPLETED：系统启动完成广播

## Q5：LocalBroadcastManager实现原理

LocalBroadcastManager内部维护了mReceivers和mActions两个HashMap。mReceivers 是接收器和IntentFilter的对应表，主要作用是方便在unregisterReceiver(…)取消注册，同时作为对象锁限制注册接收器、发送广播、取消接收器注册等几个过程的并发访问。mActions以Action为key，注册这个Action的BroadcastReceiver链表为value。mActions的主要作用是方便在广播发送后快速得到可以接收它的BroadcastReceiver。在注册广播时，其实是在更新这两个Map。

发送广播时，根据Action从mActions中取出ReceiverRecord列表，找出action匹配的广播，然后通过Handler发送消息，在Handler的handleMessage中，取出匹配的广播列表，依次回调onReceive方法。

## Q6：全局广播内部的数据结构

1. RegisteredReceivers：

这是一个维护已注册的广播接收器的信息的结构。在Android源码中，这通常是一个由广播接收器（BroadcastReceiver）和相应的过滤器（IntentFilter）组成的集合。每当应用程序注册一个广播接收器时，系统会将这个接收器及其过滤器添加到这个集合中。

2. BroadcastQueue：

这是一个管理广播消息队列的结构。系统维护了多个广播队列（通常是前台广播队列和后台广播队列），以区分不同优先级的广播消息。每个队列中包含待处理的广播消息，这些消息会按照一定的顺序（通常是FIFO，即先进先出）被处理和分发。

3. ReceiverList：

这是一个用于存储特定广播接收器列表的结构。当系统接收到一个广播消息时，会根据广播消息的Intent及其过滤器，匹配相应的广播接收器，并将它们放入这个列表中以便进一步处理。

4. IntentResolver：

这是一个用于匹配广播接收器的结构。每当有广播消息发送时，系统会通过IntentResolver根据Intent过滤器查找与之匹配的广播接收器。

当一个广播消息被发送时，系统会经历以下主要步骤：

1. Intent匹配：

系统会使用IntentResolver根据广播消息的Intent查找所有与之匹配的广播接收器。

2. 消息队列：

系统会将广播消息添加到相应的BroadcastQueue中。

3. 分发广播：

系统会从BroadcastQueue中取出广播消息，并通过ReceiverList获取相应的广播接收器，逐个调用它们的onReceive方法进行处理。
