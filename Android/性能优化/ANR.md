## Q1：什么是ANR

ANR:Application Not Responding，即应用无响应，Android系统对于一些事件需要在一定的时间范围内完成，如果超过预定时间能未能得到有效响应或者响应时间过长，都会造成ANR。一般地，这时往往会弹出一个提示框，告知用户当前xxx未响应，用户可选择继续等待或者Force Close。

## Q2：ANR出现场景

InputDispatching Timeout: 输入事件分发超时5s，包括按键分发事件的超时

Service Timeout：前台服务在20s内未执行完成，后台服务200s

BroadcastQueue Timeout：前台广播在10s内未执行完成，后台广播60s

ContentProvider Timeout：内容提供者执行超时10s

## Q3：ANR实现原理

主体实现在系统层，核心原理是消息调度和超时处理。经系统进程system_server调度，派发到应用进程完成对消息的实际处理，同时系统进程设计了不同的超时限制来跟踪消息的处理。一旦消息处理不当，则会触发超时限制，收集当前的系统状态，然后报告给用户有进程无响应。

Android系统ANR的实现，基本都是基于Handler消息机制来完成的。前面说过响应超时的定义，那么在一个事件执行开始时，通过Handler去post一个对应时间的延迟消息，如果事件在规定事件内执行完成，就remove掉这个message，否则，Handler就会收到这个ANR的Message，做进一步处理，dump日志，弹出ANR对话框。

![image](https://github.com/user-attachments/assets/3369d71a-c838-4b3d-b446-f4f8be08213e)

## Q4：ANR分析方法

- traces.txt文件

应用ANR产生的时候，ActivityManagerService的appNotResponding方法就会被调用,然后在/data/anr/traces.txt文件中写入ANR相关信息。最新的ANR信息在最开始部分。

- log分析

通常发生了ANR，ActivityManager会打印报错信息：ANR in com.xxx.xxxx // ANR出现的进程包名

可以在log中搜索“ANR in”或“am_anr”，会找到ANR发生的log，该行会包含了ANR的时间、进程、是何种ANR等信息，如果是BroadcastReceiver的ANR，可以怀疑是BroadcastReceiver.onReceive()方法的问题。如果是Service或Provider就怀疑是否其onCreate()的问题。

在该条log之后会有CPU usage的信息，表明了CPU在ANR前后的用量（log会表明截取ANR的时间），从各种CPU Usage信息中大概可以分析如下几点：

(1). 如果某些进程的CPU占用百分比较高，几乎占用了所有CPU资源，而发生ANR的进程CPU占用为0%或非常低，则认为CPU资源被占用，进程没有被分配足够的资源，从而发生了ANR。这种情况多数可以认为是系统状态的问题，并不是由本应用造成的。

(2). 如果发生ANR的进程CPU占用较高，如到了80%或90%以上，则可以怀疑应用内一些代码不合理消耗掉了CPU资源，如出现了死循环或者后台有许多线程执行任务等等原因，这就要结合trace和ANR前后的log进一步分析了。

(3). 如果CPU总用量不高，该进程和其他进程的占用过高，这有一定概率是由于某些主线程的操作耗时过长，或者是由于主进程被锁造成的。

除了上述的情况(1)以外，分析CPU usage之后，确定问题需要我们进一步分析trace文件。trace文件记录了发生ANR前后该进程的各个线程的stack。对我们分析ANR问题最有价值的就是其中主线程的stack，一般主线程的trace可能有如下几种情况：

(1). 主线程是running或者native而对应的栈对应了我们应用中的函数，则很有可能就是执行该函数时候发生了超时。

(2). 主线程被block:非常明显的线程被锁，这时候可以看是被哪个线程锁了，可以考虑优化代码。如果是死锁问题，就更需要及时解决了。

(3). 由于抓trace的时刻很有可能耗时操作已经执行完了（ANR -> 耗时操作执行完毕 ->系统抓trace），这时候的trace就没有什么用了，主线程的stack就是这样的：

```
"main" prio=5 tid=1 Native
  | group="main" sCount=1 dsCount=0 obj=0x757855c8 self=0xb4d76500
  | sysTid=3276 nice=0 cgrp=default sched=0/0 handle=0xb6ff5b34
  | state=S schedstat=( 50540218363 186568972172 209049 ) utm=3290 stm=1764 core=3 HZ=100
  | stack=0xbe307000-0xbe309000 stackSize=8MB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/3276/stack)
  native: #00 pc 0004099c  /system/lib/libc.so (__epoll_pwait+20)
  native: #01 pc 00019f63  /system/lib/libc.so (epoll_pwait+26)
  native: #02 pc 00019f71  /system/lib/libc.so (epoll_wait+6)
  native: #03 pc 00012ce7  /system/lib/libutils.so (_ZN7android6Looper9pollInnerEi+102)
  native: #04 pc 00012f63  /system/lib/libutils.so (_ZN7android6Looper8pollOnceEiPiS1_PPv+130)
  native: #05 pc 00086abd  /system/lib/libandroid_runtime.so (_ZN7android18NativeMessageQueue8pollOnceEP7_JNIEnvP8_jobjecti+22)
  native: #06 pc 0000055d  /data/dalvik-cache/arm/system@framework@boot.oat (Java_android_os_MessageQueue_nativePollOnce__JI+96)
  at android.os.MessageQueue.nativePollOnce(Native method)
  at android.os.MessageQueue.next(MessageQueue.java:323)
  at android.os.Looper.loop(Looper.java:138)
  at android.app.ActivityThread.main(ActivityThread.java:5528)
  at java.lang.reflect.Method.invoke!(Native method)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:740)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:630)
```

当然这种情况很有可能是由于该进程的其他线程消耗掉了CPU资源，这就需要分析其他线程的trace以及ANR前后该进程自己输出的log了。

## Q5：traces文件位置，如何获取traces文件

在/data/anr/目录下

如果手机已经root，可以直接通过adb pull /data/anr/traces.txt d:/导出

通过adb bugreport E:\bugs导出

## Q6：traces文件有哪些信息

ANR的进程id、时间和进程名称

线程的基本信息

线程的优先级（默认5）、线程锁id和线程状态

线程的调用栈信息(这里可查看导致ANR的代码调用流程，分析ANR最重要的信息)

## Q7：traces文件中线程的可能状态

```
ThreadState (defined at “dalvik/vm/thread.h “)
THREAD_UNDEFINED = -1, /* makes enum compatible with int32_t */
THREAD_ZOMBIE = 0, /* TERMINATED */( 线程死亡，终止运行)
THREAD_RUNNING = 1, /* RUNNABLE or running now */(线程可运行或正在运行)
THREAD_TIMED_WAIT = 2, /* TIMED_WAITING in Object.wait() */(执行了带有超时参数的wait、sleep或join函数)
THREAD_MONITOR = 3, /* BLOCKED on a monitor */(线程阻塞，等待获取对象锁)
THREAD_WAIT = 4, /* WAITING in Object.wait() */( 执行了无超时参数的wait函数)
THREAD_INITIALIZING= 5, /* allocated, not yet running */(新建，正在初始化，为其分配资源)
THREAD_STARTING = 6, /* started, not yet on thread list */(新建，正在启动)
THREAD_NATIVE = 7, /* off in a JNI native method */(正在执行JNI本地函数)
THREAD_VMWAIT = 8, /* waiting on a VM resource */(正在等待VM资源)
THREAD_SUSPENDED = 9, /* suspended, usually by GC or debugger */(线程暂停，通常是由于GC或debug被暂停)
```

## Q8：可能导致ANR的原因

IO操作，如数据库、文件、网络

CPU不足，一般是别的App占用了大量的CPU，导致App无法及时处理

硬件操作，如camera

线程问题，如主线程被join/sleep，或wait锁等导致超时

Service问题，如service忙导致超时无响应，或service binder的数量达到上限

system server问题，如WatchDog发现ANR

## Q9：哪些地方是执行在主线程的？

1.Activity的所有生命周期回调都是执行在主线程的.

2.Service默认是执行在主线程的.

3.BroadcastReceiver的onReceive回调是执行在主线程的.

4.没有使用子线程的Looper的Handler的handleMessage, post(Runnable)是执行在主线程的.

5.AsyncTask的回调中除了doInBackground, 其他都是执行在主线程的.

6.View的post(Runnable)是执行在主线程的.

## Q10：日常开发中如何避免ANR

避免在主线程进行复杂耗时的操作，特别是文件读取或者数据库操作。

避免频繁实时更新UI。

在设计及代码编写阶段避免出现出现同步/死锁或者错误处理不恰当等情况。

使用Thread或者HandlerThread时，调用Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，因为默认Thread的优先级和主线程相同。

BroadcastReceiver 要进行复杂操作的的时候，可以在onReceive()方法中启动一个Service来处理；

避免在IntentReceiver里启动一个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢夺焦点。如果你的应用程序在响应Intent广播时需要向用户展示什么，你应该使用Notification Manager来实现。
