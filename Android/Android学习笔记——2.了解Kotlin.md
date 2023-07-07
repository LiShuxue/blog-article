## 启动

main() 函数是运行 kotlin 的启动函数，用 println()打印

```kotlin
fun main() {
    println("hello")
}
```

## 变量，常量

用 val 或者 var 定义变量，一般使用 val，类似 js 中都使用 const。

```kotlin
var a = 1; // 可重新赋值
val b = 2; // 不可重新赋值
```

定义常量的关键字是 const， 注意只有在单例类、companion object 或顶层方法中才可以使用 const 关键字。

```kotlin
const val TYPE = 0
```

## 字符串 String 和 StringBuilder

不可变性：String 是不可变的，意味着一旦创建，就不能修改其内容。每次对 String 进行修改时，实际上是创建一个新的 String 对象。而 StringBuilder 是可变的，可以在原始对象上进行修改，而不会创建新的对象。StringBuilder 在需要频繁修改字符串时效率更高。

性能：由于 String 的不可变性，每次修改字符串时都需要创建一个新的 String 对象，这可能会导致内存分配和垃圾回收的开销。对于频繁的字符串拼接操作，使用 StringBuilder 通常比使用 String 更高效，因为它可以在原地修改字符串而不会创建新的对象。

线程安全性：String 是线程安全的，可以在多个线程中共享而无需担心数据竞争。而 StringBuilder 是非线程安全的，如果多个线程同时修改同一个 StringBuilder 对象，可能会导致不可预期的结果。如果在多线程环境下进行字符串操作，应该使用线程安全的 StringBuffer 类。

### 延迟初始化

全局变量延迟初始化使用的是 lateinit 关键字，它可以告诉 Kotlin 编译器，我会在晚些时候对这个变量进行初始化，这样就不用在一开始的时候将它赋值为 null 了。后面会减少很多判空处理。

当你对一个全局变量使用了 lateinit 关键字时，请一定要确保它在被任何地方调用之前已经完成了初始化工作，否则 Kotlin 将无法保证程序的安全性。

`::xxx.isInitialized` 可以用来判断某个变量是否已经初始化。

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

when 一般都有 else 分支，除非是密封类。

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

### 区间

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

### 静态方法，伴生对象

Java 中定义一个静态方法非常简单，只需要在方法上声明一个 static 关键字就可以，静态方法非常适合用于编写一些工具类的功能。使用 Util.doAction()的方式来调用。

```java
public class Util {
    public static void doAction() {
        System.out.println("do action");
    }
}
```

像工具类这种功能，在 Kotlin 中就非常推荐使用单例类的方式来实现。虽然这里的 doAction()方法并不是静态方法，仍然使用 Util.doAction()的方式来调用。

```kotlin
object Util {
    fun doAction() {
        println("do action")
    }
}
```

不过，使用单例类的写法会将整个类中的所有方法全部变成类似于静态方法的调用方式，而如果我们只是希望让类中的某一个方法变成静态方法的调用方式该怎么办呢？这个时候就可以使用 companion object。

所有定义在 companion object 中的方法都可以使用类似于 Java 静态方法的形式调用。companion object 这个关键字实际上会在类的内部创建一个伴生类，而 doAction2()方法就是定义在这个伴生类里面的实例方法。Kotlin 规定这个类始终只会存在一个伴生类对象，因此调用 Util.doAction2()方法实际上就是调用了 Util 类中伴生对象的 doAction2()方法。

```kotlin
class Util {
    fun doAction1() {
        println("do action1")
    }
    companion object {
        fun doAction2() {
            println("do action2")
        }
    }
}
```

如果你在 Java 代码中以静态方法的形式去调用的话，你会发现这些方法并不存在，因为这是 Kotlin 语法上模拟的静态方法。如果你确确实实需要定义真正的静态方法，Kotlin 仍然提供了两种实现方式：注解和顶层方法。

如果我们给单例类或 companion object 中的方法加上 `@JvmStatic` 注解，那么 Kotlin 编译器就会将这些方法编译成真正的静态方法。只能给这些方法加，如果你尝试加在一个普通方法上，会直接提示语法错误。

顶层方法指的是那些没有定义在任何类中的方法，比如我们编写的 main()方法。Kotlin 编译器会将所有的顶层方法全部编译成静态方法，因此只要你定义了一个顶层方法，那么它就一定是静态方法。想要定义一个顶层方法，首先需要创建一个普通的 Kotlin 文件，我们在这个文件中定义的任何方法都会是顶层方法。如果是在 Kotlin 代码中调用的话，所有的顶层方法都可以在任何位置被直接调用，不用管包名路径，也不用创建实例，直接调用方法 `doSomething()` 即可。如果是在 Java 代码中调用，使用 `文件名Kt.xxx()`调用。比如使用 `HelperKt.doSomething()` 的写法来调用

### 单例类

当我们希望某个类在全局最多只能拥有一个实例，这时就可以使用单例模式。在 Kotlin 中，只需要使用 object 关键字即可。Kotlin 在背后自动帮我们创建了一个 Singleton 类的实例，并且保证全局只会存在一个 Singleton 实例。

```kotlin
object Singleton {
    var name = "test"
}

// 无需初始化，直接使用
Singleton.name
```

### 内部类

使用 inner class 关键字来定义内部类

### 密封类

密封类的关键字是 sealed class。

当在 when 语句中传入一个密封类变量作为条件时，Kotlin 编译器会自动检查该密封类有哪些子类，并强制要求你将每一个子类所对应的条件全部处理。这样就可以保证，即使没有编写 else 条件，也不可能会出现漏写条件分支的 情况。

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

用 interface 定义接口，实现接口跟继承一样，也是用冒号，如果是多个接口，中间用逗号。

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

## 标准函数，let，with, run，apply，repeat

### let

let 扩展函数实际上是一个作用域函数，当你需要去定义一个变量在一个特定的作用域范围内，let 函数的是一个不错的选择；

let 函数另一个作用就是可以避免写一些判断 null 的操作。let 函数的特性配合?.操作符可以在空指针检查的时候起到很大的作用。

```kotlin
obj.let { obj2 ->
    // 编写具体的业务逻辑
}

fun doStudy(study: Study?) {
    study?.let { stu ->
        stu.readBooks()
        stu.doHomework()
    }
}
// 省略参数
study?.let {
    it.readBooks()
    it.doHomework()
}
```

### with

with 函数接收两个参数，第一个参数可以是一个任意类型的对象，第二个参数是一个 Lambda 表达式。

with 函数会在 Lambda 表达式中提供第一个参数对象的上下文，并使用 Lambda 表达式中的最后一行代码作为返回值返回。

```kotlin
val result = with(obj) { // 这里是obj的上下文
    "value" // with函数的返回值
}

// 原写法
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val builder = StringBuilder()
builder.append("Start eating fruits.\n")
for (fruit in list) {
    builder.append(fruit).append("\n")
}
println(builder.toString())

// 使用 with
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val builder = StringBuilder()
val result = with(builder) {
    append("Start eating fruits.\n")
    for (fruit in list) {
        append(fruit).append("\n")
    }
    toString()
}
println(result)
```

### run

run 函数的用法和使用场景其实和 with 函数是非常类似的，只是稍微做了一些语法改动而已。

首先 run 函数通常不会直接调用，而是要在某个对象的基础上调用。其次 run 函数只接收一个 Lambda 参数，并且会在 Lambda 表达式中提供调用对象的上下文。

```kotlin
val result = obj.run { // 这里是obj的上下文
    "value" // run函数的返回值
}

// 使用 run
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val builder = StringBuilder()
// 只是将调用with函数并传入builder对象改成了调用 builder对象的run方法，其他都没有任何区别
val result = builder.run {
    append("Start eating fruits.\n")
    for (fruit in list) {
        append(fruit).append("\n")
    }
    toString()
}
println(result)
```

### apply

apply 函数和 run 函数也是极其类似的，都要在某个对象上调用，并且只接收一个 Lambda 参数，也会在 Lambda 表达式中提供调用对象的上下文，但是 apply 函数无法指定返回值，而是会自动返回调用对象本身。

```kotlin
val result = obj.apply {
    // 这里是obj的上下文
}
// result == obj

// 使用apply
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val builder = StringBuilder()
val result = builder.apply {
    append("Start eating fruits.\n")
    for (fruit in list) {
        append(fruit).append("\n")
    }
}
println(result.toString())
```

### repeat

repeat 函数允许你传入一个数值 n，然后会把 Lambda 表达式中的内容执行 n 遍。

```kotlin
repeat(2) {
    println("1")
    println("2")
}
```

## 扩展函数

扩展函数表示即使在不修改某个类的源码的情况下，仍然可以打开这个类，向该类添加新的函数。

比如希望向 Int 类中添加一个扩展函数，因此需要先创建一个 Int.kt 文件。文件名虽然并没有固定的要求，但是最好向哪个类中添加扩展函数，就定义一个同名的 Kotlin 文件，这样便于你以后查找。函数体内部使用 this 关键字来引用该对象。

扩展函数的作用域限定在声明它的作用域内或导入它的作用域内。最好将它定义成顶层方法，这样可以让扩展函数拥有全局的访问域。

```kotlin
fun Int.isEven(): Boolean {
    return this % 2 == 0
}

val number = 10
val isNumberEven = number.isEven() // 调用扩展函数
```

## 运算符重载

常用运算符，比如加+，减-，乘\*，除/，余%，递增++，递减--等等，都可以重新定义其实现方式，运算符重载使用的是 operator 关键字，只要在指定函数的前面加上 operator 关键字，就可以实现运算符重载的功能了。

重载函数的作用域是声明重载函数的类之间。

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

Kotlin 中的 javaClass 表示获取当前实例的 Class 对象，相当于在 Java 中调用 getClass()方法;

`SecondActivity::class.java` 的写法就相当于 Java 中 `SecondActivity.class` 的写法。

Kotlin 允许我们将没有用到的参数使用下划线来替代。
