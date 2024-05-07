# 1.Activity生命周期

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

（3）onCreate能做的事onStart其实都能做，但是onstart能做的事onCreate却未必适合做。如setContentView和资源初始化在两者都能做，然而想动画的初始化在onStart中做比较好。

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

（1）调用onSaveInstance保存当前Activity状态。注意，它与onPause方法没有先后之分。

（2）调用onStop方法做后续处理。

（3）调用onDestroy方法销毁当前活动。

（4）重新onCreate该活动。

（5）调用onStart方法之后，再调用onRestoreInstance方法加载保存的数据。

（6）接下来就与正常的一样了，调用onResume，然后运行。

## Q7：onSaveInstanceState和onRestoreInstanceState调用时机

当某个activity变得可能会被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。有这么几种情况： 

（1）当用户按下HOME键时。这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。

（2）长按HOME键，选择运行其他的程序时。 

（3）按下电源按键（关闭屏幕显示）时。 

（4）从activity A中启动一个新的activity时。 

（5）屏幕方向切换时，例如从竖屏切换到横屏时。在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState一定会被执行。 

总而言之，onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据。 

需要注意的是，onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的，onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的onRestoreInstanceState方法不会被执行。 

另外，onRestoreInstanceState的bundle参数也会传递到onCreate方法中，也可以选择在onCreate方法中做数据还原。

# 2.Activity启动模式

# 3.IntentFilter匹配规则

# 4.启动Activity的方式

# 5.Activity启动过程


