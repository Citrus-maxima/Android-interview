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

## Q6：对外提供数据共享时，如何限制对方的使用呢？

设置android:exported属性，这个属性用于指示该服务是否能够被其他应用程序组件调用或跟它交互。如果设置为true，则能够被调用或交互。设置为false时，只有同一个应用程序的组件或带有相同用户ID的应用程序才能启动或绑定该服务。

对于需要开放的组件应设置合理的权限，如果只需要对同一个签名的其它应用开放ContentProvider，则可以设置signature级别的权限。参考：

```xml
<permission android:name="com.android.gallery3d.filtershow.permission.READ"
            android:protectionLevel="signature" />

<permission android:name="com.android.gallery3d.filtershow.permission.WRITE"
            android:protectionLevel="signature" />

<provider
    android:name="com.android.gallery3d.filtershow.provider.SharedImageProvider"
    android:authorities="com.android.gallery3d.filtershow.provider.SharedImageProvider"
    android:grantUriPermissions="true"
    android:readPermission="com.android.gallery3d.filtershow.permission.READ"
    android:writePermission="com.android.gallery3d.filtershow.permission.WRITE" />
```

## Q7：如果我们只需要开放部份的 URI 给其他的应用访问呢？

设置<path-permission>标签，参考：

```xml
<provider android:name="ContactsProvider2"
    android:authorities="contacts;com.android.contacts"
    android:label="@string/provider_label"
    android:multiprocess="false"
    android:exported="true"
    android:grantUriPermissions="true"
    android:readPermission="android.permission.READ_CONTACTS"
    android:writePermission="android.permission.WRITE_CONTACTS">
    <path-permission
            android:pathPrefix="/search_suggest_query"
            android:readPermission="android.permission.GLOBAL_SEARCH" />
    <path-permission
            android:pathPrefix="/search_suggest_shortcut"
            android:readPermission="android.permission.GLOBAL_SEARCH" />
    <path-permission
            android:pathPattern="/contacts/.*/photo"
            android:readPermission="android.permission.GLOBAL_SEARCH" />
    <grant-uri-permission android:pathPattern=".*" />
</provider>
```

## Q8：ContentProvider的数据传输过程

1.	客户端请求：

当一个应用程序需要访问另一个应用程序的数据时，它会构造一个URI，并调用相应的ContentResolver方法（例如query、insert、update、delete）来发起请求。

2.	ContentResolver：

ContentResolver是应用程序访问ContentProvider的入口点。它会解析请求的URI，确定应该访问哪个ContentProvider。

3.	URI匹配：

ContentProvider在接收到请求时，会使用UriMatcher类来匹配传入的URI，以确定要操作的数据类型和具体的数据项。

4.	数据操作：

ContentProvider根据匹配结果，调用相应的方法来执行数据操作（例如查询数据库、插入新记录、更新现有记录或删除记录）。

5.	结果返回：

ContentProvider将操作结果（例如查询结果的Cursor对象或操作成功的状态）返回给ContentResolver，ContentResolver再将结果返回给最初发起请求的应用程序。

## Q9：Android的数据存储方式

- SharedPreferences

  本质是基于XML文件存储key-value键值对数据，通常用来存储一些少量的简单的配置信息。

  使用步骤：

  （1）根据Context获取SharedPreferences对象

  （2）利用edit()方法获取Editor对象。

  （3）通过Editor对象存储key-value键值对数据。

  （4）通过commit()方法提交数据。

  缺点：

  只能存储boolean，int，float，long和String五种简单的数据类型，无法进行条件查询。

- 文件

  文件可用来存放大量数据，如文本、图片、音频等。

  写文件：调用Context.openFileOutput()方法根据指定的路径和文件名来创建文件，这个方法会返回一个FileOutputStream对象。

  读取文件：调用Context.openFileInput()方法通过制定的路径和文件名来返回一个标准的FileInputStream对象。

- SQLite

  优点：

  （1）效率出众

  （2）十分适合存储结构化数据

  （3）方便在不同的Activity，甚至不同的应用之间传递数据

  （4）面向资源有限的设备

  （5）没有服务器进程

  （6）所有数据存放在同一文件中跨平台

  （7）可自由复制

- ContentProvider

  Android系统中能实现所有应用程序共享的一种数据存储方式。每个ContentProvider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享时，就需要使用ContentProvider为这些数据定义一个URI，然后其他的应用程序就通过Content Provider传入这个URI来对数据进行操作。

- 网络存储

  网络一般用于需要实时传输数据


## 10: 说说 ContentProvider、ContentResolver、ContentObserver 之间的关系

ContentProvider：内容提供者，用于对外提供数据

ContentResolver：内容解析者，用于获取内容提供者提供的数据

ContentObserver：内容监听器，可以监听数据的改变状态

ContentResolver.registerContentObserver()监听消息

ContentResolver.notifyChange(uri)发出消息

## Q11：为什么要用ContentProvider？它和sql的实现上有什么差别？

ContentProvider屏蔽了数据存储的细节,内部实现对用户完全透明,用户只需要关心操作数据的uri就可以了，ContentProvider可以实现不同app之间的共享。Sql也有增删改查的方法，但是sql只能查询本应用下的数据库。而 ContentProvider还可以去增删改查本地文件。
