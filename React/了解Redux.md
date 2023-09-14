# 了解 Redux

## 三大核心概念

### Store

store 用来存储应用的 state，是一个普通的 js 对象。一个应用只有一个 Store 对象，用 createStore 方法创建 store。

```js
import { createStore, applyMiddleware } from 'redux';
import reducers from './reducers';
const store = createStore(reducers, initState, applyMiddleware(...));
```

State 是只读的，唯一改变 state 的方法就是触发 action

```js
store.dispatch(action);
```

Store 相关方法

1. `store.getState()` 获取 state；
2. `store.dispatch(action)` 更新 state，返回值是那个被 dispatch 的 action；
3. `store.subscribe(listener)` 注册监听器;
4. `store.subscribe(listener)()` 返回的函数注销监听器。

### Action

描述发生的事件，也是一个对象，包括 type 和 'payload' 等字段，可以定义很多个 action。也可以用 Action 创建函数生成 action

```js
const action = { type: 'ADD_TODO', text: 'Go to swimming pool' };
```

Action creaters  
用来创建 action 对象的函数，很多时候我们并不会写一个个的对象作为我们的 action。而是用 action creaters 去生成 action。

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text,
  };
}
```

触发 action

```js
store.dispatch({ type: 'ADD_TODO', text: 'Go to swimming pool' });

store.dispatch(addTodo(text));
```

FSA，即 Flux Standard Action，一个用户友好的 Flux action 对象标准

1. 一个 action 必须是一个普通的 JavaScript 对象，有一个 type 字段。
2. 一个 action 可能有 error 字段、payload 字段、meta 字段。
3. 一个 action 必须不能包含除 type、payload、error 及 meta 以外的其他字段。

### Reducer

描述 action 如何改变 state ，就是一个函数，接收 action 和旧的 state 对象，改变 state 的值，并返回新的 state。

这个 state 并不是所有的 state 对象，而是需要修改的这个 state，这个 state 可以传入一个默认值，即为这个 state 的初始值。

```js
function reducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false,
        },
      ];
    default:
      return state;
  }
}
```

> 注意：不能直接修改旧 state，必须先拷贝一份 state，再进行修改。
>
> 1. `newState = JSON.parse(JSON.stringfy(state))`
> 2. 使用`Object.assign`函数生成新的 state
> 3. 使用扩展运算符`...state`

一个应用可以只有一个 Reducer，但随着应用的增大，我们不可能将所有 state 的修改逻辑全部写在一个方法里。  
这时我们就需要编写很多小的 reducer 函数来分别管理 state 的每一部分。  
但是创建 store 的时候，直接收一个 reducer,所以我们要把这些小的 reducer 组成一个总的 reducer。这个时候就要用到 combineReducers()方法。

```js
function reducer1(state, action) {
  switch(action.type) {
    case 'ADD_TEXT':
      return [...state, action.text];
    default
      return state;
  }
}

function reducer2(state, action) {
  switch(action.type) {
    case 'ADD_TODO':
      return [...state, action.todo];
    default
      return state;
  }
}

// ---
import { combineReducers } from 'redux'
const reducer = comebineReducer({ reducer1, reducer2 });
const store = createStore(reducer, preloadedState);
```

> 注意使用 comineReducer 后，如果传入 preloadedState，preloadedState 中的 key 必须与 combineReducers 中传入的对象的 key 一样。  
> 如果不传入这个值，所有 state 的初始值将使用在 reducer 中定义的默认值。

---

## Middleware

在 Express 或者 Koa 等框架中，middleware 是指可以被嵌入在框架接收请求到产生响应过程之中的代码。  
Redux middleware 提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。 主要用来自定义扩展 Redux。你可以利用 Redux middleware 来进行**日志记录、创建崩溃报告、调用异步接口或者路由**等等。

根据 Redux middleware API，每个 middleware 接受 Store 的 dispatch 和 getState 函数作为命名参数，并返回一个函数。

返回的函数接受一个 next 类型的参数（这个 next 是一个函数，如果调用了它，就代表着这个中间件完成了自己的职能，并将对 action 控制权交予下一个中间件），并且再返回一个函数，去操作 action。

它再返回的这个函数，才是真正处理 action 的函数，他接受 action 为参数，处理一些自定义逻辑，然后通过 next 方法再把 action 交给下一个中间件去处理。

所以，middleware 的函数签名是：

```js
({ getState, dispatch }) =>
  (next) =>
  (action) =>
    next(action);
```

调用链中最后一个 middleware 会接受真实的 store 的 dispatch 方法作为 next 方法，然后结束调用链。也就是在最后才真正的 dispatch action。

根据上面对 middleware 的解释来自定义一个 middleware。

```js
// 先定义一个什么也没干的middleware
const doNothingMiddleware = (dispatch, getState) => next => action => {
  //  to do
  return next(action);
}

// 自定义一个打印state信息的middleware，在真正的dispatch前后打印
// 我们已经知道，真正的dispath发生在最后的 next(action).所以我们要在这个语句前后打印。
const loggerMiddleware = (dispatch, getState) => next => action => {
  console.log('will dispatch', action)；// 先打印action
  const returnValue = next(action); // 调用下一个middleware的dipatch方法
  console.log('state after dispatch', getState())；
  return returnValue;
}
```

我们可以通过`applyMiddleware(...middleware)`来使用一些开源的或者自定义的 middleware。

```js
const store = createStore(todos, ['Use Redux'], applyMiddleware(loggerMiddleware));

store.dispatch({
  type: 'ADD_TODO',
  text: 'Understand the middleware',
});
```

---

## 异步 action

根据前面的描述，我们目前为止所有介绍的都是同步的写法，比如

```js
import { createStore } from 'redux';

// 创建reducer
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

// 生成store
let store = createStore(counter);

// 监听store的变化
store.subscribe(() => console.log(store.getState()));

// 触发store改变
store.dispatch({ type: 'INCREMENT' });
```

假如我们想要在 dispatch 一个 action 的时候，触发一个 http 请求，或者做一些异步的操作，应该怎么办呢？

有的同学可能会想到，在 reducer 中做这些异步不就行了，但是真的可以吗？  
根据 redux 的定义，首先 reducer 的返回值必须是 state，不能是 promise 之类的，所以如果我们在里面写了异步代码，返回 promise 之类的不会生效。  
其次 reducer 是纯函数，如果我们在里面写了异步的代码，并将结果反映在 state 中，那将不可预测，不符合纯函数。  
所以 reducer 中是不可以写异步的。

那如果我们想要执行异步怎么办呢？我们想到是否可以在 action creater 中去操作。

在 action creater 中去发 http 请求，当请求成功之后，才创建 action。  
想法很好，但是我们知道 redux 本身的 store.dispatch 只支持同步的 action 对象。所以我们必须要借助一个库实现这个功能，就是`redux-thunk`，它也是 Redux 推荐的实现异步 action 的标准方法。

### `redux-thunk`

Redux 默认的 action creaters 行为都是返回一个 action，但 Redux Thunk middleware 允许创建一个返回函数的 action creater，而不是返回 action。这就叫 thunk。

返回的函数接收 dispatch 和 getState 作为参数。

可以用来延迟 dispatch action，所以可以写一些异步代码。  
也可以 dispach 一个其他的 thunk。

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

const store = createStore(rootReducer, applyMiddleware(thunk));

function asyncActionCreator() {
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

store.dispatch(asyncActionCreator());
```

---

## 其他一些常用的 middleware

### `redux-promise` 或者 `redux-promise-middleware`，异步 action

redux-thunk 可以 dispatch 函数，而用 redux-promise 或者 redux-promise-middleware 可以 dispatch 一个 Promise

写法一，Action creator 返回值是一个 Promise 对象。可以 dispatch promise resovle 的 value，如果 promise reject，不会 dispacth 任何东西。

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

写法二，Action 对象的 payload 属性是一个 Promise 对象。这需要借助**redux-actions**模块引入 createAction 方法，生成 FSA

1. dispatch a copy of the action with the resolved value of the promise, and set status to success.
2. dispatch a copy of the action with the rejected value of the promise, and set status to error.

```js
import { createAction } from 'redux-actions';

store.dispatch(
  createAction(
    'FETCH_POSTS',
    fetch(`/some/API/${postTitle}.json`).then((response) => response.json())
  )
);
```

### `redux-actions`

创建标准的 action，以及处理 action

`createAction(type, payloadCreator, metaCreator)`: action 生成器，创建 action

1. 第一个参数：action 类型，必须参数
2. 第二个参数：生成 payload 的函数，可选参数。
3. 第三个参数：生成 meta 字段的函数，可选

`createActions` 创建多个 action

```js
const increment = createAction('INCREMENT');
const increment2 = createAction('INCREMENT', () => {
  return payload;
});
const { actionOne, actionTwo, actionThree } = createActions(
  {
    ACTION_ONE: (key, value) => ({ [key]: value }),
    ACTION_TWO: [
      (first) => [first], // payload
      (first, second) => ({ second }), // meta
    ],
  },
  'ACTION_THREE'
);
```

`handleAction(type, reducer, defaultState)`: 处理 action，相当于普通 reducer 的功能，返回值就是 reducer

1. 第一个参数：action 类型，必须参数
2. 第二个参数：真正的 reducer 函数，接收 state 和 action，返回新的 state，必须参数。
3. 第三个参数：state 的初始值，可选

`handleActions` 处理多个 action

```js
import { handleAction, handleActions } from 'redux-actions';
const reducer = handleAction(
  'ADDNUM',
  (state, action) => {
    let newState = { ...state };
    newState.n++;
    return newState;
  },
  defaultState
);

const reducer = handleActions(
  {
    INCREMENT: (state, action) => ({
      counter: state.counter + action.payload,
    }),

    DECREMENT: (state, action) => ({
      counter: state.counter - action.payload,
    }),
  },
  { counter: 0 }
);
```

`combineActions(...types)` 合并多个 action type，使他们公用一个 reducer

```js
const reducer = handleAction(
  combineActions(increment, decrement),
  (state, action) => {
    let newState = { ...state };
    newState.n++;
    return newState;
  },
  { counter: 10 }
);
```

### 用 redux-observable 来 dispatch Observable

### 用 redux-saga 中间件来创建更加复杂的异步 action

### 用 redux-pack 中间件 dispatch 基于 Promise 的异步 Action

### 用 redux-logger 中间件记录显示 action、state 的变化
