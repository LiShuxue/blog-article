[第一行代码 Android（第三版）](https://cdn.lishuxue.site/blog/image/Android/%E7%AC%AC%E4%B8%80%E8%A1%8C%E4%BB%A3%E7%A0%81Android%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89.pdf)

## Android 架构

Android 大致可以分为 4 层架构: Linux 内核层、系统运行库层、应用框架层和应用层。

- Linux 内核层: 这一层为 Android 设备的各种硬件提供了底层的驱动， 如显示驱动、音频驱动、照相机驱动、蓝牙驱动、Wi-Fi 驱动、电源管理等。

- 系统运行库层: 这一层通过一些 C/C++ 库为 Android 系统提供了主要的特性支持。如 SQLite 库提供了数据库的支持，OpenGL|ES 库提供了 3D 绘图的支持，Webkit 库提供了浏览器内核的支持等。
  在这一层还有 Android 运行时库，它主要提供了一些核心库，允许开发者使用 Java 语言来 编写 Android 应用。另外，Android 运行时库中还包含了 Dalvik 虚拟机(5.0 系统之后改为 Android Runtime(ART)运行环境)，它使得每一个 Android 应用都能运行在独立的进程中，并且拥有一个自己 的虚拟机实例。

- 应用框架层: 这一层主要提供了构建应用程序时可能用到的各种 API，开发者可以使用这些 API 来构建自己的应用程序。如 Activity Manager，Package Manager，Location Manager 等等

- 应用层：所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序， 或者是你从 Google Play 上下载的小游戏，当然还包括你自己开发的程序。

## 四大组件

Android 系统四大组件分别是 Activity、Service 、BroadcastReceiver 和 ContentP rovider。

- Activity 是所有 Android 应用程序的门面，凡是在应用中你看得到 的东西，都是放在 Activity 中的。

- Service 在后台默默地运行，即使用户退出了应用，Service 仍然是可以继续运行的。

- BroadcastReceiver 允许你的应用接收来自各处的广播消息，比如电话、短信等，当然， 你的应用也可以向外发出广播消息。

- ContentProvider 则为应用程序之间共享数据提供了 可能，比如你想要读取系统通讯录中的联系人，就需要通过 ContentProvider 来实现。

## 目录结构

- .gradle 和.idea ： 这两个目录下放置的都是 Android Studio 自动生成的一些文件，我们无须关心，也不要去手动编辑。

- build：这个目录主要包含了一些在编译时自动生成的文件，你也不需要过多关心。

- gradle ：这个目录下包含了 gradle-wrapper 的配置文件和 jar 包，gradle-wrapper.jar 是一个脚本，调用了配置文件中声明的 Gradle 版本，并且我们编译时需要事先下载它。所以，开发者能够快速的启动并且运行 Gradle 项目，不用再手动安装，从而节省了时间成本。

- build.gradle： 全局的 gradle 构建脚本，可以配置构建脚本的公共部分。

- gradle.properties: 全局的 gradle 配置文件，在这里配置的属性将会影响到项目中所有的 gradle 编译脚本。

- gradlew 和 gradlew .bat：这两个文件是用来在命令行界面中执行 gradle 命令的，其中 gradlew 是在 Linux 或 Mac 系统中使用的，gradlew .bat 是在 Windows 系统中使用的。

- local.properties：指定本机中的 Android SDK 路径，通常内容是自动生成的，我们并不需要修改。

- settings.gradle：指定项目中所有引入的模块。

- app/build：这个目录和外层的 build 目录类似，也包含了一些在编译时自动生成的文件，或者是手动生成的 apk。

- app/libs：如果你的项目中使用到了第三方 jar 包，就需要把这些 jar 包都放在 libs 目录下，放在这个目 录下的 jar 包会被自动添加到项目的构建路径里。

- app/src: 主要的 android 代码和测试用例。

- app/src/java：java 目录是放置我们所有 Java 代码的地方(Kotlin 代码也放在这里)

- app/src/res：项目中使用到的所有图片、布局、字符串，应用图标等资源都要存放在这个目录下。所有 以“drawable” 开头的目录都是用来放图片的，所有以“mipmap” 开头的目录都是用来放应用图 标的，所有以“values” 开头的目录都是用来放字符串、样式、颜色等配置的，所有以“layout” 开头的目录都是用来放布局文件的。

- app/src/AndroidManifest.xml：Android 应用的配置文件，你在程序中定义的所有四大组件都需要在这个文件里注册，另外还可以在这个文件中给应用程序添加权限声明。没有在 AndroidManifest.xml 里注册的 Activity 是不 能使用的。

- app/build.gradle：这是 app 模块的 gradle 构建脚本，这个文件中会指定很多项目构建相关的配置

- app/proguard-rules.pro：指定项目代码的混淆规则

### AndroidManifest.xml

AndroidManifest.xml 中 声明了 MainActivity 是这个 项目的主 Activity ，在手机上点击应用图标，首先启动的就是这个 Activity 。

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

### MainActivity

MainActivity 是继承自 AppCompatActivity 的。AppCompatActivity 是 AndroidX 中提供的一种向下兼容的 Activity ，可以使 Activity 在不同系统版本中的功能保持一致性。Activity 类是 Android 系统提供的一个基类，我们项目中所有自定义的 Activity 都必须 继承它或者它的子类才能拥有 Activity 的特性(AppCompatActivity 是 Activity 的子类)。

### res

Android 程序的设计讲究逻辑和视图分离，因此是不推荐在 Activity 中直接编写界面的。更加通用的做法是，在布局文件中编写界面，然后在 Activity 中引入进来。 比如在 onCreate 方法中执行 `setContentView(R.layout.activity_main)`

res 中定义的资源，我们有以下两种方式来引用它。

- 在代码中通过 R.string.app_name 可以获得该字符串的引用。

- 在 XML 中通过@string/app_name 可以获得该字符串的引用。

基本的语法就是上面这两种方式，其中 string 部分是可以替换的，如果是引用的图片资源就可 以替换成 drawable，如果是引用的应用图标就可以替换成 mipmap，如果是引用的布局文件就 可以替换成 layout，以此类推。

## 日志

Android 中的日志工具类是 Log(android.util.Log)，可以用来打印一些信息帮助我们调试代码。

在 Logcat 中查看：package:mine 就可以看到当前应用的日志信息，也可以使用 level:debug 来查看不同的日志等级。

## 其他

Mac 上是 Ctrl + Shift + R 重新运行 main()函数，Ctrl + R 可以重新运行 Android 程序。
