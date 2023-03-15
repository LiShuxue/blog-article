## Hook 简介
Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。即可以在函数式组件中使用state和生命周期等。

* 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
* 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。

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

在函数式组件中通过useEffect给函数组件增加了操作副作用的能力。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。

默认情况下，它在第一次渲染之后和每次更新之后都会执行。

如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（[]）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行。

useEffect的第二个参数可用于定义其依赖的所有变量。如果其中一个变量发生变化，则useEffect会再次运行。如果包含变量的数组为空，则在更新组件时useEffect不会再执行，因为它不会监听任何变量的变更。

useEffect函数还可以通过返回一个函数来指定如何“清除”副作用，React 会在组件卸载时执行你返回的函数。

在一个函数中也可以使用多个 useEffect。

```jsx
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  const [result, setResult] = useState({});

  useEffect(() => {
    fetch(xxx).then(result => {
      setResult(result)
    })
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
},[dep1]);
```
我们可能会习惯性的把 useEffect 当成一个 watch 来用，但每次我们 setState 过后，函数组件又会重新执行一遍，useEffect 也会重新跑一遍。所以对于这种，直接可以将dep2的计算放到组件外层执行。
```js
const Demo = () => {
  const [dep1,setDep1] = useState(0);
  const dep2 = compute(dep1); // 因为dep1更新后，这段代码肯定重新执行，而且此时dep1是更新后的值。
}
```

不要同时使用一堆依赖项 和 多个 useEffect。这样的代码非常容易造成循环依赖的问题，而且一旦出了问题，非常难排查很解决，整个 state 的更新很难预测。

## useContext
接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。

当组件上层最近的 <MyContext.Provider> 更新时，该 Hook 会触发重渲染，并使用最新传递给 MyContext provider 的 context value 值。
```js
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```
注意：useContext 的参数必须是 context 对象本身。
* 正确： useContext(MyContext)
* 错误： useContext(MyContext.Consumer)
* 错误： useContext(MyContext.Provider)

## useReducer
```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
useReducer是react提供的，类似redux中reducer的操作。也是管理内部state的，可以用来替代useState。

它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。
```js
const [state, dispatch] = useReducer(reducer);
```
直接提供一个state, 作为初始化state
```js
const [state, dispatch] = useReducer(reducer, {count: 0});
``` 
可以提供一个init函数作为第三个函数，来惰性地创建初始 state, 这样初始 state 将被设置为 init(initialArg)。
```js
function init(initialCount) {
  return {count: initialCount};
}
function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // ...
}

```

使用useState的例子,
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
使用useReducer改写，
```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

> useContext结合useReducer，可以模拟redux的操作，就可以抛弃redux库，完全拥抱react了。

## useCallback
传入一个操作函数，和一个依赖数组，返回该操作函数的“记忆”版本。该函数只有在某个依赖项改变的时候才会更新。

使用场景是：有一个父组件，其中包含子组件，子组件接收一个函数作为props，通常而言，如果父组件更新了，子组件也会执行更新。但是大多数场景下，子组件更新是没有必要的，所以我们使用memo将子组件包裹，他会浅比较props是否改变。但是由于我们传递的props是一个函数，默认每次父组件更新创建的函数都不一样。所以我们可以借助useCallback来返回函数，然后把这个函数作为props传递给子组件；这样，子组件就能避免不必要的更新。

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
传入一个函数（这个函数返回一个值），和一个依赖数组，useMemo返回这个值的“记忆”版本，该值只有在某个依赖项改变的时候才会更新。相当于Vue中的计算属性，当某个依赖项改变时才重新计算值，这种优化有助于避免在每次渲染时都进行<b>高开销</b>的计算。

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

  return <div>
      <h4>{count}-{val}-{expensive()}</h4>
      <div>
          <button onClick={() => setCount(count + 1)}>+c1</button>
          <input value={val} onChange={event => setValue(event.target.value)}/>
      </div>
  </div>;
}
```
不使用useMemo的时候，val和count改变，组件重新渲染，都会触发expensive的执行。
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

  return <div>
      <h4>{count}-{expensive}</h4>
      {val}
      <div>
          <button onClick={() => setCount(count + 1)}>+c1</button>
          <input value={val} onChange={event => setValue(event.target.value)}/>
      </div>
  </div>;
}
```
val改变，组件重新渲染，expensive不会执行，返回上次缓存的值。count改变，组件重新渲染，expensive会重新执行。

>useMemo和useCallback都会在组件第一次渲染的时候执行，之后会在其依赖的变量发生改变时再次执行；并且这两个hooks都返回缓存的值，useMemo返回缓存的变量，useCallback返回缓存的函数。

>useMemo主要用来优化自身，假如自己组件内部有一个高开销的函数，这样在组件每次状态改变，重新渲染的时候，这个高开销的函数都会重新执行，浪费性能。用useMemon包裹这个高开销的函数，避免在每次渲染的时候都重新执行，只有依赖项改变的时候才重新执行。

>useCallback主要用来优化子组件，避免子组件的重新渲染，一般来说父组件状态更新并重新渲染的时候，子组件也会更新并重新渲染。通常我们可以使用memo将子组件包裹，他会浅比较props是否改变来决定是否重新渲染。但是由于我们传递的props是一个函数，默认每次父组件更新创建的函数都不一样，所以子组件都会重新渲染。所以我们可以借助useCallback来返回一个不经常变的函数，然后把这个函数作为props传递给子组件，这样，子组件就能避免不必要的更新。

>useCallback通常跟memo一起使用

## useRef
useRef(initValue)可以创建一个ref对象，也可以接收一个参数作为.current属性的值。
>什么是ref  
ref可以用来获取DOM对象或者class组件的实例(函数式组件没有实例)。  
在class组件中可以用React.createRef()来创建ref  
通过 `ref={myRef}` 绑定在DOM 对象或者React组件上  
通过 `this.myRef.current` 访问DOM对象或者class组件的实例

useRef创建的ref， 除了可以挂载在DOM对象或者class组件上之外，还可以用来保存一个不变的数据。这个数据在整个生命周期中都不变。

在函数式组件中，如果用ref = React.createRef()去创建ref，每次重新渲染， 这个ref都会重新生成， 所以是不对的。（但是class组件中没有这个问题， 因为class组件将各个生命周期都分开了，你在最开始创建的ref，组件mount的时候不会被改变。）

所以只能用useRef，useRef创建的ref就像外部定义的一个全局变量，不会随着组件的更新而重新创建。

## 自定义Hook
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
