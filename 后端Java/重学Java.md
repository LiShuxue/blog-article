## Java 简介

Java 介于编译型语言和解释型语言之间。

编译型语言如 C、C++，代码是直接编译成机器码执行，但是不同的平台（x86、ARM 等）CPU 的指令集不同，因此，需要编译出每一种平台的对应机器码。

解释型语言如 Python、Ruby 没有这个问题，可以由解释器直接加载源码然后运行，代价是运行效率太低。

而 Java 是将代码编译成一种“字节码”，它类似于抽象的 CPU 指令，然后，针对不同平台编写虚拟机，不同平台的虚拟机负责加载字节码并执行，这样就实现了“一次编写，到处运行”的效果。

### Java 平台

Java SE (Java Platform, Standard Edition)：
Java SE 是标准版本，适用于通用的应用程序开发。它包括了基本的 Java 类库和 API，使开发者能够构建桌面应用、控制台应用、小型服务器等。Java SE 包括 Java Development Kit (JDK) 和 Java Runtime Environment (JRE)。

Java EE (Java Platform, Enterprise Edition)：
Java EE 是企业级版本，用于构建大规模、分布式、事务性的企业应用程序。它在 Java SE 的基础上添加了更多的服务器端组件和 API，用于处理企业级的需求，如 Web 服务、消息队列、事务管理等。不用关注这个了，它现在由社区维护，而不是官方，他提供的功能都可以通过 Spring 全家桶实现。（另外，早期的 J2EE，也是指的它，最开始是 JDK1.2 版本时的叫法，后来由于 Java 版本升级，这种叫法带来了不少困惑，所以发布 Java 5 后正式将 J2EE 改名为 Java EE。）

Java ME (Java Platform, Micro Edition)：
Java ME 是微型版本，用于嵌入式设备、移动设备和物联网设备的开发。它针对资源受限的设备，提供了小型的 Java 运行环境（JRE）和 API。不幸的是，Java ME 从来没有真正流行起来，反而是 Android 开发成为了移动平台的标准之一，因此，没有特殊需求，不建议学习 Java ME。

### JDK、JRE、JVM

JDK（Java Development Kit）是用于开发和运行 Java 应用程序的软件环境。里面包含运行时环境（JRE）和其他 Java 开发所需的一些常用的命令行工具，比如说程序运行（java）、程序编译（javac）、文档生成（javadoc）、归档（jar）等等。

JRE（Java Runtime Environment）是只用于运行 Java 应用程序的软件环境。也就是说，如果只想运行 Java 程序而不需要开发 Java 程序的话，只需要安装 JRE 就可以了。

JVM (Java Virtual Machine) ，也就是 Java 虚拟机，由一套字节码指令集、一组寄存器、一个栈、一个垃圾回收堆和一个存储方法域等组成，屏蔽了不同操作系统（macOS、Windows、Linux）的差异性，使得 Java 能够“一次编译，到处运行”。

### Java 版本

说 Java 版本，其实就是说的 JDK 版本。

1995 Java 1.0 发布。后面发布了 1.1，1.2，1.3，1.4，1.5...

从 1.5 开始，命名规范不再用 1.x，而是直接用 Java 5。如果见到 1.7，1.8，其实就是 Java 7，Java 8。

Java 8 新增了很多好用的改动，成为大家非常喜欢的版本。Java 8 是 LTS 版本，从 Java 8 之后，几乎每年都会推出很多新的版本，到 2023 年，已经是 Java 20 了。不过你依然可以只使用 Java 8，老话常说，“新版任你发，我用 Java 8”。

### 安装

从官网下载安装：[https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/)

一般会有 arm64 或者 x64 的版本，ARM64（也称为 AArch64）和 x64（也称为 AMD64 或 x86-64）是不同的处理器架构，如果是苹果的 M1 芯片选 arm64，如果是 intel 或者 amd 的处理器，选 x64。

如果是解压版，需要配置环境变量 JAVA_HOME，然后将 $JAVA_HOME/bin 添加到系统的 PATH 环境变量中。如果是安装版，不需要配置。

执行 `java -version` 查看已安装的版本。

## Java 基础

### 数据类型

基本数据类型是 CPU 可以直接进行运算的类型。一个字节是 1 byte，是一个 8 位二进制数，即 8 个 bit。

- 整数类型：byte(1 byte)，short(2 byte)，int(4 byte)，long(8 byte)

- 浮点数类型：float(4 byte)，double(8 byte)

- 字符类型：char(2 byte)，使用单引号'，且仅有一个字符。

- 布尔类型：boolean，理论上存储布尔类型只需要 1 bit，但是通常 JVM 内部会把 boolean 表示为 4 字节整数

除了上述基本类型的变量，剩下的都是引用类型。如 String，数组，集合，类，接口等。

### 包装器

Java 中的基本数据类型都有对应的包装类，用于将基本数据类型（如 int、float、char 等）转换为对应的对象类型。包装类与基本数据类型之间可以使用自动装箱和拆箱进行转换。

### 变量 && 常量

变量一般会先声明类型。也可以用 var 定义，来省略类型，编译器会自动推断。

变量计算时，低精度的数据类型自动转换为高精度的数据类型。如，int 转 long，float 转 double，int 转 double。也可以进行强制类型转换，不过可能会导致数据丢失或精度降低。

```java
int a = 1;
var b = 2;
```

加上 final 修饰符就变成了常量，通常在类中作为静态常量使用。常量只能进行一次赋值，把 final 加在类上，类不能被继承；final 加在方法上，方法不能被重写；参数 final，则这个方法里这个参数也不能被赋值。

变量之间的赋值是赋引用。

对于参数传递，如果是简单基本数据类型和 String 类型，那么它传递的是值拷贝。对于对象，它传递的是对象的引用，即里面的变了，外面的也跟着变。需要注意的是，这条规则只适用于参数传递。

```java
public static final double PI = 3.14159;
```

### 字符串 String，StringBuffer，StringBuilder

字符串 String 是不可变的，一旦创建，就不能修改其内容。重新赋值只是相当于指向了另一个字符串。

字符串可以用 + 拼接，从 Java 13 开始，可以用 """..."""（三个引号）表示多行字符串，多行字符串前面共同的空格会被去掉。

基本类型比较用==，字符串或者对象比较，用 equals() 方法。

```java
String str = "hello";
String s = """
            SELECT * FROM
                users
            WHERE id > 100
            ORDER BY name DESC
            """;
```

StringBuffer 和 StringBuilder 都是用于操作可变字符串的类，它们提供了一系列方法来高效地进行字符串的修改和拼接。

StringBuffer 适用于多线程环境，因为它的方法都是同步的，即线程安全的。

StringBuilder 是 StringBuffer 的非线程安全版本，它提供了类似的方法，但不进行同步操作。由于不需要同步，所以在单线程环境下使用 StringBuilder 可以更快地执行操作。

```java
StringBuffer buffer = new StringBuffer();
buffer.append("Hello");
buffer.append(" ");
buffer.append("World");
String result = buffer.toString();

StringBuilder builder = new StringBuilder();
builder.append("Hello");
builder.append(" ");
builder.append("World");
String result = builder.toString();
```

### 流程控制

if-else，switch，for 循环，for-each 循环，while 循环，do while 循环。

break 关键字通常用于中断循环或 switch 语句，continue 关键字可以在循环中立即跳转到下一个循环。

```java
// for-each循环遍历数组或者集合
for (int element : anOtherArray) {
    System.out.println(element);
}
```

### 数组 和 集合

数组是一个固定大小的数据结构，一旦创建，其大小就不能再改变。

Arrays.asList() 方法可以用于将一系列元素转换为集合 List。接受一个可变参数，或者一个数组作为参数。返回的列表具有固定大小，不能进行添加或删除操作。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
```

常用的集合有 List(ArrayList)，Set(HashSet)，Map(HashMap)

```java
List<String> list = new ArrayList<>();
Set<String> set = new HashSet<>();
Map<Integer, String> map = new HashMap<>();
```

Java 8 引入了 forEach 方法，允许在集合类上进行迭代操作，通过 Lambda 表达式来定义遍历逻辑。

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.forEach(name -> System.out.println(name));
```

Java 8 也提供了新的 Stream API 来处理集合数据，它允许通过链式操作处理集合，进行过滤、映射、排序、归约等操作。

- map 方法用于将集合中的每个元素映射为另一种形式，生成一个新的集合。
- filter 方法用于根据指定条件过滤集合中的元素，生成一个满足条件的新集合。
- sorted 方法用于对集合元素进行排序。
- collect 方法用于将流中的元素收集到一个集合中。
- findFirst 和 findAny，findFirst 返回流中的第一个元素，而 findAny 返回任意一个满足条件的元素。
- allMatch、anyMatch 和 noneMatch，检查流中的元素是否满足指定条件。

### 时间和日期

从 Java 8 开始，推荐使用新的日期和时间 API，如 LocalDate、LocalTime、LocalDateTime 等，来替代旧的 Date 类。新的日期和时间 API 更加易用、类型安全。

### 输入输出（IO）

Java 中的输入输出流（I/O 流）是用于处理数据输入和输出的机制。它们允许你从文件、网络、内存等源读取数据，或将数据写入到这些目标中。在 Java 中，I/O 流被组织成输入流（Input Stream）和输出流（Output Stream），分别用于数据的读取和写入。

#### 字节输入流（Input Stream）

- InputStream：字节输入流的基类。
- FileInputStream：从文件中读取字节数据的输入流。
- BufferedInputStream：对其他输入流添加缓冲功能，提高读取效率。

#### 字节输出流（Output Stream）

- OutputStream：字节输出流的基类。
- FileOutputStream：将字节数据写入文件的输出流。
- BufferedOutputStream：对其他输出流添加缓冲功能，提高写入效率。

#### 字符输入流（Reader）

- Reader：字符输入流的基类。
- FileReader：从文件中读取字符数据的输入流。
- BufferedReader：对其他字符输入流添加缓冲功能，提高读取效率。

#### 字符输出流（Writer）

- Writer：字符输出流的基类。
- FileWriter：将字符数据写入文件的输出流。
- BufferedWriter：对其他字符输出流添加缓冲功能，提高写入效率。

### 线程

创建线程有三种方式，继承 Thread 类，实现 Runnable 接口，用 lambda 表达式。

实现 Runnable 接口的类的对象不能直接调 start()方法，需要用这个对象当参数再实例化一个 Thread 类的对象，这个 Thread 类的对象再去调 start()方法。

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable is running");
    }
}

// 使用lambda
Runnable runnable = () -> {
    System.out.println("Lambda thread is running");
};

Thread thread = new Thread(runnable);
thread.start();
```

#### 线程同步

线程同步是指在多线程环境中，为了避免数据竞争和保证共享资源的正确访问，采取的一种机制。

常用的线程同步机制包括 synchronized 关键字和 Lock 接口（java.util.concurrent 包中的锁实现，如 ReentrantLock）。这些机制确保了只有一个线程可以访问共享资源，从而防止并发问题。同步后，线程之间的访问需要排队的，前者完成后，后者才开始。

synchronized 关键字用于修饰方法或代码块，确保同一时间只有一个线程可以进入被 synchronized 修饰的区域。

Lock 接口提供了更灵活的同步机制，比 synchronized 关键字更强大。它允许你更精确地控制锁的获取和释放。

当两个线程竞争时，join 方法允许一个线程等待另一个线程完成执行。

#### 线程通信

wait：使当前线程进入等待状态，同时释放锁，直到其他线程调用相同对象的 notify 或 notifyAll 方法来唤醒等待的线程。

notify：唤醒一个正在等待相同对象锁的线程。

### 反射

允许在运行时检查和操作类、方法、字段等。它使你可以在编译时未知类的情况下，动态地获取类的信息并操作它们。通过反射可以根据一个类路径（或对象，类名）来获取这个类中的所有信息。

使用反射，首先要得到这个类的 Class 对象。反射得到方法才是最重要的。

```java
Class<?> clazz = MyClass.class; // MyClass是你要反射的类
String className = clazz.getName();
// 获取类的所有公共方法（包括继承的方法）
Method[] publicMethods = clazz.getMethods();
```

## 面向对象

### 类

类是一种用户定义的数据类型，用于描述对象的属性和行为。类是对象的模板，它包括构造函数、成员变量、方法等。

#### 构造函数

构造函数是类的一个特殊方法，用于创建对象时初始化对象的状态。构造函数的名称与类名相同，没有返回类型。

#### 成员变量

成员变量是类中定义的变量，用于表示对象的状态。每个对象都有自己的一组成员变量。

当方法里定义了局部变量，并且和类的属性同名的时候，方法内默认采用局部变量。在属性或变量前加上 "this."，来表示调用当前类的属性或变量，而不是局部变量。

#### 方法

方法是类中的函数，用于定义对象的行为。方法包括构造函数在内。

#### 修饰符

修饰符用于控制类、变量和方法的可访问性和行为。常见的修饰符包括 public、private、protected 和 static 等。

1. 什么都不写：包可见
2. private：只能在自己的类中
3. public：任意位置都可以访问
4. protected：子类，同包中

#### 静态变量

静态变量属于类而不是对象，它们在类加载时初始化，所有对象共享相同的静态变量。

程序启动时，所有的静态资源都会被加载到内存，使用的时候内存直接调用，而非静态的资源只有在你调用的时候，才会加载至内存。用类名去访问。

#### 静态方法

静态方法属于类而不是对象，可以通过类名调用，无需创建类的实例。

静态方法中只能直接访问静态成员，不能使用非静态成员

静态方法不能使用 this，super 关键字

静态方法不能被重写，不能修饰构造器

#### 静态块

静态块主要用来实例化当前对象之前做的初始化工作，不必写在构造方法之前，位置可以随便写，每一次创建对象之前，块都会执行一次。

#### 静态类

在 Java 中，类可以嵌套在其他类中。静态类是一种嵌套类，它使用 static 修饰符来声明，通常用于封装辅助功能。

#### 内部类

内部类是定义在其他类内部的类。它可以访问外部类的成员，包括私有成员。

#### 匿名内部类

匿名内部类是一种没有类名的内部类，通常用于创建接口的实例。

```java
Runnable r = new Runnable() {
    public void run() {
        System.out.println("Running...");
    }
};
```

#### 抽象类

抽象类是一种不能实例化的类，通常用作其他类的基类。包含一个抽象方法就是抽象类。

抽象类包含的抽象方法必须在其子类中实现，否则该子类也是抽象类。抽象方法不能是 static。

类实现了接口，接口中的抽象方法必须全部实现，不然报错。而如果抽象类实现了这个接口，可以不用全部实现这些方法，这个抽象类的子类如果不实现，那么这个子类也是抽象类。

#### 重写

重写是子类重新定义父类方法的过程。子类的重写方法必须与父类的方法具有相同的名称、参数列表和返回类型。

#### 重载

重载是在同一类中定义多个具有相同名称但参数列表不同的方法。返回值类型可同可不同。构造方法也是可以重载的。

### 对象

对象是类的实例，它是类定义的模板的具体实现。对象具有状态和行为，并且可以通过方法来访问和操作。

在创建对象的时候也可以用接口创建：【接口名】 【对象名】= new 【实现接口的类】，这样你想用哪个类的对象就可以 new 哪个对象了，不需要改原来的代码，这个叫统一访问。

### 接口

接口是一种抽象类型，用于定义一组方法的签名，但没有提供实现。类可以实现一个或多个接口。

接口中的常量：public static final。接口中方法：public abstract 。都会自动补全这些，且必须是这样的。

### 封装

封装是一种将数据和方法封装在类中的机制，以隐藏数据的实现细节。它通过私有成员变量和公共方法实现。

### 继承

继承是一种机制，允许一个类继承另一个类的属性和方法。子类可以扩展父类并添加新的功能。

子类继承父类所有的属性和方法，但不能继承父类的构造方法，同时也可以增加自己的属性和方法。

实例化子类时，先实例化父类。super() 调用父类的无参构造或有参构造，且只能写在子类构造的第一行。子类创建对象时，会先调用父类的构造。

### 多态

多态是一种特性，允许不同类的对象对同一方法做出不同的响应。它通过方法的重写实现。

通过父类型去代表某个子类型：父类 Person p = new 子类 Student()

参数，返回值的多态：参数是父类型，可以传入任意子类型。返回值是父类型，可以返回任意子类型。

子类型转父类型，向上会自动转换，叫多态，向下就得强制转换。

## Java 8 新特性

### Lambda 表达式

语法：`(parameters) -> expression` 或者 `(parameters) -> { statements; }`

Lambda 表达式在许多情况下都可以代替传统的匿名内部类，使代码更加简洁和易读，只能用于函数式接口（只有一个抽象方法的接口），否则会导致编译错误。

```java
// 传统的匿名内部类方式
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, World!");
    }
};

// 使用 Lambda 表达式
Runnable lambdaRunnable = () -> System.out.println("Hello, Lambda!");
```

### 方法引用

方法引用是 Java 8 引入的一种语法糖，用于简化 Lambda 表达式的写法，使代码更加简洁和易读。方法引用允许你通过方法的名称来引用一个已经存在的方法，而不需要像 Lambda 表达式那样提供方法的具体实现。主要作用是将这个方法作为参数传递给其他方法。

使用一对冒号 :: 声明该方法是引用的。静态方法引用 `类名::静态方法名`，实例方法引用 `实例::实例方法名`，对象方法引用 `类名::实例方法名`，构造方法引用 `类名::new`。

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 对象方法引用
names.forEach(System.out::println);
```

### 函数式接口

函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。函数式接口可以被隐式转换为 lambda 表达式。

使用 @FunctionalInterface 注解来显式声明一个接口为函数式接口。

```java
@FunctionalInterface
interface GreetingService
{
    void sayMessage(String message);
}
```

使用 Lambda 表达式来表示该接口的一个实现(Java 8 之前一般是用匿名类实现的)

```java
// 使用lambda表达式实现GreetingService 接口
GreetingService greetService1 = message -> System.out.println("Hello " + message);

// 使用匿名内部类实现 GreetingService 接口
GreetingService greetService2 = new GreetingService() {
    @Override
    public void sayMessage(String message) {
        System.out.println("Hello " + message);
    }
};
```

### 默认方法

接口可以自己有默认实现方法，而且不需要实现类去实现其方法。

只需在方法名前面加个 default 关键字即可实现默认方法。也可以有静态默认方法。

```java
public interface Vehicle {
   default void print(){
      System.out.println("我是一辆车!");
   }

   // 静态默认方法
   static void blowHorn(){
      System.out.println("按喇叭!!!");
   }
}
```

### Stream API

Stream 用来处理集合数据，它允许通过链式操作处理集合，进行过滤、映射、排序、归约等操作。

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count();
```
