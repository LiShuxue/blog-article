# 在React中使用Redux

上一篇文章中所介绍的内容，与React没有关系，上面都是在Redux中如何操作状态，其他框架也可以使用Redux。

如果想在react中使用redux，需要借助<u>react-redux</u>库。

## react-redux
### Provider
React Redux 提供了 `<Provider />`, 能够使你的Redux store中的数据在整个app是可用的。
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
React Redux 还提供一个 `connect()` 方法能够让你把组件和store连接起来，这样就可以在你的组件中使用store中的相关信息。
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

作用：将state映射成组件的<b>额外的</b>props传递给组件
  * 输入state和组件本身的props，然后返回一个对象，对象里的每一项都会变成组件额外的props传递给组件
  * 每次state改变，这个方法都会被调用，所以组件额外的props都会被更新，导致组件重新渲染
  * 这个方法应作为第一个参数传递给connect，如果你不希望订阅store，那么请传递null或者undefined作为connect的第一个参数。  
```js
function mapStateToProps(state, ownProps) {
  const { test } = state
  const { id } = ownProps
  const todo = getTodoById(state, id)
  return { test, todo }
}

// component will receive: props.test, props.todo
```

第二个参数是 `mapDispatchToProps` 方法或者对象

作用：实现向store中dispatch acions。

* 学习redux的时候我们知道，一般需要用store.dispatch(action)来改变state，比如在components中调用store.dispatch。但是当我们使用了react-redux库的`connect()`方法以后，我们的组件就不再需要直接和store打交道了，不需要在组件中写store.dispatch了。

* 默认的，如果你不为connect()指明第二个参数，即不使用mapDispatchToProps，你的组件会默认接收props.dispatch，然后你可以在组件中用dispatch(action)去分发action，改变state

* 如果你指明了mapDispatchToProps参数，这个参数中所包含的方法，在被调用时会自动dispatch actions，你就无需再手写dispatch了。

mapDispatchToProps有三种形式

1. 定义为一个函数

接收dispatch和ownProps?作为参数，并且返回一个能够使用dispatch来分发actions的若干函数组成的对象。  
函数返回值也会合并到你的组件props中去。你就能够直接调用它们来分发action。

优点：可以自定义一些方法让组件去调用。
```js
// action creaters
const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })

// 定义mapDispatchToProps
const mapDispatchToProps = dispatch => {
  return {
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    reset: () => dispatch({ type: "RESET" })
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

bindActionCreators 将 action creators 转化为一个对象, 这个对象跟action creator 有同样的key, 并且每一个action creator会被包裹在一个dispatch方法中。

简而言之，就是上面第一种形式的简化版。

```js
import { bindActionCreators } from 'redux'

// action creaters
const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })

// 定义mapDispatchToProps
const mapDispatchToProps = dispatch => {
  return bindActionCreators({ increment, decrement, reset }, dispatch)
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

根据上面的写法，在React中调用redux的过程非常相似，都是先定义一个action creator, 然后将他包裹在另一个方法中，如`(…args) => dispatch(actionCreator(…args))`, 然后将这个方法当成props传递给组件。

因为这在react使用中很常见，所以react-redux的connect方法支持简写形式，即如果你的mapDispatchToProps传递的是action creator构成的对象， connect内部会自动给你调用bindActionCreators去绑定dispatch。

```js
// action creaters
const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })

// 定义mapDispatchToProps
const mapDispatchToProps = { increment, decrement }
// 
```

> 我们推荐总是使用这种对象简写形式，除非你有特殊的原因需要自定义你的dispatch行为

---

## Dumb 组件（展示组件） 和 Smart 组件（容器组件）
Dumb组件，又叫展示组件，无状态组件
1. 关注页面的展示效果（外观）
2. 内部可以包含展示组件和容器组件，通常会包含一些自己的DOM标记和样式(style)
3. 通常允许通过this.props.children方式来包含其他组件。
4. 对应用程序的其他部分没有依赖关系，例如Flux操作或store。
5. 不用关心数据是怎么加载和变动的。
6. 只能通过props的方式接收数据和进行回调(callback)操作。
7. 很少拥有自己的状态，即使有也是用于展示UI状态的。
8. 会被写成函数式组件除非该组件需要自己的状态，生命周期或者做一些性能优化。

Smart组件，又叫容器组件，有状态组件
1. 关注应用是如何工作的
2. 内部可以包含容器组件和展示组件，但通常没有任何自己的DOM标记，除了一些包装divs，并且从不具有任何样式。
3. 提供数据和行为给其他的展示组件或容器组件。
4. 调用Flux操作并将它们作为回调函数提供给展示组件。
5. 往往是有状态的，因为它们倾向于作为数据源
6. 通常使用高阶组件生成，例如React Redux的connect（），Relay的createContainer（）或Flux Utils的Container.create（），而不是手工编写。

## 使用Immutable.JS
### 永远不要将普通的 JavaScript 对象与 Immutable.JS 混合使用

### 使整个 Redux state tree 成为 Immutable.JS 对象

### 在除了 Dumb 组件外的组件使用 Immutable.JS
