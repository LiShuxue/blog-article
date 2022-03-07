## 什么是虚拟DOM

虚拟dom就是一个普通的javascript对象，一般包含tag，props，children三个属性。
```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>
```
转化成虚拟dom
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

## 虚拟dom是如何工作的
* 当数据发生变化，比如setState时，会引起组件重新渲染，react会重新构建虚拟dom树(以该组件为根，重新渲染整个组件子树)。
* diff新dom树与旧dom树的差异。
* 最后将差异更新到真实的dom。

将虚拟DOM树转换成真实DOM树的最少操作的过程 称为 调和 。

## react diff 工作原理
diff算法的作用是计算出Virtual DOM中真正变化的部分，并只针对该部分进行原生DOM操作，而非重新渲染整个页面。

传统diff算法
通过循环递归对节点进行依次对比，算法复杂度达到 O(n^3)。为了解决这个问题，React中定义了三种策略，在对比时，根据策略只需遍历一次树就可以完成对比，将复杂度降到了O(n)
### diff 策略
1. tree diff: 在两个树对比时，只会比较同一层级的节点，会忽略掉跨层级的操作。
2. component diff: 拥有相同类的两个组件将会生成相似的树形结构， 拥有不同类的两个组件会生成不同的树形结构。
3. element diff: 对于同一层级的一组节点，可以通过唯一id来区分。
### tree diff
* 对树进行分层比较，两棵树只会对同一层次的节点进行比较。
* 如果该节点不存在时，则该节点及其子节点会被完全删除，不会再进一步比较。
* 只需遍历一次，就能完成整棵DOM树的比较。
* diff只简单考虑同层级的节点位置变换，如果是跨层级的话，只有创建节点和删除节点的操作(先创建再删除)。因此 官方建议不要进行DOM节点跨层级操作，可以通过CSS隐藏、显示节点，而不是真正地移除、添加DOM节点。
### component diff
* 同一类型的两个组件，按原策略（层级比较）继续比较Virtual DOM树即可。
* 同一类型的两个组件，组件A变化为组件B时，可能Virtual DOM没有任何变化，如果知道这点（变换的过程中，Virtual DOM没有改变），可节省大量计算时间，所以 用户 可以通过 shouldComponentUpdate() 来判断是否需要更新。
* 不同类型的组件，将一个（将被改变的）组件判断为dirty component（脏组件），从而替换 整个组件的所有节点。删除组件， 创建新组件。
### element diff
* 同一级节点比较时，diff提供三种节点操作：删除、插入、移动。同一节点有唯一的key。
* 删除： 直接删除节点
* 插入： 插入节点
* 移动： 根据key进行移动

## Vue 和 React 的 Diff 算法有哪些区别
相同点：
1. 都只对同级节点进行对比
2. 都用key做为标识

不同点：
1. 列表对比，react 采用单指针从左向右进行遍历，vue采用双指针，从两头向中间进行遍历。
2. 节点对比，当节点元素类型相同，但是className不同，vue认为是不同类型元素，删除重建，而react会认为是同类型节点，只是修改节点属性。

## vue和react的区别
1. 模板 与 JSX
2. 复用 mixin与HOC， hooks和setup
3. 通信方面 
    * 父子，vue, data down, event up. react props 和 callback
    * 同级，provide/inject, context
4. 数据绑定， vue双向，v-modal. react单向

## React 中 keys 的作用是什么？
key是react元素或组件的唯一标识，用于diff的时候，追踪元素或组件的删除，插入，移动等。

## 触发多次setstate，那么render会执行几次？
多次setState会合并为一次render，因为setState并不会立即改变state的值，而是将其放到一个任务队列里，最终将多个setState合并，一次性更新页面

## setState为什么异步？能不能同步？什么时候异步？什么时候同步？
* setState只在合成事件和钩子函数中是异步的，在原生事件与setTimeout中都是同步的
* setState本身的执行都是同步的，并不是由异步代码实现。只是因为合成事件和钩子函数的执行顺序在更新之前，所以合成事件和钩子函数中无法拿到更新后的值。形成了异步。可以通过setState的第二个callback参数拿到更新后的值。
* 在合成事件与钩子函数中会对多次setState进行更新优化，只执行最后一次；在原生事件与setTimeout内不会进行批量更新优化；

## React 中什么是合成事件
React 合成事件（SyntheticEvent）是 React 模拟原生 DOM 事件所有能力的一个事件对象。即浏览器原生事件的跨浏览器包装器。它根据 W3C 规范 来定义合成事件，兼容所有浏览器，拥有与浏览器原生事件相同的接口。

在 React 中，“合成事件”会以事件委托（Event Delegation）方式绑定在组件最上层，并在组件卸载（unmount）阶段自动销毁绑定的事件。

一般如果 DOM 上绑定了过多的事件处理函数，整个页面响应以及内存占用可能都会受到影响。React 为了避免这类 DOM 事件滥用，同时屏蔽底层不同浏览器之间的事件系统差异，实现了一个中间层——SyntheticEvent。

合成事件的一套机制：React 并不是将 click 事件直接绑定在 dom 上面，而是采用事件冒泡的形式冒泡到 documnet 上面，然后 React 将事件封装给正式的函数处理运行和处理

1. 当用户在为 onClick 添加函数时，React 并没有将 Click 事件绑定在 DOM 上面
2. 而是在 document 处监听所有支持的事件，当事件发生并冒泡至 document 处时，React 将事件内容封装交给中间层 SynthticEvent(负责所有事件合成)
3. 所以当事件触发的时候，对使用统一的分发函数 dispatchEvent 将指定函数执行

### 那么 React 为什么使用合成事件？其主要有三个目的：
1. 进行浏览器兼容，实现更好的跨平台

    React 采用的是顶层事件代理机制，能够保证冒泡一致性，可以跨浏览器执行。React 提供的合成事件用来抹平不同浏览器事件对象之间的差异，将不同平台事件模拟合成事件。

2. 避免垃圾回收

    事件对象可能会被频繁创建和回收，因此 React 引入事件池，在事件池中获取或释放事件对象。即 React 事件对象不会被释放掉，而是存放进一个数组中，当事件触发，就从这个数组中弹出，避免频繁地去创建和销毁(垃圾回收)。

3. 方便事件统一管理和事务机制

## 类组件和函数组件之间的区别是啥（不考虑hook）？
1. 类组件有state和生命周期， 函数式组件又叫无状态组件， 没有state和生命周期。
2. 类组件使用时需要先实例化， 函数式组件不能实例化，他是一个函数， 传参， 执行，返回结果。
3. 无状态组件由于没有实例化过程，所以无法访问组件this中的对象。

## hooks
hooks为函数组件提供了状态，也支持在函数组件中获取数据等。
* 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
* 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。
### useState
在函数式组件中使用state
### useEffect
* 执行副作用操作，比如请求数据， 绑定解绑事件，设置取消定时器等。
* 默认第一次渲染和每次更新都会执行useEffect
* 如果想只第一次渲染执行，依赖项传空数组。数组不为空时，将会依赖项变化后执行。
* 可以返回一个函数来清楚副作用。将会在组件卸载时执行。
### useCallback
用于缓存函数，如果依赖项发生了变化，该函数才发生变化。一般用于将此函数传入子组件中，因为默认每次父组件更新，传入子组件的该函数都不一样。配合React.memo()一起使用
### useMemo
用于缓存函数返回值。如果依赖项发生变化，该函数才重新执行，计算返回值。一般用于该函数比较复杂，开销较大。避免每次更新此函数都重新执行。
### useRef
* ref 用来获取类组件实例，或者DOM对象
* useRef 用来创建ref对象
* 通过 ref={myRef} 绑定在DOM 对象或者React组件上
* 通过 this.myRef.current 访问DOM对象或者class组件的实例
* useRef()可以用来保存一个不变的值。简单来说，我们可以将useRef返回值看作一个组件内部全局共享变量，它会在多次渲染内部共享一个相同的值。

## 如何创建 refs
* 类组件中： `this.myRef = React.createRef();`
* 函数式组件中： `const ref = useRef();`
* 或者在类组件中通过回调函数： `<input type='text' ref={(input) => this.input = input} />`

## 用React hook实现一个计数器组件
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

## 如何实现一个自定义hook，useEventListener
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
    }
  }, [eventName, dom])
}
```

## useEffect中如何使用async/await
在useEffect内部或者外部，重新封装一个async函数，在async函数内部使用await。在useEffect里面调用这个async函数。
```js
async function fetchMyAPI() {
  let response = await fetch("api/data");
}

useEffect(() => {
  fetchMyAPI();
}, []);

// 或者
useEffect(() => {
  const fetchMyAPI = async () => {
    let response = await fetch("api/data");
  }
  fetchMyAPI();
}, []);
```

## react hooks 的原理是什么
闭包

## state 和 props 的区别是什么
* props和state都是普通的 JS 对象
* state 是组件内部状态，可变
* props 是外部传入的参数， 不可变

## 什么是受控组件
在html中，表单元素如input, textarea，通常自己维护自己的状态，由用户输入自动更新。他们都是有自己的value的，这些元素是可以被直接设置value属性值的。 

但在React中，数据是单向的，所以不应该直接去修改这种组件的属性值，而应该用组件内部的state中的值。对于这种遵从React规定，使用state为唯一数据源，被 React 控制取值赋值的表单元素，我们称为受控组件。

* state中包含这个表单元素的value值
* 这个表单元素的value值要取自state中的value
* 元素value改变时，要赋值给state中的value，通过setState
* 通过e.target.value获取元素的value
```html
<input
    type="text"
    value={this.state.value}
    onChange={(e) => {
        this.setState({
            value: e.target.value.toUpperCase()
        })
    }}
/>
```

## 什么是jsx
jsx将html和js混在一起编写，需要通过babel和webpack编译来转化成js。
* 在html中可以写任何的js
* js变量或者js表达式，语句，要用大括号{}包裹
* html标签的属性值可以直接用双引号包裹字符串，也可以用大括号包裹js表达式
* html标签的属性名使用驼峰命名，如html 的class，在JSX中是`className`
* 如果标签没有内容， 可以用 `/>` 来闭合标签
* 事件的命名也使用驼峰命名，而不是纯小写。如html中的onclick，在JSX中是onClick
* 事件处理应该传入一个函数作为事件处理函数，而不是一个字符串
* JSX 可以赋值给变量，也可以把 JSX 当作参数传入，以及从函数中返回 JSX
* 组件名称必须以大写字母开头，React会将以小写字母开头的组件视为原生DOM标签
* 注释也要用{}包裹

## 如何避免组件的重新渲染？
* React.memo(): 这可以防止不必要地重新渲染函数组件。对子组件默认使用浅比较对比前后两次 props 的变更，若未发生变更则不会重新渲染。
* PureComponent: 这可以防止不必要地重新渲染类组件。对子组件默认使用浅比较对比前后两次 props 的变更。

## 为什么要给所有的实例方法绑定this呢？
```js
class ToggleButton extends React.Component{
    constructor(props){
        super(props);
        this.state = {isToggleOn: true};
        this.handleClick = this.handleClick.bind(this); 
    }
    handleClick(){
        this.setState(prevState => ({
            isToggleOn: !preveState.isToggleOn
        }));
    }
    render(){
        return(
          <button onClick={this.handleClick}>
              {this.state.isToggleOn ? 'ON':'OFF'}
          </button>
        )
    }
}
```
为了提前规避this指针丢失的问题，不然在程序执行过程中this就可能会指向window而不是该实例。

可以使用箭头函数， 他会默认指向上下文。

React在事件发生时调用onClick，由于onClick只是中间变量，如果不绑定this, 所以处理函数中的this指向会丢失。其实真正调用时并不是this.handleClick(),如果是这样调用那么this指向就不会有问题。真正调用的是onClick()，而onclick是dom事件，并不是类中的方法，简单可以理解为是在类外面的调用，此时的this其实指向的是全局作用域，而这个作用域下并没有setstate方法，所以自然而然地报undefined错误。

## react组件通信
### 父组件向子组件通信
父组件通过向子组件传递 props，子组件得到 props 后进行相应的处理。
### 子组件向父组件通信
父组件将一个函数作为 props 传递给子组件，子组件调用该回调函数，便可以向父组件通信。
### 跨级组件通信
使用context
### 兄弟组件
1. 利用二者共同父组件的 context 对象进行通信
2. 自定义event bus
3. redux

## react性能优化
* render里面尽量减少新建变量和bind函数，传递参数时尽量减少传递参数的数量。
* 定制shouldComponentUpdate函数
* React.memo 和 React.PureComponent
* key不要用index
* useMemo, useCallback

## 组件插槽
使用props.children
```js
// 父组件中
<Parent>
    <h1>this is children</h1>
</Parent>

function Parent(props) {
    return (
        <div>
            <h1>below is children</h1>
            {props.children}
        </div>
    );
}
```

## React组件生命周期
### 挂载阶段
constructor()   
static getDerivedStateFromProps() 从props中获取state，将传入的props映射到state  
render()   
componentDidMount()
### 更新阶段
static getDerivedStateFromProps()  从props中获取state，将传入的props映射到state 
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
React.memo默认情况下其只会对复杂对象做浅层对比，如果你想要控制对比过程，那么请将自定义的比较函数通过第二个参数传入来实现。
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

## 代码分割, 组件懒加载
import()， React.lazy()

## 怎么开发错误边界组件
如果一个 class 组件中定义了 static getDerivedStateFromError() 或 componentDidCatch() 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界组件。

## 使用 PropTypes 进行类型检查
```js
Component.propTypes = {
  name: PropTypes.string
};
```

## 如何创建，使用Context
### 创建
```jsx
const initValue = {a: 'test'}
const MyContext = React.createContext(a)
```
### 使用
```jsx
// 父组件中
<MyContext.Provider value={initValue}>
  <Son />
</MyContext.Provider>

// 后代组件中， hook写法
const initValue = useContext(MyContext);

// Class组件中使用
Son.contextType = MyContext // 先挂载在类上
this.context // 组件中就可以使用
```

## render prop
render prop组件， 将渲染逻辑改成方法传入， 而不是写死， 可以做到动态渲染。
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

## 什么是高阶组件HOC
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
* 代码重用、逻辑和引导抽象
* 渲染劫持
* state 抽象和操作
* props 处理

## React 中各种组件复用的优劣势（mixin、render props、hoc、hook）
mixins: 变量来源不清、属性重名、Mixins引入过多会导致顺序冲突

HOC：优点：提取公共逻辑，降低耦合度。缺点：组件嵌套过多，调试溯源不清晰，state、内部方法相同的话会进行覆盖。

Render props：优点：清楚知道这个state来自哪里。缺点：使用起来会嵌套地狱。不能在return外使用数据。

hook: Hooks 就是让你不必写class组件就可以用state和其他的React特性；也可以编写自己的hooks在不同的组件之间复用。代码量少，复用性高，易拆分，也有一些闭包的坑

## React 路由有3种渲染方式——render，children，component，到底用哪一个？用任何一个都可以吗？
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
<Route path="path" children={ props => (
  <div className={props.match? "active": ''}>
    <Link to="path" />
  </div>
) />
```
### component
只有在当 location 匹配时才渲染
```js
<Route path="path" component={Comp} />
```

但当他们同时存在时，优先渲染component的值，其次是render属性的值，而children属性的值优先级最低，为了避免 不必要的错误，尽量每个Route中只是用他们三个中的其中一个。
## React 的 Fiber 是干什么的
因为 JavaScript 单线程的特点，每个同步任务不能耗时太长，不然就会让程序不会对其他输入作出相应，React 的更新过程就是犯了这个禁忌，而 React Fiber 就是要改变现状。 而可以通过分片来破解 JavaScript 中同步操作时间过长的问题。

把一个耗时长的任务分成很多小片，每一个小片的运行时间很短，虽然总时间依然很长，但是在每个小片执行完之后，都给其他任务一个执行的机会，这样唯一的线程就不会被独占，其他任务依然有运行的机会。

React Fiber 把更新过程碎片化，每执行完一段更新过程，就把控制权交还给 React 负责任务协调的模块，看看有没有其他紧急任务要做，如果没有就继续去更新，如果有紧急任务，那就去做紧急任务。

维护每一个分片的数据结构，就是 Fiber。

## React 性能优化
* Code Splitting
* shouldComponentUpdate 避免重复渲染
* 异步按需加载，路由懒加载，路由监听器
* 组件尽可能的进行拆分、解耦
* 列表类组件优化，
比如：shouldComponentUpdate(nextProps, nextState)：React.PureComponent、Immutable，React.memo
* bind 函数优化
* 非可控组件和可控组件区别是否受 state 影响
* ReactDOMServer 进行服务端渲染组件

