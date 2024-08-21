## 常用控件

### TextView

TextView 主要用于在界面上显示一段文本信息。属性有 id，layout_width，layout_height，text，gravity（对齐方式），textColor，textSize 等等

宽高的值：match_parent 表示让当前控件的大小和父布局的大小一样。wrap_content 表示让当前控件的大小能够刚好包含住里面的内容。固定值表示表示给控件指定一个固定的尺寸，单位一般用 dp。

android:background 用于为布局或控件指定一个背景。android:padding 表示给控件的周围加上补白，这样不至于让文本内容紧靠在边缘上; android:maxLines 设置为 1 表示让这个 TextView 只能单行显示; android:ellipsize 用于设定当文本内容超出控件宽度时文本的缩略方式。

```xml
<TextView
    android:id="@+id/textView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:textColor="#00ff00"
    android:textSize="24sp"
    android:text="This is TextView"/>
```

### Button

Android 系统默认会将按钮上的英文字母全部转换成大写，如果不想要，可以添加 `android:textAllCaps="false"` 这个属性。

```xml
<Button
    android:id="@+id/button"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Button" />
```

监听事件，两种写法。其实所有的控件都可以添加点击事件监听。

```kotlin
// 第一种
button.setOnClickListener {
    // 在此处添加逻辑
}

// 第二种，实现接口
class MainActivity : AppCompatActivity(), View.OnClickListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button.setOnClickListener(this)
    }
    override fun onClick(v: View?) {
        when (v?.id) {
            R.id.button -> {
                // 在此处添加逻辑
            }
        }
    }
}
```

### EditText

EditText 允许用户在控件里输入和编辑内容，并可以在程序中对这些内容进行处理。属性有 hint，相当于 placeholder，通过 android:maxLines 指定了 EditText 的最大行数。

通过 `val inputText = editText.text.toString()` 可以获取输入框的内容。

```xml
<EditText
    android:id="@+id/editText"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="Type something here"
    android:maxLines="2" />
```

### ImageView

ImageView 是用于在界面上展示图片的一个控件。现在最主流的手机屏幕分辨率大多是 xxhdpi 的，所以我们在 res 目录下新建一个 drawable-xxhdpi 目录，并将图片放进去。

xml 中通过 src 指定图片，也可以通过代码动态地更改 ImageView 中的图片 `imageView.setImageResource(R.drawable.img_2)` 。

```xml
<ImageView
    android:id="@+id/imageView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/img_1" />
```

### ScrollView

内容超出屏幕宽度或者高度时，滚动的显示屏幕外的内容

### ProgressBar

ProgressBar 用于在界面上显示一个进度条，表示我们的程序正在加载一些数据。默认是一个圆形的标志。通过 style 属性可以将它指定成水平进度条。通过 android:max 属性给进度条设置一个最大值，然后在代码中动态地更改进度条的进度。

```xml
<ProgressBar
    android:id="@+id/progressBar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    style="?android:attr/progressBarStyleHorizontal"
    android:max="100" />
```

### AlertDialog

AlertDialog 可以在当前的界面上显示一个对话框，这个对话框是置顶于所有界面元素之上的，能够屏蔽掉其他控件的交互能力，因此 AlertDialog 一般是用于提示一些非常重要的内容或者警告信息。

AlertDialog 并不需要到布局文件中创建，而是在代码中通过构造器（AlertDialog.Builder）来构造标题 setTitle、内容 setMessage、图标 setIcon 和按钮 setPositiveButton/setNegativeButton 等内容。通过 show 来展示，dismiss() 来关闭对话框。

内容区域可以是消息，可以是列表，也可以是自定义布局。如果是列表，用 setItems() 来设置数组。如果是自定义布局，通过 setView() 设置布局文件。

```kotlin
class MainActivity : AppCompatActivity(), View.OnClickListener {
    ...
    override fun onClick(v: View?) {
        when (v?.id) {
            R.id.button -> {
                AlertDialog.Builder(this).apply {
                    setTitle("This is Dialog")
                    setMessage("Something important.")
                    setCancelable(false)
                    setPositiveButton("OK") { dialog, which ->
                    }
                    setNegativeButton("Cancel") { dialog, which ->
                    }
                    show()
                }
            }
        }
    }
}
```

### 动态展示一个控件

Android 控件的可见属性通过 android:visibility 控制，可选值有 3 种: visible、invisible 和 gone，默认可见 visible。invisible 表示控件不可见，但是它仍然占据着原来的位置和大小。gone 则表示控件不仅不可见，而且不再占用任何屏幕空间。

我们可以通过代码来设置控件的可见性，使用的是 setVisibility() 方法，允许传入 View.VISIBLE、View.INVISIBLE 和 View.GONE 这 3 种值。

```kotlin
if (progressBar.visibility == View.VISIBLE) {
    progressBar.visibility = View.GONE
}
```

## 布局

### LinearLayout

LinearLayout 又称作线性布局，可以让控件在水平（horizontal）或者竖直（vertical）方向排列。通过 android:orientation 属性指定排列方向。

如果 LinearLayout 的排列方向是 horizontal，内部的控件就绝对不能将宽度指定为 match_parent，否则，单独一个控件就会将整个水平方向占满，其他的控件就没有可放置的位置了。同样的道理，如果 LinearLayout 的排列方向是 vertical，内部的控件就不能将高度指定为 match_parent。

控件上添加 android:layout_gravity 用于指定控件在布局中的对齐方式。当 LinearLayout 的排列方向是 horizontal 时，只有垂直方向上的对齐方式才会生效。因为此时水平方向上的长度是不固定的，每添加一个控件，水平方向上的长度都会改变，因而无法指定该方向上的对齐方式。同样的道理，当 LinearLayout 的排列方向是 vertical 时，只有水平方向上的对齐方式才会生效。

控件上添加 android:layout_weight 这个属性允许我们使用比例的方式来指定控件的大小，需要将宽度指定成了 0 dp。使用了 android:layout_weight 属性，此时控件的宽度就不应该再由 android:layout_width 来决定了。类似于 flex，指定为 1，控件会自动充满剩余空间，如果是两个，就平分。

### RelativeLayout

RelativeLayout 又称作相对布局，可以通过相对定位的方式让控件出现在布局的任何位置。

相对父布局定位：值是 true/false，表示贴着父布局的上下左右还是中间。

```xml
android:layout_alignParentTop
android:layout_alignParentBottom
android:layout_alignParentLeft
android:layout_alignParentRight
android:layout_centerInParent
```

相对于控件进行定位：值是某个控件的 id，表示放在这个控件的上下左右。

```xml
android:layout_above
android:layout_below
android:layout_toLeftOf
android:layout_toRightOf
```

另外一组相对于控件进行定位的属性，让控件跟目标控件的上下左右边缘对齐。

```xml
android:layout_alignTop
android:layout_alignBottom
android:layout_alignLeft
android:layout_alignRight
```

### FrameLayout

FrameLayout 又称作帧布局，它相比于前面两种布局就简单太多了，因此它的应用场景少了很多。这种布局没有丰富的定位方式，所有的控件都会默认摆放在布局的左上角。默认会重叠。

控件上可以使用 layout_gravity 属性来指定该控件在布局中的对齐方式，这和 LinearLayout 中的用法是相似的。

### ConstraintLayout

约束布局 ConstraintLayout 是一个 ViewGroup，可以在 API9 以上的 Android 系统使用它，它的出现主要是为了解决布局嵌套过多的问题，使用“相对定位”灵活地确定控件的位置和大小的一个布局。

相对定位约束：值是控件 id，表示控件的哪个位置与目标控件的哪个位置对齐。

```xml
app:layout_constraintTop_toTopOf=""           我的顶部和谁的顶部对齐
app:layout_constraintBottom_toBottomOf=""     我的底部和谁的底部对齐
app:layout_constraintLeft_toLeftOf=""         我的左边和谁的左边对齐
app:layout_constraintRight_toRightOf=""       我的右边和谁的右边对齐
app:layout_constraintStart_toStartOf=""       我的开始位置和谁的开始位置对齐
app:layout_constraintEnd_toEndOf=""           我的结束位置和谁的结束位置对齐
app:layout_constraintTop_toBottomOf=""        我的顶部位置在谁的底部位置
app:layout_constraintStart_toEndOf=""         我的开始位置在谁的结束为止
<!-- ...以此类推 -->

layout_constraintBaseline_toBaselineOf        我的文字基线跟谁的文字基线对齐
```

角度约束：一个控件在某个控件的某个角度的位置。

```xml
app:layout_constraintCircle=""         目标控件id
app:layout_constraintCircleAngle=""    对于目标的角度(0-360)
app:layout_constraintCircleRadius=""   到目标中心的距离
```

百分比偏移：让控件在父布局的水平方向或垂直方向的百分之多少的位置。

```xml
app:layout_constraintHorizontal_bias=""   水平偏移 取值范围是0-1的小数
app:layout_constraintVertical_bias=""     垂直偏移 取值范围是0-1的小数
```

控件内边距、外边距、GONE Margin：内边距和外边距的使用方式其实是和其他布局一致的。GONE Margin，当依赖的目标 view 隐藏时会生效的属性，例如 B 被 A 依赖约束，当 B 隐藏时 B 会缩成一个点，自身的 margin 效果失效，A 设置的 GONE Margin 就会生效

```xml
<!--  外边距  -->
android:layout_margin="0dp"
android:layout_marginStart="0dp"
android:layout_marginLeft="0dp"
android:layout_marginTop="0dp"
android:layout_marginEnd="0dp"
android:layout_marginRight="0dp"
android:layout_marginBottom="0dp"

<!--  内边距  -->
android:padding="0dp"
android:paddingStart="0dp"
android:paddingLeft="0dp"
android:paddingTop="0dp"
android:paddingEnd="0dp"
android:paddingRight="0dp"
android:paddingBottom="0dp"

<!--  GONE Margin  -->
app:layout_goneMarginBottom="0dp"
app:layout_goneMarginEnd="0dp"
app:layout_goneMarginLeft="0dp"
app:layout_goneMarginRight="0dp"
app:layout_goneMarginStart="0dp"
app:layout_goneMarginTop="0dp"
```

尺寸限制：用来限制最大、最小宽高度，这些属性只有在给出的宽度或高度为 wrap_content 时才会生效。

```xml
android:minWidth=""   设置view的最小宽度
android:minHeight=""  设置view的最小高度
android:maxWidth=""   设置view的最大宽度
android:maxHeight=""  设置view的最大高度
```

更多内容参考：<https://juejin.cn/post/6949186887609221133>

## 自定义控件

所用的所有控件都是直接或间接继承自 View 的，所用的所有布局都是直接或间接继承自 ViewGroup 的。

### 单独布局

比如我们需要实现页面的标题栏，正常需要在每个 Activity 中为这些控件单独编写一次事件注册的代码，比如标题栏中的返回按钮。比较繁琐，代码重复度高，所以我们用自定义控件。

在 layout 目录下新建一个 title.xml 布局，在其他布局中 include 这个文件即可。`<include layout="@layout/title" />`。（别忘了 supportActionBar?.hide()隐藏当前系统自带的标题栏）。

### 自定义

需要写一个类，在布局中引入自定义 TitleLayout 控件时就会调用这个构造函数。可以将点击事件定义在这里。

```kotlin
class TitleLayout(context: Context, attrs: AttributeSet) : LinearLayout(context, attrs) {
    init {
        LayoutInflater.from(context).inflate(R.layout.title, this)
        titleBack.setOnClickListener {
            val activity = context as Activity
            activity.finish()
        }
        titleEdit.setOnClickListener {
            Toast.makeText(context, "You clicked Edit button", Toast.LENGTH_SHORT).show()
        }
    }
}
```

使用时只需要在布局文件中添加这个自定义控件即可。

```xml
<com.example.uicustomviews.TitleLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

## ListView

ListView 可以显示列表数据，超出屏幕可以滚动。

```xml
<ListView
    android:id="@+id/listView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

指定数据，需要构建 adapter 对象，赋值给 listView 的 adapter。可以使用 ArrayAdapter 来构建。

android.R.layout.simple_list_item_1 作为 ListView 子项布局的 id，这是一个 Android 内置的布局文件，里面只有一个 TextView ，可用于简单地显示一段文本。

```xml
class MainActivity : AppCompatActivity() {
    private val data = listOf("Apple", "Banana", "Orange", "Watermelon", "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango", "Apple", "Banana", "Orange", "Watermelon", "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val adapter = ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, data)
        listView.adapter = adapter
    }
}
```

### 定制 ListView 的界面

假设我们需要将上面的列表名称左边显示一个图片。首先需要定义一个实体类，作为 ListView 适配器的适配类型。

```kotlin
class Fruit(val name:String, val imageId: Int)
```

然后需要为 ListView 的子项指定一个我们自定义的布局，在 layout 目录下新建 fruit_item.xml，里面添加 ImageView 和 TextView

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60dp">

    <ImageView
        android:id="@+id/fruitImage"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp" />

    <TextView
        android:id="@+id/fruitName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp" />
</LinearLayout>
```

接下来需要创建一个自定义的适配器，这个适配器继承自 ArrayAdapter，并将泛型指定为 Fruit 类。新建类 FruitAdapter。

```kotlin
class FruitAdapter(activity: Activity, val resourceId: Int, data: List<Fruit>) :
    ArrayAdapter<Fruit>(activity, resourceId, data) {

    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view = LayoutInflater.from(context).inflate(resourceId, parent, false)
        val fruitImage: ImageView = view.findViewById(R.id.fruitImage)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
        val fruit = getItem(position) // 获取当前项的Fruit实例
        if (fruit != null) {
            fruitImage.setImageResource(fruit.imageId)
            fruitName.text = fruit.name
        }
        return view
    }
}
```

最后修改给 ListView 赋值数据的地方，将数据变为所有的 Fruit 对象。

```kotlin
class MainActivity : AppCompatActivity() {
    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        fruitList.add(Fruit("Apple", R.drawable.apple_pic))
        fruitList.add(Fruit("Banana", R.drawable.banana_pic))
        fruitList.add(Fruit("Orange", R.drawable.orange_pic))

        val adapter = FruitAdapter(this, R.layout.fruit_item, fruitList)
        listView.adapter = adapter
    }
}
```

### 优化 ListView

目前我们 ListView 的运行效率是很低的，因为在 FruitAdapter 的 getView() 方法中，每次都将布局重新加载了一遍，当 ListView 快速滚动的时候，这就会成为性能的瓶颈。

getView() 方法中还有一个 convertView 参数，这个参数用于将之前加载好的布局进行缓存，以便之后进行重用，我们可以借助这个参数来进行性能优化。

```kotlin
val view: View
if (convertView == null) {
    view = LayoutInflater.from(context).inflate(resourceId, parent, false)
} else {
    view = convertView
}
```

虽然现在已经不会再重复去加载布局，但是每次在 getView() 方法中仍然会调用 View 的 findViewById() 方法来获取一次控件的实例。 我们可以借助一个 ViewHolder 来对这部分性能进行优化。

我们新增一个内部类 ViewHolder，用于对 ImageView 和 TextView 的控件实例进行缓存。当 convertView 为 null 的时候，创建一个 ViewHolder 对象，并将控件的实例存放在 ViewHolder 里，然后调用 View 的 setTag()方法，将 ViewHolder 对象存储在 View 中。当 convertView 不为 null 的时候，则调用 View 的 getTag()方法，把 ViewHolder 重新取出。这样所有控件的实例都缓存在了 ViewHolder 里，就没有必要每次都通过 findViewById() 方法来获取控件实例了。

```kotlin
class FruitAdapter(activity: Activity, val resourceId: Int, data: List<Fruit>) :
    ArrayAdapter<Fruit>(activity, resourceId, data) {

    inner class ViewHolder(val fruitImage: ImageView, val fruitName: TextView)

    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view: View
        val viewHolder: ViewHolder
        if (convertView == null) {
            view = LayoutInflater.from(context).inflate(resourceId, parent, false)
            val fruitImage: ImageView = view.findViewById(R.id.fruitImage)
            val fruitName: TextView = view.findViewById(R.id.fruitName)
            viewHolder = ViewHolder(fruitImage, fruitName)
            view.tag = viewHolder
        } else {
            view = convertView
            viewHolder = view.tag as ViewHolder
        }

        val fruit = getItem(position) // 获取当前项的Fruit实例
        if (fruit != null) {
            viewHolder.fruitImage.setImageResource(fruit.imageId)
            viewHolder.fruitName.text = fruit.name
        }
        return view
    }
}
```

### ListView 的点击事件

使用 setOnItemClickListener() 方法为任何子项添加点击监听。

```kotlin
val adapter = FruitAdapter(this, R.layout.fruit_item, fruitList)
listView.adapter = adapter
listView.setOnItemClickListener { parent, view, position, id ->
    val fruit = fruitList[position]
    Toast.makeText(this, fruit.name, Toast.LENGTH_SHORT).show()
}
```

## 更强大的滚动控件: RecyclerView

ListView 并不是完美无缺的，比如如果不使用一些技巧来提升它的运行效率，那么 ListView 的性能就会非常差。还有，ListView 的扩展性也不够好， 它只能实现数据纵向滚动的效果，如果我们想实现横向滚动的话，ListView 是做不到的。

为此，Android 提供了一个更强大的滚动控件——RecyclerView。它可以说是一个增强版的 ListView ，不仅可以轻松实现和 ListView 同样的效果，还优化了 ListView 存在的各种不足之处。目前 Android 官方更加推荐使用 RecyclerView。

首先需要在项目的 build.gradle 中添加 RecyclerView 库的依赖，就能保证在所有 Android 系统版本上都可以使用 RecyclerView 控件了。然后点击“Sync Now” 就可以了，然后 gradle 会开始进行同步，把我们新添加的 RecyclerView 库引入项目当中。

```gradle
implementation 'androidx.recyclerview:recyclerview:1.3.0'
```

更多细节参考：《第一行代码》第三版，第四章，第 6 小节：更强大的滚动控件:RecyclerView

## 编写页面的最佳实践

当图片需要拉伸的时候，制作 9-Patch 图。图片上右键，Create 9-Patch file。最后记得要将原来的图片删除，只保留制作好的 xxxx.9.png 图片即可，因为 Android 项目中不允许同一文件夹下有两张相同名称的图片(即使后缀名不同也不行)。
