## Activity

Activity 是所有 Android 应用程序的门面，凡是在应用中你看得到 的东西，都是放在 Activity 中的。所有自定义的 Activity 都必须 继承它或者它的子类才能拥有 Activity 的特性。

AppCompatActivity 是 AndroidX 中提供的一种向下兼容的 Activity ，可以使 Activity 在不同系统版本中的功能保持一致性。

必须重写 onCreate 方法。

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout);
    }
}
```

## 使用 ViewBinding

findViewById()方法的作用就是获取布局文件中控件的实例，比较繁琐。

我们可以在 module 的 build.gradle 中开启 viewBinding

```xml
buildFeatures {
    viewBinding true
}
```

初始化 binding，并 setContentView 为 binding.root。Binding 类的命名规则是将布局文件按驼峰方式重命名后，再加上 Binding 作为结尾。

```kotlin
// 老式写法
setContentView(R.layout.first_layout);
val button1: Button = findViewById(R.id.button1)
button1.setOnClickListener {

}

// 新写法，Activity
class NewsContentActivity : AppCompatActivity() {
    private var _binding: ActivityNewsContentBinding? = null
    private val binding = _binding!!
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _binding = ActivityNewsContentBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}

// 新写法， Fragment
class NewsContentFragment : Fragment() {
    private var _binding: NewsContentFragBinding? = null
    private val binding = _binding!!
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        _binding = NewsContentFragBinding.inflate(layoutInflater, container, false)
        return binding.root
    }
}
```

## Toast 显示

```kotlin
Toast.makeText(this, "You clicked Button 1", Toast.LENGTH_SHORT).show()
```

## 使用菜单 Menu

需要建立对应的 menu 资源文件，里面包含 item

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/add_item"
        android:title="Add"/>
    <item
        android:id="@+id/remove_item"
        android:title="Remove"/>
</menu>
```

然后在 Activity 中复写 onCreateOptionsMenu 方法，return true 则显示菜单，false 不显示。

重写 onOptionsItemSelected()方法，来定义菜单响应事件。

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {
    menuInflater.inflate(R.menu.main, menu)
    return true
}

override fun onOptionsItemSelected(item: MenuItem): Boolean {
    when (item.itemId) {
        R.id.add_item -> Toast.makeText(this, "You clicked Add", Toast.LENGTH_SHORT).show()
        R.id.remove_item -> Toast.makeText(this, "You clicked Remove", Toast.LENGTH_SHORT).show()
    }
    return true
}
```

## 销毁 Activity

返回键，或者调用 finish()方法。

## 页面跳转 Intent

Intent 是 Android 程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。

Intent 一般可用于启动 Activity、启动 Service 以 及发送广播等场景。

### 显式 Intent

通过 Intent()方法，传入当前 Activity 的 context this 和目标 Activity 的 class，然后`startActivity(intent)`

SecondActivity::class.java 的写法就相当于 Java 中 SecondActivity.class 的写法。

```kotlin
button1.setOnClickListener {
    val intent = Intent(this, SecondActivity::class.java)
    startActivity(intent)
}
```

### 隐式 Intent

不在 kotlin 代码中明确指出想要启动哪一个 Activity，而是在 AndroidManifest.xml 文件中给该 Activity 添加 intent-filter，指定 action 和 category 等信息，然后在 kotlin 代码中，指定该 action，intent 添加 category，再使用`startActivity(intent)`跳转。

只有`<action>`和`<category>`中的内容同时匹配 Intent 中指定的 action 和 category 时，这个 Activity 才能响应该 Intent 。android.intent.category.DEFAULT 是一种默认的 category， 在调用 startActivity()方法的时候会自动将这个 category 添加到 Intent 中。

intent.addCategory("com.example.activitytest.MY_CATEGORY")方法来添加 category。

使用隐式 Intent ，不仅可以启动自己程序内的 Activity，还可以启动其他程序的 Activity，这就 使多个应用程序之间的功能共享成为了可能。

```xml
<activity android:name=".SecondActivity" >
    <intent-filter>
        <action android:name="com.example.activitytest.ACTION_START" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

```kotlin
// 程序内部的跳转
button1.setOnClickListener {
    val intent = Intent("com.example.activitytest.ACTION_START")
    startActivity(intent)
}

// 程序外部跳转
button1.setOnClickListener {
    // 系统内置浏览器
    val intent = Intent(Intent.ACTION_VIEW)
    intent.data = Uri.parse("https://www.baidu.com")
    startActivity(intent)
}

button1.setOnClickListener {
    // 拨打电话
    val intent = Intent(Intent.ACTION_DIAL)
    intent.data = Uri.parse("tel:10086")
    startActivity(intent)
}
```

### intent 传输数据

putExtra()方法可以把我们想要传递的数据暂存在 Intent 中，在启动另一个 Activity 后，只需要把这些数据从 Intent 中取出就可以了。使用 getStringExtra / getIntExtra / getBooleanExtra 等

```kotlin
// 第一个activity中
intent.putExtra("extra_data", data)

// 第二个activity中
val extraData = intent.getStringExtra("extra_data")
```

### 返回上一个 Activity，并携带数据

返回上一个 Activity，只需要按一下 back 键就行，或者是主动 finish()销毁该 Activity。

如果需要返回的时候携带数据，则需要在开始打开新的 Activity 的时候，就用`startActivityForResult(intent, 1)` 来打开。在第二个 Activity 中使用 setResult 即可向上一个 Activity 返回数据，同时第一个 Activity 监听数据，重写 onActivityResult 方法。

```kotlin
// 第一个Activity
button1.setOnClickListener {
    val intent = Intent(this, SecondActivity::class.java)
    startActivityForResult(intent, 1)
}

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    // 第一个参数requestCode，是我们在启动Activity 时传入的请求码; 第二个参数resultCode，即我们在返回数据时传入的处理结果; 第三个参数data，即携带着返回数据的Intent 。
    super.onActivityResult(requestCode, resultCode, data)
    when (requestCode) {

        1 -> if (resultCode == RESULT_OK) {
            val returnedData = data?.getStringExtra("data_return")
            Log.d("FirstActivity", "returned data is $returnedData")
        }
    }
}

// 第二个Activity
button2.setOnClickListener {
    val intent = Intent()
    intent.putExtra("data_return", "Hello FirstActivity")
    setResult(RESULT_OK, intent)
    finish()
}
```

如果用户在 SecondActivity 中并不是通过点击按钮，而是通过按下 Back 键回到 FirstActivity ，这样可以重写 onBackPressed()方法来返回数据。

```kotlin
override fun onBackPressed() {
    val intent = Intent()
    intent.putExtra("data_return", "Hello FirstActivity")
    setResult(RESULT_OK, intent)
    finish()
}
```

### 使用 Intent 传递对象

使用 Intent 来传递对象通常有两种实现方式：Serializable 和 Parcelable。

序列化后的对象可以在网络上进行传输，也可以存储到本地，让一个类去实现 Serializable 这个接口就可以。

getSerializableExtra()方法来获取通过参数传递过来的序列化对象，接着再将它向下转型成 Person 对象。

Parcelable 方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是 Intent 所支持的数据类型，这样就能实现传递对象的功能了。

## Activity 的生命周期

Activity 的存储是栈结构，每启动一个新的 Activity ，该 Activity 会入栈，覆盖在原 Activity 之上，然后点击 Back 键会销毁最上面的 Activity ，下面的一个 Activity 就会重新显示出来。

### Activity 的状态

运行状态： 当一个 Activity 位于返回栈的栈顶时，Activity 就处于运行状态。

暂停状态：当一个 Activity 不再处于栈顶位置，但仍然可见时，Activity 就进入了暂停状态。因为并不是每一个 Activity 都会占满整个屏幕，比如对话框形式的 Activity 只会占用屏幕中间的部分区域。

停止状态：当一个 Activity 不再处于栈顶位置，并且完全不可见的时候，就进入了停止状态。系统仍然会为这种 Activity 保存相应的状态和成员变量，但是这并不是完全可靠的，当其他地方需要内存时，处于停止状态的 Activity 有可能会被系统回收。

销毁状态：一个 Activity 从返回栈中移除后就变成了销毁状态。

### 生命周期方法

onCreate()： 每个 Activity 中都重写了这个方 法，它会在 Activity 第一次被创建的时候调用。你应该在这个方法中完成 Activity 的初始化 操作，比如加载布局、绑定事件等。

onStart()：这个方法在 Activity 由不可见变为可见的时候调用。

onResume()：这个方法在 Activity 准备好和用户进行交互的时候调用。此时的 Activity 一 定位于返回栈的栈顶，并且处于运行状态。

onPause()：这个方法在系统准备去启动或者恢复另一个 Activity 的时候调用。我们通常会在这个方法中将一些消耗 CPU 的资源释放掉，以及保存一些关键数据，但这个方法的执行速度一定要快，不然会影响到新的栈顶 Activity 的使用。

onStop()：这个方法在 Activity 完全不可见的时候调用。它和 onPause()方法的主要区 别在于，如果启动的新 Activity 是一个对话框式的 Activity ，那么 onPause()方法会得到执 行，而 onStop()方法并不会执行。

onDestroy()：这个方法在 Activity 被销毁之前调用，之后 Activity 的状态将变为销毁状 态。

onRestart()：这个方法在 Activity 由停止状态变为运行状态之前调用，也就是 Activity 被重新启动了。

**完整生存期**：Activity 在 onCreate()方法和 onDestroy()方法之间所经历的就是完整生存期。一般情况下，一个 Activity 会在 onCreate()方法中完成各种初始化操作，而在 onDestroy()方法中完成释放内存的操作。

**可见生存期**：Activity 在 onStart()方法和 onStop()方法之间所经历的就是可见生存期。在可见生存期内，Activity 对于用户总是可见的，即便有可能无法和用户进行交互。我们可以通过这两个方法合理地管理那些对用户可见的资源。比如在 onStart()方法中对资源进行加载，而在 onStop()方法中对资源进行释放，从而保证处于停止状态的 Activity 不会占用过多内存。

**前台生存期**：Activity 在 onResume()方法和 onPause()方法之间所经历的就是前台生存期。在前台生存期内，Activity 总是处于运行状态，此时的 Activity 是可以和用户进行交互的，我们平时看到和接触最多的就是这个状态下的 Activity 。

### Dialog Activity

```xml
<activity
    android:name=".DialogActivity"
    android:theme="@style/Theme.AppCompat.Dialog">
</activity>
```

### Activity 被回收了怎么办

用户在 Activity A 的基础上启动了 Activity B ，Activity A 就进入了停止状态，这个时候由于系统内存不足，将 Activity A 回收掉了，然后用户按下 Back 键返回 Activity A ，会出现什么情况呢？其实还是会正常显示 Activity A 的，只不过这时并不会执行 onRestart()方法，而是会执行 Activity A 的 onCreate()方法，因为 Activity A 在这种情况下会被重新创建一次，数据和状态都会消失。

Activity 中提供了一个 onSaveInstanceState()回调方法，这个方法可以保证在 Activity 被回收之前一定会被调用，因此我们可以通过这个方法来解决问题。方法有一个 Bundle 类型的参数用于保存数据，类似于 Intent，可以用 putString，putInt 等保存数据。

onCreate()方法其实也有一个 Bundle 类型的参数。这个参数在一般情况下都是 null，但是如果在 Activity 被系统回收之前，你通过 onSaveInstanceState()方法保存数据，这个参数就会带有之前保存的全部数据，我们只需要再通过相应的取值方法将数据取出即可。

```kotlin
override fun onSaveInstanceState(state: Bundle) {
    super.onSaveInstanceState(state)
    val tempData = "Something you just typed"
    state.putString("data_key", tempData)
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    if (savedInstanceState != null) {
        val tempData = savedInstanceState.getString("data_key")
        Log.d(tag, "tempData is $tempData")
    }
}
```

## Activity 的启动模式

启动模式一共有 4 种，分别是 standard、singleTop、
singleTask 和 singleInstance ，可以在 AndroidManifest.xml 中通过给`<activity>`标签指定 android:launchMode 属性来选择启动模式。

**standard** 模式，是 Activity 默认的启动模式，在不进行显式指定的情况下，所有 Activity 都会自动使用这种启动模式。在 standard 模式下，每当启动一个新的 Activity ，它就会在返回栈中入栈，并处于栈顶的位置。对于使用 standard 模式的 Activity ，系统不会在乎这个 Activity 是否已经在返回栈中存在，是否在栈顶，每次启动都会创建一个该 Activity 的新实例，放入栈顶。

**singleTop** 模式，在启动 Activity 时如果发现栈顶已经是该 Activity，则认为可以直接使用它，不会再创建新的 Activity 实例。如果该 Activity 并没有处于栈顶的位置，还是会创建一个该 Activity 的新实例，放入栈顶。

**singleTask** 模式，每次启动该 Activity 时，系统首先会在返回栈中检查是否存在该 Activity 的实例，如果发现已经存在则直接使用该实例，并把在这个 Activity 之上的所有其他 Activity 统统出栈，如果没有发现就会创建一个新的 Activity 实例。

**singleInstance** 模式，会有一个单独的返回栈来管理这个 Activity ，不管是哪个应用程序来访问这个 Activity ，都共用同一个返回栈，也就解决了共享 Activity 实例的问题。

## Activity 的最佳实践技巧

### 时刻知晓当前页面是哪个 Activity

创建一个 BaseActivity 基类，在 onCreate 中打印 javaClass.simpleName，获取当前实例的类名。让所有的 Activity 都继承这个。观察 Logcat 中的打印信息，每当我们进入一个 Activity 的界面，该 Activity 的类名就会被打印出来。

Kotlin 中的 javaClass 表示获取当前实例的 Class 对象，相当于在 Java 中调用 getClass()方法;  
Kotlin 中的 BaseActivity::class.java 表示获取 BaseActivity 类的 Class 对象，相当于在 Java 中调用 BaseActivity.class。

```kotlin
open class BaseActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("BaseActivity", javaClass.simpleName)
    }
}
```

### 随时随地退出程序

如果 Activity 跳转很多，想退出程序是非常不方便的，需要多次按 Back 键才行(按 Home 键只是把程序挂起，并没有退出程序)。如果我们的程序需要注销或者退出的功能，需要销毁所有 Activity。可以创建一个管理 Activity 的单例类。

```kotlin
object ActivityCollector {
    private val activities = ArrayList<Activity>()

    fun addActivity(activity: Activity) {
        activities.add(activity)
    }
    fun removeActivity(activity: Activity) {
        activities.remove(activity)
    }
    fun finishAll() {
        for (activity in activities) {
            if (!activity.isFinishing) {
                activity.finish()
            }
        }
        activities.clear()
    }
}
```

接下来修改 BaseActivity 中的代码

```kotlin
open class BaseActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("BaseActivity", javaClass.simpleName)
        ActivityCollector.addActivity(this)
    }
    override fun onDestroy() {
        super.onDestroy()
        ActivityCollector.removeActivity(this)
    }
}
```

从此以后，不管你想在什么地方退出程序，只需要调用 ActivityCollector.finishAll() 方法就可以了。

还可以在销毁所有 Activity 的代码后面再加上杀掉当前进程的代码，以保证程序完全退出。killProcess()方法用于杀掉一个进程，它接收一个进程 id 参数，可以通过 myPid()方法来获得当前程序的进程 id。需要注意的是，killProcess()方法只能用于杀掉当前程序的进程，不能用于杀掉其他程序。

```kotlin
android.os.Process.killProcess(android.os.Process.myPid())
```

finishAffinity()：关闭你当前 Activity 所在栈中的所有的 Activity，如果 Activity 不在同一个栈，其他栈中的会保留。如果所有 activity 都保存在默认栈中，则使用该方法会直接退出程序。

system.exit(0)，退出程序。

### 启动 Activity 的最佳写法

修改即将需要跳转的 SecondActivity 中的代码

```kotlin
class SecondActivity : BaseActivity() {
    ...
    companion object {
        fun actionStart(context: Context, data1: String, data2: String) {
            val intent = Intent(context, SecondActivity::class.java)
            intent.putExtra("param1", data1)
            intent.putExtra("param2", data2)
            context.startActivity(intent)
        }
    }
}
```

然后在第一个 Activity 中，直接调用该方法。养成一个良好的习惯，给你编写的每个 Activity 都添加类似的启动方法，这样可以让启动 Activity 变得非常简单。

```kotlin
button1.setOnClickListener {
    SecondActivity.actionStart(this, "data1", "data2")
}
```

### 全局获取 Context 的技巧

Android 提供了一个 Application 类，每当应用程序启动的时候，系统就会自动将这个类进行初始化。而我们可以定制一个自己的 Application 类，以便于管理程序内一些全局的状态信息，比如全局 Context。

```kotlin
class MyApplication : Application() {
    companion object {
        lateinit var context: Context
    }

    override fun onCreate() {
        super.onCreate()
        context = applicationContext
    }
}
```

接下来我们还需要告知系统，当程序启动的时候应该初始化 MyApplication 类，而不是默认的 Application 类。这一步也很简单，在 AndroidManifest.xml 文件的`<application>`标签下进行指定就可以了。

之后不管你想在项目的任何地方使用 Context，只需要调用一下 MyApplication.context 就可以。

## Application 类

Application 类是 Android 应用程序中的一个重要组件，它是所有应用程序组件的基类。

它的一个主要作用是提供应用程序的全局上下文。这个上下文可以在整个应用程序中使用，而不受活动（Activity）或服务（Service）生命周期的限制。这使得你可以在任何地方访问应用程序级别的资源和数据。

也可用于管理应用程序的全局状态。你可以在 Application 类中创建成员变量和方法，用于存储和管理应用程序范围内的数据，例如全局配置信息、共享数据等。

你可以创建自定义的 Application 子类，以便扩展其功能。要使用自定义的 Application 类，需要在 AndroidManifest.xml 文件中的`<application>`元素的 android:name 属性中指定你的自定义 Application 类的名称。

生命周期与应用程序一致，它的实例在应用程序启动时创建，并在应用程序退出时销毁。所以也只有 `onCreate()` 和 `onTerminate()`。
