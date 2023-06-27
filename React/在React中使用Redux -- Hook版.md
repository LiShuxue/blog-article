之前我们介绍了在 React 中如何使用 Redux，需要借助‘react-redux’库来实现，通过 Provider 提供一个 store，然后用 connect()高阶组件，来在组件中使用 state 和 dispatch action。

这篇文章主要讲如何不用上述写法，而是使用 Hooks 来实现。这些 hooks 最初在 react-redux v7.1.0 中添加。

## 提供 Store

首先跟之前一样，还是需要先给整个组件树提供 store.

```js
const store = createStore(rootReducer);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

## useSelector()

`useSelector(selector, equalityFn?)` 从 store 中提取 state。

1. 参数一 selector 是一个纯函数，接收整个 store state，返回可以是任意值。
2. 这样就会对一个 state 进行订阅，当他更新的时候，useSelector 会执行。
3. 当一个 action 被 dispatch 后，useSelector ()将对前一个选择器结果值和当前结果值进行引用比较（===比较）。如果它们是不同的，组件将被迫重新渲染。如果它们是相同的，组件将不会重新呈现。
4. 第二个参数提供一个比较函数，来确定是否需要重新渲染。

```js
import React from 'react';
import { useSelector } from 'react-redux';

export const CounterComponent = () => {
  const counter = useSelector((state) => state.counter);
  return <div>{counter}</div>;
};
```

## useDispatch()

返回 dispatch 函数，你可以在组件中使用，去 dispatch action

```js
import React from 'react';
import { useDispatch } from 'react-redux';

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch();

  return (
    <div>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment counter</button>
    </div>
  );
};
```
