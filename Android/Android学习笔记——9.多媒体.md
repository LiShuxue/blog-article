## 通知 Notification

最开始应用程序可以随意给用户推送通知，用户是没有办法对这些信息做区分的，要么同意接受所有信息，要么屏蔽所有信息，这也是 Android 通知功能的痛点。

所以 Android 8.0 系统引入了通知渠道，顾名思义，就是每条通知都要属于一个对应的渠道。每个应用程序都可以自由地创建当前应用拥有哪些通知渠道，但是这些通知渠道的控制权是掌握在用户手上的。用户可以自由地选择这些通知渠道的重要程度，是否响铃、是否振动或者是否要关闭这个渠道的通知。

对于每个应用来说，通知渠道的划分是非常考究的，因为通知渠道一旦创建之后就不能再修改了，因此开发者需要仔细分析自己的应用程序一共有哪些类型的通知，然后再去创建相应的通知渠道。

### 创建通知渠道

使用 NotificationChannel 类构建一个通知渠道，并调用 NotificationManager 的 createNotificationChannel() 方法完成创建。由于是 Android 8.0 系统中新增的 API，因此我们在使用的时候还需要进行版本判断。

其中渠道 ID 可以随便定义，只要保证全局唯一性就可以。渠道名称是给用户看的，需要可以清楚地表达这个渠道的用途。通知的重要等级主要有 IMPORTANCE_HIGH、IMPORTANCE_DEFAULT、 IMPORTANCE_LOW、IMPORTANCE_MIN 这几种，对应的重要程度依次从高到低。不同的重要等级会决定通知的不同行为。这是初始的等级，用户可以随时手动更改某个通知渠道的重要等级，开发者是无法干预的。

通知渠道发出的通知可以弹出横幅(像接收到微信消息一样的提示)、发出声音，而低重要等级的通知渠道发出的通知不仅可能会在某些情况下被隐藏，而且可能会被改变显示的顺序，将其排在更重要的通知之后。

```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val channel = NotificationChannel(channelId, channelName, importance)
    manager.createNotificationChannel(channel)
}
```

### 创建通知

使用 AndroidX 库中的 NotificationCompat 类创建 Notification 对象，可以设置 title，content 等，通过 notify(id, notification) 方法让通知显示出来。每个通知是个唯一的 id。

```kotlin
val notification = NotificationCompat.Builder(context, channelId)
    .setContentTitle("This is content title")
    .setContentText("This is content text")
    .setSmallIcon(R.drawable.small_icon)
    .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.large_icon))
    .build()

manager.notify(1, notification)
```

### 通知的交互，点击，取消

实现通知的点击效果，我们需要在代码中进行相应的设置，这就涉及了一个新的概念——PendingIntent。PendingIntent 倾向于在某个合适的时机执行某个动作。所以，也可以把 PendingIntent 简单地理解为延迟执行的 Intent。

PendingIntent 类提供了几个静态方法用于获取 PendingIntent 的实例，可以根据需求来选择是使用 getActivity() 方法、getBroadcast() 方法，还是 getService() 方法。这几个方法所接收的参数都是相同的：第一个参数依旧是 Context，第二个参数一般用不到，传入 0 即可，第三个参数是一个 Intent 对象，我们可以通过这个对象构建出 PendingIntent 的“意图”，第四个参数用于确定 PendingIntent 的行为，有 FLAG_ONE_SHOT、FLAG_NO_CREATE、FLAG_CANCEL_CURRENT 和 FLAG_UPDATE_CURRENT 这 4 种值可选，每种值的具体含义你可以查看文档，通常情况下这个参数传入 0 就可以了。

NotificationCompat.Builder 通过 setContentIntent() 方法接收这个意图。

setAutoCancel()方法传入 true，就表示当点击这个通知的时候，通知会自动取消。也可以通过 manager.cancel(id) 去取消，这个 id 就是创建通知的那个 id。

```kotlin
// 打开特定的Activity
val intent = Intent(this, NotificationActivity::class.java)
val pi = PendingIntent.getActivity(this, 0, intent, 0)

val notification = NotificationCompat.Builder(this, "normal")
    ...
    .setContentIntent(pi)
    .build()
```

## 调用摄像头和相册

### 拍照

先创建一个 File 对象，用于存放摄像头拍下的图片，再将 File 对象转换成 Uri 对象，这个 Uri 对象标识着 output_image.jpg 这张图片的本地真实路径。接下来再构建一个 Intent 对象，并将这个 Intent 的 action 指定为 android.media.action.IMAGE_CAPTURE，再调用 Intent 的 putExtra()方法指定图片的输出地址。

```kotlin
outputImage = File(externalCacheDir, "output_image.jpg")
if (outputImage.exists()) {
    outputImage.delete()
}
outputImage.createNewFile()

imageUri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    FileProvider.getUriForFile(this, "com.example.cameraalbumtest.fileprovider", outputImage)
} else {
    Uri.fromFile(outputImage)
}

val intent = Intent("android.media.action.IMAGE_CAPTURE")
intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri)
startActivityForResult(intent, 1)
```

externalCacheDir 应用关联缓存目录：`/sdcard/Android/data/<packagename>/cache`，从 Android 6.0 系统开始，读写 SD 卡被列为了危险权限，如果将图片存放在 SD 卡的任何其他目录，都要进行运行时权限处理才行，而使用应用关联目录则可以跳过这一步。另外，从 Android 10.0 系统开始，公有的 SD 卡目录已经不再允许被应用程序直接访问了，而是要使用作用域存储才行。

从 Android 7.0 系统开始，直接使用本地真实路径的 Uri 被认为是不安全的，会抛出一个 FileUriExposedException 异常。而 FileProvider 则是一种特殊的 ContentProvider，它使用了和 ContentProvider 类似的机制来对数据进行保护，可以选择性地将封装过的 Uri 共享给外部，从而提高了应用的安全性。

调用照相机程序去拍照有可能会在一些手机上发生照片旋转的情况。这是因为这些手机认为打开摄像头进行拍摄时手机就应该是横屏的，因此回到竖屏的情况下就会发生 90 度的旋转。

### 打开相册

先构建一个 Intent 对象，并将它的 action 指定为 Intent.ACTION_OPEN_DOCUMENT，表示打开系统的文件选择器。接着给这个 Intent 对象设置一些条件过滤，只允许可打开的图片文件显示出来，然后调用 startActivityForResult() 方法即可

```kotlin
// 打开文件选择器
val intent = Intent(Intent.ACTION_OPEN_DOCUMENT)
intent.addCategory(Intent.CATEGORY_OPENABLE)
// 指定只显示图片
intent.type = "image/*"
startActivityForResult(intent, 2)
```

## 播放音频视频

在 Android 中播放音频文件一般是使用 MediaPlayer 类实现的，它提供了开始，暂停，重置，停止等方法。

音乐存放位置，可以创建一个 assets 目录，并在这个目录下存放任意文件和子目录，这些文件和子目录在项目打包时会一并被打包到安装文件中，然后我们在程序中就可以借助 AssetManager 这个类提供的接口对 assets 目录下的文件进行读取。

播放视频文件其实并不比播放音频文件复杂，主要是使用 VideoView 类来实现的。这个类将视频的显示和控制集于一身。提供了开始，暂停，重新播放等。

VideoView 不支持直接播放 assets 目录下的视频资源，所以我们在 res 目录下再创建一个 raw 目录，像诸如音频、视频之类的资源文件也可以放在这里，并且 VideoView 是可以直接播放这个目录下的视频资源的。
