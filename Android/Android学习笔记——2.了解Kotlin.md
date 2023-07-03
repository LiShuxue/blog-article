## 启动

main() 函数是运行 kotlin 的启动函数，用 println()打印

```kotlin
fun main() {
    println("hello")
}
```

## 变量，常量

用 val 定义常量，用 var 定义变量，一般使用 val，类似 js 中都使用 const。

```kotlin
var a = 1; // 可重新赋值
val b = 2; // 不可重新赋值
```

### 类型

变量可以主动声明类型，也可以让 kotlin 自己推导。

Int, Long, Short, Float, Double, Boolean, Char, Byte。

```kotlin
val c: Int = 3;
```

## 函数

使用 fun 来声明函数，参数需要声明类型，如果有返回值，返回值也需要声明类型，如果没有返回值，可以不写。

```kotlin
fun methodName(param1: Int, param2: Int): Int {
    return 0
}
```

当一个函数中只有一行代码时，可以不写函数体，直接将唯一的一行代码写在函数定义的尾部，中间用等号连接，省略 return 关键字。

```kotlin
fun largerNumber(num1: Int, num2: Int): Int = max(num1, num2)
```

## 逻辑控制

### 条件语句，if / when

if 跟我们普通使用的 if 基本一样，但是 Kotlin 中的 if 语句有一个额外的功能，它是可以有返回值的，返回值就是 if 语句每一个条件中最后一行代码的返回值。

```kotlin
fun largerNumber(num1: Int, num2: Int): Int {
    val value = if (num1 > num2) {
        num1
    } else {
        num2
    }
    return value
}
```

当你的 if 判断条件非常多的时候，就是应该考虑使用 when 语句的时候。

when 类似于 switch case 语句。语法：`匹配值 -> { 执行逻辑 }`。 when 和 if 一样，也是有返回值的。

```kotlin
fun getScore(name: String) : Int {
    val value = if (name == "Tom") {
        1
    } else if (name == "Jim") {
        2
    } else if (name == "Jack") {
        3
    } else {
        0
    }

    return value
}
// 修改为when
fun getScore(name: String) : Int {
    val value = when (name) {
        "Tom" -> 1
        "Jim" -> 2
        "Jack" -> 3
        else -> 0
    }
    return value
}
```

除了精确匹配之外，when 语句还允许进行类型匹配。is 关键字就是类型匹配的核心，它类似于 instanceof 关键字。

```kotlin
fun checkNumber(num: Number) {
    when (num) {
        is Int -> println("number is Int")
        is Double -> println("number is Double")
        else -> println("number not support")
    }
}
```

### 循环语句 while / for in

while 循环跟我们 js 中一致。

for in 也是类似于 js 中的。可以对区间进行遍历，还可以用于遍历数组和集合。

循环中使用 step=2，相当于 i=i+2。

```kotlin
fun main() {
    val range = 0 until 10
    for (i in range step 2) {
        println(i)
    }
}
```

#### 区间

可以用 .. 表示一个两头头闭合的区间。`val range = 0..10`

可以用 until 表示一个左闭右开的区间。`val range = 0 until 10`

可以用使用 downTo 关键字，创建一个降序的区间，两头闭合的。`val range = 10 downTo 1`

## 类

使用 class 创建类，类似 js，里面使用 var 定义成员变量。如果使用 val 关键字的话，初始化之后就不能再重新赋值了。

当一个类中没有任何代码时，还可以将尾部的大括号省略。

```kotlin
class Person {
    var name = ""
    var age = 0
    fun eat() {
        println(name + " is eating. He is " + age + " years old.")
    }
}
```

类声明的时候也可以有参数，有参数的话就不用声明成员变量了。

```kotlin
class Person(var name: String, var age: Int) {
    fun eat() {
        println(name + " is eating. He is " + age + " years old.")
    }
}
```

如果需要对初始化参数做一些额外的操作，可以用 init 结构体。

```kotlin
class Person(var name: String, var age: Int) {
    init {
        println("name is " + name)
        println("age is " + age)
    }

    fun eat() {
        println(name + " is eating. He is " + age + " years old.")
    }
}
```

实例化，不需要使用 new 关键字。对成员变量的访问，像 js 中访问属性一样简单。不需要手动写 setter，getter

```kotlin
val p = Person()
p.name = "Tom" // set
println(p.name) // get
```

### public、private、protected 和 internal 修饰符

这些修饰符，可以加在 fun 或者属性的前面。

- public 表示对所有类可见，这也是 Kotlin 默认的。

- private 表示只在类内部可见。

- protected 表示只对当前类和子类可见。

- internal 表示只对同一模块中的类可见。

### 数据类

数据类通常需要重写 equals()、hashCode()、toString()这几个方法。其中，equals() 方法用于判断两个数据类是否相等。hashCode()方法作为 equals()的配套方法，也需要一起 重写，否则会导致 HashMap、HashSet 等 hash 相关的系统类无法正常工作。toString()方法 用于提供更清晰的输入日志，否则一个数据类默认打印出来的就是一行内存地址。

当在一个类前 面声明了 data 关键字时，就表明你希望这个类是一个数据类，Kotlin 会根据主构造函数中的参 数帮你将 equals()、hashCode()、toString()等固定且无实际逻辑意义的方法自动生成， 从而大大减少了开发的工作量。

```kotlin
data class Cellphone(val brand: String, val price: Double)
```

### 单例类

当我们希望某个类在全局最多只能拥有一个实例，这时就可以使用单例模式。在 Kotlin 中，只需要使用 object 关键字即可。Kotlin 在背后自动帮我们创建了一个 Singleton 类的实例，并且保证全局只会存在一个 Singleton 实例。

```kotlin
object Singleton {
    var name = "test"
}

// 无需初始化，直接使用
Singleton.name
```

## 继承

Kotlin 中任何一个非抽象类默认都是不可以被继承的，相当于 Java 中给类声明了 final 关键字。类的前面加上 open 关键字就可以继承了。

继承的时候使用 冒号+父类构造函数，如果有参数就传参数

```kotlin
// Person构造函数没有参数
open class Person1 {
    ...
}
// Person构造函数有参数
open class Person2(var name: String, var age: Int) {
    ...
}

// Student构造函数继承没有参数
class Student : Person1() {
    var sno = ""
    var grade = 0
}

// Student构造函数继承有参数
class Student : Person2("Tom", 32) {
    var sno = ""
    var grade = 0
}
```

## 接口

用 interface 定义接口，实现接口跟继承一样，也是用冒号。

类中使用 override 关键字来重写父类或者实现接口中的函数。

允许对接口中定义的函数进行默认实现。

```kotlin
interface Study {
    fun readBooks()
    fun doHomework() {
        println("do homework default implementation.")
    }
}

class Student(name: String, age: Int) : Person(name, age), Study {
    override fun readBooks() {
        println(name + " is reading.")
    }
}
```

## 集合 List(ArrayList, LinkedList)，Set(HashSet)，Map(HashMap)

### List 创建，添加，修改，删除，遍历

ArrayList 是基于数组的，LinkedList 是基于链表的。所以 ArrayList 适合查询，LinkedList 适合插入。

可以直接用类的构造方法创建，也可以用 listOf()函数创建。

listOf()函数创建的是一个不可变的集合。该集合只能用于读取， 我们无法对集合进行添加、修改或删除操作。如果我们确实需要创建一个可变的集合，使用 mutableListOf()函数。

添加用 add()，修改用 set()，删除用 remove()/removeAt()，遍历 for-in

```kotlin
// 普通创建
val list = ArrayList<String>()
list.add("Apple")
list.add(1, "Banana")

list.set(1, "bana")
list.removeAt(0)

// listOf
val list = listOf("Apple", "Banana", "Orange")

// 遍历
for (fruit in list) {
    println(fruit)
}
```

### Set 创建，添加，删除，遍历

Set 无序，不重复。Set 的创建跟 List 差不多，只是用 setOf()和 mutableSetOf()，或者 hashSetOf()

添加用 add()，删除用 remove()，遍历用 for-in

```kotlin
// 普通创建
val set = HashSet<String>()
set.add("test")
set.add("test2")
set.remove("test")
for (xx in set) {
    println(xx)
}

// mutableSetOf
val set = mutableSetOf("Apple", "Banana", "Orange", "Pear", "Grape")
```

### Map 创建，添加，修改，删除，遍历

Map 的创建可以用 mapOf()和 mutableMapOf()函数，或者 hashMapOf()

添加使用 put()或者直接指定 map["xx"] = "xxx"，修改用 set()或者直接指定 map["xx"] = "xxx"，删除用 remove()，遍历用 for-in，可以同时遍历 key 和 value

```kotlin
// 普通创建
val map = HashMap<String, String>()
map.put("1", "test");
map["2"] = "test2"
map.set("2", "test3")
map["2"] = "test4"
map.remove("1")

// mapOf
val map = mapOf("Apple" to 1, "Banana" to 2, "Orange" to 3, "Pear" to 4, "Grape" to 5)

// 遍历
for ((fruit, number) in map) {
    println("fruit is " + fruit + ", number is " + number)
}
```

### 集合拷贝

toList()、toMutableList()、toSet()、toMutableSet()、toHashSet()、toMap()、toMutableMap()、

### Lambda 表达式

Lambda 表达式：`{参数名1: 参数类型, 参数名2: 参数类型 -> 函数体}`，类似于我们 js 的箭头函数。

首先是参数列表，参数列表的结尾使用一个 `->` 符号，表示参数列表的结束以及函数体的开始，函数体中可以编写任意行代码，最后一行代码会自动作为 Lambda 表达式的返回值。

Lambda 表达式简化：获取名字最长的水果字符串长度

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
// 源表达式
val lambda = { fruit: String -> fruit.length }
val maxLengthFruit = list.maxBy(lambda)

// 简化，第一次，不声明变量
val maxLengthFruit = list.maxBy({ fruit: String -> fruit.length })

// 简化，第二次，当Lambda 参数是函数的最后一个参数时，可以将Lambda 表达式移到函数括号的外面
val maxLengthFruit = list.maxBy(){ fruit: String -> fruit.length }

// 简化，第三次，如果Lambda 参数是函数的唯一一个参数的话，还可以将函数的括号省略
val maxLengthFruit = list.maxBy{ fruit: String -> fruit.length }

// 简化，第四次，不必声明String，因为Kotlin会自动推导
val maxLengthFruit = list.maxBy { fruit -> fruit.length }

// 简化，第五次，当Lambda 表达式的参数列表中只有一个参数时，也不必声明参数名，而是可以使用it 关键字来代替
val maxLengthFruit = list.maxBy { it.length }
```

### List 集合的函数式 API

forEach() / forEachIndexed()

```kotlin
val list = listOf("test1", "test2")
list.forEach { print("$it ") }
list.forEachIndexed { index, value -> println("index:$index, value:$value") }
```

map() / mapIndexed()

```kotlin
val list = listOf("test1", "test2")
val newList = list.map { it.uppercase() }
```

filter() / filterIndexed()

```kotlin
val list = listOf("test1", "test222")
val newList = list.filter { it.length <= 5 }
```

find()、count()、all()、any()

- find() 获取集合满足条件的首个元素，若无则返回 null
- count() 获取集合满足条件的元素的个数
- any() 判断集合中是否至少存在一个元素满足条件
- all() 判断集合中是否所有元素都满足条件

## Java 函数式 API 的使用

如果我们在 Kotlin 代码中调用了一个 Java 方法，并且该方法接收一个 Java 单抽象方法接口参数，就可以使用函数式 API。Java 单抽象 方法接口指的是接口中只有一个待实现方法，如果接口中有多个待实现方法，则无法使用函数式 API。

Java 原生 API 中有一个最为常见的单抽象方法接口——Runnable 接口。这个接口 中只有一个待实现的 run()方法，定义如下:

```kotlin
// 接口
public interface Runnable {
    void run();
}

// 使用
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}).start();

// kotlin改写，舍弃new，匿名类实例用object
Thread(object : Runnable {
    override fun run() {
        println("Thread is running")
    }
}).start()

// 简化
Thread {
    println("Thread is running")
}.start()
```

Android 中的点击事件

```kotlin
// 接口
public interface OnClickListener {
    void onClick(View v);
}

// 使用
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
    }
});

// kotlin简写
button.setOnClickListener {

}
```

## 其他

字符串都用双引号

比较相等用两个等号“==”

Mac 上是 Ctrl + Shift + R 重新运行 main()函数

Kotlin 默认所有的参数和变量都不可为空，如果想为 null，需要声明的时候加上 `?` 如，`val test: Int? = null`。

如果防止某一个数据为空，可以使用 `?.` 来判断，当对象为空时则什么都不做。

`?:` 这个操作符的左右两边都接收一个表达式， 如果左边表达式的结果不为空就返回左边表达式的结果，否则就返回右边表达式的结果。

非空断言工具，写法是在对象的后面加上 `!!`，告诉 Kotlin，我非常确信这里的对象不会为空。

在字符串里嵌入${}这种语法结构的表达式，`"hello, ${obj.name}. nice to meet you!"`，当表达式中仅有一个变量的时候，还可以将两边的大括号省略。

函数也可以有参数默认值。调用方法可以用 key/value 形式指定参数传参。
