## Hook 简介

Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。即可以在函数式组件中使用 state 和生命周期等。

- 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
- 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。

## useState

useState 就是一个 Hook ，可以给函数式组件里添加一些内部 state。React 会在重复渲染时保留这个 state。

useState() 方法里面唯一的参数就是初始 state。

useState 会返回一对值：当前状态和一个让你更新它的函数，你可以在事件处理函数中或其他一些地方调用这个函数。它类似 class 组件的 this.setState，但是它不会把新的 state 和旧的 state 进行合并。

在初始化渲染期间，返回的状态 (state) 与传入的第一个参数 (initialState) 值相同。setState 函数用于更新 state。它接收一个新的 state 值并将组件的一次重新渲染加入队列。

在后续的重新渲染中，useState 返回的第一个值将始终是更新后最新的 state。

尽量不要直接在组件顶层 block 中调用 setState。

```jsx
function ExampleWithManyStates() {
  // 声明多个 state 变量！
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

## useEffect

你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”。

在函数式组件中通过 useEffect 给函数组件增加了操作副作用的能力。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。

默认情况下，它在第一次渲染之后和每次更新之后都会执行。

如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（[]）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行。

useEffect 的第二个参数可用于定义其依赖的所有变量。如果其中一个变量发生变化，则 useEffect 会再次运行。如果包含变量的数组为空，则在更新组件时 useEffect 不会再执行，因为它不会监听任何变量的变更。

useEffect 函数还可以通过返回一个函数来指定如何“清除”副作用，React 会在组件卸载时执行你返回的函数。

在一个函数中也可以使用多个 useEffect。

```jsx
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  const [result, setResult] = useState({});

  useEffect(() => {
    fetch(xxx).then((result) => {
      setResult(result);
    });
  }, []);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
    return () => {
      document.title = `reset title`;
    };
  }, [count]);

  // ...
}
```

所有组件内部状态的转换都应该归于纯函数中，不要把 useEffect 当成 watch 来用。如

```js
useEffect(() => {
  const dep2 = compute(dep1);
  setDep2(dep2);
}, [dep1]);
```

我们可能会习惯性的把 useEffect 当成一个 watch 来用，但每次我们 setState 过后，函数组件又会重新执行一遍，useEffect 也会重新跑一遍。所以对于这种，直接可以将 dep2 的计算放到组件外层执行。

```js
const Demo = () => {
  const [dep1, setDep1] = useState(0);
  const dep2 = compute(dep1); // 因为dep1更新后，这段代码肯定重新执行，而且此时dep1是更新后的值。
};
```

不要同时使用一堆依赖项 和 多个 useEffect。这样的代码非常容易造成循环依赖的问题，而且一旦出了问题，非常难排查很解决，整个 state 的更新很难预测。

## useContext

1、父组件导入并调用 createContext 方法，得到 Context 对象，并导出

```js
import { createContext } from 'react';
export const MyContext = createContext();
```

2、在父组件中使用 Provider 组件包裹需要接收数据的后代组件，并通过 value 属性提供要共享的数据

```js
function App() {
  return (
    <MyContext.Provider value={ ...something }>
      <Toolbar />
    </MyContext.Provider>
  );
}

function Toolbar() {
  return (
    <div>
      <MyButton />
    </div>
  );
}

```

3、需要获取共享数据的后代组件：导入 useContext，并按需导入根组件中导出的 Context 对象；调用 useContext，得到 value 的值

```js
function MyButton() {
  const something = useContext(MyContext);

  // ...
}
```

注意：useContext 的参数必须是 context 对象本身。

- 正确： useContext(MyContext)
- 错误： useContext(MyContext.Consumer)
- 错误： useContext(MyContext.Provider)

## useReducer

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

useReducer 是 react 提供的，类似 redux 中 reducer 的操作。也是管理内部 state 的，可以用来替代 useState。

它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。

```js
const [state, dispatch] = useReducer(reducer);
```

直接提供一个 state, 作为初始化 state

```js
const [state, dispatch] = useReducer(reducer, { count: 0 });
```

可以提供一个 init 函数作为第三个函数，来惰性地创建初始 state, 这样初始 state 将被设置为 init(initialArg)。

```js
function init(initialCount) {
  return { count: initialCount };
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // ...
}
```

使用 useState 的例子,

```js
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(0)}>Reset</button>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(count + 1)}>+</button>
    </>
  );
}
```

使用 useReducer 改写，

```js
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}
```

> useContext 结合 useReducer，可以模拟 redux 的操作，就可以抛弃 redux 库，完全拥抱 react 了。

## useCallback

传入一个操作函数，和一个依赖数组，返回该操作函数的“记忆”版本。该函数只有在某个依赖项改变的时候才会更新。

使用场景是：有一个父组件，其中包含子组件，子组件接收一个函数作为 props，通常而言，如果父组件更新了，子组件也会执行更新。但是大多数场景下，子组件更新是没有必要的，所以我们使用 memo 将子组件包裹，他会浅比较 props 是否改变。但是由于我们传递的 props 是一个函数，默认每次父组件更新创建的函数都不一样。所以我们可以借助 useCallback 来返回函数，然后把这个函数作为 props 传递给子组件；这样，子组件就能避免不必要的更新。

```jsx
function Parent() {
  const [count, setCount] = useState(1);
  const [val, setVal] = useState('');

  const callback = useCallback(() => {
      return count;
  }, [count]);

  return (
    <div>
      <h4>{count}</h4>
      <Child callback={callback}/>
      <div>
          <button onClick={() => setCount(count + 1)}>+</button>
          <input value={val} onChange={event => setVal(event.target.value)}/>
      </div>
    </div>
  );
}

function Child = memo(({ callback }) => {
    const [count, setCount] = useState(() => callback());

    return <div>{count}</div>
});
```

## useMemo

传入一个函数（这个函数返回一个值），和一个依赖数组，useMemo 返回这个值的“记忆”版本，该值只有在某个依赖项改变的时候才会更新。相当于 Vue 中的计算属性，当某个依赖项改变时才重新计算值，这种优化有助于避免在每次渲染时都进行<b>高开销</b>的计算。

```jsx
function WithoutMemo() {
  const [count, setCount] = useState(1);
  const [val, setValue] = useState('');

  function expensive() {
    console.log('compute');
    let sum = 0;
    for (let i = 0; i < count * 100; i++) {
      sum += i;
    }
    return sum;
  }

  return (
    <div>
      <h4>
        {count}-{val}-{expensive()}
      </h4>
      <div>
        <button onClick={() => setCount(count + 1)}>+c1</button>
        <input value={val} onChange={(event) => setValue(event.target.value)} />
      </div>
    </div>
  );
}
```

不使用 useMemo 的时候，val 和 count 改变，组件重新渲染，都会触发 expensive 的执行。

```jsx
function WithMemo() {
  const [count, setCount] = useState(1);
  const [val, setValue] = useState('');

  const expensive = useMemo(() => {
    console.log('compute');
    let sum = 0;
    for (let i = 0; i < count * 100; i++) {
      sum += i;
    }
    return sum;
  }, [count]);

  return (
    <div>
      <h4>
        {count}-{expensive}
      </h4>
      {val}
      <div>
        <button onClick={() => setCount(count + 1)}>+c1</button>
        <input value={val} onChange={(event) => setValue(event.target.value)} />
      </div>
    </div>
  );
}
```

val 改变，组件重新渲染，expensive 不会执行，返回上次缓存的值。count 改变，组件重新渲染，expensive 会重新执行。

> useMemo 和 useCallback 都会在组件第一次渲染的时候执行，之后会在其依赖的变量发生改变时再次执行；并且这两个 hooks 都返回缓存的值，useMemo 返回缓存的变量，useCallback 返回缓存的函数。

> useMemo 主要用来优化自身，假如自己组件内部有一个高开销的函数，这样在组件每次状态改变，重新渲染的时候，这个高开销的函数都会重新执行，浪费性能。用 useMemon 包裹这个高开销的函数，避免在每次渲染的时候都重新执行，只有依赖项改变的时候才重新执行。

> useCallback 主要用来优化子组件，避免子组件的重新渲染，一般来说父组件状态更新并重新渲染的时候，子组件也会更新并重新渲染。通常我们可以使用 memo 将子组件包裹，他会浅比较 props 是否改变来决定是否重新渲染。但是由于我们传递的 props 是一个函数，默认每次父组件更新创建的函数都不一样，所以子组件都会重新渲染。所以我们可以借助 useCallback 来返回一个不经常变的函数，然后把这个函数作为 props 传递给子组件，这样，子组件就能避免不必要的更新。

> useCallback 通常跟 memo 一起使用

## useRef

useRef(initValue)可以创建一个 ref 对象，也可以接收一个值，作为 返回值.current 的值。组件重新渲染时，useRef 每次都会返回相同的引用，所以可以在多次渲染中保存同一个值。

useRef 变化不会主动使页面渲染。current 属性值可以被主动改变，但是不会跟 useState 或者 useReducer 一样触发页面变化。

1. 获取 DOM 对象。可以直接用在 html 标签上，如果用在子组件上，获取不到子组件的 DOM，需要借助 forwardRef 来实现。
2. 保存一个不变的数据。

```jsx
function CustomTextInput(props) {
  // 创建一个 ref
  const textInput = useRef(null);

  function handleClick() {
    textInput.current.focus(); // 获取dom对象
  }

  return (
    <div>
      {// 在html标签使用ref }
      <input type="text" ref={textInput} />
    </div>
  );
}

// 用在子组件上，需要配合forwardRef
<Child ref={textInput} />
const Child = forwardRef((props, ref) => {
  return <input type="text" ref={ref} />;
});
```

## 自定义 Hook

有时候我们会想要在组件之间重用一些状态逻辑。目前为止，有两种主流方案来解决这个问题：高阶组件和 render props。自定义 Hook 可以让你在不增加组件的情况下达到同样的目的。

自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook。

```jsx
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```
