## 数据持久化的方式

Android 系统中主要提供了 3 种方式用于简单地实现数据持久化功能：文件存储、SharedPreferences 存储以及数据库存储。

## 文件存储

比较适合存储一些简单的文本数据或二进制数据，所有的文件都默认存储到`/data/data/<packagename>/files/` 目录下。

### 存

openFileOutput()方法，可以用于将数据存储到指定的文件中。第一个参数是文件名，第二个参数是文件的操作模式，主要有 MODE_PRIVATE 和 MODE_APPEND 两种模式可选，默认是 MODE_PRIVATE，表示当指定相同文件名的时候，所写入的内容将会覆盖原文件中的内容，而 MODE_APPEND 则表示如果该文件已存在，就往文件里面追加内容，不存在就创建新文件。

use 函数，会保证在 Lambda 表达式中的代码全部执行完之后自动将外层的流关闭。

```kotlin
fun save(inputText: String) {
    val output = openFileOutput("data", Context.MODE_PRIVATE)
    val writer = BufferedWriter(OutputStreamWriter(output))
    writer.use {
        it.write(inputText)
    }
}
```

借助 Device File Explorer 工具查看一下`/data/data/<packagename>/files/`目录，在右下角。

### 取

openFileInput()方法，用于从文件中读取数据。它只接收一个参数，即要读取的文件名。

```kotlin
fun load(): String {
    val content = StringBuilder()
    val input = openFileInput("data")
    val reader = BufferedReader(InputStreamReader(input))
    reader.use {
        reader.forEachLine {
        content.append(it)
    }
    return content.toString()
}
```

读取数据使用了一个 forEachLine 函数，它会将读到的每行内容都回调到 Lambda 表达式中。

## SharedPreferences 存储

SharedPreferences 是使用键值对的方式来存储数据的。当保存一条数据的时候，需要给这条数据提供一个对应的键，在读取数据的时候就可以通过这个键把相应的值取出来。

SharedPreferences 支持多种不同的数据类型存储，Boolean, String, Int 等

SharedPreferences 文件都是存放在`/data/data/<packagename>/shared_prefs/` 目录下的。

### 存

首先需要获取 SharedPreferences 对象，Android 中主要提供了以下两种方法用于得到 SharedPreferences 对象：

- 在 Context 类中的调用 getSharedPreferences()方法，需要传两个参数，第一个参数是文件名称，第二个是操作模式，目前只有默认的 MODE_PRIVATE 这一种模式。

- 在 Activity 类中的 getPreferences()方法，它只接收一个操作模式参数，因为使用这个方法时会自动将当前 Activity 的类名作为 SharedPreferences 的文件名。

存储数据，主要可以分为 3 步实现。

调用 SharedPreferences 对象的 edit()方法获取一个 SharedPreferences.Editor 对象。

向 SharedPreferences.Editor 对象中添加数据，比如添加一个布尔型数据就使用 putBoolean()方法，添加一个字符串则使用 putString()方法，以此类推。

调用 apply()方法将添加的数据提交，从而完成数据存储操作。

```kotlin
val editor = getSharedPreferences("data", Context.MODE_PRIVATE).edit()
editor.putString("name", "Tom")
editor.putInt("age", 28)
editor.putBoolean("married", false)
editor.apply()
```

借助 Device File Explorer 来进行查看`/data/data/<packagename>/shared_prefs/`目录，这个文件是个 xml 文件。不能在实际的项目中直接存储敏感信息，因为会以明文的形式存储在 SharedPreferences 文件中。

### 取

```kotlin
val prefs = getSharedPreferences("data", Context.MODE_PRIVATE)
val name = prefs.getString("name", "")
val age = prefs.getInt("age", 0)
val married = prefs.getBoolean("married", false)
```

### KTX 扩展库简化书写 edit

```kotlin
getSharedPreferences("data", Context.MODE_PRIVATE).edit {
    putString("name", "Tom")
    putInt("age", 28)
    putBoolean("married", false)
}
```

## SQLite 数据库存储

SQLite 是一款轻量级的关系型数据库，它的运算速度非常快，占用资源很少。SQLite 不仅支持标准的 SQL 语法，还遵循了数据库的 ACID 事务。

### 创建数据库

Android 为了让我们能够更加方便地管理数据库，专门提供了一个 SQLiteOpenHelper 帮助类，借助这个类可以非常简单地对数据库进行创建和升级。

SQLiteOpenHelper 是一个抽象类，如果我们想要使用它，就需要创建一个自己的帮助类去继承它。SQLiteOpenHelper 中有两个抽象方法：onCreate() 和 onUpgrade()。我们必须在自己的帮助类里重写这两个方法，然后分别在这两个方法中实现创建和升级数据库的逻辑。

SQLiteOpenHelper 中还有两个非常重要的实例方法：getReadableDatabase()和 getWritableDatabase()。这两个方法都可以创建或打开一个现有的数据库(如果数据库已存在则直接打开，否则要创建一个新的数据库)，并返回一个可对数据库进行读写操作的对象。不同的是，当数据库不可写入的时候(如磁盘空间已满)，getReadableDatabase()方 法返回的对象将以只读的方式打开数据库，而 getWritableDatabase()方法则将出现异常。

数据库文件会存放在`/data/data/<packagename>/databases/` 目录下。

```kotlin
class MyDatabaseHelper(val context: Context, name: String, version: Int) : SQLiteOpenHelper(context, name, null, version) {
    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," + "author text," +
            "price real," +
            "pages integer," +
            "name text)"

    private val createCategory = "create table Category (" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(createBook)
        db.execSQL(createCategory)
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        db.execSQL("drop table if exists Book")
        db.execSQL("drop table if exists Category")
        onCreate(db)
    }
}


// 创建
val dbHelper = MyDatabaseHelper(this, "BookStore.db", 1)
dbHelper.writableDatabase
```

version 表示当前数据库的版本号，之前我们传入的是 1 ，现在只要传入一个比 1 大的数，就可以让 onUpgrade()方法得到执行了。

可以使用 Device File Explorer ，但是这个工具最多只能看到 databases 目 录下出现了一个 BookStore.db 文件，是无法查看 Book 表的。因此我们还需要借助一个叫作 Database Navigator 的插件工具。安装之后左上角侧边栏会出现 DB Browser。

### 添加数据

添加数据时可以使用 insert 语句，也可以用 SQLiteDatabase 中提供的 insert()方法，专门用于添加数据。

它接收 3 个参数：第一个参数是表名，第二个参数用于在未指定添加数据的情况下给某些可为空的列自动赋值 NULL，一般我们用不到这个功能，直接传入 null 即可，第三个参数是一个 ContentValues 对象，它提供了一系列的 put()方法重载，用于向 ContentValues 中添加数据，只需要将表中的每个列名以及相应的待添加数据传入即可。

```kotlin
val db = dbHelper.writableDatabase
val values1 = ContentValues().apply {
    // 开始组装第一条数据
    put("name", "The Da Vinci Code")
    put("author", "Dan Brown")
    put("pages", 454)
    put("price", 16.96)
}
db.insert("Book", null, values1) // 插入一条数据
```

id 那一列并没给它赋值，这是因为在前面创建表的时候，我们就将 id 列设置为自增长了，它的值会在入库的时候自动生成。

### 更新数据

SQLiteDatabase 中提供了一个非常好用的 update()方法，用于对数据进行更新。

这个方法 接收 4 个参数：第一个参数和 insert()方法一样，也是表名，第二 个参数是 ContentValues 对象，要把更新数据在这里组装进去，第三、第四个参数用于约束更新某一行或某几行中的数据，不指定的话默认会更新所有行。

```kotlin
val db = dbHelper.writableDatabase
val values = ContentValues()
values.put("price", 10.99)
db.update("Book", values, "name = ?", arrayOf("The Da Vinci Code"))
```

### 删除数据

SQLiteDatabase 中提供了一个 delete()方法，专门用于删除数据。这个方法接收 3 个参数：第一个参数仍然是表名，第二、第三个参数用于约束删除某一行或某几行的数据，不指定的话默认会删除所有行。

```kotlin
val db = dbHelper.writableDatabase
db.delete("Book", "pages > ?", arrayOf("500"))
```

### 查询数据

SQLiteDatabase 中还提供了一个 query()方法用于对数据进行查询。这个方法的参数非常复杂，最短的一个方法重载也需要传入 7 个参数。

- 第一个参数不用说，当然还是表名。

- 第二个参数用于指定去查询哪几列，如果不指定则默认查询所有列。

- 第三、第四个参数用于约束查询某一行或某几行的数据，不指定则默认查询所有行的数据。

- 第五个参数用于指定需要去 group by 的列，不指定则表示不对查询结果进行 group by 操作。

- 第六个参数用于对 group by 之后的数据进行进一步的过滤，不指定则表示不进行过滤。

- 第七个参数用于指定查询结果的排序方式，不指定则表示使用默认的排序方式。

调用 query()方法后会返回一个 Cursor 对象，查询到的所有数据都将从这个对象中取出。

```kotlin
val db = dbHelper.writableDatabase
// 查询Book表中所有的数据
val cursor = db.query("Book", null, null, null, null, null, null)
if (cursor.moveToFirst()) {
    do {
        // 遍历Cursor对象，取出数据并打印
        val name = cursor.getString(cursor.getColumnIndex("name"))
        val author = cursor.getString(cursor.getColumnIndex("author"))
        val pages = cursor.getInt(cursor.getColumnIndex("pages"))
        val price = cursor.getDouble(cursor.getColumnIndex("price"))
    } while (cursor.moveToNext())
}
cursor.close()
```

moveToFirst()方法，将数据的指针移动到第一行的位置，然后进入一个循环当中，去遍历查询到的每一行数据。在这个循环中可以通过 Cursor 的 getColumnIndex()方法获取某一列在表中对应的位置索引，然后将这个索引传入相应的取值方法中，就可以得到从数据库中读取到的数据了。

### 使用 SQL 语句操作数据库

添加数据:

```kotlin
db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)", arrayOf("The Da Vinci Code", "Dan Brown", "454", "16.96") )
```

更新数据:

```kotlin
db.execSQL("update Book set price = ? where name = ?", arrayOf("10.99", "The Da Vinci Code")
```

删除数据:

```kotlin
db.execSQL("delete from Book where pages > ?", arrayOf("500"))
```

查询数据:

```kotlin
val cursor = db.rawQuery("select * from Book", null)
```

### 使用事务

```kotlin
val db = dbHelper.writableDatabase
db.beginTransaction() // 开启事务
try {
    db.delete("Book", null, null)
    if (true) {
        // 手动抛出一个异常，让事务失败
        throw NullPointerException()
    }
    val values = ContentValues().apply {
        put("name", "Game of Thrones")
        put("author", "George Martin")
        put("pages", 720)
        put("price", 20.85)
    }
    db.insert("Book", null, values)
    db.setTransactionSuccessful() // 事务已经执行成功
} catch (e: Exception) {
    e.printStackTrace()
} finally {
    db.endTransaction() // 结束事务
}
```

### 升级数据库

每一个数据库版本都会对应一个版本号，当指定的数据库版本号大于当前数据库版本号的时候，就会进入 onUpgrade()方法中执行更新操作。这里需要为每一个版本号赋予其所对应的数据库变动，然后在 onUpgrade()方法中对当前数据库的版本号进行判断，再执行相应的改变就可以了。

```kotlin
override fun onCreate(db: SQLiteDatabase) {
    db.execSQL(createBook)
    db.execSQL(createCategory)
}
override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
    if (oldVersion <= 1) {
        db.execSQL(createCategory)
    }
}
```

这样当用户直接安装第 2 版的程序时，就会进入 onCreate()方法，将两张表一起创建。而当用户使用第 2 版的程序覆盖安装第 1 版的程序时，就会进入升级数据库的操作中。

### 使用 KTX 库简化书写 ContentValues

```kotlin
val values = contentValuesOf("name" to "Game of Thrones", "author" to "George Martin", "pages" to 720, "price" to 20.85)

db.insert("Book", null, values)
```

## Room

现在 Google 推出了一个专门用于 Android 平台的数据库框架——Room。相比于传统的数据库 API，Room 的用法要更加复杂一些，但是却更加科学和规范，也更加符合现代高质量 App 的开发标准。

使用面向对象的思维去编写程序，而完全不用考虑数据库相关的逻辑和实现。

### 定义实体类

```kotlin
@Entity
data class User(var firstName: String, var lastName: String, var age: Int) {
    @PrimaryKey(autoGenerate = true)
    var id: Long = 0
}
```

### 定义 Dao

所有访问数据库的操作都是在这里封装的。业务方永远只需要与 Dao 层进行交互，而不必和底层的数据库打交道。

数据库操作通常有增删改查这 4 种，因此 Room 也提供了@Insert、@Delete、@Update 和@Query 这 4 种相应的注解。

```kotlin
@Dao
interface UserDao {
    @Insert
    fun insertUser(user: User): Long

    @Update
    fun updateUser(newUser: User)

    @Query("select * from User")
    fun loadAllUsers(): List<User>

    @Query("select * from User where age > :age")
    fun loadUsersOlderThan(age: Int): List<User>

    @Delete
    fun deleteUser(user: User)

    @Query("delete from User where lastName = :lastName")
    fun deleteUserByLastName(lastName: String): Int
}
```

### 定义 Database

这部分内容的写法是非常固定的，只需要定义好 3 个部分的内容：数据库的版本号、包含哪些实体类，以及提供 Dao 层的访问实例。

```kotlin
@Database(version = 1, entities = [User::class])
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao

    companion object {
        private var instance: AppDatabase? = null
        @Synchronized
        fun getDatabase(context: Context): AppDatabase {
            instance?.let {
                return it
            }
            return Room.databaseBuilder(context.applicationContext,
                AppDatabase::class.java, "app_database")
                .build().apply {
                instance = this
            }
        }
    }
}
```

### 数据库升级

Room 在数据库升级方面设计得非常烦琐，基本上没有 比使用原生的 SQLiteDatabase 简单到哪儿去，每一次升级都需要手动编写升级逻辑才行。
