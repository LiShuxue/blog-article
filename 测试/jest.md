## Jest 安装使用

安装 `yarn add jest -D`

然后在项目目录下执行 `jest`

Jest 会寻找项目中所有的 test 文件来执行，根据下面的规则寻找

- `__tests__ ` 文件夹下的所有`.js`文件
- 所有 `.test.js` 文件
- 所有 `.spec.js`文件

## 测试文件放在哪

推荐在所写的每一个组件旁边，创建一个**test**文件夹，然后在下面创建 xxx.test.js

因为**test**文件夹下还可能会放测试的其他东西，比如快照 snapshot，而且这样在 test 文件中引用组件也会很方便，不用写一大堆相对路径 “../../../...”

> 个人观点，我并不觉得这是一种好的实践，如果对于 UI 组件库项目或许可以，如果对于一个大型应用，这样无疑会使代码结构变的很混乱，不易理解。而且我们基于代码搜索的时候，会把测试文件的内容也包含在内，不利于快速搜索到想要的内容。  
> 所以推荐，把测试文件单独放在一个跟 src 同级的目录下面。

## 常用的 CLI 配置所代表的意思

我们以命令行来运行 jest 的时候，可能需要传入一些 options.

- "--runInBand", // 按顺序执行测试，而不是默认的并行执行所有的，对调试有帮助
- "--no-cache", // 不使用缓存，默认使用上次的缓存
- "--watch", // 观察文件的变化，并重新执行变化的文件对应的测试
- "--watchAll" // 观察文件的变化，并重新执行所有的测试
- "--coverage" // 输出测试覆盖率报告
- 更多选项请看：https://jestjs.io/docs/en/cli

## 怎么编写测试文件

用 `it()` 或者 `test()` 来编写测试用例

用 `expect()` 函数来断言，即测试自己写的代码或者函数

```js
import sum from './sum';
it('sums numbers', () => {
  expect(sum(1, 2)).toEqual(3);
  expect(sum(2, 2)).toEqual(4);
});
```

## 匹配器

用来验证断言的执行结果，比如上述的 .toEqual()。Jest 本身提供了很多的匹配器，如

- .toBe 精确判断，判断值和引用
- .toEqual 判断值
- .not 用来取反
- toBeNull 只匹配 null
- toBeUndefined 只匹配 undefined
- toBeTruthy 匹配任何 if 语句为真
- toBeFalsy 匹配任何 if 语句为假
- .toBeGreaterThan 比某个数字大
- .toBeLessThan 比某个数字小
- .toMatch 字符串是否匹配某个正则表达式
- .toContain 判断数组或者可迭代对象是否包含某一项
- .toThrow 判断方法是否抛出了特定的异常信息
- .resolves 获得一个 Promise 的 resolve 结果，然后进行判断
- .rejects 获得一个 Promise 的 reject 结果，然后进行判断
- .toHaveBeenCalled() 验证 mock 方法被调用
- .toHaveBeenCalledTimes(number) 验证方法被调用了几次
- .toHaveBeenCalledWith(arg1, arg2...) 验证方法被调用的时候的参数
- .toMatchSnapsho 验证是否匹配最近的快照
- Jest 还提供了很多的匹配器供你选择，如果没有合适的，也可以自己写匹配器。更多匹配器信息参考 https://jestjs.io/docs/en/expect

## 如何测试异步代码

### 回调函数形式

假如我们有一个异步方法 fetchData，获取一些数据并在完成时调用一个回调函数来处理返回值 callback(data)，即`fetchData(callback)`

下面这种写法，断言并不会执行，因为默认情况下 Jest 按同步代码执行到最后就算结束

```js
test('test callback', () => {
  function callback(data) {
    expect(data).toBe('result'); // 不会执行
  }
  fetchData(callback);
});
```

可以使用传入一个 done 方法来解决，Jest 会等待 done 方法执行完毕后结束测试

```js
test('test callback', (done) => {
  function callback(data) {
    expect(data).toBe('result'); // 会执行
    done();
  }
  fetchData(callback);
});
```

### Promise 形式

如果 fetchData 不使用回调函数，而是返回一个 Promise，可以这样写

```js
test('test promise', () => {
  return fetchData().then((data) => {
    // 注意要加return关键字
    expect(data).toBe('result');
  });
});
```

不要忘记加 return 关键字，把 promise return 出去，不然 Jest 执行到 then 方法就完成了，里面的测试语句不会执行。

也可以使用 `.resolves / .rejects` 匹配器，也要加 return 关键字

```js
test('test promise', () => {
  return expect(fetchData()).resolves.toBe('result');
});
```

### Async/Await 形式

这种形式跟我们自己平时写代码是一样的，很简单

```js
test('test async await', async () => {
  const data = await fetchData();
  expect(data).toBe('result');
});
```

更多关于异步测试的信息，参考：https://jestjs.io/docs/en/asynchronous

> 个人思考，为什么 callback 形式和 promise 形式需要特殊处理才能按预期执行？

> 个人认为，这跟 JS 代码的执行原理有关系，我们知道在浏览器中执行异步代码，会通过事件循环来处理，也会分为宏任务，微任务。先执行同步代码，同步代码执行完后再执行微任务代码，然后是宏任务代码。这样保证了回调函数或者是 promise 等异步代码的执行，但是这个事件循环机制是 V8 引擎提供的。个人认为 Jest 并没有提供事件循环机制，所以要通过额外的操作，保证异步代码的执行。

> 可能有的会认为，为什么 async/await 的写法却跟我们在浏览器中执行代码一样，个人还是认为，这个是 Jest 自己实现的功能，跟浏览器保持了一致，而不是浏览器中的那个 async/await。

如果个人的理解有错的地方，欢迎指正

## Jest 的一些钩子,全局方法

### 执行前后的钩子

比如你可能需要在测试用例执行前，做一些准备工作，或者用例执行之后的一些收尾工作。

Jest 提供了`beforeEach()`, `afterEach()`, `beforeAll()`, `afterAll()` 来在不同时间段执行。区别是...Each 会在每一个 test 用例执行前后生效，而...All 一般写在文件的开头，会在整个文件的<b>所有用例</b>开始执行前和执行完毕后生效。

如果用于一组用例，可以用 describe 来分组，写在 describe 里面。

```js
beforeEach(() => {
  // do something
});
afterEach(() => {
  // do something
});
beforeAll(() => {
  // do something
});
afterAll(() => {
  // do something
});
```

### 只执行哪个，跳过哪个，以及哪个需要完善

`test.only，it.only`可以指定只执行这个用例，通常用在你这个测试文件比较大，里面有多恨测试用例，但是你为了方便调试，只想执行某一个或者某几个。

`test.skip，it.skip`指定跳过哪些用例，这些将不会执行

`test.todo` 声明的用例，会在最后控制台输出，表明这个测试用例还没有实现，是需要去做的

```js
test.only('test1', () => {
  ....
});

test.skip('test2', () => {
  ...
});

test.todo('test3');
```

更多全局方法函数参考：https://jestjs.io/docs/en/api

## Mock 函数, jest.fn, jest.mock

### 为什么需要 mock 函数？

在项目中，一个模块的方法内常常会去调用另外一个模块的方法。在单元测试中，我们可能并不需要关心内部调用的方法的执行过程和结果，只想知道它是否被正确调用即可，甚至会指定该函数的返回值。此时，使用 Mock 函数是十分有必要。

### jest.fn

使用`jest.fn`可以创建一个假的方法，我们可以对这个假的方法设置返回值，也可以对这个假方法添加一些具体实现，还可以通过这个假的方法的.mock 属性，获取这个方法被调用时的参数，this 指向，函数返回值等。

创建一个 mock 方法

```js
const myMock = jest.fn();

// 或者
const myMock = jest.fn(() => console.log('mock')); // 包含具体实现

// 或者
const myMock = jest.fn();
myMock.mockImplementation(() => console.log('mock')); // 通过mockImplementation添加具体实现

//或者
const myMock = jest.fn();
myMock
  .mockImplementationOnce(() => console.log('mock1'))
  .mockImplementationOnce(() => console.log('mock2')); // 通过mockImplementationOnce可以添加多个实现，用来多次调用
```

设置返回值

```js
const myMock = jest.fn();
myMock.mockReturnValue(0); // 这个方法被调用后将返回0

// 或者
const myMock = jest.fn();
myMock.mockReturnValueOnce(1);
      .mockReturnValueOnce(2); // mockReturnValueOnce 来添加多个返回值。这个方法被多次调用时，第一次返回1，第二次返回2
```

.mock 属性，获取这个方法被调用时的参数，this 指向，函数返回值等

```js
const myMock = jest.fn();
console.log(myMock.mock.calls); // 被调用时的参数
console.log(myMock.mock.instance); // this指向
console.log(myMock.mock.results); // 返回值
```

### jest.mock 来模拟模块

有时候我们引用了第三方的模块，或者自己写的一些模块，它里面也是包含自己的方法，我们可能并不想真正去执行这些方法，这个时候就需要根据这个原始模块创建一个假的替身。

比如我们用了 axios 库发请求，我们就可以模拟它。

```js
import axios from 'axios';
jest.mock('axios'); // 先模拟这个模块

test('mock', () => {
  axios.get.mockImplementation(() => Promise.resolve('result')); // 再重新定义模块中方法的实现

  return axios.get('https://xxx/xx/xx').then((result) => expect(result).toEqual('result')); // 调用这个模块的时候其实用的已经是我们mock的假的替身
});
```

如果我们自己写了一个模块，也可以模拟

```js
import Foo from './foo';
jest.mock('./foo');

test('mock', () => {
  Foo.mockImplementation(() => 1);
  expect(Foo()).toEqual(1);
});
```

虽然在代码中我们把 jest.mock 写在了后面，但实际上 Jest 会把 jest.mock 放在任何 import 的前面去调用，详见 https://github.com/kentcdodds/how-jest-mocking-works

### 手动 mock 模块

我们可以在所要测试的模块旁边，创建一个**mocks**文件夹，里面放一个跟我们所要测试的模块同名的 js 文件，对这个模块进行手动模拟。

这样当我们在测试文件中使用 jest.mock()方法时，它将使用我们手动创建的 mock 模块，而非 Jest 自动创建。

Jest 可以通过配置文件设置进行相应的设置，其中有一个选项是 automock，当设置为 true 时。在测试文件中所有 import 的包都将被自动 mock，默认是 false。

当 automock 设置为 true 时，即使未调用 jest.mock（'moduleName'），也会使用手动模拟实现而不是自动创建的模拟。要选择退出此行为，需要在应该使用实际模块实现的测试中显式调用 jest.unmock（'moduleName'）。一般无需使用 automock。

## 定时器模拟

我们在代码中可能会使用`setTimeout, setInterval`做一些定时任务，这些方法并不适合测试，因为我们的测试代码需要等待相应的延时。

Jest 提供了`jest.useFakeTimers()`来模拟定时器函数，它可以模拟 setTimeout 和其他的定时器函数。

如果你需要在一个文件或一个 describe 块中运行多次测试，在<b>每次测试前</b>手动添加`jest.useFakeTimers()`或者在 beforeEach 中添加。

```js
// timerGame.js
function timerGame() {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
  }, 1000);
}
module.exports = timerGame;

// __tests__/timerGame-test.js
jest.useFakeTimers();
test('waits 1 second before ending the game', () => {
  const timerGame = require('../timerGame');
  timerGame();

  expect(setTimeout).toHaveBeenCalledTimes(1);
  expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);
});
```

### 定时器控制函数

- `jest.runAllTimers()`  
  所有的 timer 都立即执行完毕(当前测试的目标模块中的所有 timer)，而无需等待那些延时。
- `jest.runOnlyPendingTimers()`  
  当前正在执行的 timer 立即执行完毕
- `jest.advanceTimersByTime(1000)`  
  将时间向前进行 1000ms,这个值可以自己设置。所有通过 setTimeout() 或 setInterval() 而处于<b>任务队列中等待中的</b>“宏任务”和一切其他<b>应该在本时间片中被执行的</b>东西都应该被执行。

## 覆盖率报表 Coverage Reporting

Jest 内部集成了 Istanbul，我们可以通过 `jest --coverage` 输出测试覆盖率报告。

会在控制台打印形如下列的信息。也会输出这些在 `/coverage` 文件夹下，可以打开 `/coverage/lcov-report/index.html` 查看详细信息。

```
----------|----------|----------|----------|----------|-------------------|
File      |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
----------|----------|----------|----------|----------|-------------------|
All files |    33.33 |      100 |        0 |    33.33 |                   |
 foo.js   |    33.33 |      100 |        0 |    33.33 |               3,4 |
----------|----------|----------|----------|----------|-------------------|
```

它包含 4 个维度的覆盖率，和未覆盖到的行。

- `% Lines` 行覆盖率（line coverage）：是否每一行都执行了？
- `% Funcs` 函数覆盖率（function coverage）：是否每个函数都调用了？
- `% Branch` 分支覆盖率（branch coverage）：是否每个 if 代码块都执行了？
- `% Stmts` 语句覆盖率（statement coverage）：是否每个语句都执行了？
