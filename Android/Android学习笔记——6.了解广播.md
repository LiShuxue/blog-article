## 广播机制

每个应用程序都可以对自己感兴趣的广播进行注册，这样该程序就只会收到自己所关心的广播内容，这些广播可能是来自于系统的，也可能是来自于其他应用程序的。Android 提供了一套完整的 API，允许应用程序自由地发送和接收广播。

发送广播用 Intent，接收广播用 BroadcastReceiver。

### 广播类型

Android 中的广播主要可以分为两种类型：标准广播和有序广播。

标准广播(normal broadcasts)：是一种完全异步执行的广播，在广播发出之后，所有的 BroadcastReceiver 几乎会在同一时刻收到这条广播消息，因此它们之间没有任何先后顺序可言。这种广播的效率会比较高，但同时也意味着它是无法被截断的。

有序广播(ordered broadcasts)：是一种同步执行的广播，在广播发出之后，同一时刻只会有一个 BroadcastReceiver 能够收到这条广播消息，当这个 BroadcastReceiver 中的逻辑执行完毕后，广播才会继续传递。所以此时的 BroadcastReceiver 是有先后顺序的，优先级高的 BroadcastReceiver 就可以先收到广播消息，并且前面的 BroadcastReceiver 还可以截断正在传递的广播，这样后面的 BroadcastReceiver 就无法收到广播消息了。

## 系统广播

Android 内置了很多系统级别的广播，我们可以在应用程序中通过监听这些广播来得到各种系统的状态信息。

比如手机开机完成后会发出一条广播，电池的电量发生变化会发出一条广播，系统时间发生改变也会发出一条广播，亮屏熄屏、网络变化等场景下也会发出广播。等等。我们可以根据自己感兴趣的广播，自由地注册 BroadcastReceiver。

注册 BroadcastReceiver 的方式一般有两种：在代码中注册和在 AndroidManifest.xml 中注册。其中前者被称为动态注册，后者被称为静态注册。

具体的系统广播列表在：

```xml
<Android SDK>/platforms/<任意android api版本>/data/broadcast_actions.txt
```

### 动态注册

新建一个类，让它继承自 BroadcastReceiver ，并重写父类的 onReceive() 方法就行了。然后再 MainActivity 的 onCreate 方法中，调用 registerReceiver() 注册。

系统每隔一分钟就会发出一条 android.intent.action.TIME_TICK 的广播，可以监听时间变化。

```kotlin
inner class TimeChangeReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Time has changed", Toast.LENGTH_SHORT).show()
    }
}

// onCreate中，注册广播接收器，接收的是 android.intent.action.TIME_TICK
val intentFilter = IntentFilter()
intentFilter.addAction("android.intent.action.TIME_TICK")
timeChangeReceiver = TimeChangeReceiver()
registerReceiver(timeChangeReceiver, intentFilter)
```

### 静态注册

动态注册的 BroadcastReceiver 可以自由地控制注册与注销，在灵活性方面有很大的优势。但是它必须在程序启动之后才能接收广播，让程序在未启动的情况下也能接收广播需要使用静态注册的方式，静态注册需要在 AndroidManifest.xml 中用 receiver 标签注册。

从理论上来说，动态注册能监听到的系统广播，静态注册也应该能监听到，在过去的 Android 系统中确实是这样的。但是由于大量恶意的应用程序利用这个机制在程序未启动的情况下监听系统广播，从而使任何应用都可以频繁地从后台被唤醒，严重影响了用户手机的电量和性能，因此 Android 系统几乎每个版本都在削减静态注册 BroadcastReceiver 的功能。

在 Android 8.0（API level 26） 系统之后，所有隐式广播都不允许使用静态注册的方式来接收了。隐式广播指的是那些没有具体指定发送给哪个应用程序的广播，大多数系统广播属于隐式广播，但是少数特殊的系统广播目前仍然允许使用静态注册的方式来接收。这些特殊的系统广播列表详见 <https://developer.android.google.cn/guide/components/broadcast-exceptions.html> 。

开机自启动可以使用静态注册的方式来接收开机广播，然后在 onReceive() 方法里执行相应的逻辑。

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

<receiver
    android:name=".BootCompleteReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

```kotlin
class BootCompleteReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Boot Complete", Toast.LENGTH_LONG).show()
    }
}
```

## 自定义广播

使用 intent 发送一个自定义的 action。intent 中也可以携带自定义数据。

默认情况下我们发出的自定义广播恰恰都是隐式广播。因此这里一定要调用 setPackage() 方法，指定这条广播是发送给哪个应用程序的，从而让它变成一条显式广播。

通过 `sendBroadcast(intent)` 发送一条标准广播。

```kotlin
val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
intent.setPackage(packageName)
sendBroadcast(intent)
```

通过 `sendOrderedBroadcast(intent, null)` 可以发送一条有序广播。我们通过 android:priority 属性给 BroadcastReceiver 设置优先级，优先级比较高的 BroadcastReceiver 就可以先收到广播。通过 `abortBroadcast()` 可以终止广播传递下去。

## 注册时机

比如强制下线的广播，可以在 BaseActivity 中声明 BroadcastReceiver 内部类，因为每个 Activity 都继承他。

我们始终需要保证只有处于栈顶的 Activity 才能接收到广播，非栈顶的 Activity 不应该也没必要接收这条广播，所以在 onResume() 中 registerReceiver 和 onPause() 方法里 unregisterReceiver 就可以很好地解决这个问题，当一个 Activity 失去栈顶位置时就会自动取消 BroadcastReceiver 的注册。
