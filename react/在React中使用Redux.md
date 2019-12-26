# 在React中使用Redux

上一篇文章中所介绍的内容，与React没有关系，上面都是在Redux中如何操作状态，其他框架也可以使用Redux。

如果想在react中使用redux，需要借助<u>react-redux</u>库。

## react-redux
React Redux 提供了 `<Provider />`, 能够使你的整个app访问到Redux store中的数据
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

React Redux 提供一个 `connect` 方法能够让你把组件和store连接起来

mapStateToProps(state, ownProps?)：第一个参数是state，可选的第二个参数时ownProps，然后返回一个被连接组件所需要的数据的纯对象。  
这个方法应作为第一个参数传递给connect，并且会在每次Redux store state改变时被调用。如果你不希望订阅store，那么请传递null或者undefined替代mapStateToProps作为connect的第一个参数。  
将state映射成组件的<b>额外的</b>props，因为组件本身有自己的props

mapDispatchToProps：实现向store中分发acions，你的组件就不再需要直接和store打交道了，不需要在组件中写store.dispatch  
默认的，如果你不为connect()指明第二个参数，即不使用mapDispatchToProps，你的组件会默认接收dispatch  
一旦你以这种方式连接了你的组件，你的组件就会接收props.dispatch。你可以用它来向store中分发actions。

mapDispatchToProps有两种形式，函数或者对象  
如果是一个函数，一旦该组件被创建，就会被调用。接收dispatch作为一个参数，并且返回一个能够使用dispatch来分发actions的若干函数组成的对象。  
函数返回值会合并到你的组件props中去。你就能够直接调用它们来分发action。
```js
const mapDispatchToProps = dispatch => {
  return {
    increment: () => dispatch({ type: "INCREMENT" }),
    decrement: () => dispatch({ type: "DECREMENT" }),
    reset: () => dispatch({ type: "RESET" })
  };
};

function Counter({ count, increment, decrement, reset }) {
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={reset}>reset</button>
    </div>
  );
}
```  
如果是一个action creator构成的对象，每一个action creator将会转化为一个prop function并会在调用时自动分发actions。注意： 我们建议使用这种形式。
```js
import { increment, decrement, reset } from "./counterActions";

const actionCreators = {
  increment,
  decrement,
  reset
}

export default connect(mapState, actionCreators)(Counter);

function Counter({ count, increment, decrement, reset }) {
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={reset}>reset</button>
    </div>
  );
}
```

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
