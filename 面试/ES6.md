## let、const

let 表示变量，const 表示常量，都是块级作用域。

- 不存在变量提升
- 不允许重复声明

## 解构赋值，数组解构，对象解构

- 数组解构：直接从数组中取值，按照对应位置，赋值给变量
- 对象解构：对象解构不需要严格按照顺序取值，而只要按照变量名去取对应属性名的值

```js
let [a, b] = [b, a];
let { a, b } = { a: 1, b: 2 };
```

## ES6 新增字符串操作

1. `str.includes(str)` 返回布尔值，表示是否找到了参数字符串。
2. `str.startsWith(str)` 返回布尔值，表示参数字符串是否在原字符串的头部。
3. `str.endsWith(str)` 返回布尔值，表示参数字符串是否在原字符串的尾部。
4. `str.repeat(n)` 方法返回一个新字符串，表示将原字符串重复 n 次。n 为 0 则返回空串
5. `str.padStart(n, str)`，`str.padEnd(n, str)` 字符串补全长度。如果某个字符串不够指定长度，会在头部或尾部补全。
6. `trimStart()`和`trimEnd()` 与 trim()一致，trimStart()消除字符串头部的空格，trimEnd()消除尾部的空格。
7. `str.replaceAll(regexp/substr, str2)` 替换所有

## 模板字符串

反引号包裹，用`${}`包裹变量

## 数值的扩展

1. `Number.isFinite()` 用来检查一个数值是否为有限的
2. `Number.isNaN()` 用来检查一个值是否为 NaN
3. ES6 将全局方法`parseInt()`和`parseFloat()`，移植到 Number 对象上面，行为完全保持不变。
4. `Number.isInteger()` 用来判断一个数值是否为整数。
5. ES6 引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`这两个常量，用来表示 -2^53 到 2^53 之间范围的上下限。
6. `Number.isSafeInteger()` 则是用来判断一个整数是否落在这个范围之内

## 参数默认值，剩余参数

参数默认值一般用在尾参数。  
rest 参数形式为(...变量名)，用于获取函数的多余参数。

## ES6 新增数组操作

1. `Array.from()` 将对象转为数组：string，arguments, Set, Map, key 是数字的对象
2. `Array.of()` 用于将一组值，转换为数组。`Array.of(3, 11, 8) // [3,11,8]`
3. `arr.copyWithin(target, start, end)` 将指定位置的成员(从 start 到 end，不包含 end 的成员)复制到其他位置(从 target 开始)（会覆盖原有成员），然后返回当前数组。

   - target（必需）：从该位置开始替换数据。
   - start（可选）：从该位置开始读取数据，默认为 0。
   - end（可选）：到该位置前停止读取数据，默认等于数组长度。

   `[1, 2, 3, 4, 5].copyWithin(0, 3) // [4, 5, 3, 4, 5]` 用 4，5 把 1，2 替换
   `[1, 2, 3, 4, 5].copyWithin(0, 3, 4) // [4, 2, 3, 4, 5]` 用 4 把 1 替换

4. `arr.find((value, index, arr) => { ... })` 找出第一个符合条件的数组元素，然后返回该元素
5. `arr.findIndex((value, index, arr) => { ... })` 找出第一个符合条件的数组元素，然后返回该元素的位置
6. `arr.includes()` 返回一个布尔值，表示某个数组是否包含给定的值，与字符串的 includes 方法类似
7. `arr.entries()`，`arr.keys()`和`arr.values()`用于遍历数组。keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。
8. `arr.flat()` 用于将嵌套的数组“拉平”。默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以给 flat()方法的参数传入一个整数，表示想要拉平的层数。  
   `[1, 2, [3, 4]].flat()   //[1, 2, 3, 4]`  
   `[1, 2, [3, [4, 5]]].flat()   //[1, 2, 3, [4, 5]]`  
   `[1, 2, [3, [4, 5]]].flat(2)   //[1, 2, 3, 4, 5]`
9. `arr.flatMap((value, index, arr) => { ... })` 先执行 map，然后对返回值组成的新数组执行 flat()方法。

## 表达式作为属性名或方法名

```js
obj['a' + 'bc'] = 123;

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123,
};
```

## Symbol 类型

ES6 引入了一种新的原始数据类型 Symbol，表示独一无二的值。

Symbol 值通过 Symbol 函数生成。

```js
let s = Symbol();
typeof s; // "symbol"
console.log(s); // Symbol()
```

Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。

```js
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1; // Symbol(foo)
s2; // Symbol(bar)
```

ES2019 提供了一个实例属性 description，直接返回 Symbol 的描述。

```js
const sym = Symbol('foo');
sym.description; // "foo"
```

Symbol 用于对象的属性名，就能保证不会出现同名的属性。  
不能使用点运算符，这样会把他当成字符串。Symbol 值必须放在方括号之中。

```js
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!',
};
```

Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名。

`Object.getOwnPropertySymbols`方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。

`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。

我们希望重新使用同一个 Symbol 值，Symbol.for 方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。

## Set, Map, WeakSet, WeakMap

### Set

Set 类似于数组，但是成员的值都是唯一的，没有重复的值。  
Set 函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。

- `Set.prototype.size`：返回 Set 实例的成员总数。
- `Set.prototype.add(value)`：添加某个值，返回 Set 结构本身。
- `Set.prototype.delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功。
- `Set.prototype.has(value)`：返回一个布尔值，表示该值是否为 Set 的成员。
- `Set.prototype.clear()`：清除所有成员，没有返回值。
- `Set.prototype.keys()`：返回键名的遍历器
- `Set.prototype.values()`：返回键值的遍历器
- `Set.prototype.entries()`：返回键值对的遍历器
- `Set.prototype.forEach()`：使用回调函数遍历每个成员

### WeakSet

与 Set 类似，有两点不同。

1. 但 WeakSet 的成员只能是对象，而不能是其他类型的值。
2. WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。因此 ES6 规定 WeakSet 不可遍历。

### Map

Map 类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。

- `Map.prototype.size`：返回 Map 实例的成员总数。
- `Map.prototype.set(key, value)`：添加某个值，返回 Map 结构本身。如果 key 已经有值，则键值会被更新，否则就新生成该键。
- `Map.prototype.get(key)`：读取 key 对应的键值，如果找不到 key，返回 undefined。
- `Map.prototype.delete(key)`：删除某个值，返回一个布尔值，表示删除是否成功。
- `Map.prototype.has(key)`：返回一个布尔值，表示某个键是否在当前 Map 对象之中。
- `Map.prototype.clear()`：清除所有成员，没有返回值。
- `Map.prototype.keys()`：返回键名的遍历器
- `Map.prototype.values()`：返回键值的遍历器
- `Map.prototype.entries()`：返回键值对的遍历器
- `Map.prototype.forEach()`：使用回调函数遍历每个成员

### WeakMap

与上面的 WeakSet 类似

### WeakSet/WeakMap 的 key 被垃圾处理器回收了，访问 get 会是什么表现

```js
var obj = { a: 'a' };
var ws = new WeakSet();
var map = new WeakMap();
ws.add(obj);
map.set(obj, 'test');
obj = null; // 模拟垃圾回收，或者手动在chrom > devtools > performance > collect garbage

console.log(ws); // WeakSet {}
ws.has(obj); // false

console.log(map); // WeakMap {}
map.get(obj); // undefined
map.has(obj); // false
```

## 用 Set 获取两个数组的并集，交集，差集，补集

```js
// 并集
let union = Array.from(new Set([...a, ...b]));
// 交集
let intersect = Array.from(new Set(a)).filter((x) => b.includes(x));
// 差集
let difference = Array.from(new Set(a)).filter((x) => !b.includes(x));
// 补集
let complement = [...a.filter((x) => !b.includes(x)), ...b.filter((x) => !a.includes(x))];
```

## Proxy

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

Proxy 属于一种“元编程“, 实际上重载了点运算符。

生成 Proxy 实例: `var proxy = new Proxy(target, handler);`

```js
var target = {};
var proxy = new Proxy(target, {
  set(target, key, value, receiver) {
    console.log(target);
    console.log(key);
    console.log(value);
    console.log(receiver);
    return Reflect.set(target, key, value, receiver);
  },
  get(target, key, receiver) {
    console.log(target);
    console.log(key);
    console.log(receiver);
    return Reflect.get(target, key, receiver);
  },
});

proxy.test = 1;
// {}
// test
// 1
// Proxy {}
proxy.test;
// {test: 1}
// test
// Proxy {test: 1}
// 1
target;
// {test: 1}
target.test2 = 2;
target;
// {test: 1, test2: 2}
proxy;
// Proxy {test: 1, test2: 2}
```

在 Proxy 代理的情况下，目标对象内部的 this 关键字会指向 Proxy 代理。谁调用指向谁。

```js
const target = {
  m: function () {
    console.log(this === proxy);
  },
};
const handler = {
  set(target, key, value, receiver) {
    console.log(target);
    console.log(key);
    console.log(value);
    console.log(receiver);
    return Reflect.set(target, key, value, receiver);
  },
  get(target, key, receiver) {
    console.log(target);
    console.log(key);
    console.log(receiver);
    return Reflect.get(target, key, receiver);
  },
};

const proxy = new Proxy(target, handler);

target.m(); // false
proxy.m(); // true
```

一共支持 13 种拦截操作

1. get(target, propKey, receiver)：拦截对象属性的读取，比如 proxy.foo 和 proxy['foo']。

2. set(target, propKey, value, receiver)：拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v，返回一个布尔值。

3. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。

4. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)。

5. has(target, propKey)：拦截 propKey in proxy 的操作，返回一个布尔值。

6. deleteProperty(target, propKey)：拦截 delete proxy[propKey]的操作，返回一个布尔值。

7. defineProperty(target, propKey, propDesc)：拦截 Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。

8. getOwnPropertyDescriptor(target, propKey)：拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。

9. getPrototypeOf(target)：拦截 Object.getPrototypeOf(proxy)，返回一个对象。

10. setPrototypeOf(target, proto)：拦截 Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

11. preventExtensions(target)：拦截 Object.preventExtensions(proxy)，返回一个布尔值。

12. isExtensible(target)：拦截 Object.isExtensible(proxy)，返回一个布尔值。

13. ownKeys(target)：拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。

## Reflect

在 ES6 中官方新定义了 Reflect 对象，在 ES6 之前对象上的所有的方法都是直接挂载在对象这个构造函数的原型身上，而未来对象可能还会有很多方法，如果全部挂载在原型上会显得比较臃肿，而 Reflect 对象就是为了分担 Object 的压力，如下。

1. 将 Object 对象的一些明显属于语言内部的方法（比如 Object.defineProperty），放到 Reflect 对象上。
2. 修改某些 Object 方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而 Reflect.defineProperty(obj, name, desc)则会返回 false。
3. 让 Object 操作都变成函数行为。某些 Object 操作是命令式，比如 name in obj 和 delete obj[name]，而 Reflect.has(obj, name)和 Reflect.deleteProperty(obj, name)让它们变成了函数行为。
4. Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法。这就让 Proxy 对象可以方便地调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础。也就是说，不管 Proxy 怎么修改默认行为，你总可以在 Reflect 上获取默认行为。

Reflect 对象一般搭配 Proxy 使用。

### Proxy 中为啥要搭配 Reflect 使用

在复杂的使用场景保持正确的上下文和 this。

```js
var target = {};
var proxy = new Proxy(target, {
  set(target, key, value, receiver) {
    return Reflect.set(target, key, value, receiver);
  },
  get(target, key, receiver) {
    return Reflect.get(target, key, receiver);
  },
});
```

有 13 个静态方法

1. Reflect.apply(target, thisArg, args)
1. Reflect.construct(target, args)
1. Reflect.get(target, name, receiver)
1. Reflect.set(target, name, value, receiver)
1. Reflect.defineProperty(target, name, desc)
1. Reflect.deleteProperty(target, name)
1. Reflect.has(target, name)
1. Reflect.ownKeys(target)
1. Reflect.isExtensible(target)
1. Reflect.preventExtensions(target)
1. Reflect.getOwnPropertyDescriptor(target, name)
1. Reflect.getPrototypeOf(target)
1. Reflect.setPrototypeOf(target, prototype)

上面这些方法的作用，大部分与 Object 对象的同名方法的作用都是相同的，而且它与 Proxy 对象的方法是一一对应的。

Reflect.apply，可以改写以前的 call, apply 写法

```js
// old
var a = {
  name: 'test',
  say() {
    console.log(this.name);
  },
};
var b = {
  name: 'b',
};
a.say.call(b); //b
a.say.apply(b); //b
Reflect.apply(a.say, b, []); //b
```

## Promise 有哪些方法

- `Promise.all([p1, p2, p3])` 都 resolve 则 resolve, 有一个 reject 即 rejefct
- `Promise.race([p1, p2, p3])` 有任何一个先 resolve，或者 reject，则整体即为 resolve 或 reject
- `Promise.allSettled([p1, p2, p3])` ES2020 引入。只有等到所有这些参数实例都返回结果，不管是 resolve 还是 rejected，整体才会结束。一旦结束，状态总是 resolve，值是各个 promise 的状态和 value;
- `Promise.any([p1, p2, p3])` 目前还是 stage3 提案。只要参数实例有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。跟 Promise.race()的不同就是不会因为某个 Promise 变成 rejected 状态而结束。
- `Promise.resolve()` 将现有对象转为 Promise 对象,并且是 resolve 状态。不带参数，直接返回一个 resolved 状态的 Promise 对象。
- `Promise.reject()` 返回一个新的 Promise 实例，该实例的状态为 rejected。
- `Promise.prototype.then()` 为 Promise 实例添加状态改变时的回调函数。前面说过，then 方法的第一个参数是 resolved 状态的回调函数，第二个参数（可选）是 rejected 状态的回调函数。
- `Promise.prototype.catch()` 如果异步操作抛出错误，状态就会变为 rejected，就会调用 catch 方法指定的回调函数，处理这个错误。另外，then 方法指定的成功或者失败的回调函数，如果运行中抛出错误，也会被 catch 方法捕获。
- `Promise.prototype.finally()` 用于指定不管 Promise 对象最后状态如何，都会执行的操作。

## then 参数

then 方法接受的参数是函数，而如果传递的并非是一个函数，它实际上会将其解释为 then(null)，相当于忽略了，直接执行后面的，这就会导致前一个 Promise 的结果会穿透下面。

```js
Promise.resolve(1).then(2).then(Promise.resolve(3)).then(console.log); // 1
```

## Promise 的执行顺序

1. 同步代码最先执行
2. Promise 新建后立即执行, Promise.resolve()也是立即执行
3. then 方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行
4. setTimeout 等异步方法会最后执行。
5. 如果有 async/await，await 关键字后面的方法也是立即执行，执行完之后会继续执行全局同步代码，等同步代码执行完毕, 才会将 await 语句后面的代码放入微任务队列。不是立即放入。

## 如何限制 Promise 请求并发数

通过 Promise.all()实现

1. 初始化 limit 个 Promise 对象，作为 Promise.all 的参数
2. Promise.all，执行这 limit 个数的 Promise，等待 resolve
3. 重复步骤 1，2

## Iterator 遍历器和 for...of

Iterator 它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作。用 for...of 遍历。

当使用 for...of 循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。

原生具备 Iterator 接口的数据结构如下：

- Array
- Map
- Set
- String
- TypedArray
- 函数的 arguments 对象
- NodeList 对象

## Generator

Generator 函数是一个状态机，封装了多个内部状态。  
形式上，Generator 函数是一个普通函数，但是有两个特征。一是，function 关键字与函数名之间有一个星号；二是，函数体内部使用 yield 表达式，定义不同的内部状态。  
调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是上一章介绍的遍历器对象。  
调用遍历器对象的 next 方法，使得指针移向下一个状态。也就是说，每次调用 next 方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个 yield 表达式（或 return 语句）为止。

```js
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}
var test = helloWorldGenerator();

test.next(); // { value: 'hello', done: false }
test.next(); // { value: 'world', done: false }
test.next(); // { value: 'ending', done: true }
test.next(); // { value: undefined, done: true }
```

next 方法可以带一个参数，该参数就会被当作上一个 yield 表达式的返回值。

for...of 循环可以自动遍历 Generator 函数运行时生成的 Iterator 对象，且此时不再需要调用 next 方法。

```js
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

next 方法的返回对象的 done 属性为 true，for...of 循环就会中止，且不包含该返回对象。所以上面代码的 return 语句返回的 6，不包括在 for...of 循环之中。

Generator 函数的执行必须靠执行器，所以才有了 co 模块。co 模块可以让你不用编写 Generator 函数的执行器。Generator 函数只要传入 co 函数，就会自动执行。co 函数返回一个 Promise 对象。

```js
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

var co = require('co');
co(gen).then(function () {
  console.log('Generator 函数执行完成');
});
```

## async await

async 就是 Generator 函数的语法糖。

1. async 函数自带执行器。async 函数的执行，与普通函数一模一样。
2. async 和 await，比起星号和 yield，语义更清楚。
3. co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，而 async 函数的 await 命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。
4. async 函数的返回值是 Promise 对象，比 Generator 函数的返回值是 Iterator 对象方便多了

async/await 的注意事项：

1. async 函数返回一个 Promise 对象。
2. async 函数内部 return 语句返回的值，会成为 then 方法回调函数的参数。
3. async 函数返回的 Promise 对象，必须等到内部所有 await 命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到 return 语句或者抛出错误。
4. await 命令后面的 Promise 对象如果变为 reject 状态，则 reject 的参数会被 catch 方法的回调函数接收到
5. 把 await 命令放在 try...catch 代码块中
6. 多个 await 命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
7. await 命令只能用在 async 函数之中，如果用在普通函数，就会报错。
8. async 函数可以保留运行堆栈。

## await 能不能被 return，return await promise 和 return promise 的区别

<https://stackoverflow.com/questions/38708550/difference-between-return-await-promise-and-return-promise>

1. 大体上没啥区别，效果一样
2. 一般 await 用 try catch 包裹，如果两个都用 try catch 包裹，如果 promise reject，return await promise 会走到 catch， return promise 不会。

## Class 与继承

### class 的用法

1. class 声明类
2. constructor 为构造方法，可传参。constructor 方法是类的默认方法，如果没有显式定义，一个空的 constructor 方法会被默认添加。
3. 类可以定义实例属性和方法
4. 类内部可以使用 get 和 set 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。
5. 类的属性名，可以采用表达式
6. 类的方法内部如果含有 this，它默认指向类的实例
7. 用 static 表示静态方法。静态方法可以被子类继承。如果静态方法包含 this 关键字，这个 this 指的是类，而不是实例。
8. 实例属性除了定义在 constructor()方法里面的 this 上面，也可以定义在类的最顶层。
9. 目前只能通过在类名上面加属性。现在有一个提案在实例属性的前面，加上 static 关键字来表示静态属性。
10. 私有属性和私有方法，名字前面加下划线。或者用 Symbol 值命名私有方法名。因为 Symbol 作为属性名，不会被遍历出来。但可以用 getOwnPropertySymbols 遍历出来。
11. ES6 为 new 命令引入了一个 new.target 属性，该属性一般用在构造函数之中，返回 new 命令作用于的那个构造函数。

### 继承

通过 extends 关键字实现继承。

子类必须在 constructor 方法中调用 super 方法，只有调用 super 之后，才可以使用 this 关键字，否则会报错。super()代表父类的构造函数。相当于 A.prototype.constructor.call(this)。

```js
class Father {
  constructor() {
    this.name = 'father';
  }
}
class Son extends Father {
  constructor() {
    super();
    this.name = 'son';
  }
}
```

### ES5 的组合继承

子类必须调用父类的 call，同时子类原型 prototype 是父类实例，子类原型 prototype 的构造函数是本身。

```js
var Father = function () {
  this.name = '爸爸';
};
var Son = function () {
  Father.call(this);
  this.name = '儿子';
};
Son.prototype = new Father();
Son.prototype.constructor = Son;
```

优点：

- 父类的方法可以被复用
- 父类的引用属性不会被共享
- 子类构建实例时可以向父类传递参数

缺点：

调用了两次父类的构造函数，一次是设置子类实例的原型的时候，一次在创建子类型实例的时候。这种情况造成了性能上的浪费。

### ES5 的寄生组合继承

跟上面的区别是用 Object.create(Father.prototype)来创建一个新的原型赋值给子类原型

```js
Son.prototype = Object.create(Father.prototype);
```

### class 中的原型链

同时存在两条继承链：

1. 类的**proto**属性，表示构造函数的继承，总是指向父类。
2. 实例的**proto**属性，表示方法的继承，总是指向类的 prototype 属性。

```js
class A {}
class B extends A {}
let a = new A();
let b = new B();

B.__proto__ === A; // true
A.__proto__ === Function.prototype; // true

b.__proto__ === B.prototype; // true
a.__proto__ === A.prototype; // true
B.prototype.__proto__ === A.prototype; // true
A.prototype.__proto__ === Object.prototype; // true
```

## class 中的箭头函数和普通函数有什么区别，如下

```js
class A {
  a = 1;
  fn() {
    console.log(this.a);
  }
  f = () => {
    console.log(this.a);
  };
}
let a = new A();
```

1. 两种写法都是原型方法，实例可以调用。
2. 普通函数写法，`a.fn()`这样调用没问题，如果是改下引用， `let b = a.fn; b();`这样调用会报错。
3. 箭头函数会绑定 this，以后不管什么方式调用 this 都不会丢失。
