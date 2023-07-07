## Fragment

Fragment 是一种可以嵌入在 Activity 当中的 UI 片段，它能让程序更加合理和充分地利用大屏幕的空间，因而在平板上应用得非常广泛。

它和 Activity 很像，都能包含布局，都有自己的生命周期。可以将 Fragment 理解成一个迷你型的 Activity，虽然这个迷你型的 Activity 有可能和普通的 Activity 是一样大的。

## 基本创建和使用

先创建布局文件 xml，跟之前创建布局一样。

然后创建 Fragment 类，让他继承自 AndroidX 库中的 Fragment。

```kotlin
class LeftFragment : Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
            savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.left_fragment, container, false)
    }
}
```

最后，在 Activity 中使用这个 Fragment，修改 activity 的布局文件，添加`<fragment>`标签或者`<androidx.fragment.app.FragmentContainerView>`，通过 android:name 属性来显式声明要添加的 Fragment 类名，要将类的包名也加上。

```xml
<androidx.fragment.app.FragmentContainerView
    android:id="@+id/leftFrag"
    android:name="com.example.fragmenttest.LeftFragment"
    android:layout_width="0dp"
    android:layout_height="match_parent"
    android:layout_weight="1" />
```

## 动态添加 Fragment

Fragment 真正的强大之处在于，它可以在程序运行时动态地添加到 Activity 当中。根据具体情况来动态地添加 Fragment ，你就可以将程序界面定制得更加多样化。

一共可分为 6 步：

- 在主布局中，添加 FrameLayout 布局，用来承接动态添加或者替换的 Fragment

  ```xml
  <FrameLayout
      android:id="@+id/rightLayout"
      android:layout_width="0dp"
      android:layout_height="match_parent"
      android:layout_weight="1" >
  </FrameLayout>
  ```

- 创建待添加 Fragment 的实例。

- 获取 FragmentManager ，在 Activity 中可以直接调用 getSupportFragmentManager()
  方法获取。
- 开启一个事务，通过调用 beginTransaction() 方法开启。

- 向容器内添加或替换 Fragment ，一般使用 replace()方法实现，需要传入容器的 id 和待添加的 Fragment 实例。

- 提交事务，调用 commit()方法来完成。

  ```kotlin
  val fg = LeftFragment()
  val fragmentManager = supportFragmentManager
  val transaction = fragmentManager.beginTransaction()
  transaction.replace(R.id.rightLayout, fg)
  transaction.commit()
  ```

## 返回上一个 Fragment

FragmentTransaction 中提供了一个 addToBackStack()方法，可以用于将一个事务添加到返回栈中。按下 Back 键可以回到上一个 Fragment。

## Fragment 和 Activity 之间的交互

FragmentManager 提供了一个类似于 findViewById()的方法 findFragmentById，专门用于从布局文件中获取 Fragment 的实例，然后就能轻松地调用 Fragment 里的方法了。

使用 viewBinding 的话得用 androidx.fragment.app.FragmentContainerView

```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.leftFrag) as LeftFragment
```

在每个 Fragment 中都可以通过调用 getActivity()方法来得到和当前 Fragment 相关联的 Activity 实例。由于 getActivity()方法有可能返回 null，因此我们需要先进行一个判空处理。

```kotlin
if (activity != null) {
    val mainActivity = activity as MainActivity
}
```

## Fragment 的生命周期

Fragment 也有自己的生命周期，并且它和 Activity 的生命周期很像。

### Fragment 状态

运行状态：当一个 Fragment 所关联的 Activity 正处于运行状态时，该 Fragment 也处于运行状态。

暂停状态：当一个 Activity 进入暂停状态时(由于另一个未占满屏幕的 Activity 被添加到了栈顶)，与它相关联的 Fragment 就会进入暂停状态。

停止状态：当一个 Activity 进入停止状态时，与它相关联的 Fragment 就会进入停止状态，或者通过调用 FragmentTransaction 的 remove()、replace()方法将 Fragment 从 Activity 中移除，但在事务提交之前调用了 addToBackStack()方法，这时的 Fragment 也会进入停止状态。总的来说，进入停止状态的 Fragment 对用户来说是完全不可见的，有可能会被系统回收。

销毁状态：Fragment 总是依附于 Activity 而存在，因此当 Activity 被销毁时，与它相关联的 Fragment 就会进入销毁状态。或者通过调用 FragmentTransaction 的 remove()、 replace()方法将 Fragment 从 Activity 中移除，但在事务提交之前并没有调用 addToBackStack()方法，这时的 Fragment 也会进入销毁状态。

### 生命周期函数

Activity 中有的回调方法， Fragment 中基本上也有，而且 Fragment 还提供了一些附加的回调方法。

onAttach()：当 Fragment 和 Activity 建立关联时调用。

onCreateView()：为 Fragment 创建视图(加载布局)时调用。

onActivityCreated()：确保与 Fragment 相关联的 Activity 已经创建完毕时调用。

onDestroyView()：当与 Fragment 关联的视图被移除时调用。 onDetach():当 Fragment 和 Activity 解除关联时调用。

### Fragment 被回收后保存数据

也可以通过 onSaveInstanceState()方法来保存数据，因为进入停止状态的 Fragment 有可能在系统内存不足的时候被回收。保存下来的数据在 onCreate()、onCreateView()和 onActivityCreated()这 3 个方法中你都可以重新得到，它们都含有一个 Bundle 类型的 savedInstanceState 参数。

## 动态加载布局的技巧

### 限定符

平板一般采用双页模式，左侧显示列表，右侧显示内容。怎样才能在运行时判断程序应该是使用双页模式还是单页模式呢?这就需要借助限定符 (qualifier )来实现了。

新建一个 layout-large 文件夹，同样创建一个 activity_main.xml 布局，里面采用平板的左右设计，包含两个 fragment。而 layout/activity_main.xml 中就是普通的手机布局，只包含一个 fragment。

其中，large 就是一个限定符，那些屏幕被认为是 large 的设备就会自动加载 layout-large 文件夹下的布局，小屏幕的设备则还是会加载 layout 文件夹下的布局。

Android 中一些常见的限定符：

| 屏幕特征 | 限定符 | 描述                                             |
| -------- | ------ | ------------------------------------------------ |
| 大小     | small  | 提供给小屏幕设备的资源                           |
| 大小     | normal | 提供给中等屏幕设备的资源                         |
| 大小     | large  | 提供给大屏幕设备的资源                           |
| 大小     | xlarge | 提供给超大屏幕设备的资源                         |
| -        | -      | -                                                |
| 分辨率   | ldpi   | 提供给低分辨率设备的资源(120 dpi 以下 )          |
| 分辨率   | mdpi   | 提供给中等分辨率设备的资源(120 dpi ~ 160 dpi )   |
| 分辨率   | hdpi   | 提供给高分辨率设备的资源(160 dpi ~ 240 dpi )     |
| 分辨率   | xhdpi  | 提供给超高分辨率设备的资源(240 dpi ~ 320 dpi )   |
| 分辨率   | xxhdpi | 提供给超超高分辨率设备的资源(320 dpi ~ 480 dpi ) |
| -        | -      | -                                                |
| 方向     | land   | 提供给横屏设备的资源                             |
| 方向     | port   | 提供给竖屏设备的资源                             |

### 最小宽度限定符

有时候希望可以更加灵活地为不同设备加载布局，不管它们是不是被系统认定为 large ，这时就可以使用最小宽度限定符(smallest-width qualifier)。最小宽度限定符允许我们对屏幕的宽度指定一个最小值(以 dp 为单位)，然后以这个最小值为临界点，屏幕宽度大于这个值的设备就加载一个布局，屏幕宽度小于这个值的设备就加载另一个布局。

需要在 res 目录创建新的文件夹：layout-sw600dp，在其中添加对应布局即可。当程序运行在屏幕宽度大于等于 600 dp 的设备上时，会加载 layout-sw600dp/activity_main.xml 布局。
