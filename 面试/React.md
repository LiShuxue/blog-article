<https://juejin.cn/post/7258071726227849277>

## 什么是虚拟 DOM

虚拟 dom 就是一个普通的 javascript 对象，一般包含 tag，props，children 三个属性。

```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>
```

转化成虚拟 dom

```js
{
  tag: 'div',
  props: {
    id: 'app'
  },
  children: [
    {
      tag: 'p',
      props: {
        className: 'text'
      }
      children: 'hello world!!!'
    }
  ]
}
```

## 虚拟 dom 是如何工作的

- 当数据发生变化，比如 setState 时，会引起组件重新渲染，react 会重新构建虚拟 dom 树(以该组件为根，重新渲染整个组件子树)。
- diff 新 dom 树与旧 dom 树的差异。
- 最后将差异更新到真实的 dom。

将虚拟 DOM 树转换成真实 DOM 树的最少操作的过程 称为 调和 。

## react diff 工作原理

diff 算法的作用是计算出 Virtual DOM 中真正变化的部分，并只针对该部分进行原生 DOM 操作，而非重新渲染整个页面。

传统 diff 算法
通过循环递归对节点进行依次对比，算法复杂度达到 O(n^3)。为了解决这个问题，React 中定义了三种策略，在对比时，根据策略只需遍历一次树就可以完成对比，将复杂度降到了 O(n)

### diff 策略

1. component diff: 拥有相同类的两个组件将会生成相似的树形结构， 拥有不同类的两个组件会生成不同的树形结构。
1. tree diff: 在两个树对比时，只会比较同一层级的节点，会忽略掉跨层级的操作。
1. element diff: 对于同一层级的一组节点，可以通过唯一 id 来区分。

### component diff

- 同一类型的两个组件，按原策略（层级比较）继续比较 Virtual DOM 树即可。
- 同一类型的两个组件，组件 A 变化为组件 B 时，可能 Virtual DOM 没有任何变化，如果知道这点（变换的过程中，Virtual DOM 没有改变），可节省大量计算时间，所以 用户 可以通过 shouldComponentUpdate() 来判断是否需要更新。
- 不同类型的组件，将一个（将被改变的）组件判断为 dirty component（脏组件），从而替换 整个组件的所有节点。删除组件， 创建新组件。

### tree diff

- 对树进行分层比较，两棵树只会对同一层次的节点进行比较。
- 如果该节点不存在时，则该节点及其子节点会被完全删除，不会再进一步比较。
- 只需遍历一次，就能完成整棵 DOM 树的比较。
- diff 只简单考虑同层级的节点位置变换，如果是跨层级的话，只有创建节点和删除节点的操作(先创建再删除)。因此 官方建议不要进行 DOM 节点跨层级操作，可以通过 CSS 隐藏、显示节点，而不是真正地移除、添加 DOM 节点。

### element diff

- 同一级节点比较时，diff 提供三种节点操作：删除、插入、移动。同一节点有唯一的 key。
- 删除： 直接删除节点
- 插入： 插入节点
- 移动： 根据 key 进行移动

## Vue 和 React 的 Diff 算法有哪些区别

相同点：

1. 都只对同级节点进行对比
2. 都用 key 做为标识

不同点：

1. 列表对比，react 采用单指针从左向右进行遍历，vue 采用双指针，从两头向中间进行遍历。
2. 节点对比，当节点元素类型相同，但是 className 不同，vue 认为是不同类型元素，删除重建，而 react 会认为是同类型节点，只是修改节点属性。

## vue 和 react 的区别

1. 模板 与 JSX
2. 复用 mixin 与 HOC， hooks 和 setup
3. 通信方面
   - 父子，vue, data down, event up. react props 和 callback
   - 同级，provide/inject, context
4. 数据绑定， vue 双向，v-modal。 react 单向

## React 中 keys 的作用是什么？

key 是 react 元素或组件的唯一标识，用于 diff 的时候，追踪元素或组件的删除，插入，移动等。

## React 中什么是合成事件

React 合成事件是模拟原生 DOM 事件所有能力的一个事件对象。兼容所有浏览器，拥有与浏览器原生事件相同的接口。

“合成事件”会以事件委托（Event Delegation）方式绑定在组件最上层，并在组件卸载（unmount）阶段自动销毁绑定的事件。

一般如果 DOM 上绑定了过多的事件处理函数，整个页面响应以及内存占用可能都会受到影响。React 为了避免这类 DOM 事件滥用，同时屏蔽底层不同浏览器之间的事件系统差异，实现了一个中间层——SyntheticEvent。

合成事件的一套机制：React 并不是将 click 事件直接绑定在 dom 上面，而是采用事件冒泡的形式冒泡到 documnet 上面，然后 React 将事件封装给正式的函数处理运行和处理

1. 当用户在为 onClick 添加函数时，React 并没有将 Click 事件绑定在 DOM 上面
2. 而是在 document 处监听所有支持的事件，当事件发生并冒泡至 document 处时，React 将事件内容封装交给中间层 SynthticEvent(负责所有事件合成)
3. 所以当事件触发的时候，对使用统一的分发函数 dispatchEvent 将指定函数执行

<https://segmentfault.com/a/1190000039108951>

### 那么 React 为什么使用合成事件？其主要有三个目的

1. 进行浏览器兼容，实现更好的跨平台

   React 采用的是顶层事件代理机制，能够保证冒泡一致性，可以跨浏览器执行。React 提供的合成事件用来抹平不同浏览器事件对象之间的差异，将不同平台事件模拟合成事件。

2. 避免垃圾回收

   事件对象可能会被频繁创建和回收，因此 React 引入事件池，在事件池中获取或释放事件对象。即 React 事件对象不会被释放掉，而是存放进一个数组中，当事件触发，就从这个数组中弹出，避免频繁地去创建和销毁(垃圾回收)。

3. 方便事件统一管理和事务机制

## 类组件和函数组件之间的区别是啥（不考虑 hook）？

1. 类组件有 state 和生命周期， 函数式组件又叫无状态组件， 没有 state 和生命周期。
2. 类组件使用时需要先实例化， 函数式组件不能实例化，他是一个函数， 传参， 执行，返回结果。
3. 无状态组件由于没有实例化过程，所以无法访问组件 this 中的对象。

## hooks

hooks 为函数组件提供了状态，也支持在函数组件中获取数据等。

- 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
- 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。

### useState

在函数式组件中使用 state。

React 会在重复渲染时保留这个 state，也就是说后续渲染时，useState 返回的第一个值将始终是更新后最新的 state

### useEffect

- 执行副作用操作，比如请求数据， 绑定解绑事件，设置取消定时器等。
- 默认第一次渲染和每次更新都会执行 useEffect
- 如果想只第一次渲染执行，依赖项传空数组。数组不为空时，将会依赖项变化后执行。
- 可以返回一个函数来清楚副作用。将会在组件卸载时执行。

我们可能会习惯性的把 useEffect 当成一个 watch 来用，但每次我们 setState 过后，函数组件又会重新执行一遍，useEffect 也会重新跑一遍。所以对于这种，直接可以将 useEffect 内部的的计算放到组件外层执行。

### useCallback

用于缓存函数，如果依赖项发生了变化，该函数才发生变化。使用场景：

1. 我们知道 react 在父组件更新的时候，会对子组件进行全量的更新，我们可以使用 memo 对子组件进行缓存，在更新的时候浅层的比较一下 props，如果 props 没有变化，就不会更新子组件，那如果 props 中有函数，我们就需要使用 useCallback 缓存一下这个父组件传给子组件的函数。

2. 我们的 useEffect 中可能会有依赖函数的场景，这个时候就需要使用 useCallback 缓存一下函数，避免 useEffect 的无限调用

3. 我们在自定义 hooks 需要返回函数的时候，建议使用 useCallback 缓存一下，因为我们不知道用户拿返回的函数去干什么，万一加到 useEffect 的依赖里面就有问题了。

### useMemo

用于缓存函数返回值。如果依赖项发生变化，该函数才重新执行，计算返回值。一般用于该函数比较复杂，开销较大。避免每次更新此函数都重新执行。

### useRef

- useRef 用来创建 ref 对象
- 通过 ref={myRef} 绑定在 html 标签或者 React 组件上（组件需要配合 forwardRef 使用），来获取 DOM 对象
- useRef()也可以用来保存一个不变的值。简单来说，我们可以将 useRef 返回值看作一个组件内部全局共享变量，它会在多次渲染内部共享一个相同的值。

### useContext

用来在多个组件中共享数据。

1. 使用 createContext 来创建 Context
2. 使用 Context.Provider 来提供数据
3. 使用 useContext 来获取数据

### useReducer

useReducer 可以替代 useState，多一个页面使用很多 useState 时，而一些状态之间也是有关联的，可以用 useRedcuer 来重构。

1、定义 reducer

```js
const reducer = (state, action) => {
  switch (action.type) {
    case 'xxx':
      return { ...state, ...action.payload };
    case 'yyy':
      return { ...state, yyy };
  }
};
```

2、使用 useReducer 来初始化 state 和改变 state 的方法

```jsx
const initState = { a: 'a', b: 'b' };
const [state, dispatch] = useReducer(reducer, initState);
```

3、使用 state 和 dispatch

```js
// state可以直接解构
const [{ a, b }, dispatch] = useReducer(reducer, initState);

// 改变state
const changeState = () => {
  dispatch({ type: 'xxx', payload: { test: 'test' } });
};
<div onClick={changeState}>{a}</div>;
```

## hooks 为什么不能用在循环或条件中

hooks 是在函数式组件中引申出来的，函数本身不能保存状态，为了在函数式组件中有状态，我们需要额外维护一个有序的链表，在初始化 useState 之类的 hook 时，将它们保存到这个链表里，之后 setState 也是更新这个表。

比如第一次执行函数组件时，我们用 useState 设置状态 count（初始值为 0 ）和 isDone（初始值为 false），它们其实被保存到一个有序链表中，它们的值会记录下来： [0, false] 。

我们执行 setIsDone(true)来更新 state 的值，则表中数据变为[0,true]，同时导致组件重新渲染。

第二次重新渲染时，useState 会 按顺序 从这个表中拿出 0 和 true，赋值给 count 和 isDone。

如果你把 hook 写到判断条件下，初始化表的时候顺序会有问题，后续执行会出现问题。所以要求每次函数组件的 hook 执行的位置相同，数量正确，否则会导致错位，不能拿到预期的状态值。

## 如何创建 ref

- 类组件中： `this.myRef = React.createRef();`
- 函数式组件中： `const ref = useRef();`

## 用 React hook 实现一个计数器组件

```js
fucntion Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count => count+1)
    }, 1000)
  },[])
  return () => {
    clearInterval(timer)
  }
}
```

## 如何实现一个自定义 hook，useEventListener

```js
function useEventListener(eventName, handler, dom) {
  const savedHandler = useRef();

  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);

  useEffect(() => {
    const eventListener = (event) => savedHandler.current(event);
    dom.addEventLister(eventName, eventListener);
    return () => {
      dom.removeEventListener(eventName, eventListener);
    };
  }, [eventName, dom]);
}
```

## useEffect 中如何使用 async/await

在 useEffect 内部或者外部，重新封装一个 async 函数，在 async 函数内部使用 await。在 useEffect 里面调用这个 async 函数。不能直接在 useEffect 的函数上面加 async。

```js
async function fetchMyAPI() {
  let response = await fetch('api/data');
}

useEffect(() => {
  fetchMyAPI();
}, []);

// 或者
useEffect(() => {
  const fetchMyAPI = async () => {
    let response = await fetch('api/data');
  };
  fetchMyAPI();
}, []);
```

## react hooks 的原理是什么

闭包

## state 和 props 的区别是什么

- props 和 state 都是普通的 JS 对象
- state 是组件内部状态，可变
- props 是外部传入的参数， 不可变

## 什么是受控组件

在 html 中，表单元素如 input, textarea，通常自己维护自己的状态，由用户输入自动更新。他们都是有自己的 value 的，这些元素是可以被直接设置 value 属性值的。

但在 React 中，数据是单向的，所以不应该直接去修改这种组件的属性值，而应该用组件内部的 state 中的值。对于这种遵从 React 规定，使用 state 为唯一数据源，被 React 控制取值赋值的表单元素，我们称为受控组件。

- state 中包含这个表单元素的 value 值
- 这个表单元素的 value 值要取自 state 中的 value
- 元素 value 改变时，要赋值给 state 中的 value，通过 setState
- 通过 e.target.value 获取元素的 value

```html
<input type="text" value={this.state.value} onChange={(e) => { this.setState({ value:
e.target.value.toUpperCase() }) }} />
```

## 什么是 jsx

jsx 将 html 和 js 混在一起编写，需要通过 babel 和 webpack 编译来转化成 js。

- 在 html 中可以写任何的 js
- js 变量或者 js 表达式，语句，要用大括号{}包裹
- html 标签的属性值可以直接用双引号包裹字符串，也可以用大括号包裹 js 表达式
- html 标签的属性名使用驼峰命名，如 html 的 class，在 JSX 中是`className`
- 如果标签没有内容， 可以用 `/>` 来闭合标签
- 事件的命名也使用驼峰命名，而不是纯小写。如 html 中的 onclick，在 JSX 中是 onClick
- 事件处理应该传入一个函数作为事件处理函数，而不是一个字符串
- JSX 可以赋值给变量，也可以把 JSX 当作参数传入，以及从函数中返回 JSX
- 组件名称必须以大写字母开头，React 会将以小写字母开头的组件视为原生 DOM 标签
- 注释也要用{}包裹

## 组件什么时候会重新渲染

1. 组件内部 state 变化，组件重新渲染，函数组件重新开始执行，但是 state 保留的最新的。
2. 父组件渲染，子组件也会重新渲染
3. Context.Provider 提供的 value 变化时，所有使用这个 value 的都会重新渲染。
4. 组件的 props 变化。其实 props 是由父组件传下去的，父组件重新渲染，子组件肯定也会渲染，如果不想子组件渲染，可以用 memo 包裹。

## 如何避免组件的重新渲染？

- React.memo(): 这可以防止不必要地重新渲染函数组件。对子组件默认使用浅比较对比前后两次 props 的变更，若未发生变更则不会重新渲染。
- PureComponent: 这可以防止不必要地重新渲染类组件。对子组件默认使用浅比较对比前后两次 props 的变更。

## 为什么要给所有的实例方法绑定 this 呢？

```js
class ToggleButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isToggleOn: true };
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    this.setState((prevState) => ({
      isToggleOn: !preveState.isToggleOn,
    }));
  }
  render() {
    return <button onClick={this.handleClick}>{this.state.isToggleOn ? 'ON' : 'OFF'}</button>;
  }
}
```

为了提前规避 this 指针丢失的问题，不然在程序执行过程中 this 就可能会指向 window 而不是该实例。

可以使用箭头函数， 他会默认指向上下文。

React 在事件发生时调用 onClick，由于 onClick 只是中间变量，如果不绑定 this, 所以处理函数中的 this 指向会丢失。其实真正调用时并不是 this.handleClick(),如果是这样调用那么 this 指向就不会有问题。真正调用的是 onClick()，而 onclick 是 dom 事件，并不是类中的方法，简单可以理解为是在类外面的调用，此时的 this 其实指向的是全局作用域，而这个作用域下并没有 setstate 方法，所以自然而然地报 undefined 错误。

## react 组件通信

### 父组件向子组件通信

父组件通过向子组件传递 props，子组件得到 props 后进行相应的处理。

### 子组件向父组件通信

父组件将一个函数作为 props 传递给子组件，子组件调用该回调函数，便可以向父组件通信。

### 跨级组件通信

使用 context

### 兄弟组件

1. 利用二者共同父组件的 context 对象进行通信
2. 自定义 event bus
3. redux

## react 性能优化

- render 里面尽量减少新建变量和 bind 函数，传递参数时尽量减少传递参数的数量。
- 定制 shouldComponentUpdate 函数
- React.memo 和 React.PureComponent
- key 不要用 index
- useMemo, useCallback

## 组件插槽

使用 props.children

```js
// 父组件中
<Parent>
  <h1>this is children</h1>
</Parent>;

function Parent(props) {
  return (
    <div>
      <h1>below is children</h1>
      {props.children}
    </div>
  );
}
```

## React 组件生命周期

### 挂载阶段

constructor()  
static getDerivedStateFromProps() 从 props 中获取 state，将传入的 props 映射到 state  
render()  
componentDidMount()

### 更新阶段

static getDerivedStateFromProps() 从 props 中获取 state，将传入的 props 映射到 state
shouldComponentUpdate()  
render()  
getSnapshotBeforeUpdate()  
componentDidUpdate()

### 卸载

componentWillUnmount()

### 错误处理

static getDerivedStateFromError()  
componentDidCatch()

## 如何在 hooks 中实现 shouldComponentUpdate 这个生命周期

React.memo 默认情况下其只会对复杂对象做浅层对比，如果你想要控制对比过程，那么请将自定义的比较函数通过第二个参数传入来实现。

```js
function MyComponent(props) {
  /* 使用 props 渲染 */
}
function areEqual(prevProps, nextProps) {
  /*
  如果把 nextProps 传入 render 方法的返回结果与
  将 prevProps 传入 render 方法的返回结果一致则返回 true，
  否则返回 false
  */
}
export default React.memo(MyComponent, areEqual);
```

## 触发多次 setstate，那么 render 会执行几次？

多次 setState 会合并为一次 render，因为 setState 并不会立即改变 state 的值，而是将其放到一个任务队列里，最终将多个 setState 合并，一次性更新页面

## setState 为什么异步？能不能同步？什么时候异步？什么时候同步？

- setState 只在合成事件和钩子函数中是异步的，在原生事件与 setTimeout 中都是同步的
- setState 本身的执行都是同步的，并不是由异步代码实现。只是因为合成事件和钩子函数的执行顺序在更新之前，所以合成事件和钩子函数中无法拿到更新后的值。形成了异步。可以通过 setState 的第二个 callback 参数拿到更新后的值。
- 在合成事件与钩子函数中会对多次 setState 进行更新优化，只执行最后一次；在原生事件与 setTimeout 内不会进行批量更新优化；

## 代码分割, 组件懒加载

import()， React.lazy()

## 怎么开发错误边界组件

如果一个 class 组件中定义了 static getDerivedStateFromError() 或 componentDidCatch() 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界组件。

## 使用 PropTypes 进行类型检查

```js
Component.propTypes = {
  name: PropTypes.string,
};
```

## 如何创建，使用 Context

### 创建

```jsx
const initValue = { a: 'test' };
const MyContext = React.createContext(a);
```

### 使用

```jsx
// 父组件中
<MyContext.Provider value={initValue}>
  <Son />
</MyContext.Provider>;

// 后代组件中， hook写法
const initValue = useContext(MyContext);

// Class组件中使用
Son.contextType = MyContext; // 先挂载在类上
this.context; // 组件中就可以使用
```

## render prop

render prop 组件， 将渲染逻辑改成方法传入， 而不是写死， 可以做到动态渲染。

```jsx
// 创建 render prop组件
render() {
  return(
    <div>
      {this.props.render(this.state)}
    </div>
  )
}

// 使用
<MyComponent render={(data) => {
  <h1>Hello {data}</h1>
}} />
```

## 什么是高阶组件 HOC

高阶组件(HOC)是接受一个组件并返回一个新组件的函数。基本上，这是一个模式，是从 React 的组合特性中衍生出来的，称其为纯组件，因为它们可以接受任何动态提供的子组件，但不会修改或复制输入组件中的任何行为。

```js
function higherOrderComponent(wrappedComponent) {
  return class HOCComponent extends Component {
    static displayName = `HOC(${wrappedComponent.displayName})`

    constructor(props){
      super(props)
      // 操作state
      this.state = {
        name:'alien'
      }
     }

    // 操作props
    let newProps = { ...this.props, title: 'new title'}

    return <WrappedComponent {...newProps} { ...this.state } />;
  }
}
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

## 高阶组件可以用来做什么

- 代码重用、逻辑和引导抽象
- 渲染劫持
- state 抽象和操作
- props 处理

## React 中各种组件复用的优劣势（mixin、render props、hoc、hook）

mixins: 变量来源不清、属性重名、Mixins 引入过多会导致顺序冲突

HOC：优点：提取公共逻辑，降低耦合度。缺点：组件嵌套过多，调试溯源不清晰，state、内部方法相同的话会进行覆盖。

Render props：优点：清楚知道这个 state 来自哪里。缺点：使用起来会嵌套地狱。不能在 return 外使用数据。

hook: Hooks 就是让你不必写 class 组件就可以用 state 和其他的 React 特性；也可以编写自己的 hooks 在不同的组件之间复用。代码量少，复用性高，易拆分，也有一些闭包的坑

## React 路由有 3 种渲染方式——render，children，component，到底用哪一个？用任何一个都可以吗？

### render

给 render 传递的是一个函数，它和 component 一样，在函数的参数中可以访问到所有的 route props

```js
//内联方式
<Route path="path" render={() => <div>这是内联组件写法</div>} />
//嵌套组合方式
<Route path="path" render={ props => (
  <ParentComp>
    <Comp {...props} />
  </ParentComp>
) />
```

### children

无论 location 是否匹配路由，你都需要渲染一些内容，这时候你可以使用 children。

```js
<Route
  path="path"
  children={(props) => (
    <div className={props.match ? 'active' : ''}>
      <Link to="path" />
    </div>
  )}
/>
```

### component

只有在当 location 匹配时才渲染

```js
<Route path="path" component={Comp} />
```

但当他们同时存在时，优先渲染 component 的值，其次是 render 属性的值，而 children 属性的值优先级最低，为了避免 不必要的错误，尽量每个 Route 中只是用他们三个中的其中一个。

## React 的 Fiber 是干什么的

因为 JavaScript 单线程的特点，每个同步任务不能耗时太长，不然就会让程序不会对其他输入作出相应，React 的更新过程就是犯了这个禁忌，而 React Fiber 就是要改变现状。 而可以通过分片来破解 JavaScript 中同步操作时间过长的问题。

把一个耗时长的任务分成很多小片，每一个小片的运行时间很短，虽然总时间依然很长，但是在每个小片执行完之后，都给其他任务一个执行的机会，这样唯一的线程就不会被独占，其他任务依然有运行的机会。

React Fiber 把更新过程碎片化，每执行完一段更新过程，就把控制权交还给 React 负责任务协调的模块，看看有没有其他紧急任务要做，如果没有就继续去更新，如果有紧急任务，那就去做紧急任务。

维护每一个分片的数据结构，就是 Fiber。

## React 性能优化

- Code Splitting
- shouldComponentUpdate 避免重复渲染
- 异步按需加载，路由懒加载，路由监听器
- 组件尽可能的进行拆分、解耦
- 列表类组件优化，
  比如：shouldComponentUpdate(nextProps, nextState)：React.PureComponent、Immutable，React.memo
- bind 函数优化
- 非可控组件和可控组件区别是否受 state 影响
- ReactDOMServer 进行服务端渲染组件
