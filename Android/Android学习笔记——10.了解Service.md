## Service

Service 是 Android 中实现程序后台运行的解决方案，它非常适合执行那些不需要和用户交互而且还要求长期运行的任务。Service 的运行不依赖于任何用户界面，即使程序被切换到后台，或者用户打开了另外一个应用程序，Service 仍然能够保持正常运行。

Service 并不是运行在一个独立的进程当中的，而是依赖于创建 Service 时所在的应用程序进程。当某个应用程序进程被杀掉时，所有依赖于该进程的 Service 也会停止运行。

实际上 Service 并不会自动开启线程，所有的代码都是默认运行在主线程当中的。也就是说，我们需要在 Service 的内部手动创建子线程，并在这里执行具体的任务，否则就有可能出现主线程被阻塞的情况。

## 多线程编程

当我们需要执行一些耗时操作，比如发起一条网络请求时，考虑到网速等其他原因，服务器未必能够立刻响应我们的请求，如果不将这类操作放在子线程里运行，就会导致主线程被阻塞。定义一个线程只需要新建一个类继承自 Thread，然后重写父类的 run()方法，并在里面编写耗时逻辑即可。使用继承的方式耦合性有点高，所以我们更多会选择使用实现 Runnable 接口的方式来定义一个线程。更常用的是使用 Lambda 表达式，或者 Kotlin 提供的更简单的 thread 函数。

```kotlin
// 继承类
class MyThread : Thread() {
    override fun run() {
        // 耗时操作
    }
}
// 启动线程
MyThread().start()

// 实现接口
class MyThread : Runnable {
    override fun run() {
        // 耗时操作
    }
}
// 启动线程
val myThread = MyThread()
Thread(myThread).start()

// Lambda表达式
Thread {
    // 编写具体的逻辑
}.start()

// thread函数
thread {
    // 编写具体的逻辑
}
```

## 更新 UI

Android 的 UI 也是线程不安全的。也就是说，如果想要更新应用程序里的 UI 元素，必须在主线程中进行。

但是有些时候，我们必须在子线程里执行一些耗时任务，然后根据任务的执行结果来更新相应的 UI 控件，对于这种情况，Android 提供了一套异步消息处理机制 Handler。

### 异步消息处理

异步消息处理主要由 4 个部分组成：Message 、Handler 、MessageQueue 和 Looper 。

Message 是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间传递数据。可以使用 Message 的 what 字段，除此之外还可以使用 arg1 和 arg2 字段来携带一些整型数据，使用 obj 字段携带一个 Object 对象。

Handler 主要用于发送和处理消息。发送消息一般是使用 Handler 的 sendMessage()方法、post()方法等，而发出的消息经过一系列地辗转处理后，最终会传递到 Handler 的 handleMessage()方法中。

MessageQueue 是消息队列的意思，它主要用于存放所有通过 Handler 发送的消息。这部分消息会一直存在于消息队列中，等待被处理。每个线程中只会有一个 MessageQueue 对象。

Looper 是每个线程中的 MessageQueue 的管家，调用 Looper 的 loop()方法后，就会进入一个无限循环当中，然后每当发现 MessageQueue 中存在一条消息时，就会将它取出，并传递到 Handler 的 handleMessage()方法中。每个线程中只会有一个 Looper 对象。

```kotlin
changeTextBtn.setOnClickListener {
    thread {
        // 子线程中发送消息
        val msg = Message()
        msg.what = updateText
        handler.sendMessage(msg) // 将Message对象发送出去
    }
}

// 主线程中创建一个Handler对象，用来发送和接收消息
val handler = object : Handler(Looper.getMaininLooper()) {
    override fun handleMessage(msg: Message) {
        // 在这里可以进行UI操作
        when (msg.what) {
            updateText -> textView.text = "Nice to meet you"
        }
    }
}
```

### AsyncTask

AsyncTask 是基于异步消息处理机制封装的。AsyncTask 是一个抽象类，所以如果我们想使用它，就必须创建一个子类去继承它。

在继承时我们可以为 AsyncTask 类指定 3 个泛型参数：

- Params：用于在后台任务中使用。

- Progress：在后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位。

- Result：当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型。

同时我们也需要重写几个方法才能完成对任务的定制：

- onPreExecute()：这个方法会在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框等。

- doInBackground(Params...)：这个方法中的所有代码都会在子线程中运行，我们应该在这里去处理所有的耗时任务。任务一旦完成，就可以通过 return 语句将任务的执行结果返回，如果 AsyncTask 的第三个泛型参数指定的是 Unit，就可以不返回任务执行结果。注意，在这个方法中是不可以进行 UI 操作的，如果需要更新 UI 元素，比如说反馈当前任务的执行进度，可以调用 publishProgress (Progress...)方法来完成。

- onProgressUpdate(Progress...)：当在后台任务中调用了 publishProgress(Progress...)方法后，onProgressUpdate (Progress...)方法就会很快被调用，该方法中携带的参数就是在后台任务中传递过来的。在这个方法中可以对 UI 进行操作，利用参数中的数值就可以对界面元素进行相应的更新。

- onPostExecute(Result)：当后台任务执行完毕并通过 return 语句进行返回时，这个方法就很快会被调用。返回的数 据会作为参数传递到此方法中，可以利用返回的数据进行一些 UI 操作，比如说提醒任务执 行的结果，以及关闭进度条对话框等。

启动任务只需要调用 execute() 方法即可。也可以给 execute() 方法传入任意数量的参数，这些参数将会传递到 doInBackground()方法当中。

```kotlin
// 第一个泛型参数指定为Unit，表示在执行AsyncTask 的时候不需要传入参数给后台任务。第二个泛型参数指定为Int，表示使用整型数据来作为进度显示单位。第三个泛型参数指定为Boolean，则表示使用布尔型数据来反馈执行结果。
class DownloadTask : AsyncTask<Unit, Int, Boolean>() {
    ...
}

// 启动任务
DownloadTask().execute()
```

### runOnUiThread

Android 是不允许在子线程中进行 UI 操作的，当我们在子线程中做了一些任务，想更新 UI 时，除了利用上面的一些方法，还有一个更简便的方法 runOnUiThread。在里面可以直接更新主线程的 UI。

```kotlin
thread {
    // 做一些耗时任务

    ...

    // 更新UI
    runOnUiThread {
        // 在这里进行UI操作，将结果显示到界面上
        responseText.text = response
    }
}
```

## 创建 Service

继承自系统的 Service 类，重写 onCreate()、onStartCommand()和 onDestroy()方法，实现 onBind() 抽象方法。

onCreate() 方法是在 Service 第一次创建的时候调用的，onStartCommand() 方法在每次启动 Service 的时候都会调用。onDestroy() 方法会在 Service 销毁的时候调用，可以回收那些不再使用的资源。onBind()用来和 Activity 通信。

Service 都需要在 AndroidManifest.xml 文件中进行注册才能生效。

启动和停止 Service 的方法主要是借助 Intent 来实现的，通过 startService 或者 stopService，在 Service 内部调用 stopSelf()方法也可以进行停止。

从 Android 8.0 系统开始，应用的后台功能被大幅削减。现在只有当应用保持在前台可见状态的情况下，Service 才能保证稳定运行，一旦应用进入后台之后，Service 随时都有可能被系统回收。

```kotlin
class MyService : Service() {
    ...
}

startServiceBtn.setOnClickListener {
    val intent = Intent(this, MyService::class.java)
    startService(intent) // 启动Service
}
stopServiceBtn.setOnClickListener {
    val intent = Intent(this, MyService::class.java)
    stopService(intent) // 停止Service
}
```

### 跟 Service 绑定

首先创建一个 ServiceConnection 的匿名类实现，并在里面重写 onServiceConnected() 方法和 onServiceDisconnected() 方法。

onServiceConnected() 方法会在 Activity 与 Service 成功绑定的时候调用，而 onServiceDisconnected() 方法只有在 Service 的创建进程崩溃或者被杀掉的时候才会调用。

bindService()方法接收 3 个参数，第一个参数是 Intent 对象，第二个参数是前面创建出的 ServiceConnection 的实例，第三个参数则是一个标志位，这里传入 BIND_AUTO_CREATE 表示在 Activity 和 Service 进行绑定后自动创建 Service 。

这样 Activity 就能自由地和 Service 进行通信了。只要调用方和 Service 之间的连接没有断开， Service 就会一直保持运行状态，直到被系统回收。

```kotlin
private val connection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName, service: IBinder) {
        // service就是 Service类中onBind方法返回的对象。
    }
    override fun onServiceDisconnected(name: ComponentName) {

    }
}

bindServiceBtn.setOnClickListener {
    val intent = Intent(this, MyService::class.java)
    bindService(intent, connection, Context.BIND_AUTO_CREATE) // 绑定Service
}

unbindServiceBtn.setOnClickListener {
    unbindService(connection) // 解绑Service
}
```

## 生命周期

两种启动方式的生命周期不同：

1、startService() 的生命周期 ：onCreate() -> onStartCommand() -> onDestroy()。

此方式的特征：与调起该服务的 context 没有任何关系，调起的 Context（Activity 等）销毁后，该服务也可以存活，但是无法与调起的 Context 进行通讯。每调用一次 startService()方法，onStartCommand()就会执行一次。

2、bindService()的生命周期：onCreate() -> onBind() -> onUnbind() -> onDestroy()

此方式的特征：1. 与调起该服务的 Context（Activity 等）绑定在一起的，可以与调起 Context 进行通讯，且调起 Context 销魂时该服务会自动解绑。2. 绑定服务时不会调用 onstartcommand()方法。

## 前台 Service

从 Android 8.0 系统开始，应用的后台功能被大幅削减。现在只有当应用保持在前台可见状态的情况下，Service 才能保证稳定运行，一旦应用进入后台之后，Service 随时都有可能被系统回收。如果你希望 Service 能够一直保持运行状态，就可以考虑使用前台 Service 。即使你退出应用程序，MyService 也会一直处于运行状态，而且不用担心会被系统回收。当然，MyService 所对应的通知也会一直显示在状态栏上面。如果用户不希望我们的程序一直运行，也可以选择手动杀掉应用，这样 MyService 就会跟着一起停止运行了。

前台 Service 和普通 Service 最大的区别就在于，它一直会有一个正在运行的图标在系统的状态栏显示。

构建 Notification 对象，并且不使用 NotificationManager 的 notify 将通知显示出来，而是调用了 startForeground()方法。

另外，从 Android 9.0 系统开始，使用前台 Service 必须在 AndroidManifest.xml 文件中进行权限声明才行。`<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />`

```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val channel = NotificationChannel("my_service", "前台Service通知", NotificationManager.IMPORTANCE_DEFAULT)
    manager.createNotificationChannel(channel)
}
val intent = Intent(this, MainActivity::class.java)
val pi = PendingIntent.getActivity(this, 0, intent, 0)
val notification = NotificationCompat.Builder(this, "my_service")
    .setContentTitle("This is content title")
    .setContentText("This is content text")
    .setSmallIcon(R.drawable.small_icon)
    .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon))
    .setContentIntent(pi)
    .build()

startForeground(1, notification)
```

## Service 中处理耗时操作

Service 中的代码都是默认运行在主线程当中的，如果直接在 Service 里处理一些耗时的逻辑，就很容易出现 ANR(Application Not Responding)的情况。

所以我们应该在 Service 的每个具体的方法里开启一个子线程，然后在这里处理那些耗时的逻辑。这种 Service 一旦启动，就会一直处于运行状态，必须调用 stopService()或 stopSelf()方法，或者被系统回收，Service 才会停止。所以我们可以在执行完逻辑后自动停止。

```kotlin
override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
    thread {
        // 处理具体的逻辑
        ...
        // 停止
        stopSelf()
    }
    return super.onStartCommand(intent, flags, startId)
}
```

### IntentService 类

也可以用 IntentService 类，创建一个异步的、会自动停止的 Service。

在子类中实现 onHandleIntent()这个抽象方法，这个方法中可以处理一些耗时的逻辑，而不用担心 ANR 的问题，因为这个方法已经是在子线程中运行的了。

```kotlin
class MyIntentService : IntentService("MyIntentService") {
    override fun onHandleIntent(intent: Intent?) { // 打印当前线程的id
        Log.d("MyIntentService", "Thread id is ${Thread.currentThread().name}")
    }
    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyIntentService", "onDestroy executed")
    }
}

// 启动Service
val intent = Intent(this, MyIntentService::class.java)
startService(intent)
```
