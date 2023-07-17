## Material Design

Material Design 是由 Google 的设计工程师们基于传统优秀的设计原则，结合丰富的创意和科学技术所开发的一套全新的界面设计语言，包含了视觉、运动、互动效果等特性。Google 从 Android 5.0 系统开始，就将所有内置的应用都使用 Material Design 风格进行设计。

2015 年的 Google I/O 大会上推出了一个 Design Support 库，这个库将 Material Design 中最具代表性的一些控件和效果进行了封装，使得开发者即使在不了解 Material Design 的情况下，也能非常轻松地将自己的应用 Material 化。后来 Design Support 库又改名成了 Material 库。

## 标题栏 ToolBar

ToolBar 由 AndroidX 库提供的。Toolbar 的强大之处在于，它不仅继承了 ActionBar 的所有功能，而且灵活性很高，可以配合其他控件完成一些 Material Design 的效果。

每个 Activity 最顶部的那个标题栏其实就是 ActionBar，不过 ActionBar 由于其设计的原因，被限定只能位于 Activity 的顶部，从而不能实现一些 Material Design 的效果，因此官方现在已经不再建议使用 ActionBar 了。更加推荐使用 Toolbar 。

任何一个新建的项目，默认都是会显示 ActionBar 的，其实这是根据项目中指定的主题来显示的，在 android:theme 属性，制定了 res/values/styles.xml 下面的文件。使用 Toolbar 来替代 ActionBar，因此需要指定一个不带 ActionBar 的主题，通常有 Theme.AppCompat.NoActionBar 和 Theme.AppCompat.Light.NoActionBar 这两种主题可选。

当你引入了 Material 库之后，还需要将 res/values/styles.xml 文件中 AppTheme 的 parent 主题改成 Theme.MaterialComponents.Light.NoActionBar ，否则在使用接下来的一些控件时可能会遇到崩溃问题。

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
</FrameLayout>
```

`xmlns:app`，`xmlns:android`来指定命名空间，才能使用`android:id、android: layout_width`等写法，由于许多 Material 属性是在新系统中新增的，老系统中并不存在，那么为了能够兼容老系统，我们就不能使用 android:attribute 这样的写法了，而是应该使用 app:attribute。

使用：`setSupportActionBar(toolbar)`显示在页面。调用 supportActionBar 的 setDisplayHomeAsUpEnabled()方法让导航按钮显示出来，调用 setHomeAsUpIndicator()方法来设置一个导航按钮图标。

## 滑动菜单 DrawerLayout

所谓的滑动菜单，就是将一些菜单选项隐藏起来，而不是放置在主屏幕上，然后可以通过滑动的方式将菜单显示出来。这种方式既节省了屏幕空间，又实现了非常好的动画效果，是 Material Design 中推荐的做法。

AndroidX 库中提供了一个 DrawerLayout 控件。它是一个布局，在布局中允许放入两个直接子控件：第一个子控件是主屏幕中显示的内容，第二个子控件是滑动菜单中显示的内容。

调用 DrawerLayout 的 openDrawer()方法将滑动菜单展示出来，closeDrawers()方法将滑动菜单关闭。

## 菜单内容 NavigationView

NavigationView 是 Material 库中提供的一个控件，它不仅是严格按照 Material Design 的要求来设计的，而且可以将滑动菜单页面的实现变得非常简单。

需要提前定义好 menu 和 headerLayout 文件。 menu 是用来在 NavigationView 中显示具体的菜单项的，headerLayout 则是用来在 NavigationView 中显示头部布局的。通过 app:menu 和 app:headerLayout 属性将我们准备好的 menu 和 headerLayout 设置进去。

```xml
<com.google.android.material.navigation.NavigationView
    android:id="@+id/navView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    app:menu="@menu/nav_menu"
    app:headerLayout="@layout/nav_header" />
```

调用了 NavigationView 的 setCheckedItem()方法设置默认选中项。调用 setNavigationItemSelectedListener()方法设置菜单项选中的事件监听器

## CircleImageView

开源项目 CircleImageView，它可以用来轻松实现图片圆形化的功能

## 悬浮按钮 FloatingActionButton

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:id="@+id/fab"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|end"
    android:layout_margin="16dp"
    android:src="@drawable/ic_done" />
```

## 提示工具 Snackbar

Snackbar 并不是 Toast 的替代品，它们有着不同的应用场景。Toast 的作用是告诉用户现在发生了什么事情，但用户只能被动接收这个事情。而 Snackbar 则在这方面进行了扩展，它允许在提示中加入一个可交互按钮，当用户点击按钮的时候，可以执行一些额外的逻辑操作。

Snackbar 的用法和 Toast 是基本相似的。调用 setAction()方法来设置一个动作，从而让 Snackbar 不仅仅是一个提示，而是可以和用户进行交互的。

```kotlin
Snackbar.make(view, "Data deleted", Snackbar.LENGTH_SHORT)
    .setAction("Undo") {
        Toast.makeText(this, "Data restored", Toast.LENGTH_SHORT).show()
    }
    .show()
```

## CoordinatorLayout

CoordinatorLayout 可以说是一个加强版的 FrameLayout，由 AndroidX 库提供。它在普通情况下的作用和 FrameLayout 基本一致，但是它拥有一些额外的 Material 能力。

CoordinatorLayout 可以监听其所有子控件的各种事件，并自动帮助我们做出最为合理的响应。

举个简单的例子，刚才弹出的 Snackbar 提示将悬浮按钮遮挡住了，而如果我们能让 CoordinatorLayout 监听到 Snackbar 的弹出事件，那么它会自动将内部的 FloatingActionButton 向上偏移，从而确保不会被 Snackbar 遮挡。

## 卡片式布局 MaterialCardView

MaterialCardView 是用于实现卡片式布局效果的重要控件，由 Material 库提供。实际上，MaterialCardView 也是一个 FrameLayout，只是额外提供了圆角和阴影等效果，看上去会有立体的感觉。

```xml
<com.google.android.material.card.MaterialCardView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:cardCornerRadius="4dp"
    app:elevation="5dp">

    <TextView
        android:id="@+id/infoText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</com.google.android.material.card.MaterialCardView>
```

## Glide

Glide 是一个超级强大的开源图片加载库，它不仅可以用于加载本地图片，还可以加载网络图片、GIF 图片甚至是本地视频。

图片像素非常高，如果不进行压缩就直接展示的话，很容易引起内存溢出。而使用 Glide 就完全不 需要担心这回事 Glide 在内部做了许多非常复杂的逻辑操作，其中就包括了图片压缩，我们只需要安心按照 Glide 的标准用法去加载图片就可以了。

## AppBarLayout

AppBarLayout 实际上是一个垂直方向的 LinearLayout ，它在内部做了很多滚动事件的封装，并应用了一些 Material Design 的设计理念。AppBarLayout 必须是 CoordinatorLayout 的子布局。

第一步将 Toolbar 嵌套到 AppBarLayout 中，第二步给 RecyclerView 指定一个布局行为 `app:layout_behavior="@string/appbar_scrolling_view_behavior"`。

当 RecyclerView 滚动的时候就已经将滚动事件通知给 AppBarLayout。可以让标题栏随着滚动消失或者出现。

## 下拉刷新 SwipeRefreshLayout

想要实现下拉刷新功能的控件放置到 SwipeRefreshLayout 中，就可以迅速让这个控件支持下拉刷新。

调用 SwipeRefreshLayout 的 setColorSchemeResources()方法来设置下拉刷新进度条的颜色，调用 setOnRefreshListener()方法来设置一个下拉刷新的监听器。

## 可折叠式标题栏 CollapsingToolbarLayout

CollapsingToolbarLayout 是一个作用于 Toolbar 基础之上的布局，它也是由 Material 库提供的。只能作为 AppBarLayout 的直接子布局来使用。而 AppBarLayout 又必须是 CoordinatorLayout 的子布局
