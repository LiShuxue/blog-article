之前我们介绍了在React中如何使用Redux，需要借助‘react-redux’库来实现，通过Provider提供一个store，然后用connect()高阶组件，来在组件中使用state和dispatch action。

这篇文章主要讲如何不用上述写法，而是使用Hooks来实现。这些hooks最初在react-redux v7.1.0中添加。

## 提供Store
首先跟之前一样，还是需要先给整个组件树提供store.
```js
const store = createStore(rootReducer)

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

## useSelector()
`useSelector(selector, equalityFn?)` 从store中提取state。
1. 参数一selector是一个纯函数，接收整个store state，返回可以是任意值。
2. 这样就会对一个state进行订阅，当他更新的时候，useSelector会执行。
3. 当一个action被dispatch后，useSelector ()将对前一个选择器结果值和当前结果值进行引用比较（===比较）。如果它们是不同的，组件将被迫重新渲染。如果它们是相同的，组件将不会重新呈现。
4. 第二个参数提供一个比较函数，来确定是否需要重新渲染。
```js
import React from 'react'
import { useSelector } from 'react-redux'

export const CounterComponent = () => {
  const counter = useSelector(state => state.counter)
  return <div>{counter}</div>
}
```

## useDispatch()
返回dispatch函数，你可以在组件中使用，去dispatch action
```js
import React from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()

  return (
    <div>
      <button onClick={() => dispatch({ type: 'increment' })}>
        Increment counter
      </button>
    </div>
  )
}
```
