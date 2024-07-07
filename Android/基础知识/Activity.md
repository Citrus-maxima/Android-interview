## Q1. 请简述Activity的生命周期，并列出主要的回调方法。

![image](https://github.com/Citrus-maxima/Android-interview/assets/46516051/0a82a02c-e0d7-40d6-b0ec-fdb3389f7807)

onCreate:表示创建，用于进行Activity的一些初始化工作，比如使用setContentView加载布局，对一些控件和变量进行初始化等。

onStart:表示启动，此时Activity已经可见了，但是还没出现在前台，用户还看不到，无法与Activity交互。

onResume:表示继续、重新开始。Activity在这个阶段已经出现在前台并且可见了。这个阶段可以打开独占设备。

onPause:表示暂停，当前Activity要跳到另一个Activity或应用正常退出时都会执行这个方法。此时Activity在前台并可见，我们可以进行一些轻量级的存储数据和去初始化的工作，不能太耗时，因为在跳转Activity时只有当一个Activity执行完了onPause方法后另一个Activity才会启动，而且android中指定如果onPause在500ms内没有执行完毕的话就会强制关闭Activity。从生命周期图中发现可以在这快速重启，但这种情况其实很罕见，比如用户切到下一个Activity的途中按back键快速得切回来。

onStop：表示停止，此时Activity已经不可见了，但是Activity对象还在内存中，没有被销毁。这个阶段的主要工作也是做一些资源的回收工作。

onDestroy：表示销毁，这个阶段Activity被销毁，不可见，我们可以将还没释放的资源释放，以及进行一些回收工作。

onRestart：表示重新开始，Activity在这时可见，当用户按Home键切换到桌面后又切回来或者从后一个Activity切回前一个Activity就会触发这个方法。这里一般不做什么操作。

## Q1：onCreate和onStart之间有什么区别？

（1）可见与不可见的区别。前者不可见，后者可见。

（2）执行次数的区别。onCreate方法只在Activity创建时执行一次，而onStart方法在Activity的切换以及按Home键返回桌面再切回应用的过程中被多次调用。因此Bundle数据的恢复在onStart中进行比onCreate中执行更合适。

（3）onCreate能做的事onStart其实都能做，但是onstart能做的事onCreate却未必适合做。如setContentView和资源初始化在两者都能做，然而像动画的初始化在onStart中做比较好。

## Q2：onStart方法和onResume方法有什么区别？

（1）是否在前台。onStart方法中Activity可见但不在前台，不可交互，而onResume中的Activity在前台。

（2）职责不同。onStart方法中主要还是进行初始化工作，而onResume方法，根据官方的建议，可以做开启动画和独占设备的操作。

## Q3：onPause方法和onStop方法有什么区别？

是否可见。onPause时Activity可见，onStop时Activity不可见，但Activity对象还在内存中。

## Q4：onStop方法和onDestroy方法有什么区别？

onStop阶段Activity还没有被销毁，对象还在内存中，此时可以通过切换Activity再次回到该Activity，而onDestroy阶段Acivity被销毁。

## Q5：为什么切换Activity时各方法的执行次序是(A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop而不是(A)onPause→(A)onStop→(B)onCreate→(B)onStart→(B)onResume

（1）一个Activity或多或少会占有系统资源，而在官方的建议中，onPause方法将会释放掉很多系统资源，为切换Activity提供流畅性的保障，而不需要再等多两个阶段，这样做切换更快。

（2）按照生命周期图的表示，如果用户在切换Activity的过程中再次切回原Activity，是在onPause方法后直接调用onResume方法的，这样比onPause→onStop→onRestart→onStart→onResume要快得多。

## Q6：资源相关的系统配置发生改变导致Activity被杀死并重新创建，Activity的生命周期

如竖屏切换到横屏，由于系统配置发生了改变，在默认情况下，Activity就会被销毁并重新创建。A销毁后立刻创建B，A中的一些信息会在B中恢复。

（1）调用onPause方法确保未保存的状态得到存储，以及释放那些不再需要的资源。

（2）调用onSaveInstanceState保存当前Activity状态。

（3）调用onStop方法做后续处理。

（4）调用onDestroy方法销毁当前活动。

（5）重新onCreate该活动。

（6）调用onStart方法之后，再调用onRestoreInstanceState方法加载保存的数据。

（7）接下来就与正常的一样了，调用onResume，然后运行。

## Q7：在Android开发中，你如何确保Activity在横竖屏切换时不重新创建？

在AndroidManifest.xml文件中的Activity标签中指定android:configChanges属性。当声明了此属性后，系统不会再在横竖屏切换时销毁并重新创建Activity，而是会调用Activity的onConfigurationChanged()方法。

## Q8：onSaveInstanceState和onRestoreInstanceState调用时机

当某个activity变得可能会被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。有这么几种情况： 

（1）当用户按下HOME键时。这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。

（2）长按HOME键，选择运行其他的程序时。 

（3）按下电源按键（关闭屏幕显示）时。 

（4）从activity A中启动一个新的activity时。 

（5）屏幕方向切换时，例如从竖屏切换到横屏时。在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState一定会被执行。 

总而言之，onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据。 

需要注意的是，onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的，onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的onRestoreInstanceState方法不会被执行。 

另外，onRestoreInstanceState的bundle参数也会传递到onCreate方法中，也可以选择在onCreate方法中做数据还原。

## Q9：以下场景中，Activity生命周期的调用

1. Activity A中启动Activity B：

   A: opPause() -> B: onCreate() -> B: onStart() -> B: onResume() -> A: onStop()

2. home键：

   onPause() -> onstop() -> onRestart() -> onStart() -> onResume()

3. finish()：

   onCreate()中执行：onCreate() -> onDestroy()

   onStart()中执行：onCreate() -> onStart() -> onStop() -> onDestroy()

   onResume()中执行：onCreate() -> onStart() -> onResume() -> onPause() -> onStop() -> onDestroy()

4. 物理返回键onKeyDown()：

   onPause() -> onStop() -> onDestroy()

5. 息屏再打开：

   onPause() -> onStop() -> onRestart() -> onStart() -> onResume()

6. 在当前Activity界面进入设置界面更改了一些已有的设置，或者Activity发生异常崩溃重建：

   onPause() -> onStop() -> onDestroy() -> onCreate() -> onStart() -> onResume()

## Q10：Activity启动模式

- 标准模式（Standard）：默认启动模式，每次启动Activity都会创建一个新的Activity实例，并置于栈顶。

- 栈顶复用模式（SingleTop）：如果要启动的Activity已经在栈顶，则不会重新创建Activity，而是复用栈顶的实例，同时该Activity的onNewIntent()方法会被调用。如果要启动的Activity不在栈顶，则会重新创建该Activity的实例，并置于栈顶。

- 栈内复用模式（SingleTask）：如果要启动的Activity已经存在于它想要归属的栈中，那么将栈中位于该Activity上的所有Activity出栈，同时该Activity的onNewIntent()方法会被调用。如果要启动的Activity不存在于它想要归属的栈中，如果该栈存在，则创建该Activity的实例，如果该栈不存在，则首先要创建一个新栈，然后创建该Activity实例并压入到新栈中。

- 单例模式（SingleInstance）：启动Activity时，首先要创建一个新栈，然后创建该Activity实例并压入新栈中，新栈中只会存在这一个Activity实例。一旦该模式的Activity实例已经存在于某个栈中，任何应用激活该Activity时都会重用该栈中的实例，以及进入到该应用中。即多个应用共享该栈中的该Activity实例。

## Q11：Activity各种启动模式的使用场景

- standard模式

使用场景：这是默认的启动模式，当没有特殊要求时，Activity通常会使用此模式。

特点：每次启动一个Activity时，系统都会创建一个新的Activity实例，无论该Activity的实例是否已经存在。

示例：当用户在应用程序中浏览一系列列表项，点击每个列表项都会打开一个新的详情页面时，这些详情页面Activity可以使用standard模式。

- singleTop模式

使用场景：当希望避免在任务栈顶部重复创建相同Activity的实例时，可以使用此模式。

特点：如果新的Activity已经位于任务栈的栈顶，那么此Activity不会重新创建实例，而是复用栈顶的Activity实例，同时会调用它的onNewIntent()方法。

示例：通知栏弹出Notification，点击Notification跳转到指定Activity，但是如果我现在页面就停留在那个指定的Activity，就不会再次打开我当前的Activity。

- singleTask模式

使用场景：当希望某个Activity作为任务的入口点，并且希望在整个任务中只存在一个实例时，可以使用此模式。

特点：只要Activity实例在栈中存在，那么多次启动此Activity都不会重新创建此Activity的实例，而是将已存在的Activity实例移到栈顶，并调用它的onNewIntent()方法。

示例：在应用中，通常会有一个主界面（如MainActivity），用户可以通过主界面进入其他功能页面。为了保持主界面的唯一性，并避免在主界面和其他页面之间频繁切换时重复创建主界面实例，可以将主界面Activity设置为singleTask模式。

- singleInstance模式

使用场景：当希望某个Activity作为单独的任务存在，并且只在整个系统中存在一个实例时，可以使用此模式。

特点：除了具有singleTask模式的特性外，还具有更严格的限制。具有此启动模式的Activity只能单独存在于一个任务栈中，其他任务栈中的Activity不能与其位于同一个任务栈中。

示例：在一些需要全局唯一性的场景中，如音乐播放器或来电界面，可以使用singleInstance模式来确保这些Activity在整个系统中只有一个实例存在。

## Q12：描述一个场景，在这个场景中你可能需要使用到FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_SINGLE_TOP这两个标志

- 场景：在一个Android应用中，用户从侧边栏或菜单中选择一个功能项，打开对应的Activity，并且希望按返回键时能够回到主界面而不是菜单或侧边栏Activity。

- 使用场景描述

假设我们有一个Android应用，其主界面为Activity A，侧边栏或菜单中包含了多个功能项，每个功能项对应一个Activity（例如Activity B、Activity C等）。用户从侧边栏或菜单中选择一个功能项，点击后启动Activity B。用户在Activity B中完成操作后，如果直接按返回键，用户会先返回到之前的菜单或侧边栏Activity，而不是直接回到主界面。为了改善用户体验，我们希望用户能够直接返回到主界面。为此，我们可以给Activity A设置FLAG_ACTIVITY_CLEAR_TOP标志，这样返回时如果Activity A（主界面）已经在任务栈中存在，那么Activity A之上的所有Activity实例（包括Activity B）都会被销毁。然后，Activity A会被置于栈顶，用户按返回键时就会直接退出应用，而不是返回到之前的菜单或侧边栏Activity。

如果Activity A被设置为singleTop启动模式，并且它已经在栈顶时，再次启动Activity A不会创建新的Activity A实例，而是会复用现有的实例。
这可以避免不必要的Activity创建和销毁，提高应用的性能和用户体验。

- 总结
在这个场景中，FLAG_ACTIVITY_CLEAR_TOP用于确保用户能够直接返回到主界面而不是经过中间的Activity，而FLAG_ACTIVITY_SINGLE_TOP（如果适用）则用于优化主界面的启动行为。这两个标志的结合使用可以为用户提供更加流畅和直观的应用导航体验。

## Q13：简述Intent在Android中的作用及其分类

- 作用

1. 启动活动（Activity）：Intent可以指定要启动的目标Activity，并携带额外的数据。

2. 启动服务（Service）：通过Intent，可以在后台启动Service以执行长时间运行的操作。

3. 发送广播（Broadcast）：Intent还可以用于发送广播，通知其他应用或应用内的组件有特定的事件发生。

- 分类

显式Intent：直接指定了要启动的组件名称（如Activity、Service等），主要用于应用内部组件的启动。

隐式Intent：不直接指定组件名称，而是设置Action、Data、Category，让系统来筛选出合适的Activity。主要用于启动其他应用的组件或进行某些动作。

## Q14：请描述Intent中能够携带的数据类型

1. 基本数据类型及其包装类：如int、String等。

2. Serializable对象：实现了Serializable接口的Java对象可以通过Intent传递。

3. Parcelable对象：实现了Parcelable接口的Java对象也是Intent可以携带的数据类型。相比于Serializable，Parcelable更加高效，常用于进程间通信（IPC）。

4. Bundle：一个可以携带多个键值对的容器，可以包含以上所有类型的数据。

## Q15：请解释Intent Filter在Android中的作用

Intent Filter在Android中用于描述一个组件（如Activity、Service、BroadcastReceiver）能够响应哪些类型的隐式Intent。它定义了组件能够处理的动作（action）、数据（data）、类别（category）等。当系统接收到一个隐式Intent时，会查找所有注册了该Intent Filter的组件，并将Intent发送给能够响应的组件。

例如，一个浏览器应用可能会在其Activity的Manifest中定义一个Intent Filter，用于捕获处理HTTP或HTTPS链接的隐式Intent，从而在用户点击链接时启动浏览器应用。

## Q16：请简述IntentFilter的匹配规则

IntentFilter主要包括：action、category、data。只有Intent完全匹配三者，才能成功启动Activity。一个Activity可以拥有多个IntentFilter，一个Intent只要能匹配其中一个，就能成功启动Activity。

（1）action匹配规则

action根据name属性值进行匹配。Intent的action需要与name的值完全一样，才算匹配成功。一个IntentFilter可以有多个action, Intent的action只要和其中一个action匹配成功就可以。一个Intent如果没有指定action，那么匹配失败。

（2）category匹配规则

category根据name属性值进行匹配。Intent要么不携带category参数，直接默认匹配。要么携带的category每一个都必须在IntentFilter中的category能匹配到。
Intent不携带category也能匹配成功是因为: 调用startActivity和startActivityForResult时，系统会默认为Intent添加\<category android:name="android.intent.category.DEFAULT" />category。因此IntentFilter中也通常需要包含这个默认的category。

（3）data匹配规则

data的匹配和action类似，也是只要Intent的data能匹配多个data中的一个就匹配成功。data由两部分组成：mimeType和URI。

（4）优先级

如果存在多个IntentFilter都满足匹配条件，系统会根据在\<intent-filter>标签中定义的优先级标签（\<priority>）来对IntentFilter进行排序。优先级最高的IntentFilter将被选择来处理该Intent。

## Q17：Activity启动过程

应用程序调用startActivity()方法，传入一个Intent，表示要启动的Activity。AMS接收到启动Activity的请求后，会检查Activity所属的应用程序是否已经启动。如果没有启动，AMS会通知Zygote进程启动一个新的应用进程来承载该Activity。

AMS会通过Binder机制向目标应用进程中的ActivityThread发送启动Activity的命令。ActivityThread是应用进程中的主线程，它接收到AMS的启动命令后，会在主线程中创建Activity实例。
 
ActivityThread会调用Instrumentation类的callActivityOnCreate()方法，最终触发Activity的生命周期方法，如onCreate()。

## Q18：请解释什么是Activity的透明主题（transparent theme），并给出一个应用场景。

透明主题（Transparent Theme）是一种特殊的主题，它使得Activity的背景完全透明。使用透明主题的Activity会让用户看到它背后的Activity或应用程序主屏幕。应用场景：

 1. 创建覆盖层：例如，在拍照应用中，可以使用透明Activity显示拍摄按钮和一些控制元素，而摄像头的预览内容仍然在背景中显示。
    
 2. 临时显示UI元素：例如，在地图应用中，可以使用透明Activity显示一些临时的标记或工具提示，同时让用户继续查看地图。
    
 3. 引导页面：在引导用户使用应用的过程中，可以使用透明Activity显示一些引导信息或动画，而不影响用户对应用界面的操作。
