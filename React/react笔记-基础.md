## JSX

### JSX写法
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

### JSX编译
Babel 会把 JSX 转译成一个名为 React.createElement() 函数调用，这个函数会创建出来一个虚拟dom节点
```jsx
const element = (
    <h1 className="greeting">
        Hello, world!
    </h1>
);
```
等效于
```js
const element = React.createElement(
    'h1',
    {className: 'greeting'},
    'Hello, world!'
);
```

### JSX渲染
想要将一个 React 元素渲染到 DOM 节点中，只需把它们一起传入 ReactDOM.render()
```jsx
ReactDOM.render(
    element,
    document.getElementById('root')
);
```

一般React 应用只会调用一次 ReactDOM.render()，来渲染顶层组件（一般React应用程序的顶层组件都是 App 组件），这个顶层组件会包含子组件，子组件又会包含他自己的子组件。这就是组件树，跟DOM树一样。

## 组件

### 函数组件定义
定义组件最简单的方式就是编写 JavaScript 函数

函数式组件一般是为了创建纯展示组件，这种组件只负责根据传入的props来展示，不涉及到state的操作。也叫无状态组件。
* 组件不会被实例化，整体渲染性能得到提升
* 组件不能访问this对象
* 组件无法访问生命周期的方法
* 组件只能访问输入的props，同样的props会得到同样的渲染结果，不会有副作用
```jsx
fucntion Test() {
    return <h1>Hello, Test</h1>
}
```

### class组件定义
也可以用ES6的class定义组件，这个class必须继承至React.Component，必须有render()方法，render方法里面return JSX
```jsx
class Test extends React.Component {
    render() {
        return <h1>Hello, Test</h1>;
    }
}
```
不要创建自己的组件基类去继承。 在 React 组件中，代码重用的主要方式是组合而不是继承。

只要有可能，尽量使用无状态组件创建形式。否则（如需要state、生命周期方法等），使用`React.Component`这种形式创建组件

### props
任何JSX组件所接收的属性都会转换为props传递给组件。

组件决不能修改自身的 props。

组件可以通过props向下传递数据。

props可以传状态，值，函数，组件等等。

如果你没给 prop 赋值，它的默认值是 true.

通过配置特定的 defaultProps 属性来定义 props 的默认值：
```jsx
Greeting.defaultProps = {
  name: 'Stranger'
};
```

对组件的 props 上进行类型检查，你只需配置特定的 propTypes 属性
```jsx
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```
也可以用在无状态组件上

### state
每个组件中都包含state，state 与 props 类似，但是 state 是私有的，只在当前组件中有效。
```jsx
class Test extends React.Component {
    constructor(props) {
        super(props)
        this.state = {test: 'this is test state'}
    }
    render() {
        return (
            <h1>{this.state.test}</h1>
        )
    }
}
```

不要直接修改state， 要用this.setState({test: 'test'})，setState只会更新特定的state，其他会被保留。

this.props 和 this.state 可能会异步更新，所以你不要直接依赖他们的值来更新下一个状态。如果需要依赖state或者props，则需要给setState传入一个函数
```jsx
this.setState((state, props) => {
    test: this.state.test + this.props.name
})
```

setState 其实是异步的 —— 不要指望在调用 setState 之后，this.state 会立即映射为新的值。如果你需要基于当前的 state 来计算出新的值，那你应该传递一个函数，而不是一个对象

### state 和 props 的区别
props（“properties” 的缩写）和 state 都是普通的 JavaScript 对象。它们都是用来保存信息的，这些信息可以控制组件的渲染输出，而它们的一个重要的不同点就是：props 是传递给组件的（类似于函数的形参），而 state 是在组件内被组件自己管理的（类似于在一个函数内声明的变量）。

### 常用生命周期
#### 挂载阶段：
当组件实例被创建并插入 DOM 中时
* constructor()
* render()
* componentDidMount()
#### 更新阶段：
* shouldComponentUpdate()
* render()
* componentDidUpdate()
#### 卸载阶段：
* componentWillUnmount()

### 事件处理
React 事件的命名采用小驼峰式（camelCase），而不是纯小写。

使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。

注意this的绑定，否则this有可能为undefined

#### 四种写法：
一般我们常用第一和第二种。

1.  在构造函数中绑定this，在JSX模板使用的时候直接用this.handleClick
```jsx
class Test extends React.Component {
    constructor(props) {
        super(props)
        this.handleClick = this.handleClick.bind(this)
    }
    handleClick() {
        console.log(this)
    }
    render() {
        return (
            <h1 onClick={this.handleClick}>test click</h1>
        )
    }
}
```

2. 不需要在构造函数中绑定，但是要在声明时候用箭头函数
```jsx
class Test extends React.Component {
    constructor(props) {
        super(props)
    }
    handleClick = () => {
        console.log(this)
    }
    render() {
        return (
            <h1 onClick={this.handleClick}>test click</h1>
        )
    }
}
```

3. 不需要在构造函数中绑定，也不在声明时候用箭头函数，但是在使用时候用箭头函数
```jsx
class Test extends React.Component {
    constructor(props) {
        super(props)
    }
    handleClick() {
        console.log(this)
    }
    render() {
        return (
            <h1 onClick={(e) => this.handleClick(e)}>test click</h1>
        )
    }
}
```

4. 不在构造函数中绑定，但是在JSX模板使用的时候绑定。也不使用箭头函数
```jsx
class Test extends React.Component {
    constructor(props) {
        super(props)
    }
    handleClick() {
        console.log(this)
    }
    render() {
        return (
            <h1 onClick={this.handleClick.bind(this)}>test click</h1>
        )
    }
}
```

### 事件传参
有时候我们调用事件处理函数的时候，需要传递一些参数进去，这个时候需要用到上面的第三和第四种写法。  
只能用这两种方法，因为这个大括号里面只能是一个函数的引用，而不是函数的调用。
```html
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>

<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

### 条件渲染

#### if else
```jsx
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />
  }
  return <GuestGreeting />
}
```

#### && 运算符
```jsx
return (
    <div>
        {unreadMessages.length > 0 && <h2>TEST</h2>}
    </div>
);
```

#### 三目运算符
```jsx
return (
    <div>
        <h2>isLoggedin {isLoggedIn ? 'yes' : 'not'}</h2>
    </div>
);
```

#### 阻止渲染
某些情况下，你可能希望能隐藏组件，让组件直接返回 null。
```jsx
function Hide(props) {
    return null;
}
```

### 列表渲染

要为列表项加key，最好是用元素的id，如果没有id，可以用索引index

key应该直接作用在参与循环的组件上，而不是这个组件内部。一般在 map() 方法中的元素需要设置 key 属性。

```jsx
function NumberList(props) {
    const numbers = props.numbers;
    const listItems = numbers.map((number) =>
        <li key={number.toString()}>{number}</li>
    );
    return (
        <ul>{listItems}</ul>
    );
}
```

### 受控组件
对于一些表单元素，如input等，都是有自己的value的，这些元素是可以被直接设置value属性值的。
在React中，数据是单向的，所以不应该直接去修改这种组件的属性值，而应该用组件内部的state中的值。

对于这种遵从React规定，使用state为唯一数据源，被 React 控制取值赋值的表单元素，我们称为受控组件。

具体写法为

1. state中包含这个表单元素的value值
2. 这个表单元素的value值要取自state中的value
3. 元素value改变时，要赋值给state中的value，通过setState
4. 通过e.target.value获取元素的value

```jsx
render() {
    return (
        <input
            type="text"
            value={this.state.value}
            onChange={(e) => {
                this.setState({
                    value: e.target.value.toUpperCase()
                })
            }}
        />
    );
}
```

也是双向绑定的实现

### 父子组件通信
将多个组件中需要共享的 state 向上移动到它们的最近共同父组件中，便可实现共享 state。这就是所谓的“状态提升”。

两个子组件有可能都需要使用和修改父组件的状态，这就需要用到子组件向父组件传递数据，即子组件更新父组件的状态。
* 父组件提供更新状态的方法，并通过props传递给子组件
* 子组件中通过props调用这个方法，传值进去
```jsx
// 父组件中
changeParentState(value) {
    this.setState(...);
}
<Child onChildChange={this.changeParentState} />
```
```jsx
// 子组件中
handleChange(e) {
    this.props.onChildChange(e.target.value);
}
<input onChange={this.handleChange} />
```

父组件中更新子组件的状态就很简单了
* 将父组件的state传递到子组件中，通过props
* 子组件获取这个props中的值，并使用在自己的组件中

### 组件插槽
通过插槽功能可以将某个组件的子元素或者子组件，直接渲染在组件中的固定位置。使用props.children
```jsx
// 父组件中
<Parent>
    <h1>this is children</h1>
</Parent>
```
```jsx
function Parent(props) {
    return (
        <div>
            <h1>below is children</h1>
            {props.children}
        </div>
    );
}
```

### Fragment
用空标签<>或者<React.Fragment>来包裹一些不需要父级标签的内容。

使用显式 <React.Fragment> 语法声明的片段<b>可能具有 key</b>，而且key 是目前唯一可以传递给 Fragment 的属性，未来可能会添加对其他属性的支持，例如事件。
```jsx
<>
    <td>Hello</td>
    <td>World</td>
</>

<React.Fragment key={item.id}>
    <td>Hello</td>
    <td>World</td>
</React.Fragment>
```
