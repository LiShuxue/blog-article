# 了解Redux

## 四大核心概念
### Store
存储应用的 state，是一个普通的js对象。一个应用只有一个Store对象

State 是只读的，唯一改变 state 的方法就是触发 action

### Action
描述发生的事件，也是一个对象，包括 type 和 'payload' 等字段，可以定义很多个action。也可以用 Action 创建函数生成 action

```js
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
```
触发action
```js
store.dispatch({ type: 'ADD_TODO', text: 'Go to swimming pool' })
```
FSA，即Flux Standard Action，一个用户友好的Flux action对象标准
1. 一个action必须是一个普通的JavaScript对象，有一个type字段。
2. 一个action可能有error字段、payload字段、meta字段。
3. 一个action必须不能包含除type、payload、error及meta以外的其他字段。

### Reducer
描述 action 如何改变 state ，就是一个函数，接收 action 和旧的 state 对象，改变state的值，并返回新的state。一个应用可以只有一个Reducer，但随着应用的增大，我们需要编写很多小的reducer函数来分别管理 state 的一部分。

> 注意：不能修改旧state，必须先拷贝一份state，再进行修改。
> 1. `newState = JSON.parse(JSON.stringfy(state))`
> 2. 使用`Object.assign`函数生成新的state
> 3. 使用扩展运算符`...state`
```js
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    default:
      return state
  }
}
```

### Middleware
在 Express 或者 Koa 等框架中，middleware 是指可以被嵌入在框架接收请求到产生响应过程之中的代码。  
相对于 Express 或者 Koa，Redux middleware 被用于解决不同的问题，但其中的概念是类似的。它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。 你可以利用 Redux middleware 来进行<b>日志记录、创建崩溃报告、调用异步接口或者路由</b>等等。

## 常用API

### `createStore(reducer, [preloadedState], enhancer)`
reducer： Function，接收两个参数，分别是当前的 state 树和要处理的 action，返回新的 state 树。  
preloadedState： 初始时的 state。  
enhancer： Function，Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator。这与 middleware 相似，它也允许你通过复合函数改变 store 接口。

### `combineReducers(reducers)`
reducers： 一个对象，它的值对应不同的 reducer 函数

combineReducers 辅助函数可以把多个不同 reducer 函数合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 createStore 方法。

通过为传入对象的 reducer 命名不同的 key 来控制返回 state key 的命名。例如， `combineReducers({ todos: myTodosReducer, counter: myCounterReducer })` 将 state 结构变为 `{ todos, counter }`。  

### Store相关方法
1. `store.getState()` 方法获取 state；
2. `store.dispatch(action)` 方法更新 state；
3. `store.subscribe(listener)` 注册监听器;
4. `store.subscribe(listener)()` 返回的函数注销监听器。

### `applyMiddleware(...middleware)`
...middleware: 遵循 Redux middleware API 的函数。  
每个 middleware 接受 Store 的 dispatch 和 getState 函数作为命名参数，并返回一个函数。该函数会被传入被称为 next 的下一个 middleware 的 dispatch 方法，并返回一个接收 action 的新函数，这个函数可以直接调用 next(action)，或者在其他需要的时刻调用，甚至根本不去调用它。调用链中最后一个 middleware 会接受真实的 store 的 dispatch 方法作为 next 参数，并借此结束调用链。  
所以，middleware 的函数签名是 ({ getState, dispatch }) => next => action。

## 常用的middleware
### `redux-thunk`，异步action
Redux Thunk middleware 允许创建一个返回函数的action生成器，而不是返回action。这就叫thunk。  
返回的函数接收 dispatch 和 getState 作为参数。  
可以用来延迟dispatch action，所以可以写一些异步代码。  
也可以dispach一个其他的thunk。

store.dispatch方法正常情况下，参数只能是对象，不能是函数。这时，就要使用中间件redux-thunk。
```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

const store = createStore(rootReducer, applyMiddleware(thunk));

function testActionCreator() {
  return (dispatch, getState) => {
    const { counter } = getState();

    if (counter % 2 === 0) {
      return;
    } else {
      setTimeout(() => {
        dispatch({ type: 'ADD_TODO', text: 'Go to swimming pool' });
      }, 1000);
    }
  };
}
```

### `redux-promise` 或者 `redux-promise-middleware`，异步action
redux-thunk可以dispatch函数，而用redux-promise 或者 redux-promise-middleware 可以 dispatch 一个 Promise

写法一，Action creator返回值是一个 Promise 对象。可以dispatch promise resovle的value，如果promise reject，不会dispacth任何东西。
```js
import { createStore, applyMiddleware } from 'redux';
import promiseMiddleware from 'redux-promise';
import rootReducer from './reducers/index';

const store = createStore(rootReducer, applyMiddleware(promiseMiddleware));

// 参数可以是任意的，也可以不传
function testActionCreator(dispatch, anyParam) {
  return new Promise((resolve, reject) => {
    dispatch({ type: 'ADD_TODO', text: 'Go to swimming pool' });

    return fetch(`/some/API/${anyParam}.json`)
      .then(response => {
        type: 'FETCH_POSTS',
        payload: response.json()
      });
  });
}
```

写法二，Action 对象的payload属性是一个 Promise 对象。这需要借助<u>redux-actions</u>模块引入createAction方法，生成FSA
1. dispatch a copy of the action with the resolved value of the promise, and set status to success.
2. dispatch a copy of the action with the rejected value of the promise, and set status to error.
```js
import { createAction } from 'redux-actions';

store.dispatch(createAction(
  'FETCH_POSTS', 
  fetch(`/some/API/${postTitle}.json`)
    .then(response => response.json())
));
```

### `redux-actions` 
创建标准的action，以及处理action

`createAction(type, payloadCreator, metaCreator)`: action生成器，创建action  
1. 第一个参数：action类型，必须参数   
2. 第二个参数：生成payload的函数，可选参数。 
3. 第三个参数：生成meta 字段的函数，可选  

`createActions` 创建多个action
```js
const increment = createAction('INCREMENT');
const increment2 = createAction('INCREMENT', () => {
  return payload
});
const { actionOne, actionTwo, actionThree } = createActions({
    ACTION_ONE: (key, value) => ({ [key]: value }),
    ACTION_TWO: [
      first => [first], // payload
      (first, second) => ({ second }) // meta
    ]
  },
  'ACTION_THREE'
);
```

`handleAction(type, reducer, defaultState)`: 处理action，相当于普通reducer的功能，返回值就是reducer
1. 第一个参数：action类型，必须参数   
2. 第二个参数：真正的reducer函数，接收state和action，返回新的state，必须参数。 
3. 第三个参数：state的初始值，可选  

`handleActions` 处理多个action
```js
import {handleAction, handleActions} from 'redux-actions';
const reducer = handleAction(
  'ADDNUM', 
  (state, action) => {
    let newState = {...state};
    newState.n++;
    return newState;
  },
  defaultState
)

const reducer = handleActions(
  {
    INCREMENT: (state, action) => ({
      counter: state.counter + action.payload
    }),

    DECREMENT: (state, action) => ({
      counter: state.counter - action.payload
    })
  },
  { counter: 0 }
);
```

`combineActions(...types)` 合并多个action type，使他们公用一个reducer
```js
const reducer = handleAction(
  combineActions(increment, decrement), 
  (state, action) => {
    let newState = {...state};
    newState.n++;
    return newState;
  },
  { counter: 10 }
)
```

### 用redux-observable 来 dispatch Observable。
### 用redux-saga 中间件来创建更加复杂的异步 action。
### 用redux-pack 中间件 dispatch 基于 Promise 的异步 Action。
### 用redux-logger 中间件记录显示 action、state 的变化
