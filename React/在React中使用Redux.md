# 在 React 中使用 Redux

上一篇文章中所介绍的内容，与 React 没有关系，上面都是在 Redux 中如何操作状态，其他框架也可以使用 Redux。

如果想在 react 中使用 redux，需要借助**react-redux**库。

## react-redux

### Provider

React Redux 提供了 `<Provider />`, 能够使你的 Redux store 中的数据在整个 app 是可用的。

```jsx
import React from "react";
import ReactDOM from "react-dom";

import { Provider } from "react-redux";
import store from "./store";

import App from "./App";

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root");
);
```

### `connect()`

React Redux 还提供一个 `connect()` 方法能够让你把组件和 store 连接起来，这样就可以在你的组件中使用 store 中的相关信息。

```js
import { connect } from 'react-redux'

// 一个Test组件
const Test = () => {
  render(....);
}

const mapStateToProps = (state, ownProps) => {
  // ...
}

const mapDispatchToProps = // ...

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Test)
```

### `connect()`方法接受两个参数

第一个参数是 `mapStateToProps(state, ownProps?)`方法

作用：将 state 映射成组件的**额外的**props 传递给组件

- 输入 state 和组件本身的 props，然后返回一个对象，对象里的每一项都会变成组件额外的 props 传递给组件
- 每次 state 改变，这个方法都会被调用，所以组件额外的 props 都会被更新，导致组件重新渲染
- 这个方法应作为第一个参数传递给 connect，如果你不希望订阅 store，那么请传递 null 或者 undefined 作为 connect 的第一个参数。

```js
function mapStateToProps(state, ownProps) {
  const { test } = state;
  const { id } = ownProps;
  const todo = getTodoById(state, id);
  return { test, todo };
}

// component will receive: props.test, props.todo
```

第二个参数是 `mapDispatchToProps` 方法或者对象

作用：实现向 store 中 dispatch acions。

- 学习 redux 的时候我们知道，一般需要用 store.dispatch(action)来改变 state，比如在 components 中调用 store.dispatch。但是当我们使用了 react-redux 库的`connect()`方法以后，我们的组件就不再需要直接和 store 打交道了，不需要在组件中写 store.dispatch 了。

- 默认的，如果你不为 connect()指明第二个参数，即不使用 mapDispatchToProps，你的组件会默认接收 props.dispatch，然后你可以在组件中用 dispatch(action)去分发 action，改变 state

- 如果你指明了 mapDispatchToProps 参数，这个参数中所包含的方法，在被调用时会自动 dispatch actions，你就无需再手写 dispatch 了。

mapDispatchToProps 有三种形式

1. 定义为一个函数

接收 dispatch 和 ownProps?作为参数，并且返回一个能够使用 dispatch 来分发 actions 的若干函数组成的对象。  
函数返回值也会合并到你的组件 props 中去。你就能够直接调用它们来分发 action。

优点：可以自定义一些方法让组件去调用。

```js
// action creaters
const increment = () => ({ type: 'INCREMENT' });
const decrement = () => ({ type: 'DECREMENT' });

// 定义mapDispatchToProps
const mapDispatchToProps = (dispatch) => {
  return {
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    reset: () => dispatch({ type: 'RESET' }),
  };
};

// 返回值也会变成组件的props，如props.increment, props.decrement, props.reset
function Counter({ count, increment, decrement, reset }) {
  return (
    <div>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
      <button onClick={reset}>reset</button>
    </div>
  );
}
```

2. 通过 bindActionCreators 定义函数

bindActionCreators 将 action creators 转化为一个对象, 这个对象跟 action creator 有同样的 key, 并且每一个 action creator 会被包裹在一个 dispatch 方法中。

简而言之，就是上面第一种形式的简化版。

```js
import { bindActionCreators } from 'redux';

// action creaters
const increment = () => ({ type: 'INCREMENT' });
const decrement = () => ({ type: 'DECREMENT' });

// 定义mapDispatchToProps
const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({ increment, decrement, reset }, dispatch);
  // 相当于 returns
  // {
  //   increment: (...args) => dispatch(increment(...args)),
  //   decrement: (...args) => dispatch(decrement(...args)),
  //   reset: (...args) => dispatch(reset(...args)),
  // }
};
//
```

3. 定义为一个对象

根据上面的写法，在 React 中调用 redux 的过程非常相似，都是先定义一个 action creator, 然后将他包裹在另一个方法中，如`(…args) => dispatch(actionCreator(…args))`, 然后将这个方法当成 props 传递给组件。

因为这在 react 使用中很常见，所以 react-redux 的 connect 方法支持简写形式，即如果你的 mapDispatchToProps 传递的是 action creator 构成的对象， connect 内部会自动给你调用 bindActionCreators 去绑定 dispatch。

```js
// action creaters
const increment = () => ({ type: 'INCREMENT' });
const decrement = () => ({ type: 'DECREMENT' });

// 定义mapDispatchToProps
const mapDispatchToProps = { increment, decrement };
//
```

> 我们推荐总是使用这种对象简写形式，除非你有特殊的原因需要自定义你的 dispatch 行为

---

## Dumb 组件（展示组件） 和 Smart 组件（容器组件）

Dumb 组件，又叫展示组件，无状态组件

1. 关注页面的展示效果（外观）
2. 内部可以包含展示组件和容器组件，通常会包含一些自己的 DOM 标记和样式(style)
3. 通常允许通过 this.props.children 方式来包含其他组件。
4. 对应用程序的其他部分没有依赖关系，例如 Flux 操作或 store。
5. 不用关心数据是怎么加载和变动的。
6. 只能通过 props 的方式接收数据和进行回调(callback)操作。
7. 很少拥有自己的状态，即使有也是用于展示 UI 状态的。
8. 会被写成函数式组件除非该组件需要自己的状态，生命周期或者做一些性能优化。

Smart 组件，又叫容器组件，有状态组件

1. 关注应用是如何工作的
2. 内部可以包含容器组件和展示组件，但通常没有任何自己的 DOM 标记，除了一些包装 divs，并且从不具有任何样式。
3. 提供数据和行为给其他的展示组件或容器组件。
4. 调用 Flux 操作并将它们作为回调函数提供给展示组件。
5. 往往是有状态的，因为它们倾向于作为数据源
6. 通常使用高阶组件生成，例如 React Redux 的 connect（），Relay 的 createContainer（）或 Flux Utils 的 Container.create（），而不是手工编写。

## 使用 Immutable.JS

### 永远不要将普通的 JavaScript 对象与 Immutable.JS 混合使用

### 使整个 Redux state tree 成为 Immutable.JS 对象

### 在除了 Dumb 组件外的组件使用 Immutable.JS
