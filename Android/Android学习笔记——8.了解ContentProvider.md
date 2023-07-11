## ContentProvider

ContentProvider 主要用于在不同的应用程序之间实现数据共享的功能，可以选择只对哪一部分数据进行共享，从而保证我们程序中的隐私数据不会有泄漏的风险。

## Android 运行时权限

之前我们需要在 mainfest 文件中声明权限，类似`<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />`，在安装的时候会显示该应用需要的权限。

Android 6.0 及以上系统在使用危险权限时必须进行运行时权限处理。运行时权限，用户不需要在安装软件的时候一次性授权所有申请的权限，而是可以在软件的使用过程中再对某一项权限申请进行授权。就算我拒绝了这个权限，也应该可以使用这个应用的其他功能，而不是像之前那样直接无法安装它。

当然，并不是所有权限都需要在运行时申请，对于用户来说，不停地授权也很烦琐。Android 现在将常用的权限大致归成了两类，一类是普通权限，一类是危险权限。普通权限指的是那些不会直接威胁到用户的安全和隐私的权限，对于这部分权限申请，只需要声明在 manifest 文件中，系统会自动帮我们进行授权，不需要用户手动操作。危险权限则表示那些可能会触及用户隐私或者对设备安全性造成影响的权限，如获取设备联系人信息、定位设备的地理位置等，对于这部分权限申请，必须由用户手动授权才可以，否则程序就无法使用相应的功能。

运行时权限申请，通过 checkSelfPermission 来判断，通过 requestPermissions 来申请。通过 onRequestPermissionsResult()作为权限申请成功或者失败的回调。

```kotlin
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.CALL_PHONE), 1)
} else {
    // 已经有权限
}

override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    when (requestCode) {
        1 -> {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // 已有权限，接着操作。
            } else {
                Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

## 访问其他程序的数据

如果一个应用程序通过 ContentProvider 对其数据提供了外部访问接口，那么任何其他的应用程序都可以对这部分数据进行访问。

对于每一个应用程序来说，如果想要访问 ContentProvider 中共享的数据，就一定要借助 ContentResolver 类，可以通过 Context 中的 getContentResolver()方法获取该类的实例。ContentResolver 中提供了一系列的方法用于对数据进行增删改查操作，其中 insert() 方法用于添加数据，update() 方法用于更新数据，delete() 方法用于删除数据，query() 方法用于查询数据。

通过 query 查询数据，需要指定具体的哪个包的哪个表，一般用 uri 表示，格式 `content://包名.provider/表名`，如：`content://com.example.app.provider/table1`，查询完成后返回的仍然是一个 Cursor 对象。

```kotlin
val uri = Uri.parse("content://com.example.app.provider/table1")
val cursor = contentResolver.query(
    uri, // 某个应用程序下的某一张表
    projection, // 指定查询的列名
    selection, // 指定where的约束条件
    selectionArgs, // 为where中的占位符提供具体的值
    sortOrder // 指定查询结果的排序方式
)
```

增加删除修改，跟数据库使用一致。

```kotlin
val values = contentValuesOf("column1" to "text", "column2" to 1)
contentResolver.insert(uri, values)

val values = contentValuesOf("column1" to "")
contentResolver.update(uri, values, "column1 = ? and column2 = ?", arrayOf("text", "1"))

contentResolver.delete(uri, "column2 = ?", arrayOf("1"))
```

## 创建自己的 ContentProvider

ContentProvider 类中有 6 个抽象方法，我们在使用子类继承它的时候，需要将这 6 个方法全部重写。

- onCreate()。初始化 ContentP rovider 的时候调用。通常会在这里完成对数据库的创建和
  升级等操作，返回 true 表示 ContentP rovider 初始化成功，返回 false 则表示失败。

- getType()。根据传入的内容 URI 返回相应的 MIME 类型。

另外，还有一点需要注意，ContentProvider 一定要在 AndroidManifest.xml 文件中注册才可以使用。

```kotlin
class MyProvider : ContentProvider() {
    override fun onCreate(): Boolean {
        return false
    }
    override fun query(uri: Uri, projection: Array<String>?, selection: String?, selectionArgs: Array<String>?, sortOrder: String?): Cursor? {
        return null
    }
    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        return null
    }
    override fun update(uri: Uri, values: ContentValues?, selection: String?, selectionArgs: Array<String>?): Int {
        return 0
    }
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int {
        return 0
    }
    override fun getType(uri: Uri): String? {
        return null
    }
}
```
