## 什么是虚拟DOM

虚拟dom就是一个普通的javascript对象，一般包含type，props，children三个属性。
```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>
```
转化成虚拟dom
```js
{
  type: 'div',
  props: {
    id: 'app'
  },
  children: [
    {
      type: 'p',
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

## diff 原理
### diff 策略
1. tree diff: web ui中DOM节点跨层级的移动操作特别少，可以忽略不计。
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

## React 中 keys 的作用是什么？
key是react元素或组件的唯一标识，用于diff的时候，追踪元素或组件的删除，插入，移动等。

## 触发多次setstate，那么render会执行几次？
多次setState会合并为一次render，因为setState并不会立即改变state的值，而是将其放到一个任务队列里，最终将多个setState合并，一次性更新页面

## setState为什么异步？能不能同步？什么时候异步？什么时候同步？
* setState只在合成事件和钩子函数中是异步的，在原生事件与setTimeout中都是同步的
* setState本身的执行都是同步的，并不是由异步代码实现。只是因为合成事件和钩子函数的执行顺序在更新之前，所以合成事件和钩子函数中无法拿到更新后的值。形成了异步。可以通过setState的第二个callback参数拿到更新后的值。
* 在合成事件与钩子函数中会对多次setState进行更新优化，只执行最后一次；在原生事件与setTimeout内不会进行批量更新优化；

## 类组件和函数组件之间的区别是啥（不考虑hook）？
1. 类组件有state和生命周期， 函数式组件又叫无状态组件， 没有state和生命周期。
2. 类组件使用时需要先实例化， 函数式组件不能实例化，他是一个函数， 传参， 执行，返回结果。
3. 无状态组件由于没有实例化过程，所以无法访问组件this中的对象。

## hooks
hooks为函数组件提供了状态，也支持在函数组件中进行数据获取、订阅事件、解绑事件等等。
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
用于缓存函数，如果依赖项发生了变化，该函数才发生变化。一般用于将此函数传入子组件中，因为默认每次父组件更新，传入子组件的该函数都不一样。
### useMemo
用于缓存函数返回值。如果依赖项发生变化，该函数才重新执行，计算返回值。一般用于该函数比较复杂，开销较大。避免每次更新此函数都重新执行。
### useRef
* ref 用来获取类组件实例，或者DOM对象
* useRef 用来创建ref对象
* 通过 ref={myRef} 绑定在DOM 对象或者React组件上
* 通过 this.myRef.current 访问DOM对象或者class组件的实例
* useRef(inivalue)定义的ref可以将initvalue保存为一个不变的数组。

## state 和 props 的区别是什么
* props和state都是普通的 JS 对象
* state 是组件内部状态，可变
* props 是外部传入的参数， 不可变

## 如何创建 refs
* 类组件中： `this.myRef = React.createRef();`
* 函数式组件中： `const ref = useRef();`
* 或者在类组件中通过回调函数： `<input type='text' ref={(input) => this.input = input} />`

## 什么是高阶组件
高阶组件(HOC)是接受一个组件并返回一个新组件的函数。基本上，这是一个模式，是从 React 的组合特性中衍生出来的，称其为纯组件，因为它们可以接受任何动态提供的子组件，但不会修改或复制输入组件中的任何行为。
```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```
## 高阶组件可以用来做什么
* 代码重用、逻辑和引导抽象
* 渲染劫持
* state 抽象和操作
* props 处理

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
* React.memo(): 这可以防止不必要地重新渲染函数组件
* PureComponent: 这可以防止不必要地重新渲染类组件

## 为什么要给所有的实例方法绑定this呢？
为了提前规避this指针丢失的问题，不然在程序执行过程中this就可能会指向window而不是该实例。
可以使用箭头函数， 他会默认指向上下文。

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

## vue和react的区别
1. 模板 与 JSX
2. 复用 mixin与HOC， hooks和setup
3. 通信方面 
    * 父子，vue, data down, event up. react props 和 callback
    * 同级，provide/inject, context
4. 数据绑定， vue双向，v-modal. react单向

## react性能优化
* render里面尽量减少新建变量和bind函数，传递参数时尽量减少传递参数的数量。
* 定制shouldComponentUpdate函数
* React.memo 和 React.PureComponent
* key不要用index
* useMemo, useCallback

## 组件插槽
使用props.children

## 组件生命周期
### 挂载阶段
constructor()   
static getDerivedStateFromProps()   
render()   
componentDidMount()
### 更新阶段
static getDerivedStateFromProps()   
shouldComponentUpdate()   
render()   
getSnapshotBeforeUpdate()   
componentDidUpdate()  
### 卸载
componentWillUnmount()
### 错误处理 
static getDerivedStateFromError()   
componentDidCatch()

## 代码分割, 组件懒加载
import()， React.lazy()

## 怎么开发错误边界组件
如果一个 class 组件中定义了 static getDerivedStateFromError() 或 componentDidCatch() 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界组件。

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

## render prop 和 HOC 的区别， 使用场景

## 使用 PropTypes 进行类型检查
```js
Component.propTypes = {
  name: PropTypes.string
};
```


