## 与 Javascript 的异同

### 相似点

Dart 和 JavaScript 一样，都是单线程语言，通过事件循环来处理异步任务。

Dart 和 JavaScript 都支持异步编程模型，使用类似的语法（例如，JavaScript 中的 Promise 和 async/await，Dart 中的 Future 和 async/await）来处理非阻塞的异步操作。

Dart 的设计目标之一就是支持跨平台开发，可以在不同的平台上运行。所以 Dart 也是被虚拟机编译成字节码并运行，类似于 JavaScript 在浏览器中运行时被解释执行。

Dart VM 和 V8 都支持 JIT（即时编译）技术，这意味着它们能够将源代码转换为机器代码，并在运行时执行。Dart 虚拟机在不同的操作系统上有不同的实现。

Dart 使用 Pub 作为其包管理系统，类似于 JavaScript 中的 npm。你可以方便地引入和管理依赖项。

Dart 是面向对象语言，但也支持过程式编程和函数式编程的特性。所以完全可以写函数，采用面向过程的编程风格，就像在 JavaScript 中一样。

### 不同点

Dart 默认启用空安全。JavaScript 不支持空安全。

Dart 是一门面向对象的语言，且是强类型的（但类型注释是可选的，因为 Dart 可以推断类型）。JS 更像是面向过程的，且是弱类型的。

Dart 应用程序是单一入口文件，入口点是 main()函数。

## Dart 语法

Dart 官网 <https://dart.dev/language>

Dart 在线练习：<https://dartpad.dev/?id>

### 变量 & 常量

Dart 将所有变量的作用域限制在块级别。

#### 变量

定义变量用具体的类型，也可以用 var 自动推断类型，dynamic 或者 Object 表示任何类型。

变量存储的是引用。所以下面名为 "name" 的变量包含对一个值为 "Bob" 的 String 对象的引用。

```dart
String name = 'Bob' // 主动声明类型
var name = 'Bob'; // 自动推断类型String
```

#### 常量

final 或者 const 可以定义常量，也能自动推断类型。类的实例常量，只能用 final，不能用 const。一般写代码时都使用 final 即可。

```dart
final name = 'Bob'; // 也能自动推断
final String nickname = 'Bobby';
```

#### 延迟初始化

变量在使用前必须初始化。如果不想在定义时就初始化，可以用 late 关键字。

```dart
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```

#### 解构

```dart
var numList = [1, 2, 3];
var [a, b, c] = numList;
print(a + b + c);

var (a, b) = ('left', 'right');
```

#### 空安全

Dart 语言默认启用空安全，即不可为任何变量或参数赋值为 null。

除非明确告诉 Dart 变量可以为 null，要表明变量可能具有值 null，需添加 `?` 到其类型声明中。

```dart
int? aNullableInt = null;
```

#### 空感知运算符

跟 js 中使用方式一样。`??` 运算符 (空值合并运算符)，`??=` 运算符 (空值合并赋值运算符)，`?.` 运算符 (可选链运算符)。

```dart
String? value = null;
String result = value ?? "default";
print(result);  // "default"

String? value = null;
value ??= "new";
print(value);  // "new"
```

#### 级联操作符

`..`，`?..` 允许您对同一对象进行一系列操作。`?..` 表示对象可能为空，此时这个后面的级联操作不再执行。

```dart
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;

// 前面的示例相当于以下代码：

var paint = Paint();
paint.color = Colors.black;
paint.strokeCap = StrokeCap.round;
paint.strokeWidth = 5.0;
```

### 数据类型

所有值都是对象，也就是可以放入变量中的所有内容都是对象，即使是数字、函数、null。每个对象都是类的实例。除了 null 之外，所有对象都继承自 Object 类。

null 是一个特殊的对象，它不继承自 Object 类。实际上，null 是所有类型的子类型，这意味着 null 可以被赋值给任何对象引用类型，而不会引发类型错误。

#### 常用类型

数字：int, double, num

将变量声明为 num，变量可以同时具有整数和双精度值。

```dart
int integerNumber = 42;
double floatingPointNumber = 3.14;

num x = 1;
x += 2.5;
```

字符串：String，`${}` 用于字符串插值，将表达式插入到字符串文本中。如果使用$字符串需要加反斜杠 `\`。

多行字符串可以直接定义写成多行，或者用三个引号表示多行。

```dart
String text = 'Hello, Dart!';

var food = 'bread';
var str = 'I eat ${food}';


final s1 = 'String '
    'concatenation'
    " even works over line breaks.";

final s2 = '''
You can create
multiline strings like this one.
''';
```

布尔值：bool

```dart
bool isTrue = true;
bool isFalse = false;
```

数组(有序的集合)：List，也可以使扩展运算符 `...` 和 `...?`，`...?`表示不为空就展开。也适用于 Set 和 Map。

添加：add，addAll

更新：arr[index] = 'xxx';

插入：insert 或 insertAll。

删除：remove、removeAt、removeLast、removeRange 或 removeWhere。

替换：fillRange、ReplaceRange 或 setRange。

查找：indexOf 或 lastIndexOf。

遍历：for，for-in，forEach，map，where 等。

判断为空：isEmpty，isNotEmpty

```dart
List<int> numbers = [1, 2, 3, 4, 5];
List<String> names = ['Alice', 'Bob', 'Charlie'];
```

不重复的元素的集合：Set。注意他是用大括号 `{}`

添加：add，addAll

删除：remove、removeAll、removeWhere

判断为空：isEmpty，isNotEmpty

```dart
Set<int> uniqueNumbers = {1, 2, 3, 4, 5};
```

键值对的集合：Map

是否有 Key：containsKey

添加或者更新：map['xxx'] = 'yyy';

添加另一个 Map：addAll。

删除：remove。

判断为空：isEmpty，isNotEmpty

```dart
Map<String, dynamic> person = {
  'name': 'John',
  'age': 30,
  'isStudent': false,
};
```

List，Set，Map 中可以使用 if 和 for。

#### 不可变

使用 const 关键字，则该集合具有不可变性。如果是 final，这确保了该字段不能被另一个字段覆盖，但它仍然允许修改该字段的大小或内容。

```dart
const fruits = <String>{'apple', 'orange', 'pear'};
```

也可使用 unmodifiable 方法。但是该方法会返回一个集合的拷贝。也就是说会创建一个副本。

```dart
final _set = Set<String>.unmodifiable(['a', 'b', 'c']);
```

如果不想额外创建副本，在 dart:collection 库中提供了一个 UnmodifiableListView 类，可以将原始数据包装在防止修改的视图中。

```dart
var originalList = [1, 2, 3];
var unmodifiableCopy = List<int>.unmodifiable(originalList);
var unmodifiableView = UnmodifiableListView(originalList);
```

#### 相等比较

identical 检查是否为相同的对象（相同的引用），而 == 检查是否在逻辑上相等，即它们的内容是否相同。

```dart
var a = [1, 2, 3];
var b = a;
print(identical(a, b));  // 输出 true，因为a和b引用相同的对象
```

```dart
var a = [1, 2, 3];
var b = [1, 2, 3];
print(a == b);  // 输出 true，因为a和b在逻辑上相等
```

#### 类型推断

Dart 可以使用 var 或者 final 进行推断类型。

```dart
var arguments = {'argA': 'hello', 'argB': 42}; // Map<String, Object>
```

如果显式声明类型，则可以这样写

```dart
Map<String, Object> arguments = {'argA': 'hello', 'argB': 42};
```

`as`、`is` 和 `is!` 可以方便地在运行时检查类型。

#### 任意类型

任意类型可以用 Object 或者 dynamic。dynamic 类型在编译时不进行静态类型检查，而是在运行时进行。

```dart
Object myObject = "Hello";
print(myObject.toString());  // 输出 "Hello"

dynamic myDynamic = "Hello";
print(myDynamic.toString());  // 输出 "Hello"
```

#### 泛型

Dart 中支持泛型

```dart
var names = <String>['Seth', 'Kathy', 'Lars'];
var uniqueNames = <String>{'Seth', 'Kathy', 'Lars'};
var pages = <String, String>{
  'index.html': 'Homepage',
  'robots.txt': 'Hints for web robots',
  'humans.txt': 'We are people, not machines'
};
```

#### 类型定义

定义类型别名，可以使用类型参数

```dart
typedef IntList = List<int>;
IntList il = [1, 2, 3];

typedef ListMapper<X> = Map<X, List<X>>;
ListMapper<String> m2 = {}; // 等于 Map<String, List<String>> m2 = {};
```

#### 获取对象的类型

要在运行时获取对象的类型，您可以使用 runtimeType，它返回一个 Type 对象。

```dart
print('The type of a is ${a.runtimeType}');
```

### 控制语句，流程语句，循环语句

跟 js 一样。

控制语句：if-else, switch-case

流程语句：break, continue

循环语句：for, while, do-while, for-in, forEach

Dart for-in 循环的工作方式类似于 JavaScript for-of。

```dart
for (final element in list) {
  print(element);
}
```

#### if-case 语句

如果值匹配到某个模式，分支将执行。

```dart
if (pair case [int x, int y]) return Point(x, y);
```

### 异常

Dart 提供了 Exception 和 Error 类型，以及许多预定义的子类型。也可以抛出任何非 null 对象（而不仅仅是 Exception 和 Error 对象）作为异常。

```dart
throw FormatException('Expected at least 1 section');

// 抛出任意对象：
throw 'Out of llamas!';
```

on 可以捕获指定异常，catch 捕获任意异常。

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException { // 不需要异常对象
  buyMoreLlamas();
} on Exception catch (e) { // 可以拿到异常对象
  print('Unknown exception: $e');
} catch (e) { // 捕获所有类型异常
  print('Something really unknown: $e');
}
```

#### 断言

在开发过程中，使用断言语句 assert，如果执行时这个断言返回的 false，则会中断正常执行。

```dart
var text = '123';
assert(text != null);

// 如果text是null，下面不会再执行
print(text)
```

### 函数

与 JavaScript 类似，可以在任何地方声明函数，无论是在顶层、作为类字段还是在当前的作用域范围。

Dart 中函数也是有作用域范围，也有闭包。

函数尽量声明返回值类型，当然不声明也可以正常运行。

```dart
// 有返回值
int add(int a, int b) {
  return a + b;
}

// 无返回值
void sayHello() {
  print('Hello!');
}

// 无声明
sayHello() {
  print('Hello!');
}
```

#### 函数是一等公民

将函数作为参数传给另一个函数

```dart
void printElement(int element) {
  print(element);
}

var list = [1, 2, 3];

// Pass printElement as a parameter.
list.forEach(printElement);
```

#### 匿名函数

没有名称的函数，可以将它们作为参数传递给另一个函数，或者从另一个函数返回它们。

```dart
var list = ['apples', 'bananas', 'oranges'];
list.map((item) {
  return item.toUpperCase();
});
```

直接将匿名函数赋值给变量

```dart
var multiply = (int a, int b) => a * b;
```

#### 箭头函数

Dart 中箭头函数表达式跟 js 略有不同，只有当函数包含单个表达式或 return 语句时才能使用箭头语法。

`=> expr` 是 `{ return expr; }` 的简写。

```dart
var multiply = (int a, int b) => a * b;
```

#### 可选参数

可选参数必须放在必须参数的后面，且用中括号包裹，且必须具有默认值或标记为可为 null。

```dart
multiply(int a, [int b = 5, int? c]) {
}

// 函数调用
multiply(3);
multiply(3, 5);
multiply(3, 5, 7);
```

#### 命名参数

命名参数跟可选参数类似，但是不必像可选参数那样按照定义的顺序提供，可以通过名称来引用它们。同时用大括号括起来。

默认情况下这些也都是可选的，除非它们被标记为必需。

```dart
multiply(bool x, {required int a, int b = 5, int? c}) {
}

// 函数调用
multiply(false, a: 3);
multiply(false, a: 3, b: 9);
multiply(false, c: 9, a: 3, b: 2);
```

### 异步

#### Future

类似 js 的 Promise，也是用 then 处理成功的回调，但是用 catchError 而不是 catch 来捕获异常。whenComplete 类似于 js 的 finally。

```dart
Future<String> httpResponseBody = func();

httpResponseBody.then((String value) {
    print('Future resolved to a value: $value');
}).catchError((err) {
    print('Future encountered an error before resolving.');
});
```

类似于 js 的 Promise.all 等方法，Dart 有 Future.wait 来同时处理多个 Future。同样的也有 any 方法，有一个完成就结束，不管是成功或者失败。

```dart
var value = await Future.wait([delayedNumber(), delayedString()]);
print(value); // [2, result]

final result = await Future.any([slowInt(), delayedString(), fastInt()]);
// The future of fastInt completes first, others are ignored.
print(result); // 3
```

Future 也有 value 和 error 方法，来直接返回一个 Future 的成功或者失败的对象。

```dart
return Future.value(2021);

return Future.error(Exception('Issue'));
```

Future 可以通过 sync 方法使函数立即执行，而不会导致异步行为。也可以通过 delayed 方法设置延迟一段时间再执行那些回调函数。

Future 实例还有一个 timeout 方法，可以在超过 timeout limit 后不再继续等待成功或者失败的结果。

```dart
// 立即执行
final result = await Future<int>.sync(() => 12);

// 延迟执行
Future.delayed(const Duration(seconds: 1), () {
  print('One second has passed.'); // Prints after 1 second.
});

// timeout
var result = await waitTask("completed").timeout(const Duration(seconds: 10));
print(result); // Prints "completed" after 5 seconds.
var result = await waitTask("completed").timeout(const Duration(seconds: 1), onTimeout: () => "timeout");
print(result); // Prints "timeout" after 1 second.
```

#### async && await

跟 js 一样，可以使用 async 和 await 来处理异步。

使用 try、catch 和 finally 来处理 await 的异常。

```dart
Future<void> checkVersion() async {
    try {
        var version = await lookUpVersion();
    } catch (e) {
        // ...
    } finally {
        // ...
    }
}
```

#### isolate

在应用程序中，所有 Dart 代码都在 isolate 中运行（许多 Dart 应用程序只使用一个 isolate，即主 isolate。）。每个 isolate 都有一个执行线程，并且不与其他 isolate 共享内存和变量，所以为了相互通信，isolate 使用消息传递。

如果任务只是普通的异步等待，不需要大量的 CPU 计算，那么我们可以直接使用 Future 来进行异步处理。如果是 CPU 密集型的任务，比如 IO 操作，可以考虑使用 Isolate 的方式将其放在单独的线程中执行，以避免阻塞主线程。

使用 Isolate.run() 来创建和执行 Isolate。在 Flutter 中，可以使用 compute()，compute 是 Flutter 中对 Isolate.run() 的封装。

```dart
void download() async{
    const String downloadLink = '下载链接';
    String fileContent = await Isolate.run(() => downloadAndRead(downloadLink));
    print('展示文件内容: $fileContent');
}

// flutter
void download() async{
    const String downloadLink = '下载链接';
    String fileContent = await compute(downloadAndRead,downloadLink);
    print('展示文件内容: $fileContent');
}
```

### 面向对象

Dart 是一种面向对象的语言，具有类和继承。每个对象都是一个类的实例。

this 仅在类中使用，并且始终引用当前实例。

#### 构造方法和实例

构造方法可以是默认构造方法或者命名构造方法。默认构造方法与类同名，而命名构造方法具有自定义名称。

在构造方法中，可以使用 this 关键字来引用当前实例，以初始化实例变量。

创建实例不需要 new 关键字。

```dart
class Person {
  String name;
  int age;

  // 默认构造方法
  Person(this.name, this.age);

  // 命名构造方法
  Person.guest() {
    name = 'Guest';
    age = 18;
  }
}

void main() {
  // 使用默认构造方法创建实例
  Person person1 = Person('Alice', 30);

  // 使用命名构造方法创建实例
  Person person2 = Person.guest();
}
```

#### 成员变量、成员方法

所有成员变量都会生成隐式 getter 方法，非 final 变量也会生成隐式 setter 方法。

Dart 没有访问修饰符关键字，所有类成员默认都是公共的。要在 Dart 中将类成员设置为私有，请在其名称前添加下划线 (\_)。

```dart
class Person {
  String name; // 成员变量
  int age; // 成员变量

  void sayHello() { // 成员方法
    print('Hello, my name is ${this.name}'); // 使用this引用实例变量
  }
}
```

#### this

关键字 this 用于引用当前实例。

this 可以用来区分实例变量和局部变量。

```dart
class Person {
    String name; // 实例变量

    Person(this.name); // 构造方法

    void printName() {
        String name = 'Local'; // 局部变量
        print(this.name); // 使用this引用实例变量
        print(name); // 打印局部变量
    }
}
```

#### 静态变量、静态方法

通过 static 关键字来定义静态变量和方法。

```dart
class MathUtils {
  static const double pi = 3.14; // 静态变量

  static int add(int a, int b) { // 静态方法
    return a + b;
  }
}
```

#### 抽象类、接口、mixins

抽象类是不能被实例化的类，用于定义一组方法，但不提供这些方法的具体实现。抽象方法是在抽象类中声明但没有实现的方法，它们需要在子类中被重写以提供具体的实现。

抽象方法只能在抽象类或者 mixins 中。

```dart
// 定义一个抽象类
abstract class Animal {
  // 定义一个抽象方法
  void makeSound();
}

// 定义一个继承自抽象类的子类
class Dog extends Animal {
  // 实现抽象方法
  void makeSound() {
    print('Woof! Woof!');
  }
}
```

在 Dart 中，接口和抽象类可以看作是相似的概念。接口也是一种抽象类型，用于定义类应该具有哪些方法和属性，但不提供这些方法和属性的具体实现。类可以实现一个或多个接口，以确保它们具有所需的行为。

```dart
// 定义一个接口
abstract class Shape {
  void draw(); // 定义一个抽象方法
}

// 定义一个实现了接口的类
class Circle implements Shape {
  @override
  void draw() {
    print('Drawing a circle');
  }
}
```

Mixin 是一种在类中实现代码重用的机制。一个类可以使用多个 mixin。

```dart
mixin Swim {
  void swim() {
    print('Swimming');
  }
}

class Fish with Swim {
  // Fish类使用Swim mixin中的方法
}
```

#### 继承

继承是面向对象编程中的重要概念，它允许一个类（子类）获取另一个类（父类）的属性和行为。子类可以继承父类的成员变量和成员方法，并且可以通过重写方法来修改或扩展父类的行为。

子类构造方法需要调 super 引用父类的构造函数。

```dart
// 定义一个父类
class Animal {
  String name;

  Animal(this.name);

  void eat() {
    print('${this.name} is eating');
  }
}

// 定义一个子类，并继承父类
class Dog extends Animal {
  Dog(super.name); // 等同于：Dog(String name) : super(name);

  void bark() {
    print('Woof!');
  }
}

void main() {
  // 创建子类的实例
  var dog = Dog('Buddy');
  dog.eat(); // 输出 Buddy is eating
  dog.bark(); // 输出 Woof!
}
```

#### super 参数

用 super 来初始化参数是一种机制，用于避免在构造函数的 super 调用中手动传递每个参数。这种特性使得在子类构造函数中能够将参数快速地传递给指定或默认的父类构造函数。这对于简化代码、提高可读性非常有用。

```dart
class Vector2d {
  final double x;
  final double y;

  Vector2d(this.x, this.y);
}

class Vector3d extends Vector2d {
  final double z;

  // Forward the x and y parameters to the default super constructor like:
  // Vector3d(final double x, final double y, this.z) : super(x, y);
  Vector3d(super.x, super.y, this.z);
}
```

#### 单例模式

有些类提供常量构造函数，使用常量构造函数创建编译时常量，这个实例就是相等的。

```dart
class MySingleton {
  const MySingleton();

  void doSomething() {
    print('Doing something...');
  }
}
void main() {
    var singleton1 = const MySingleton();
    var singleton2 = const MySingleton();

    print(identical(singleton1, singleton2)); // true
}
```

### 导入导出模块

import 和指令 library 可以帮助您创建模块化的代码，每个 Dart 文件都是一个模块（库），即使它不使用 library 指令。以下划线 `(_)` 开头的标识符仅在模块（库）内部可见。

#### 使用模块（库）

对于内置库，URI 有特殊的 `dart:xxx` 方案。对于其他库，您可以使用文件系统路径或 `package:xxx`，package 指定由包管理器（例如 pub 工具）提供的库。

```dart
import 'dart:html';
import 'package:test/test.dart';
```

#### 指定库前缀

如果导入两个具有冲突标识符的库，则可以为一个或两个库指定前缀。

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

lib2.Element element2 = lib2.Element();
```

#### 仅导入库的一部分

使用 `show` 或者 `hide` 可以选择导入某部分或者不导入某部分。

```dart
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```

#### 导出

export 关键字用于在导出某个库或者其中的某个方法。

```dart
// library1.dart
library library1;

void function1() {
  print('Function 1');
}

void function2() {
  print('Function 2');
}

// 使用 export 导出 library1 的内容
export 'library1.dart';
```

另一个文件中可以导入

```dart
// main.dart
import 'library1.dart';

void main() {
  function1();  // 输出 'Function 1'
  function2();  // 输出 'Function 2'
}
```

#### 文件懒加载

仅 dart compile js 支持延迟加载。 Flutter 和 Dart VM 不支持延迟加载。

延迟加载库，必须首先使用 `deferred as`。然后使用库时需要 await 或者 then

```dart
import 'package:greetings/hello.dart' deferred as hello;

await hello.loadLibrary();
```

### 包

包搜索：<https://pub.dev/>

Dart 包是一个包含 pubspec.yaml 文件的目录，pubspec.yaml 包含一些有关包的元数据。pubspec.lock 会锁包版本。

类似于 package.json。

```yaml
name: my_app

dependencies:
  js: ^0.6.0
  intl: ^0.17.0
```

使用 `dart pub get` 安装依赖，`dart pub add xxx` 添加依赖。`dart pub upgrade` 可以更新包版本。

#### 创建包

需要有 pubspec.yaml 文件和 lib 目录。如果 lib 下有 src 目录，lib/src 下的代码被认为是私有的。

`lib/<package-name>.dart` 是你的主文件，负责导出所有公共的 api。实现文件一般放在 lib/src 下，而不是放在 lib 下，防止被用户直接导入。

```dart
export 'src/cascade.dart' show Cascade;
export 'src/handler.dart' show Handler;
```

#### 常用第三方包

- archive：压缩解压相关。
- characters：用于处理 Unicode 字符和字符串的库。
- http：发送请求。
- path：处理文件路径。
- yaml：处理 yaml 文件并读取。

下面是一些扩展了 dart 核心功能的包：

async，collection，convert，io

#### 发布包

使用 `dart pub publish` 命令。
