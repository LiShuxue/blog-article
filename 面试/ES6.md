## let、const
let 表示变量，const 表示常量，都是块级作用域。  
* 不存在变量提升
* 不允许重复声明

## 解构赋值，数组解构，对象解构
* 数组解构：直接从数组中取值，按照对应位置，赋值给变量  
* 对象解构：对象解构不需要严格按照顺序取值，而只要按照变量名去取对应属性名的值  
```js
let [a, b] = [b, a];
let {a, b} = {a:1, b:2}; 
```

## ES6新增字符串操作
1. `str.includes(str)` 返回布尔值，表示是否找到了参数字符串。
2. `str.startsWith(str)` 返回布尔值，表示参数字符串是否在原字符串的头部。
3. `str.endsWith(str)` 返回布尔值，表示参数字符串是否在原字符串的尾部。
4. `str.repeat(n)` 方法返回一个新字符串，表示将原字符串重复n次。n为0则返回空串
5. `str.padStart(n, str)`，`str.padEnd(n, str)` 字符串补全长度。如果某个字符串不够指定长度，会在头部或尾部补全。
6. `trimStart()`和`trimEnd()` 与trim()一致，trimStart()消除字符串头部的空格，trimEnd()消除尾部的空格。
7. `str.replaceAll(regexp/substr, str2)` 替换所有

## 模板字符串
反引号包裹，用`${}`包裹变量

## 数值的扩展
1. `Number.isFinite()` 用来检查一个数值是否为有限的
2. `Number.isNaN()` 用来检查一个值是否为NaN
3. ES6 将全局方法`parseInt()`和`parseFloat()`，移植到Number对象上面，行为完全保持不变。
4. `Number.isInteger()` 用来判断一个数值是否为整数。
5. ES6 引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`这两个常量，用来表示 -2^53 到 2^53 之间范围的上下限。
6. `Number.isSafeInteger()` 则是用来判断一个整数是否落在这个范围之内

## 参数默认值，剩余参数
参数默认值一般用在尾参数。  
rest参数形式为(...变量名)，用于获取函数的多余参数。

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

## 表达式作为属性名或方法名
```js
obj['a' + 'bc'] = 123;

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};
```

## Symbol类型
ES6 引入了一种新的原始数据类型Symbol，表示独一无二的值。

Symbol 值通过Symbol函数生成。  
```js
let s = Symbol();
typeof s // "symbol"
console.log(s) // Symbol()
```  

Symbol函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。
```js
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)
```
ES2019 提供了一个实例属性description，直接返回 Symbol 的描述。
```js
const sym = Symbol('foo');
sym.description // "foo"
```
Symbol用于对象的属性名，就能保证不会出现同名的属性。  
不能使用点运算符，这样会把他当成字符串。Symbol 值必须放在方括号之中。
```js
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};
```

Symbol 作为属性名，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。

`Object.getOwnPropertySymbols`方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。

`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。

我们希望重新使用同一个 Symbol 值，Symbol.for方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。

## Set, Map, WeakSet, WeakMap
### Set
Set类似于数组，但是成员的值都是唯一的，没有重复的值。   
Set函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。  
* `Set.prototype.size`：返回Set实例的成员总数。
* `Set.prototype.add(value)`：添加某个值，返回 Set 结构本身。
* `Set.prototype.delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功。
* `Set.prototype.has(value)`：返回一个布尔值，表示该值是否为Set的成员。
* `Set.prototype.clear()`：清除所有成员，没有返回值。
* `Set.prototype.keys()`：返回键名的遍历器
* `Set.prototype.values()`：返回键值的遍历器
* `Set.prototype.entries()`：返回键值对的遍历器
* `Set.prototype.forEach()`：使用回调函数遍历每个成员

### WeakSet
与 Set 类似，有两点不同。
1. 但WeakSet 的成员只能是对象，而不能是其他类型的值。
2. WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。因此 ES6 规定 WeakSet 不可遍历。

### Map
Map 类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。
* `Map.prototype.size`：返回Map实例的成员总数。
* `Map.prototype.set(key, value)`：添加某个值，返回 Map 结构本身。如果key已经有值，则键值会被更新，否则就新生成该键。
* `Map.prototype.get(key)`：读取key对应的键值，如果找不到key，返回undefined。
* `Map.prototype.delete(key)`：删除某个值，返回一个布尔值，表示删除是否成功。
* `Map.prototype.has(key)`：返回一个布尔值，表示某个键是否在当前 Map 对象之中。
* `Map.prototype.clear()`：清除所有成员，没有返回值。
* `Map.prototype.keys()`：返回键名的遍历器
* `Map.prototype.values()`：返回键值的遍历器
* `Map.prototype.entries()`：返回键值对的遍历器
* `Map.prototype.forEach()`：使用回调函数遍历每个成员

### WeakMap
与上面的WeakSet类似

### WeakSet/WeakMap的key被垃圾处理器回收了，访问get会是什么表现
```js
var obj = {a: 'a'};
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
let intersect = Array.from(new Set(a)).filter(x => b.includes(x));
// 差集
let difference = Array.from(new Set(a)).filter(x => !b.includes(x));
// 补集
let complement = [...a.filter(x => !b.includes(x)), ...b.filter(x => !a.includes(x))];
```

## Proxy
Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

Proxy属于一种“元编程“, 实际上重载了点运算符。

生成 Proxy 实例: `var proxy = new Proxy(target, handler);`
```js
var target = {}
var proxy = new Proxy(target, {
    set(target, property, value, receiver) {
        console.log(target);
        console.log(property);
        console.log(value);
        console.log(receiver);
        Reflect.set(target, property, value, receiver);
    },
    get(target, property, receiver) {
        console.log(target);
        console.log(property);
        console.log(receiver);
        return Reflect.get(target, property, receiver);
    }
})

proxy.test = 1;
// {}
// test
// 1
// Proxy {}
proxy.test
// {test: 1}
// test
// Proxy {test: 1}
// 1
target 
// {test: 1}
target.test2 = 2
target 
// {test: 1, test2: 2}
proxy
// Proxy {test: 1, test2: 2}
```

在 Proxy 代理的情况下，目标对象内部的this关键字会指向 Proxy 代理。谁调用指向谁。
```js
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```

一共支持13种拦截操作
1. get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
2. set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
3. has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
4. deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。
5. ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
6. getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
7. defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
8. preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
9. getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
10. isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
11. setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
12. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
13. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。

## Reflect
Reflect对象一般搭配Proxy使用，Reflect对象的设计目的有这样几个  
1. Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。
2. 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty），放到Reflect对象上。
3. 修改某些Object方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。
4. 让Object操作都变成函数行为。某些Object操作是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。

### Proxy中为啥要搭配Reflect使用。
在复杂的使用场景保持正确的上下文和this。

```js
var target = {}
var proxy = new Proxy(target, {
    set(target, property, value, receiver) {
        Reflect.set(target, property, value, receiver);
    },
    get(target, property, receiver) {
        return Reflect.get(target, property, receiver);
    }
})
```
有13个静态方法  
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

上面这些方法的作用，大部分与Object对象的同名方法的作用都是相同的，而且它与Proxy对象的方法是一一对应的。

Reflect.apply，可以改写以前的call, apply写法
```js
// old
var a = {
	name: 'test',
	say() {
		console.log(this.name)
	}
}
var b = {
	name: 'b'
}
a.say.call(b) //b
a.say.apply(b) //b
Reflect.apply(a.say, b, []) //b
```

## Promise有哪些方法
* `Promise.all([p1, p2, p3])` 都resolve则resolve, 有一个reject即rejefct
* `Promise.race([p1, p2, p3])` 有任何一个先resolve，或者reject，则整体即为resolve或reject
* `Promise.allSettled([p1, p2, p3])` ES2020引入。只有等到所有这些参数实例都返回结果，不管是resolve还是rejected，整体才会结束。一旦结束，状态总是resolve，值是各个promise的状态和value;
* `Promise.any([p1, p2, p3])` 目前还是stage3提案。只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。跟Promise.race()的不同就是不会因为某个 Promise 变成rejected状态而结束。
* `Promise.resolve()` 将现有对象转为 Promise 对象,并且是resolve状态。不带参数，直接返回一个resolved状态的 Promise 对象。
* `Promise.reject()` 返回一个新的 Promise 实例，该实例的状态为rejected。
* `Promise.prototype.then()` 为 Promise 实例添加状态改变时的回调函数。前面说过，then方法的第一个参数是resolved状态的回调函数，第二个参数（可选）是rejected状态的回调函数。
* `Promise.prototype.catch()` 如果异步操作抛出错误，状态就会变为rejected，就会调用catch方法指定的回调函数，处理这个错误。另外，then方法指定的成功或者失败的回调函数，如果运行中抛出错误，也会被catch方法捕获。
* `Promise.prototype.finally()` 用于指定不管 Promise 对象最后状态如何，都会执行的操作。

## Promise的执行顺序
1. 同步代码最先执行  
2. Promise 新建后立即执行, Promise.resolve()也是立即执行
3. then方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行
4. setTimeout等异步方法会最后执行。
5. 如果有async/await，await关键字后面的方法也是立即执行，执行完之后会继续执行全局同步代码，等同步代码执行完毕, 才会将await语句后面的代码放入微任务队列。不是立即放入。  
```js
console.log(1);
setTimeout(() => {
    console.log(2);
}, 0)
new Promise((resolve, reject) => {
    console.log(3);
    resolve();
}).then(() => {
    console.log(4);
})

Promise.resolve().then(() => {
    console.log(5);
})
async function async1() {
	await async2()
	console.log(6)
}
async function async2() {
    await async3()
	console.log(7)
}
async function async3() {
    console.log(8)
}
async1()
console.log(9);

// 1, 3, 8, 9, 4, 5, 7, 6, 2
```

## 手写Promise
1. 首先Promise是一个类，可以new出来实例
2. new的时候，接收一个参数 `new Promise((resolve, reject)=>{})`，我们叫这个参数为executor，
3. Promise会立即执行，也就是executor会立即执行。
4. executor是一个函数，可以接收resolve和reject两个参数。
5. resolve, reject两个参数可以单独执行，所以他是两个方法
6. Promise有三种状态pending、fulfilled、rejected，pending可以变为其他状态，但其他状态不可再变
7. Promise还会有返回值，表示成功之后的值或者失败的原因。
8. resolve(value)会将Promise的状态变为fulfilled，并且将promise的值设为value
9. reject(reason)会将Promise的状态变为rejected, 并将promise的值设为reason
10. 如果excutor出错，直接变为rejected状态。
11. Promise实例有then方法，可以接收fulfilled或者rejected的回调

### 总结就是，一个 class 类， 有个constructor，参数是excutor，构造函数中执行这个excutor，excutor接受参数resolve， reject，用try catch包裹执行。同时有3个实例属性，status, successCallbacks, errorCallbacks。两个实例方法， then, catch。then里面判断状态，执行或者放入数组，catch中调用then

```js
class MyPromise{
    constructor(excutor) {
        this.status = 'pending';
        this.result = undefined;
        // 存放成功的回调
        this.successCallbacks = [];
        // 存放失败的回调
        this.errorCallbacks= [];

        let resolve = (result) => {
            if (this.status !== 'pending') return;
            this.status = 'fullfilled';
            this.result = result;
            // 依次将对应的函数执行
            this.successCallbacks.forEach(fn => fn.call(this, result));
        }

        let reject = (error) => {
            if (this.status !== 'pending') return;
            this.status = 'rejected';
            this.result = error;
            this.errorCallbacks.forEach(fn => fn.call(this, error));
        }

        try {
            excutor(resolve, reject)
        } catch(e){
            reject(e)
        }
    }

    then(success, error) {
        // 解决值穿透问题
        success = typeof success !== "function" ? (v) => v : success;
        error = typeof error !== "function" ? (err) => { throw err } : error;
        // 为了保持链式调用，继续返回promise
        return new MyPromise((resolve, reject) => {
            // 如果promise的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行
            if (this.status === 'pending' && success) {
                this.successCallbacks.push((val) => {
                    try{
                        const res = success(val);
                        res instanceof MyPromise ? res.then(resolve, reject) : resolve(res);
                    } catch(e){
                        reject(e);
                    }
                });
            }
            if (this.status === 'pending' && error) {
                this.errorCallbacks.push((val) => {
                    try{
                        const res = error(val);
                        res instanceof MyPromise ? res.then(resolve, reject) : resolve(res);
                    } catch(e){
                        reject(e);
                    }
                });
            }
            if (this.status === 'fullfilled' && success) {
                try{
                    const res = success(this.result);
                    res instanceof MyPromise ? res.then(resolve, reject) : resolve(res);
                } catch(e){
                    reject(e);
                }
            }
            if (this.status === 'rejected' && error) {
                try{
                    const res = error(this.result);
                    res instanceof MyPromise ? res.then(resolve, reject) : resolve(res);
                } catch(e){
                    reject(e);
                }
            }
        })
    }
    catch(error) {
        return this.then(undefined, error);
    } 
}
```

## 手写实现 Promise.all
1. Promise.all的返回值也是一个Promise
2. 返回一个对应的数组，数组下标的值对应其Promise的结果。
3. 如果有Promise 被reject, Promise.all失败。都成功才成功。
4. 参数数组里面可以是promise，也可是普通数据。
```js
function promiseAll(promiseList) {
    let result = [];
    let count = 0;
    return new Promise((resolve, reject) => {
        for(let i=0; i<promiseList.length; i++) {
            // 不能直接promiseList[i].then()， 因为promiseList[i]可能是普通数据
            Promise.resolve(promiseList[i]).then(res => {
                count++;
                result[i] = res;
                if (count === promiseList.length) {
                    resolve(result);
                }
            }).catch(e => {
                reject(e)
            })
        }
    })
}
```

## 如何限制 Promise 请求并发数
通过Promise.all()实现
1. 初始化limit个Promise对象，作为Promise.all的参数
2. Promise.all，执行这limit个数的Promise，等待resolve
3. 重复步骤1，2

## 实现一个带并发限制的异步调度器 `Scheduler`，保证同时运行的任务最多有两个。
```js
class Scheduler {
    add(promiseCreator) { ... }
    // ...
}

const timeout = (time) => new Promise(resolve => {
    setTimeout(resolve, time)
})

const scheduler = new Scheduler()
const addTask = (time, order) => {
    scheduler.add(() => timeout(time)).then(() => console.log(order))
}

addTask(1000, '1')
addTask(500, '2')
addTask(300, '3')
addTask(400, '4')

// 打印顺序是：2 3 1 4
// 一开始，1、2两个任务进入队列
// 500ms时，2完成，输出2，任务3进队
// 800ms时，3完成，输出3，任务4进队
// 1000ms时，1完成，输出1
// 1200ms时，4完成，输出4
```
```js
/*
    1. add方法要返回一个promise，如果执行的数量<2，要执行promise，否则放入list
    2. add方法的resolve，要在promise执行之后，所以要将resolve传下去
    3. 执行方法中执行promise，控制count，并检查list中是否还有未执行的promise
*/
class Scheduler {
    constructor() {
        this.max = 2;
        this.count = 0;
        this.list = [];
    }
    add(promise) { 
        return new Promise((resolve) => {
            promise.end = resolve;
            if (this.count < this.max) {
                this.excute(promise)
            } else {
                this.list.push(promise)
            }
        })
    }
    async excute(promise) {
        this.count++;
        await promise();
        promise.end()
        this.count--;
        if(this.list.length) {
            this.excute(this.list.shift())
        }
    }
}
```

## Iterator遍历器和for...of
Iterator它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作。用for...of遍历。

当使用for...of循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。

原生具备 Iterator 接口的数据结构如下：
* Array
* Map
* Set
* String
* TypedArray
* 函数的 arguments 对象
* NodeList 对象

## Generator
Generator 函数是一个状态机，封装了多个内部状态。  
形式上，Generator 函数是一个普通函数，但是有两个特征。一是，function关键字与函数名之间有一个星号；二是，函数体内部使用yield表达式，定义不同的内部状态。  
调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是上一章介绍的遍历器对象。  
调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。
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
next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。

for...of循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用next方法。  
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
next方法的返回对象的done属性为true，for...of循环就会中止，且不包含该返回对象。所以上面代码的return语句返回的6，不包括在for...of循环之中。

Generator 函数的执行必须靠执行器，所以才有了co模块。co 模块可以让你不用编写 Generator 函数的执行器。Generator 函数只要传入co函数，就会自动执行。co函数返回一个Promise对象。
```js
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

var co = require('co');
co(gen).then(function (){
  console.log('Generator 函数执行完成');
});
```

## async await
async就是 Generator 函数的语法糖。
1. async函数自带执行器。async函数的执行，与普通函数一模一样。
2. async和await，比起星号和yield，语义更清楚。
3. co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。
4. async函数的返回值是 Promise 对象，比 Generator 函数的返回值是 Iterator 对象方便多了

async/await的注意事项：
1. async函数返回一个 Promise 对象。
2. async函数内部return语句返回的值，会成为then方法回调函数的参数。
3. async函数返回的 Promise 对象，必须等到内部所有await命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到return语句或者抛出错误。
4. await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到
5. 把await命令放在try...catch代码块中
6. 多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
7. await命令只能用在async函数之中，如果用在普通函数，就会报错。
8. async 函数可以保留运行堆栈。

## await能不能被return，return await promise 和 return promise的区别
https://stackoverflow.com/questions/38708550/difference-between-return-await-promise-and-return-promise

1. 大体上没啥区别，效果一样
2. 一般await用try catch包裹，如果两个都用try catch包裹，如果promise reject，return await promise会走到catch， return promise不会。

## Class与继承
### class的用法
1. class声明类
2. constructor为构造方法，可传参。constructor方法是类的默认方法，如果没有显式定义，一个空的constructor方法会被默认添加。
3. 类可以定义实例属性和方法
4. 类内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。
5. 类的属性名，可以采用表达式
6. 类的方法内部如果含有this，它默认指向类的实例
7. 用static表示静态方法。静态方法可以被子类继承。如果静态方法包含this关键字，这个this指的是类，而不是实例。
8. 实例属性除了定义在constructor()方法里面的this上面，也可以定义在类的最顶层。
9. 目前只能通过在类名上面加属性。现在有一个提案在实例属性的前面，加上static关键字来表示静态属性。
10. 私有属性和私有方法，名字前面加下划线。或者用Symbol值命名私有方法名。因为Symbol作为属性名，不会被遍历出来。但可以用getOwnPropertySymbols遍历出来。
11. ES6 为new命令引入了一个new.target属性，该属性一般用在构造函数之中，返回new命令作用于的那个构造函数。

### 继承
通过extends关键字实现继承。  

子类必须在constructor方法中调用super方法。只有调用super之后，才可以使用this关键字，否则会报错。  
```js
class Father{
    constructor(){
        this.name = 'father'
    }
} 
class Son extends Father {
    constructor() {
        super();
        this.name = 'son'
    }
}
```
ES5的继承，子类必须调用父类的call，同时子类原型prototype是父类实例，子类原型prototype的构造函数是本身。
```js
var Father = function() {
    this.name = '爸爸';
}
var Son = function() {
    Father.call(this);
    this.name = '儿子';
}
Son.prototype = new Father();
Son.prototype.constructor = Son;
```

### class中的原型链
同时存在两条继承链：
1. 类的__proto__属性，表示构造函数的继承，总是指向父类。
2. 实例的__proto__属性，表示方法的继承，总是指向类的prototype属性。
```js
class A {
}
class B extends A {
}
let a = new A();
let b = new B();

B.__proto__ === A // true
A.__proto__ === Function.prototype // true

b.__proto__ === B.prototype // true
a.__proto__ === A.prototype // true
B.prototype.__proto__ === A.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

## class 中的箭头函数和普通函数有什么区别，如下
```js
class A {
  a = 1;
  fn() {
      console.log(this.a)
  }
  f = () => {
      console.log(this.a)
  };
}
let a = new A()
```
1. 两种写法都是原型方法，实例可以调用。
2. 普通函数写法，`a.fn()`这样调用没问题，如果是改下引用， `let b = a.fn; b();`这样调用会报错。
3. 箭头函数会绑定this，以后不管什么方式调用this都不会丢失。
