## Q1：Service的生命周期

![image](https://github.com/Citrus-maxima/Android-interview/assets/46516051/ec987b1b-efdc-4138-bd4f-2f98a8429d4e)

## Q2：服务启动一般有几种，服务和activty之间怎么通信，服务和服务之间怎么通信？

1、startService：

onCreate()--->onStartCommand() ---> onDestory()

如果服务已经开启，不会重复的执行onCreate()，而是会调用onStartCommand()。一旦服务开启跟调用者(开启者)就没有任何关系了，服务会在后台长期的运行。开启者不能调用服务里面的方法。

2、bindService：

onCreate() --->onBind()--->onunbind()--->onDestory()

bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉。绑定者可以调用服务里面的方法。

通信：

（1）通过Binder对象。

（2）通过broadcast(广播)。

## Q3：Service有什么类型？

![image](https://github.com/Citrus-maxima/Android-interview/assets/46516051/8ff64374-df5f-405b-a17e-dfae746a6b66)

## Q4：为什么bindService可以跟Activity生命周期联动？

1、bindService方法执行时，LoadedApk会记录ServiceConnection信息。

2、Activity执行finish方法时，会通过LoadedApk检查Activity是否存在未注销/解绑的BroadcastReceiver和ServiceConnection，如果有，那么会通知AMS注销/解绑对应的BroadcastReceiver和Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作。

## Q5：如何保证Service不被杀死？

1. 提高进程优先级，降低进程被杀死的概率
   
方法一：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的Activity，在用户解锁时将Activity销毁掉。

方法二：启动前台service。

方法三：提升service优先级。

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

2. 在进程被杀死后，进行拉活

方法一：注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等。

方法二：双进程相互唤起。

方法三：依靠系统唤起。

方法四：onDestroy方法里重启service：service + broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

3. 依靠第三方

根据终端不同，在小米手机（包括MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。

## Q7：onStartCommand 的几种模式

- START_NOT_STICKY

  如果返回 START_NOT_STICKY，表示当Service运行的进程被Android系统强制杀掉之后，不会重新创建该Service。

  如果我们某个Service执行的工作被中断几次无关紧要或者对Android内存紧张的情况下需要被杀掉且不会立即重新创建这种行为也可接受，那么我们便可将onStartCommand的返回值设置为 START_NOT_STICKY。举个例子，某个Service需要定时从服务器获取最新数据：通过一个定时器每隔指定的N分钟让定时器启动Service去获取服务端的最新数据。当执行到Service的onStartCommand时，在该方法内再规划一个N分钟后的定时器用于再次启动该Service并开辟一个新的线程去执行网络操作。假设Service在从服务器获取最新数据的过程中被Android系统强制杀掉，Service不会再重新创建，这也没关系，因为再过N分钟定时器就会再次启动该Service并重新获取数据。

- START_STICKY

  如果返回START_STICKY，表示Service运行的进程被Android系统强制杀掉之后，Android系统会将该Service依然设置为started状态（即运行状态），但是不再保存onStartCommand方法传入的intent对象，然后Android系统会尝试再次重新创建该Service，并执行onStartCommand回调方法，但是onStartCommand回调方法的Intent参数为 null，也就是onStartCommand 方法虽然会执行但是获取不到intent信息。如果这个Service可以在任意时刻运行或结束都没什么问题，而且不需要intent信息，那么就可以在onStartCommand方法中返回START_STICKY，比如一个用来播放背景音乐功能的Service就适合返回该值。

- START_REDELIVER_INTENT

  如果返回START_REDELIVER_INTENT，表示Service运行的进程被Android系统强制杀掉之后，与返回START_STICKY的情况类似，Android系统会将再次重新创建该Service，并执行onStartCommand回调方法，但是不同的是，Android系统会再次将Service在被杀掉之前最后一次传入onStartCommand方法中的Intent再次保留下来并再次传入到重新创建后的Service的onStartCommand方法中，这样我们就能读取到intent参数。只要返回START_REDELIVER_INTENT，那么onStartCommand中的intent一定不是null。如果我们的Service需要依赖具体的Intent才能运行（需要从Intent中读取相关数据信息等），并且在强制销毁后有必要重新创建运行，那么这样的Service就适合返回START_REDELIVER_INTENT。

## Q8：IntentService是什么？

IntentService是一种特殊的Service，它继承了Service并且它是一个抽象类，因此必须创建它的子类才能使用IntentService。

原理

在实现上，IntentService封装了HandlerThread和Handler。当IntentService被第一次启动时，它的onCreate()方法会被调用，onCreate()方法会创建一个HandlerThread，然后使用它的Looper来构造一个Handler对象mServiceHandler，这样通过mServiceHandler发送的消息最终都会在HandlerThread中执行。

生成一个默认的且与主线程互相独立的工作者线程来执行所有传送至onStartCommand()方法的Intetnt。

生成一个工作队列来传送Intent对象给onHandleIntent()方法，同一时刻只传送一个Intent对象，这样一来，就不必担心多线程的问题。在所有的请求(Intent)都被执行完以后会自动停止服务，所以，不需要自己去调用stopSelf()方法来停止。

该服务提供了一个onBind()方法的默认实现，它返回null。

提供了一个onStartCommand()方法的默认实现，它将Intent先传送至工作队列，然后从工作队列中每次取出一个传送至onHandleIntent()方法，在该方法中对Intent做相应的处理。

![image](https://github.com/user-attachments/assets/ad73b46f-b16a-4122-af24-de71d8a9e85a)

![image](https://github.com/user-attachments/assets/feb1d60a-1976-4e4a-b93d-68d53bba1ac8)

## Q8：为什么在mServiceHandler的handleMessage()回调方法中执行完onHandleIntent()方法后要使用带参数的stopSelf()方法？

因为stopSelf()方法会立即停止服务，而stopSelf（int startId）会等待所有的消息都处理完毕后才终服务，一般来说，stopSelf(int startId)在尝试停止服务之前会判断最近启动服务的次数是否和startId相等，如果相等就立刻停止服务，不相等则不停止服务。

