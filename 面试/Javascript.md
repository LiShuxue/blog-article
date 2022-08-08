## == 和 ===
* ==：比较值，类型不同的时候，先进行类型转换，再比较值；1 == "1"  true  
* ===：比较类型和值，不做类型转换，类型不同就是不等；1 === "1"  false

## == 比较时的隐式转换
* 基本数据类型比较，比较值
    * null == undefined比较结果是true，除此之外，null、undefined和其他任何结果的比较值都为false。
    * Boolean 和其他任何类型比较，Boolean 被转换为 Number 类型。Number(false) 是 0，Number(true) 是 1
    * String和Number比较，先将String转换为Number类型( parseInt或者Number() )。除了数字型string可以转成number，其他都是NaN。Number('') 是 0
    * NaN和其他任何类型比较永远返回false。
* 两个引用类型比较，引用地址不一样，直接false。
* 原始类型和引用类型做比较时，引用类型会依照ToPrimitive规则转换为原始类型。
    * 引用类型先调用valueOf()，如果是基本类型，直接返回
    * 不是基本类型，再调用toString
    * String再转Number，
    * [].valueOf().toString() === ''，转number是0 
    * {}.valueOf().toString() === '[object Object]'，转number是NaN 
    * [] == false // true
    * {} == false // false

## 与或非优先级运算
1. 与或非混合时，非>与>或。 
    ```js
    true && !false || false // true
    false || !false && false // false
    ```
2. && 和 || 的短路运算
    * &&的短路运算：左边能转成false，无条件返回左边式子的值。反之无条件返回右边式子的值
        ```js
        false && 1; // 输出false
        true && 1; // 输出1
        ```
    * ||的短路运算：若左边能转成true，无条件返回左边式子的值。反之无条件返回右边式子的值
        ```js
        false || 1; // 输出1
        true || 1; // 输出true
        ```

## 按位与 &， 按位或 |， 按位异或 ^，左移 <<，右移 >>，
将十进制转化为二进制，再进行与或移位运算

1. 按位与 &，如果对应的位都为1，那么结果就是1， 如果任意一个位是0 则结果就是0
```js
// 1的二进制表示为: 00000000 00000000 00000000 00000001
// 3的二进制表示为: 00000000 00000000 00000000 00000011
// 按位与 &
// 结果为：        00000000 00000000 00000000 00000001
console.log(1 & 3)     // 1
```
2. 按位或 |，对应的位中任一个数为1 那么结果就是1
```js
// 1的二进制表示为: 00000000 00000000 00000000 00000001
// 3的二进制表示为: 00000000 00000000 00000000 00000011
// 按位或 |
// 结果为：        00000000 00000000 00000000 00000011
console.log(1 | 3)     // 3
```
3. 按位异或 ^，对应两个位有且仅有一个1时结果为1，其他都是0
```js
// 1的二进制表示为: 00000000 00000000 00000000 00000001
// 3的二进制表示为: 00000000 00000000 00000000 00000011
// 按位异或 ^
// 结果为：        00000000 00000000 00000000 00000010
console.log(1 ^ 3)     // 2
```
4. 左移 <<，向左移动对应的位数，高位移出，低位的空位补零。相当于算术运算的乘法，乘以2的n次方
```js
// 1的二进制表示为: 00000000 00000000 00000000 00000001
// 左移一位
// 结果为：        00000000 00000000 00000000 00000010
console.log(1 << 1)     // 2，     1左移一位，相当于 1 * 2¹
```
5. 右移 >>，除了最左边符号位，其他位向右移动对应的位数，被移出的位被丢弃。相当于算术运算的除法，除以2的n次方并取整
```js
// 64的二进制表示为: 00000000 00000000 00000000 01000000
// 右移5位
// 结果为：         00000000 00000000 00000000 00000010
console.log(64 >> 5)     // 2，    64左移5位，相当于 64 / 2⁵
```

## 十进制转二进制 和 parseInt
1. 转二进制toString(2)：
```js
const number = 100;
number.toString(2)
```

2. parseInt(string, radix);

解析一个字符串并返回指定基数的十进制整数。 radix 是2-36之间的整数，表示被解析字符串的基数，例如指定 16 表示被解析值是十六进制数。请注意，10不是默认值！

如果 radix 是 undefined、0或未指定的，JavaScript会假定以下情况：

* 如果输入的 string以 "0x"或 "0x"（一个0，后面是小写或大写的X）开头，那么radix被假定为16，字符串的其余部分被当做十六进制数去解析。
* 如果输入的 string以 "0"（0）开头， radix被假定为8（八进制）或10（十进制）。具体选择哪一个radix取决于实现。ECMAScript 5 澄清了应该使用 10 (十进制)，但不是所有的浏览器都支持。因此，在使用 parseInt 时，一定要指定一个 radix。
* 如果输入的 string 以任何其他值开头， radix 是 10 (十进制)。

如果第一个字符不能转换为数字，parseInt会返回 NaN。

## Object.is(value1, value2)
`Object.is()`是在ES6中定义的一个新方法，它与‘===’相比，特别针对-0、+0、NaN做了处理。
```js
+0 === -0 // true
NaN === NaN // false
Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
Object.is(NaN, 0/0) // true
```

## 6种基本数据类型和3种引用类型
* number, boolean, string, null, undefined, Symbol  
* Object, Array, Function

## javascript自带的数据结构
* Object, Array, Set, Map, WeakSet, WeakMap

## null和undefined有什么区别
* null表示空对象  
* undefined表示未初始化的变量

## number的位数，最大/小值，最大/小安全整数
JS的基础类型Number，遵循 IEEE 754 规范，采用双精度存储，占用 64 bit。其中 0 到 51 存储数字（占52位），52 到 62 存储指数（占11位），63 位存储符号，如图。
![number](https://raw.githubusercontent.com/LiShuxue/blog-article/master/面试/number.png)

* Number.MAX_VALUE，
* Number.MIN_VALUE，
* Number.MAX_SAFE_INTEGER，
* Number.MIN_SAFE_INTEGER


## Object 和 Map 有什么区别
* Object 键（key）的类型只能是字符串，数字或者 Symbol；而 Map 可以是任何类型。
* Map 中的元素会保持其插入时的顺序；而 Object 则不会完全保持插入时的顺序。
* Object的key顺序：
    1. 先number类型，后字符串类型。
    1. key是整数或者整数类型的字符串，那么会按照从小到大的排序。
    2. 其它类型字符串，控制台展示的时候，按照ASC码升序排序，如果用Object.keys()获取，按照实际创建顺序排序。
* Map 是可迭代对象，所以其中的键值对是可以通过 for of 循环或 .foreach() 方法来迭代的；而普通的对象键值对则默认是不可迭代的，只能通过 for in 循环来访问。

## 一个对象作为key，会将对象toString()，转成[object Object]，第二个O大写
```js
var a = { name: "Sam" };
var b = { name: "Tom" };
var o = {};
o[a] = 1;
o[b] = 2;
console.log(o[a]); // 2
```

## function.length可以获取函数的参数个数，arguments.length也可以
```js
const sum = (a, b, c) => a + b + c
sum.length // 3
```

## 函数柯里化
柯里化是指把接收多个参数的函数变换成接收单一参数的函数，嵌套返回直到所有参数都被使用并返回最终结果。

更简单地说，柯里化是一个函数变换的过程，是将函数从调用方式：f(a,b,c)变换成调用方式：f(a)(b)(c)的过程。柯里化不会调用函数，它只是对函数进行转换。

## 基本数据类型和引用类型存储在哪里
* 基本数据类型：变量标识符和变量的值存放于栈内存。占用固定大小的空间。  
* 引用类型：变量标识符和对象的指针存放于栈，具体的对象存放于堆。

## 一个string，他不定长，放在栈中是怎么处理的？
基本数据类型不一定是直接存在栈中的。

* 字符串： 存在堆里，栈中为引用地址，如果存在相同字符串，则引用地址相同。
* 数字： 小整数（-2³¹ 到 2³¹-1（2³¹≈2*10⁹）的整数）存在栈中，其他类型存在堆中，如小数。
* 其他类型：引擎初始化时分配唯一地址，栈中的变量存的是唯一的引用。

## 函数的参数是按照什么方式传递的
### 按值传递 vs 按引用传递
* 按值传递：实际参数把它的值传递给对应的形式参数，形式参数只是用实际参数的值初始化自己的存储单元内容，是两个不同的存储单元，所以方法执行中形式参数值的改变不影响实际参数的值。按值传递的意思就是形参是实参的复制。    
* 按引用传递：就是传递对象的引用，实参和形参是完全一样的，函数内部对形参的任何改变都会影响实际参数。函数的形参接收实参的隐式引用，而不再是副本。  

### 首先理解复制变量： 
* 基本数据类型：将原始值副本赋值给新的变量，所以两个变量是完全独立的，只不过value相同。  
* 复杂类型：将指针（内存地址）赋值给新的变量，所以两个变量保存的内存地址一致，也就是指向的具体的对象一致。任何一个改变都会影响另一个。

### 传递参数
传递参数就是变量复制的过程，将实参的值赋值给形参。  
其实都是<b>按值传递</b>的。只不过有的值是基本类型，有的值是内存地址  

### 证明不是按引用传递
```js
var person = { name: "MJ" };
function changeName(obj) {
    obj.name = 'test';
    obj = new Object();
    obj.name = "EP";
}
changeName(person);
console.log(person.name); // 输出test
```
说明形参obj和实参person是两个东西，只不过值相同，都是同一个内存地址，都指向了同一个对象。  
如果是按引用传递，那他两个应该是同一个东西，修改obj=new Object()之后，person也应该是这个新的object。

## js内置对象
基本对象：数据类型对应的对象
1. Object
2. Function
3. Array
4. Number 
5. String
6. Boolean

其他对象：
1. Error
2. Date
3. Math
4. RegExp
5. JSON

ES6新增对象：
1. Symbol
2. Set & WeakSet
3. Map & WeakMap
4. Proxy
5. Reflect
6. Promise

## 闭包
一个函数包含另一个函数，并且被包含的函数使用了父函数中的变量。内部函数即为闭包。  
即有权访问另一个函数作用域内变量的函数都是闭包。 

* 闭包的特性：  
    内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回（寿命终结）了之后  

* 闭包的作用：  
    1. 使其他函数可以访问某函数内部的变量
    2. 让函数内部的变量一直隐藏/保存在内存中（但是这样会导致内存泄漏，所以在退出函数之前，应将不使用的局部变量设为null）
    3. 实现私有变量
```js
function Person(){
    var name = 'cxk';
    this.getName = function(){
        return name;
    }
    this.setName = function(value){
        name = value;
    }
}
const cxk = new Person()
console.log(cxk.getName()) //cxk
cxk.setName('jntm')
console.log(cxk.getName()) //jntm
console.log(name) //name is not defined
// 函数体内的var name = 'cxk'只有getName和setName两个函数可以访问，外部无法访问，相对于将变量私有化
```
## 立即执行函数(IIFE)
```js
(function(j){
    console.log(j)
})(i)

(function(j){
    console.log(j)
}(i))
```
作用：  
1. 创建一个独立的作用域。这个作用域里面的变量，外面访问不到（即避免「变量污染」）
2. 如果需要访问这里面的方法，可以挂载在window上，或者return给外面。以此模拟模块。

## js运行机制
JavaScript代码的整个执行过程，分为两个阶段，代码编译阶段与代码执行阶段。

### 编译阶段
* 词法分析（Tokenizing/Lexing）
* 语法分析（Parsing）
* AST 转换为字节码

### 代码执行阶段，创建执行上下文 和 代码执行
创建三种执行上下文
* 全局执行上下文
* 函数执行上下文
* eval执行上下文

执行上下文也分为两个阶段(ES6规范)：
1. 创建阶段
    * 决定 this 的指向
    * 创建词法环境(Lexical Environment) (也就是作用域) 
        1. 函数声明
        2. 变量声明
        3. 确立作用域和作用域链
    * 创建变量环境(VariableEnvironment)
        1. var类型的变量声明（变量提升）
2. 执行阶段  
    * 变量赋值
    * 函数执行

### 函数调用栈
栈里面存储的就是执行上下文。 

在代码执行阶段，先将全局执行上下文放入函数调用栈栈底，然后执行函数，并将函数执行上下文放入栈。  

处于栈顶的上下文执行完毕之后，就会自动出栈。函数中，遇到return能直接终止可执行代码的执行，因此会直接将当前上下文弹出栈。

### 作用域，作用域链
在ES6规范中，作用域更官方的叫法是词法环境（Lexical Environments），它由两部分组成：  
* 记录作用域内变量信息（我们假设let变量，const常量，函数等统称为变量）和代码结构信息的东西，称之为 Environment Record。
* 一个引用 `__outer__`，这个引用指向当前作用域的父作用域。全局作用域的 `__outer__` 为 null。

可执行上下文中的词法环境中含有外部词法环境的引用，我们可以通过这个引用获取外部词法环境的变量、声明等，这些引用串联起来一直指向全局的词法环境，因此形成了作用域链。

### 变量查找：  
先从当前的执行上下文的作用域中查找，查看当前作用域里面的 Environment Record 是否有此变量的信息，如果找到了，则返回当前作用域内的这个变量。  
如果没有查找到，则顺着 `__outer__` 到父作用域里面的 Environment Record 查找，以此递归。

### 变量提升
在执行上下文的创建阶段，会先声明var类型的变量，但不进行赋值，执行阶段才去赋值。  
所以在后面的代码中可以先使用这个变量，但是他是undefined

### var变量声明 和 函数声明都会变量提升
1. var变量声明 和 函数声明都会变量提升
2. 函数声明是整体提升，优先级高于var声明，且直接赋值
3. 具体的执行是什么取决于赋值是什么。
```js
var log = function () {
  console.log(1)
};
function log() {
  console.log(2)
}

log() // 输出 1
// function log 先提升， var log 再提升，function log 会直接给log 赋值。 var log = function 再赋值，所以最后log是consolo.log(1)的
```

### this指向
简单来说，谁调用这个函数，这个函数中的this就指向谁
* `fn()`  fn单独调用，this指向window
* `obj1.obj2.fn();` this指向obj2
* `var bar = obj.fn; bar();` this指向window
* call()、apply()、bind()，指向新的这个对象
* 箭头函数中this指向所在上下文

从原理上来说，在创建可执行上下文的时候，根据代码的执行条件，来判断分别进行默认绑定、隐式绑定、显示绑定等。
```js
var x = 0;
var obj = {
    x: 10,
    bar() {
        var x = 20;
        console.log(this.x);
    }
}

obj.bar() // 10
var foo = obj.bar;
foo() // 0
obj.bar.call(window) // 0
obj.bar.call(obj) // 10
foo.call(obj) // 10
```

## v8执行原理
1. 初始化堆栈空间，全局上下文，全局作用域，事件循环系统。
2. 输入全局的js代码，解析器(Parser)通过词法分析，生成tokens，语法分析根据tokens生成AST抽象语法树。
3. 解释器(Ignition) 会将 AST 转换为字节码，一边解释一边执行。（解释执行）
4. 在解释执行字节码的过程中，如果发现一段代码被多次重复执行，就会将其标记为热点（Hot）代码。V8 会将这段热点代码丢给优化编译器 TurboFan 编译为二进制代码。如果下次再执行时，就会直接执行二进制代码，提高执行速度。（编译执行）
5. 如果遇到普通函数，只会对其进行预解析(Pre-Parser)，验证函数的语法是否有效、解析函数声明以及确定函数作用域，并不会生成 AST，当函数被调用时，才会对其完全解析。
```js
// 源码
var test = 1;

// Tokens
[
    {
        "type": "Keyword",
        "value": "var"
    },
    {
        "type": "Identifier",
        "value": "test"
    },
    {
        "type": "Punctuator",
        "value": "="
    },
    {
        "type": "Numeric",
        "value": "1"
    },
    {
        "type": "Punctuator",
        "value": ";"
    }
]

// AST 抽象语法树
{
  "type": "Program",
  "body": [
    {
      "type": "VariableDeclaration",
      "declarations": [
        {
          "type": "VariableDeclarator",
          "id": {
            "type": "Identifier",
            "name": "test"
          },
          "init": {
            "type": "Literal",
            "value": 1,
            "raw": "1"
          }
        }
      ],
      "kind": "var"
    }
  ],
  "sourceType": "script"
}
```

![v8](https://raw.githubusercontent.com/LiShuxue/blog-article/master/面试/v8.png)

### js 到底是解释型还是编译型语言？
V8 同时采用了解释执行和编译执行这两种方式，这种混合使用的方式称为 JIT (即时编译)。  

## 什么是作用域链，什么是原型链，它们的区别
* 作用域是针对变量的，先在自己的作用域上下文寻找， 找不到再去父级作用域寻找，形成作用域链。
* 原型链是针对对象属性或者方法的， 当我们从一个对象上寻找某个属性或者方法时， 先在本身寻找， 没有的话就去父类的原型上找。父类原型找不到，再去父类的父类的原型找。形成原型链。

## 原型，原型链，构造函数
* <b>原型：</b>即prototype属性，只有函数才有，用来存放所有实例对象需要共享的属性和方法。那些不需要共享的属性和方法，就放在构造函数里面。  
* <b>构造函数：</b>用来创建实例。通过new关键字可以创建实例。命名通常首字母大写。  
* <b>原型链：</b>即__proto__属性，任何对象都有这个属性。  

当我们访问一个对象的属性或方法时，如果这个对象内部不存在这个属性，那么他就会去他的__proto__里找这个属性，也就是去父类的protorype上找，父类的prototype上如果没有，这个prototype又会去找他的__proto__，这个__proto__又会有自己的__proto__，于是就这样 一直找下去，也就是我们平时所说的原型链的概念。

一张图总结：主要是每个部分都要明白__proto__和 constructor的指向。
![proto](https://raw.githubusercontent.com/LiShuxue/blog-article/master/面试/proto3.png)

__proto__有两条链，一条是实例对象的，一条是类的。类中有Function参与，实例的没有。
```js
// 实例的，父类的prototype.__proto__直接指向Object.prototype
son.__proto__ === Son.prototype
Son.prototype.__proto__ === Father.prototype
Father.prototype.__proto__ === Object.prototype
Object.prototype.__proto__ === null

// 类的，类的__proto__属性指向父类，没有父类指向Function.prototype
Son.__proto__ === Father
Father.__proto__ === Function.prototype
Function.prototype.__proto__ === Object.prototype
Object.prototype.__proto__ === null
```

结论：
1. 对象的__proto__属性指向父类的prototype属性，没有父类则指向Object.prototype
2. 类的__proto__属性指向父类，如果没有父类指向Function.prototype。ES5中用function创建的类，类的__proto__都指向Function.prototype。  
---
3. 实例对象的.constructor指向本类
4. 类的prototype.constructor指向本类
5. 类的.constructor指向Function
---
6. 类的prototype是父类的原型的复制， 所以 instanceof 父类是true，没有父类时 instanceof Object是true

5.6 也证明在自己写ES5继承的时候， 要些这两行代码。
```js
Son.prototype = Object.create(Father.prototype);
Son.prototype.constructor = Son;
```

```js
class Father {
    constructor() {
        this.name = '爸爸';
    }
}
class Son extends Father {
    constructor() {
        super();
        this.name = '儿子';
    }
}
var ss = new Son();

// 先看__proto__属性，实例对象的__proto__链
console.log(ss.__proto__ === Son.prototype) // true
console.log(Son.prototype.__proto__ === Father.prototype) // true
console.log(Father.prototype.__proto__ === Object.prototype) // true
console.log(Object.prototype.__proto__ === null) // true
// 类的 __proto__ 链
console.log(Son.__proto__ === Father) // true， ES5创建的类这里会不一样，都是指向Function.prototype
console.log(Father.__proto__ === Function.prototype) //true
console.log(Function.prototype.__proto__ === Object.prototype) // true
console.log(Object.prototype.__proto__ === null) // true

// 再看构造函数.constructor，构造函数一定是指向某一个函数的
console.log(ss.constructor === Son) // true

console.log(Son.constructor === Function) // true
console.log(Father.constructor === Function) // true
console.log(Function.constructor === Function) // true 函数的构造还是函数
// 类的原型的构造函数
console.log(Son.prototype.constructor === Son) // true
console.log(Father.prototype.constructor === Father) // true
console.log(Function.prototype.constructor === Function) // true

// 再看原型.prototype，只有函数才有原型对象
console.log(Son.prototype instanceof Father) // true   Son的原型是Father类的实例
console.log(Father.prototype instanceof Object) // true  Father的原型是Object的实例
```

## ES5组合寄生继承
```js
// 组合寄生继承
var Father = function() {
    this.name = '爸爸';
}

var Son = function() {
    Father.call(this);
    this.name = '儿子';
}
Son.prototype = Object.create(Father.prototype);
Son.prototype.constructor = Son;

var ss = new Son();
```

## 箭头函数可以作为构造函数吗？
不可以。  

普通函数在运行时才会确定 this 的指向。箭头函数则是在函数定义的时候就确定了 this 的指向，此时的 this 指向外层的作用域。所以不可以用来做构造函数，因为如果做构造函数，生成的实例的this不是指向实例，而是指向箭头函数的上下文，这是不对的。

## obj.hasOwnProperty(prop)
hasOwnProperty() 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性，忽略那些从原型链上继承到的属性。

## typeof, instanceof, Object.prototype.toString.call()
* typeof 可以用于判断基本数据类型和函数，判断不出来引用类型null对象数组。输出的是小写的。
```js
typeof 1  // "number"
typeof '1' // "string"
typeof true  //"boolean"
typeof undefined // "undefined"
let s = Symbol()
typeof s // "symbol"

typeof function(){} // "function"
typeof Object // "function"
typeof Array // "function"
typeof Function // "function"

typeof null // "object"
typeof {} // "object"
typeof []  // "object"
typeof new Boolean(false)  // "object"
typeof new Date() // "object"
```
* Object.prototype.toString.call()，可以精确判断所有类型，前面是小写的object，后面是具体的类型
```js
Object.prototype.toString.call(1)   // "[object Number]"
Object.prototype.toString.call('1') // "[object String]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call(new Boolean(false)) // "[object Boolean]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call([])  // "[object Array]"
Object.prototype.toString.call({})  // "[object Object]"
Object.prototype.toString.call(Function) // "[object Function]"
Object.prototype.toString.call(Array) // "[object Function]"
Object.prototype.toString.call(Object) // "[object Function]"
```
* instanceof 用于判断某个对象是否是某个构造函数的实例。即检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。所以主要是用来检测对象和数组，无法准确判断Function 和基本数据类型
```js
console.log([] instanceof Array); // true
console.log({} instanceof Object); // true
console.log(/\d/ instanceof RegExp); // true

console.log(function(){} instanceof Object); // true
console.log(function(){} instanceof Function);// true

console.log('' instanceof String); // false
console.log(1 instanceof Number); // false
console.log(true instanceof Boolean); // false
```

## Object.defineProperty(obj, prop, descriptor)
在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的。
1. obj  要在其上定义属性的对象。
2. prop  要定义或修改的属性的名称。
3. descriptor  将被定义或修改的属性描述符。属性描述符有两种主要形式：数据描述符和存取描述符。描述符必须是这两种形式之一，不能同时是两者。

    存取描述符  
    * get  
    当访问该属性时，该方法会被执行，默认为 undefined。
    * set  
    当属性值修改时，触发执行该方法，该方法将接受唯一参数，即该属性新的参数值。默认为 undefined  

    数据描述符
    * writable  
    当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为 false。
    * value  
    该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。

    数据描述符和存取描述符均具有以下可选键值
    * configurable  
    当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变(也就是可以被重新defineProperty)，同时该属性也能从对应的对象上被删除。默认为 false。
    * enumerable  
    当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。默认为 false。

## call，apply，bind区别
用法：  
* call方法第一个参数是函数运行时this指向的对象，后面跟多个参数。返回值取决于原始函数的返回值。
* apply方法第一个参数也是函数运行时this指向的对象，后面跟参数数组。返回值取决于原始函数的返回值。
* bind方法返回值是一个新的函数，这个新的函数被调用时，this指向bind的第一个参数。bind方法后面也可以接收多个参数传递给原始函数。  

区别：  
* 第一个参数都是 要绑定的this指向.
* apply的第二个参数是一个参数数组,call和bind的第二个及之后的参数作为函数实参按顺序传入。
* bind不会立即调用,其他两个会立即调用。

## 深拷贝，浅拷贝，Object.assign()、ES6的扩展运算符
浅拷贝只是拷贝了指向对象的指针。  
深拷贝则是完全拷贝了整个值，创建了一个新的和原对象值一样的对象。  
* 浅拷贝 直接赋值
* 深拷贝用_.deepclone、JSON.parse(JSON.stringify(obj))  

扩展运算符和Object.assign()都是只复制最外面一层，所以根属性是深拷贝，里面的对象依然是浅拷贝

## 实现深拷贝
```js
let a = {
    a: {b: 'b'},
    c: () => { console.log('c') },
    d: [1,2],
    e: undefined,
    i: null,
    f: 1,
    K: true
}
a.g = a;
a.h = new Date();
```
1. 简易版： JSON.parse(JSON.stringify(obj));
    * 无法解决循环引用问题，会报错。 比如上面的 a.g = a
    * 无法复制值为undefined的
    * 无法复制函数
    * 无法复制一些特殊的对象，如 RegExp, Date, Set, Map等

2. 面试版：
    * 如果是对象，循环每一个key，判断每一项
        * 如果是循环引用，直接复制
        * 否则直接递归该元素
    * 如果是数组，递归每一个元素。
    * 如果是函数，函数复制
    * 如果是值类型，直接复制。

## 节流（throttle），防抖（debounce）
* 节流：频繁操作的时候，如果超过了设定的时间，就执行一次处理函数。周期性的执行。节流会稀释函数的执行频率。
* 防抖：频繁操作的时候，如果两次的间隔时间超过了设定的时间，就执行一次处理函数，如果时间没到的时候，就把timer清除。只执行最后一次。需要一个定时器。
* 函数防抖是某一段时间内只执行一次，而函数节流是间隔时间执行。

## JS 异步解决方案的发展历程以及优缺点
* 回调函数  
    缺点：
    1. 回调地狱
    2. 外层try catch捕获不到回调函数内部错误

* Promise  
    优点：解决了回调地狱的问题  
    缺点：  
    1. 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
    1. 当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
    1. Promise 真正执行回调的时候，定义 Promise 那部分实际上已经走完了，所以 Promise 的报错堆栈上下文不太友好。
    1. 无法取消Promise，一旦新建它就会立即执行，无法中途取消。（实际上没有办法取消promise的执行，其他的取消方法，也只是对结果不再处理）

* Generator  
    缺点： 使用复杂，需要配合co库

* async/await  
    优点：  
    1. 同步的写法
    2. 可以try catch
    3. 没有then的链式调用
    4. 错误堆栈信息友好
    5. 方便调试  

    缺点：  
    用async/await需要注意的是，如果里面有多个Promise，需要注意他们是否有依赖，没有依赖的话用await Promise.all([...])

## map、forEach、for in、for of、Object.keys
* for in 遍历对象的key，因为key排序的问题，for-in语句无法保证遍历顺序。   
    `for (var prop in obj) { ... }`
* for of ES6引入，用来遍历任何有Iterator接口的数据结构，如数组, Set, Map, String, Arguments等   
    `for (var value of arr) { ... }`
* forEach 遍历数组，不影响原数组  
    `arr.forEach((value, index, arr) => { ... })`
* map 遍历数组，并返回一个新的数组，新数组元素为原来的数组的每个元素调用func的结果，不影响原数组  
    `arr.map((value, index, arr) => { return ... })`
* Object.keys也是循环对象的key，返回一个数组

## js 跳出循环，break, continue, return
1. for 循环，for of循环，while循环，for in循环。都是break跳出循环，continue下一次循环
    ```js
    const arr = [0,1,2,3,4,5];
    for(let i=0; i<arr.length;i++){
        if(i===1) continue;
        if(i===4) break;
        console.log(arr[i])
    }

    for(let i of arr){
        if(i===1) continue;
        if(i===4) break;
        console.log(arr[i])
    }

    let i = 0;
    while(i < arr.length){
        console.log(arr[i])
        i++;
        if(i===1) continue;
        if(i===4) break;
    }

    const obj = {a:'a',b:'b',c:'c',d:'d',e:'e'}
    for(let key in obj){
        if(key==='a') continue;
        if(key==='e') break;
        console.log(i)
    }
    ```
2. map, forEach等循环，不能跳出循环，如果想跳出，只能主动抛出一个异常。throw new Error()

3. return 一般用来作为函数的返回值，不能单独用在循环中。写在某函数的循环中，相当于结束了函数，所以也结束了循环。
## 前端存储方式
1. cookie  
    cookie 中主要存储sessionid，过期时间，域名，路径等。会在发送请求的时候自动携带。为了维持服务器的状态。大小只有4k。

2. localStorage  
    localStorage 通过key-value存储一些值，大小为5兆，会永久存储在客户端。

3. sessionStorage  
    sessionStorage 跟localStorage相似，会话级别，关闭页面就会消失。

4. IndexDB  
    前端的非关系型数据库，用于在客户端存储大量的数据.

## 字符串操作
1. `str.charAt()` 返回在指定位置的字符
2. `str.concat(str2)` 连接字符串
3. `str.indexOf(str2)` 返回某个指定的字符串值在字符串中首次出现的位置
4. `str.lastIndexOf(str2)` 返回一个指定的字符串在字符串中最后出现的位置，从后向前搜索
5. `str.match(regexp)` 在字符串内找到一个或多个正则表达式的匹配，返回一个数组
6. `str.replace(regexp/substr, str2)` 在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串
7. `str.search(regexp/substr)` 检索字符串中指定的子字符串，或检索与正则表达式相匹配的子字符串，返回相匹配的子串的起始位置
8. `str.split()`，将字符串变为数组，以某符号分割。
9. `str.slice(start,end)` 返回一个新的字符串。从 start 开始（包括 start）到 end 结束（不包括 end）。
10. `str.substring(start, end)` 返回一个新的字符串, 子串包括 start 处的字符，但不包括 end 处的字符, 与slice() 和 substr() 方法不同的是，substring() 不接受负的参数。
11. `str.substr(start, length)` 抽取从 start 下标开始的指定数目的字符。
12. `str.toLowerCase()`	把字符串转换为小写。
13. `str.toUpperCase()`	把字符串转换为大写。
14. `str.localeCompare(str2)` 如果引用字符存在于比较字符之前则为负数; 如果引用字符存在于比较字符之后则为正数; 相等的时候返回 0

## ES6新增字符串操作
1. `str.includes(str)` 返回布尔值，表示是否找到了参数字符串。
2. `str.startsWith(str)` 返回布尔值，表示参数字符串是否在原字符串的头部。
3. `str.endsWith(str)` 返回布尔值，表示参数字符串是否在原字符串的尾部。
4. `str.repeat(n)` 方法返回一个新字符串，表示将原字符串重复n次。n为0则返回空串
5. `str.padStart(n, str)`，`str.padEnd(n, str)` 字符串补全长度。如果某个字符串不够指定长度，会在头部或尾部补全。
6. `trimStart()`和`trimEnd()` 与trim()一致，trimStart()消除字符串头部的空格，trimEnd()消除尾部的空格。
7. `str.replaceAll(regexp/substr, str2)` 替换所有

## 如何判断是否是数组？
1. `Array.isArray(arr) // es6提供`
2. `Object.prototype.toString.call(arr) === "[object Array]"` 
3. `arr instanceof Array`
4. `arr.constructor === Array`

一般用1，2方法

## 数组操作
1. `arr.push(a, b, c...)` 从后面添加一个或多个元素，返回值为添加完后的数组的长度
2. `arr.pop()`  从后面删除元素，只能是一个，返回值是删除的元素
3. `arr.shift()` 从前面删除元素，只能删除一个 返回值是删除的元素
4. `arr.unshift(a, b, c...)` 从前面添加一个或多个元素, 返回值是添加完后的数组的长度。第一个参数将成为数组的新元素 0，如果还有第二个参数，它将成为新的元素 1
5. `arr.slice(start, end)` 返回索引值start到索引值end的数组，不包含end元素，不改变原数组，如果不传参数，则返回它本身的副本。
6. `arr.splice(i, n, item1,...,itemX)` 从数组中删除从i开始的n个元素，返回被删除的数组，还可以传入其他的值来替换被删除的数组项目
7. `arr.join()` 将数组变为字符串，以传入的参数连接
8. 易混淆的一个string的方法，`str.split()`，将字符串变为数组，以某符号分割
9. `arr.concat(newArr)` 连接两个数组 返回值为连接后的新数组
10. `arr.reverse()` 将数组反转，返回值是反转后的数组
11. `arr.sort()` 将数组进行排序,返回值是排好的数组。默认是升序排列。默认将按字母顺序对数组中的元素进行排序，说得更精确点，是按照字符编码的顺序进行排序。  
    如果传入了比较函数：
    * 如果 compareFunction(a, b) 小于 0 ，那么 a 会被排列到 b 之前
    * 如果 compareFunction(a, b) 等于 0 ， a 和 b 的相对位置不变
    * 如果 compareFunction(a, b) 大于 0 ， b 会被排列到 a 之前  
    * 总结： a-b 是升序，b-a是降序
12. `arr.forEach((value, index, arr) => { ... })` 遍历数组, 无return, 不改变原数组
14. `arr.map((value, index, arr) => { ... })` 遍历数组, 返回一个新数组, 不改变原数组
15. `arr.filter((value, index, arr) => { ... })` 过滤数组，返回一个满足要求的数组
16. `arr.every((value, index, arr) => { ... })` 依据判断条件，数组的元素是否全满足，若满足则返回ture。全真则真，一假即假
17. `arr.some((value, index, arr) => { ... })` 依据判断条件，数组的元素是否有一个满足，若有一个满足则返回ture。有真则真，全假才假
18. `arr.reduce((prev, next, index, arr) = > { ... }, initValue)` 为数组中的每一个元素依次执行callback函数，并将函数的返回值作为total参数传给下次循环

## ES6新增数组操作
1. `Array.from()` 将对象转为数组：string，arguments, Set, Map, key是数字的对象
2. `Array.of()` 用于将一组值，转换为数组。`Array.of(3, 11, 8) // [3,11,8]`
3. `arr.copyWithin(target, start, end)` 将指定位置的成员(从start到end，不包含end的成员)复制到其他位置(从target开始)（会覆盖原有成员），然后返回当前数组。  
    * target（必需）：从该位置开始替换数据。
    * start（可选）：从该位置开始读取数据，默认为 0。
    * end（可选）：到该位置前停止读取数据，默认等于数组长度。

    `[1, 2, 3, 4, 5].copyWithin(0, 3) // [4, 5, 3, 4, 5]`  用4，5把1，2替换
    `[1, 2, 3, 4, 5].copyWithin(0, 3, 4) // [4, 2, 3, 4, 5]` 用4把1替换
4. `arr.find((value, index, arr) => { ... })` 找出第一个符合条件的数组元素，然后返回该元素
5. `arr.findIndex((value, index, arr) => { ... })` 找出第一个符合条件的数组元素，然后返回该元素的位置
6. `arr.includes()` 返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似
7. `arr.entries()`，`arr.keys()`和`arr.values()`用于遍历数组。keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。
8. `arr.flat()` 用于将嵌套的数组“拉平”。默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以给flat()方法的参数传入一个整数，表示想要拉平的层数。  
    `[1, 2, [3, 4]].flat()   //[1, 2, 3, 4]`  
    `[1, 2, [3, [4, 5]]].flat()   //[1, 2, 3, [4, 5]]`  
    `[1, 2, [3, [4, 5]]].flat(2)   //[1, 2, 3, 4, 5]`  
9. `arr.flatMap((value, index, arr) => { ... })` 先执行map，然后对返回值组成的新数组执行flat()方法。

## 数组去重
用Set, `let newArr = [..new Set(arr)]`

如果是对象，用reduce
```js
let temArr = [];
const uniqueList = totalList.reduce((prev, next) => {
    if (!temArr.includes(next.id)) {
        temArr.push(next.id);
        prev.push(next);
    }
    return prev;
}, []);
```

## 数组合并
concat, [...a, ...b]扩展运算符

## 数组循环中删除元素
```js
for (let i=0; i<arr.length; i++) {
    if (arr[i] === target) {
        arr.splice(i, 1);
        i = i - 1; // 一定要给下标重新赋值，减一
    }
}
```

## 函数内部 arguments 变量有哪些特性,有哪些属性,如何将它转换为数组
arguments对象是所有（非箭头）函数中都可用的局部变量。可以使用arguments[i]引用函数的参数，也可以修改函数参数。修改的只是函数的形参，并不会影响实参。  
* `arguments.callee` 指向当前执行的函数。  
* `arguments.length` 指向传递给当前函数的参数数量。
* `arguments.callee.caller` 也就是 `fn.caller` 指向调用当前函数的函数的引用，如果是在全局作用域中调用当前函数，它的值为 null。  

转换为数组：  
1. `var arr = [...arguments];`
2. `Array.prototype.slice.call(arguments);`
3. `Array.from(arguments)`

## ES6 模块和 CommonJS 模块的区别
1. CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
1. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
1. CommonJs 是单个值导出，ES6 Module可以导出多个
1. CommonJs 是动态语法可以写在判断里，ES6 Module 静态语法只能写在顶层
1. CommonJs 的 this 是当前模块，ES6 Module的 this 是 undefined

CommonJS值拷贝发生在给 module.exports 赋值的那一刻，例如：
```js
let val = 1;
module.exports = {
  val
}
```
与CommonJS 不同，ESM 中 import 的不是对象， export 的也不是对象。import 模块中引入的值就指向了 export 中导出的值。是引用。

ESM 之所以被称为 编译时输出接口，是因为它的模块解析是发生在 编译阶段。

也就是说，import 和 export 这些关键字是在编译阶段就做了模块解析，这些关键字的使用如果不符合语法规范，在编译阶段就会抛出语法错误。

与此对应的 CommonJS，它的模块解析发生在 执行阶段，因为 require 和 module 本质上就是个函数或者对象，只有在 执行阶段 运行时，这些函数或者对象才会被实例化。因此被称为 运行时加载。

## module.exports和exports的区别
1. exports是module.exports的引用, 即   
    `var exports=module.exports`
2. 初始时这两个变量是指向同一个空对象
3. 最终导出的是 module.exports，即require方能看到的只有module.exports这个对象，它是看不到exports对象的

所以
1. 当你想导出的东西可以在空对象上直接扩展就可以的时候，用exports当然省时省力。export.a = 'test'
2. 当你想导出的东西要完全覆盖掉空对象的时候，只能用module.exports了。
    ```js
    module.exports = {} // 正确
    exports = {} // 错误
    ```
3. 当你分不清楚的时候请用module.exports

## 前端模块化的发展历程
1. ### IIFE
立即执行函数定义模块
```js
var utilModule = (function() {
  var module = {};
  var name = 'utilModule';
  module.setName = function(value) {
    name = value;
  };
  module.getName = function() {
    return name;
  };
  return module;
})();

utilModule.getName() // "utilModule"
utilModule.setName('test')
utilModule.getName() // "test"
```

2. ### CommonJS
Nodejs的模块规范, 用module.exports定义当前模块对外输出的接口（不推荐直接用exports），用require加载模块。
```js
// module.js
var name = 'testModule';
var setName = function(value) {
    name = value;
}
var getName = function() {
    return name;
}
module.exports = {
    setName,
    getName
}

// main.js
var testModule = require('./module.js')
console.log(testModule.getName()); // testModule
testModule.setName('test');
console.log(testModule.getName()); // test
```

3. ### AMD 
require.js遵循了这个规范。通过define方法去定义模块，通过require方法去加载模块。
```js
// a.js
// 定义独立的模块
define({
    test: function() { console.log('this is test1'); }
})

// b.js
// 另一种定义独立模块的方式
define(function() {
    return {
        test: function() {console.log('this is test2'); }
    }
})

//c.js
// 定义非独立的模块（这个模块依赖其他模块）
define(['./a', './b'], function(a, b) {
    return {
        test: function() {
            a.test();
            b.test();
        }
    }
})

// main.js
require(['./c'], function(c) {
    c.test();
})

// index.html
<script src="require.js"></script>
<script src="main.js"></script>

// 控制台
this is test1
this is test2
```

4. ### CMD
seajs遵循了这个规范。通过define方法去定义模块，通过seajs.use去加载模块。
```js
// a.js
define(function(require, exports, module) {
    var test = function() {
        console.log('this is test1');
    }
    module.exports = { test }
});

// b.js
define(function(require, exports, module) {
    var test = function() {
        console.log('this is test2');
    }
    module.exports = { test }
});

//c.js
define(function(require, exports, module) {
    var a = require('./a');
    var b = require('./b');
    var test = function() {
        a.test();
        b.test();
    }
    module.exports = { test }
});

// main.js
seajs.use('./c', function(c) {
    c.test();
});

// index.html
<script src="sea.js"></script>
<script src="main.js"></script>

// 控制台
this is test1
this is test2
```

5. ###  ES6 module
```js
export
export default
import 
import * as ...
```

6. ### UMD
UMD是同时支持AMD和CommonJS的规范。
```js
(function (root, factory) {
  if (typeof define === "function" && define.amd) {
    // 如果环境中有define函数，并且define函数具备amd属性，则可以判断当前环境满足AMD规范
    define(["test"], factory());
  } else if (
    typeof exports === "object" &&
    typeof module.exports === "object"
  ) {
    // 是commonjs模块规范，nodejs环境
    module.exports = factory();
  } else {
    // 浏览器全局变量(root 即 window)
    root.umdModule = factory();
  }
})(this, function () {
  return {
    name: "我是一个umd模块",
  };
});

```

## AMD 和 CMD 的区别
1. 对于依赖的模块，AMD 是提前加载，CMD 是延迟加载。
2. 从规范上，CMD更贴近CommonJS的异步模块化方案

## 设计模式
* 工厂：工厂起到的作用就是隐藏了创建实例的复杂度，只需要提供一个接口，简单清晰。
* 单例：保证每次访问得到的都是同一个对象，可以用全局对象存储。
* 适配器：为了解决两个接口不兼容的情况，通过包装一层实现两个接口正常协作。
* 装饰模式：不需要改变已有的接口，作用是给对象添加功能。
* 代理模式：代理是为了控制对对象的访问，不让外部直接访问到对象。代理类可以访问并操作对象，然后暴露相关方法供外部调用。
* 发布订阅（观察者）模式：当对象发生改变时，订阅方都会收到通知。先定义一个对象，这个对象包含on,off方法和trigger方法，以及一个存储回调函数的map。on的时候往map里面push，trigger的时候再从map中拿出来并执行, off的时候删除。

## jquery插件开发
其实就是给jquery增加一种新的方法。  
1. 一种是类级别的插件开发，即给jQuery添加新的全局函数，相当于给jQuery类本身添加方法。jQuery的全局函数就是属于jQuery命名空间的函数。
```js
jQuery.foo = function() {    
    alert('This is a test.');   
}; 
// 或 
jQuery.extend({      
    foo: function() {      
        alert('This is a test.');      
    }
});
// 但是一般我们都加上命名空间去使用
jQuery.myPlugin = {           
    foo: function() {           
        alert('This is a test.');           
    }       
}; 
//调用
jQuery.foo();
$.foo();
$.myPlugin.foo();     
```
2. 另一种是对象级别的插件开发，即给jQuery对象添加方法。这就需要把方法添加到jQuery的prototype上。查看jQuery源码可以发现jQuery.fn=jQuery.prototype。
```js
$.fn.test = function() {
    alert('this is test2');
    return this; // 为了满足链式调用
}
// 或
$.fn.extends({
    test: function() {
        alert('this is test2');
        return this;
    }
})
// 但一般我们为了防止$.fn被其他库污染，都用立即执行函数定义
(function($){
    $.fn.test = function(){
        alert('this is test2');
        return this;
    }
}(jQuery));
```

## jquery绑定事件，取消绑定，触发事件
on, off, trigger
