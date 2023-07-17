## Jetpack

Jetpack 是一个开发组件工具集，它的主要目的是帮助我们编写出更加简洁的代码，并简化我们的开发过程。Jetpack 中的组件有一个特点，它们大部分不依赖于任何 Android 系统版本，这意味着这些组件通常是定义在 AndroidX 库当中的，并且拥有非常好的向下兼容性。

Jetpack 的家族还是非常庞大的，主要由基础、架构、行为、界面这 4 个部分组成。像通知、权限、Fragment 都属于 Jetpack。由此可见，Jetpack 并不全是些新东西，只要是能够帮助开发者更好更方便地构建应用程序的组件，Google 都将其纳入了 Jetpack。

在这么多的组件当中，最需要我们关注的其实还是架构组件。目前 Android 官方最为推荐的项目架构就是 MVVM，因而 Jetpack 中的许多架构组件是专门为 MVVM 架构量身打造的。如 ViewModel 等。

## ViewModel

在传统的开发模式下，Activity 的任务实在是太重了，既要负责逻辑处理，又要控制 UI 展示。如果在大型项目中仍然使用这种写法的话，那么这个项目将会变得非常臃肿并且难以维护，因为没有任何架构上的划分。

而 ViewModel 的一个重要作用就是可以帮助 Activity 分担一部分工作，它是专门用于存放与界面相关的数据的。只要是界面上能看得到的数据，它的相关变量都应该存放在 ViewModel 中，而不是 Activity 中，这样可以在一定程度上减少 Activity 中的逻辑。

当手机发生横竖屏旋转的时候， Activity 会被重新创建，同时存放在 Activity 中的数据也会丢失。而 ViewModel 的生命周期和 Activity 不同，它可以保证在手机屏幕发生旋转的时候不会被重新创建，只有当 Activity 退出的时候才会跟着 Activity 一起销毁。

比较好的编程规范是给每一个 Activity 和 Fragment 都创建一个对应的 ViewModel， 并让它继承自 ViewModel 。使用的时候不可以直接去创建 ViewModel 的实例，而是一定要通过 ViewModelProvider 来获取 ViewModel 的实例。

```kotlin
class MainViewModel : ViewModel() { }

// 使用
viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
plusOneBtn.setOnClickListener {
    viewModel.counter++
    infoText.text = viewModel.counter.toString()
}
```

### 向 ViewModel 传递参数

借助 ViewModelProvider.Factory 实现。

```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
    var counter = countReserved
}

class MainViewModelFactory(private val countReserved: Int) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return MainViewModel(countReserved) as T
    }
}

// 使用
viewModel = ViewModelProvider(this, MainViewModelFactory(countReserved))
    .get(MainViewModel::class.java)
clearBtn.setOnClickListener {
    viewModel.counter = 0
    refreshCounter()
}
```

## Lifecycles

在编写 Android 应用程序的时候，可能会经常遇到需要感知 Activity 生命周期的情况。比如说，某个界面中发起了一条网络请求，但是当请求得到响应的时候，界面或许已经关闭了，这个时候就不应该继续对响应的结果进行处理。因此，我们需要能够时刻感知到 Activity 的生命周期， 以便在适当的时候进行相应的逻辑控制。

在一个 Activity 中去感知它的生命周期非常简单，而如果要在一个非 Activity 的类中去感知 Activity 的生命周期，可以通过在 Activity 中嵌入一个隐藏的 Fragment 来进行感知，或者通过手写监听器的方式来进行感知。

Lifecycles 组件就是为了解决这个问题而出现的，它可以让任何一个类都能轻松感知到 Activity 的生命周期，同时又不需要在 Activity 中编写大量的逻辑处理。新建一个 MyObserver 类，并让 它实现 LifecycleObserver 接口，LifecycleObserver 是一个空方法接口，不用重写任何方法。我们可以在 MyObserver 中定义任何方法，但是如果想要感知到 Activity 的生命周期，还得借助额外的注解功能才行。比如这里还是定义 activityStart()和 activityStop()这两个方法。

```kotlin
class MyObserver : LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun activityStart() {
        Log.d("MyObserver", "activityStart")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun activityStop() {
        Log.d("MyObserver", "activityStop")
    }
}

// 在activity中使用
lifecycle.addObserver(MyObserver())
```

在方法上使用了@OnLifecycleEvent 注解，并传入了一种生命周期事件。生命周期事件的类型一共有 7 种：ON_CREATE、ON_START、ON_RESUME、ON_PAUSE、ON_STOP 和 ON_DESTROY 分别匹配 Activity 中相应的生命周期回调；另外还有一种 ON_ANY 类型，表示可以匹配 Activity 的任何生命周期回调。

上述代码中的 activityStart()和 activityStop()方法会分别在 Activity 的 onStart()和 onStop()触发的时候执行。

如果想主动获知当前的生命周期状态，只需要在 MyObserver 的构造函数中将 Lifecycle 对象传进来即可。就可以在任何地方调用 lifecycle.currentState 来主动获知当前的生命周期状态。

## LiveData

LiveData 是 Jetpack 提供的一种响应式编程组件，它可以包含任何类型的数据，并在数据发生变化的时候通知给观察者。LiveData 特别适合与 ViewModel 结合在一起使用，虽然它也可以单独用在别的地方，但是在绝大多数情况下，它是使用在 ViewModel 当中的。

前面我们一直使用的都是在 Activity 中手动获取 ViewModel 中的数据这种交互方式，但是 ViewModel 却无法将数据的变化主动通知给 Activity。如果我们将计数器的计数使用 LiveData 来包装，然后在 Activity 中去观察它，就可以 主动将数据变化通知给 Activity 了。

MutableLiveData 是一种可变的 LiveData ，它的用法很简单，主要有 3 种读写数据的方法，分别是 getValue()、setValue()和 postValue()方法。setValue()方法用于给 LiveData 设置数据，但是只能在主线程中调用，postValue()方法用于在非主线程中给 LiveData 设置数据。

```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
    val counter = MutableLiveData<Int>()
    init {
        counter.value = countReserved
    }
    fun plusOne() {
        val count = counter.value ?: 0
        counter.value = count + 1
    }
    fun clear() {
        counter.value = 0
    }
}

// 使用
plusOneBtn.setOnClickListener {
    viewModel.plusOne()
}
clearBtn.setOnClickListener {
    viewModel.clear()
}
// 观察到变化后直接修改
viewModel.counter.observe(this, Observer { count ->
    infoText.text = count.toString()
})
```

### map 和 switchMap

Transformations 的 map()方法来对 LiveData 的数据类型进行转换。map()方法接收两个参数：第一个参数是原始的 LiveData 对象；第二个参数是一个转换函数，在转换函数里编写具体的转换逻辑即可。

如 果 ViewModel 中的某个 LiveData 对象是调用另外的方法获取的，那么我们就可以借助 switchMap()方法，将这个 LiveData 对象转换成另外一个可观察的 LiveData 对象。

```kotlin
val userName: LiveData<String> = Transformations.map(userLiveData) { user ->
    "${user.firstName} ${user.lastName}"
}

val user: LiveData<User> = Transformations.switchMap(userIdLiveData) { userId ->
    Repository.getUser(userId)
}
```

## ORM 框架

Room

## WorkManager

基本上 Android 系统每发布一个新版本，后台权限都会被进一步收紧，开发者就很难受了，到底该如何编写后台代码才能保证应用程序在不同系统版本上的兼容性呢？为了解决这个问题，Google 推出了 WorkManager 组件。

WorkManager 很适合用于处理一些要求定时执行的任务，它可以根据操作系统的版本自动选择底层是使用 AlarmManager 实现还是 JobScheduler 实现，从而降低了我们的使用成本。另外，它还支持周期性任务、链式任务处理等功能，是一个非常强大的工具。

WorkManager 和 Service 并不相同，也没有直接的联系。Service 是 Android 系统的四大组件之一，它在没有被销毁的情况下是一直保持在后台运行的。而 WorkManager 只是一个处理定时任务的工具，它可以保证即使在应用退出甚至手机重启的情况下，之前注册的任务仍然将会得到执行。

使用 WorkManager 注册的周期性任务不能保证一定会准时执行，这并不是 bug，而是系统为了减少电量消耗，可能会将触发时间临近的几个任务放在一起执行，这样可以大幅度地减少 CPU 被唤醒的次数，从而有效延长电池的使用时间。

WorkManager 的基本用法其实非常简单，主要分为以下 3 步:

(1) 定义一个后台任务，并实现具体的任务逻辑;

(2) 配置该后台任务的运行条件和约束信息，并构建后台任务请求;

(3) 将该后台任务请求传入 WorkManager 的 enqueue()方法中，系统会在合适的时间运行。

```kotlin
class SimpleWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    override fun doWork(): Result {
        Log.d("SimpleWorker", "do work in SimpleWorker")
        return Result.success()
    }
}

val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()

WorkManager.getInstance(context).enqueue(request)
```

doWork()方法不会运行在主线程当中，因此你可以放心地在这里执行耗时逻辑。成功就返回 Result.success()，失败就返回 Result.failure()

### 高级用法

让后台任务在指定的延迟时间后运行，只需要借助 setInitialDelay()方法就可以了。

给后台任务请求添加标签 addTag("simple") ，我们可以通过标签来取消后台任务请求。

没有标签，也可以通过 id 来取消后台任务请求。

一次性取消所有后台任务请求。

结合 setBackoffCriteria()方法来重新执行任务。

beginWith()方法用于开启一个链式任务，至于后面要接上什么样的后台任务，只需要使用 then()方法来连接即可。
