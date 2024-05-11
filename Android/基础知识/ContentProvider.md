## Q1：Android为什么要设计ContentProvider这个组件？

（1）封装。对数据进行封装，提供统一的接口，使用者完全不必关心这些数据是在DB、XML、Preferences或者网络请求来的。当项目需求要改变数据来源时，使用ContentProvider的地方不需要修改。

（2）提供一种跨进程数据共享的方式。

（3）支持数据更新通知。因为数据是在多个应用程序中共享的，当其中一个应用程序改变了这些共享数据的时候，它有责任通知其它应用程序共享数据被修改了，这样其它应用程序就可以作相应的处理。

（4）更好的数据访问权限管理。ContentProvider可以对数据进行权限设置，不同的URI可以对应不同的权限，只有符合权限要求的组件才能访问到ContentProvider的具体操作。

## Q2：通过ContentResolver获取ContentProvider内容的基本步骤

（1）得到ContentResolver类对象：ContentResolver cr = getContentResolver()。

（2）定义要查询的字段String数组。

（3）使用cr.query()，返回一个Cursor对象。

（4）使用while循环得到Cursor里面的内容。

## Q3：ContentProvider是如何实现数据共享的？

1. 首先自定义一个类继承ContentProvider，然后覆写query、insert、update、delete等方法。

2. 在AndroidManifest文件中进行注册。
   
3. 把自己的数据通过uri的形式共享出去。

## Q4：多个进程同时调用一个ContentProvider的query获取数据，ContentPrvoider是如何反应的呢？

query()，insert()，delete()，update()都是在ContentProvider进程的Binder线程池中被调用执行的，而不是进程的主线程中。

## Q5：运行在主线程的ContentProvider为什么不会影响主线程的UI操作？

ContentProvider的onCreate()是运行在UI线程的，而query()、insert()、delete()、update()是运行在线程池中的工作线程的。所以调用这个方法并不会阻塞ContentProvider所在进程的主线程，但可能会阻塞调用者所在的进程的UI线程。
所以，调用ContentProvider的操作仍然要放在子线程中去做。虽然直接的CRUD的操作是在工作线程的，但系统会让调用线程等待这个异步的操作完成，才可以继续线程之前的工作。
