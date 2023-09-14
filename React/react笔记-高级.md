## 代码分割

为了避免搞出大体积的代码包，在前期就思考该问题并对代码包进行分割是个不错的选择。

引入代码分割的最佳方式是通过 import() 语法，当 Webpack 解析到该语法时，它会自动地开始进行代码分割。

```jsx
//before
import { add } from './math';
//after
import('./math').then((math) => {});
```

基于路由的代码分割是不错的选择

## 组件懒加载

React.lazy()可以在渲染组件时，才加载对应的组件。

React.lazy()需要传入一个箭头函数，调用 import()方法。 `React.lazy(() => import('./OtherComponent'));`

React.lazy 中，import()传入的参数必须是一个包含默认导出(export defaults)的组件。

如果父组件已经渲染完成，但是懒加载的组件还没有加载完成的话，可以使用加载指示器为此组件做优雅降级。使用 Suspense 组件。

```jsx
<Suspense fallback={<div>Loading...</div>}>
  <OtherComponent />
</Suspense>
```

fallback 属性接受任何在组件加载过程中你想展示的 React 元素。

可以将 Suspense 组件置于懒加载组件之上的任何位置。

可以用一个 Suspense 组件包裹多个懒加载组件。

## 错误边界

### 什么是错误边界 ？

错误边界是一种 React 组件，这种组件可以捕获并打印发生在**其子组件树**任何位置的 JavaScript 错误，并且，它会渲染出备用 UI，而不是渲染那些崩溃了的子组件树。
仅可以捕获其子组件的错误，它无法捕获其自身的错误。

### 怎么开发错误边界组件 ?

如果一个 class 组件中定义了 static getDerivedStateFromError() 或 componentDidCatch() 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界组件。

当抛出错误后，使用 static getDerivedStateFromError(error) 渲染备用 UI ，使用 componentDidCatch(error, info) 打印错误信息。

### 什么时候捕获错误 ？

错误边界会在组件渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。

### 哪些错误无法捕获 ？

- 事件处理，请在事件处理内部用 try..catch...
- 异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
- 服务端渲染
- 它自身抛出来的错误（并非它的子组件）

### 怎么使用？

像常规组件一样使用。

```jsx
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

### 错误边界应该放置在哪?

错误边界的粒度由你来决定，可以将其包装在最顶层的路由组件并为用户展示一个 “Something went wrong” 的错误信息，就像服务端框架经常处理崩溃一样。你也可以将单独的部件包装在错误边界以保护应用其他部分不崩溃。

## Context

在 react 中，数据向下传递，事件向上传递。

当一个 state 变化影响多个组件更新视图的时候，这个 state 就应该上升存到共有的父组件中。

如果此时父子组件只是 1,2 层的传递，还可以实现，如果中间有 5,6 层的 wrapper，该怎么办呢？如果我们一层层传递，很容易造成 bug。

Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

### 创建 Context

创建出来的只是默认值。真正使用时要看最近的 Provider 传递的值。

```jsx
const MyContext = React.createContext('test');
```

### Context 往下传递（生产者）

使用 Provider 可以将 Context 直接传递到某组件树上下文，然后在该组件树中的任何子组件就可以使用这个值，而不用通过一层层传递。

```jsx
<MyContext.Provider value="test2">
  <Toolbar />
</MyContext.Provider>
```

```jsx
// 中间的组件再也不必指明往下一层层传递了。
function Toolbar(props) {
  return (
    <div>
      <MyComponent />
    </div>
  );
}
```

### 子组件获取 context

需要先声明一个本组件的静态属性 contextType

```jsx
class MyComponent extends React.Component {
  static contextType = MyContext;
}
// 或者
MyComponent.contextType = MyContext;
```

挂载在 class 上的 contextType 属性会被重赋值为一个由 React.createContext() 创建的 Context 对象。这能让你使用 this.context 来消费最近 Provider 提供的那个值。你可以在任何生命周期中访问到它，包括 render 函数中。

```jsx
render() {
    let value = this.context // “test2”
}
```

静态属性 contextType，只能订阅单一 context，如果我有多个 Context 对象，那在这个组件中，这个写法就不适用了。
所以需要 Consumer 对象。

Consumer 写法：以函数作为子元素。

```jsx
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

订阅多个 Context

```jsx
<ThemeContext.Provider value={theme}>
  <UserContext.Provider value={signedInUser}>
    <MyComponent />
  </UserContext.Provider>
</ThemeContext.Provider>
```

在子组件中

```jsx
<ThemeContext.Consumer>
  {(theme) => (
    <UserContext.Consumer>
      {(user) => <ProfilePage user={user} theme={theme} />}
    </UserContext.Consumer>
  )}
</ThemeContext.Consumer>
```

### 子组件树中更新 Context

更新 Context 需要在创建 Context 的时候，再创建一个函数，现在可以是空函数。

```jsx
const ThemeContext = React.createContext({
  theme: 'test',
  toggleTheme: () => {},
});
```

然后在 Provider 中将 context 值和具体的更新函数传递进去。

```jsx
constructor(props) {
    super(props);
    // State 也包含了更新函数，因此它会被传递进 context provider。
    this.state = {
        theme: 'test2'
        toggleTheme: this.toggleTheme,
    };
}

toggleTheme = () => {
    this.setState({theme: 'test3'});
}
render() {
    return (
        <ThemeContext.Provider value={this.state}>
            <Content />
        </ThemeContext.Provider>
    )
}
```

然后再在子组件中使用这个函数

```jsx
<ThemeContext.Consumer>
  {({ theme, toggleTheme }) => <button onClick={toggleTheme}>{theme}</button>}
</ThemeContext.Consumer>
```

## Refs

Refs 是一个 获取 DOM 节点或 React 元素实例的工具。

不要过度使用 refs。你可能首先会想到使用 refs 在你的 app 中“让事情发生”。如果是这种情况，请花一点时间，认真再考虑一下 state 属性应该被安排在哪个组件层中。通常你会想明白，让更高的组件层级拥有这个 state，是更恰当的。

下面是几个适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

### 创建 Refs

Refs 是使用 React.createRef() 创建的，并通过 ref 属性附加到 React 元素。

```jsx
constructor(props) {
    super(props);
    this.myRef = React.createRef()
}
render() {
    return <div ref={this.myRef} />
}
```

### 访问 Refs

```jsx
const node = this.myRef.current;
```

当 ref 属性用于普通 HTML 元素时， ref 对象接收底层 DOM 元素作为其 current 属性。

当 ref 属性用于自定义 class 组件时，ref 对象接收组件的挂载实例作为其 current 属性。

不能在函数组件上使用 ref 属性，因为他们没有实例。

```jsx
// 为DOM元素添加ref
constructor(props) {
    super(props);
    this.myRef = React.createRef()
}
focusEvent = () => {
    this.myRef.current.focus();
}
render() {
    return(
        <div>
            <input type="text" ref={this.myRef}/>
            {// 点击 button，输入框会获取焦点}
            <button onClick={this.foucusEvent}>button</button>
        </div>
    )
}
```

```jsx
// 为 Class 组件添加ref
constructor(props) {
    super(props);
    this.myRef2 = React.createRef();
}

componentDidMount() {
    this.myRef2.current.focusTextInput();
}

render() {
    return (
        <ChildrenComp ref={ this.myRef2 } />
    );
}

// 在子组件中
constructor(props) {
    super(props);
    this.inputRef = React.createRef();
}
focusTextInput() {
    this.inputRef.current.focus();
}
render(){
    return(
        <div>
            <input type='text' ref={this.inputRef }/>
        </div>
    )
}
```

### Refs 转发

在极少数情况下，你可能希望在父组件中引用子节点的 DOM 节点。通常不建议这样做，因为它会打破组件的封装，但它偶尔可用于触发焦点或测量子 DOM 节点的大小或位置。

虽然你可以向子组件添加 ref（如上例），但这不是一个理想的解决方案，因为你只能获取组件实例而不是 DOM 节点。并且，它还在函数组件上无效。

Ref 转发是一种自动将 ref 通过组件传递给其子节点的技术。

1. 子组件需要用 React.forwardRef 创建

```jsx
const Child = React.forwardRef((props, ref) => {
  return (
    <div>
      <button ref={ref} className="btn">
        test
      </button>
    </div>
  );
});
```

2. 在父组件中创建 ref 对象并传入子组件

```jsx
constructor(props) {
    super(props);
    this.myRef3 = React.createRef();
  }

  componentDidMount() {
    this.myRef3.current.foucs() // 聚焦在子组件button上
    this.myRef3.className // "btn"
  }

  render() {
    return (
      <Child ref={ this.myRef3 } />
    );
  }
```

以下是对上述示例发生情况的逐步解释：

1. 我们通过调用 `React.createRef` 创建了一个 React ref 并将其赋值给 myRef3 变量。
2. 我们通过指定 ref 为 JSX 属性，将其向下传递给 `<Child ref={ref}>`。
3. React 传递 ref 给 fowardRef 内函数 `(props, ref) => ...`，作为其第二个参数。
4. 我们向下转发该 ref 参数到 `<button ref={ref}>`，将其指定为 JSX 属性。
5. 当 ref 挂载完成，ref.current 将指向 `<button>` DOM 节点。

## 高阶组件

高阶组件是参数为组件，返回值为新组件的函数。

在高阶组件中不应直接**修改**传入的组件，也不应使用继承来复制其行为。而应该使用组合的方式。

```jsx
const WrappedComponent = function (props) {
  return <div>这是原始组件</div>;
};
const HOCComponent = function (WrappedComponent) {
  return (
    <div>
      <WrappedComponent />
      <div>这是组合后的高阶组件</div>
    </div>
  );
};

const hoc = HOCComponent(WrappedComponent);
```

注意事项

1. 不要修改原始组件

   不要通过修改原组件的 prototype 来重写其生命周期方法等（如给 WrappedComponent.prototype.componentWillReceiveProps 重新赋值）。请使用纯函数返回新的组件，因为一旦修改原组件，就失去了组件复用的意义。

2. 将不相关的 props 传递给被包裹的组件

   HOC 应该透传与自身无关的 props。HOC 返回的组件与原组件应保持类似的接口。

3. 保持可组合性

4. displayName

   为了方便调试，最常见的高阶组件命名方式是将子组件名字包裹起来。

   ```jsx
   HOCComponent.displayName = `HOCComponent(${WrappedComponent.name})`;
   ```

5. 不要在 render 方法内部使用高阶组件

   render 中的高阶组件会在每次 render 时重新 mount，之前组件内部的 state 也会丢失。

   在 render 方法之外调用。

   ```jsx
   const HOCTest = HOCComponent(WrappedComponent)
   render(
       return (<HOCTest />)
   )
   ```

## Portals

Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。

```jsx
ReactDOM.createPortal(child, container);
```

第一个参数（child）是任何可渲染的 React 子元素，例如一个元素，字符串或 fragment。第二个参数（container）是一个 DOM 元素。

通常来讲，当你从组件的 render 方法返回一个元素时，该元素将被挂载到 DOM 节点中离其最近的父节点。

```jsx
render() {
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

然而，有时候我们需要将子元素插入到 DOM 节点中的不同位置。

```jsx
render() {
  return ReactDOM.createPortal(
    <Test />,
    domNode
  );
}
```

## render prop

具有 render prop 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑。

```jsx
<DataProvider render={(data) => <h1>Hello {data.target}</h1>} />
```

```jsx
render() {
    return (
      <div>
        {/*
          Instead of providing a static representation of what renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
}
```

## Component 和 PureComponent

React.PureComponent 与 React.Component 几乎完全相同，但 React.PureComponent 通过 props 和 state 的浅对比来实现 shouldComponentUpate()。

> 如果定义了 shouldComponentUpdate()，无论组件是否是 PureComponent，它都会执行 shouldComponentUpdate 结果来判断是否 update。如果组件未实现 shouldComponentUpdate() ，则会判断该组件是否是 PureComponent，如果是的话，会对新旧 props、state 进行 shallowEqual 比较，一旦新旧不一致，会触发 update。

在 PureComponent 中，如果包含比较复杂的数据结构，可能会因深层的数据不一致而产生错误的否定判断。从而 shouldComponentUpdate 结果返回 false，界面得不到更新。

继承 PureComponent 之后的注意事项：

1. 如果你在`render`方法中创建一个新的函数，对象或者是数组那么你的做法（可能）是错误的。因为这些只比较引用。
2. 事件传参的时候不要用箭头函数，他会导致每次父组件`render`方法被调用时，一个新的函数被创建。可以使用 DOM API 来存储事件处理器需要的数据。如果需要优化大量元素或使用依赖于 React.PureComponent 相等性检查的渲染树，请考虑使用此方法。

```jsx
<Com  onClick={this.handleClick} data-param={param} />

handleClick(e) {
    this.setState({
        justClicked: e.target.dataset.param
    });
}
```

## 静态类型检查

在 Create React App 中使用 TypeScript

```sh
npx create-react-app my-app --typescript
```

## 使用 PropTypes 进行类型检查

要在组件的 props 上进行类型检查，你只需配置特定的 propTypes 属性：

```jsx
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

Greeting.propTypes = {
  name: PropTypes.string,
};
```

```jsx
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  // 这些属性都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message),
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number,
  }),

  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number,
  }),

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `onOfType` 中不会起作用。
  customProp: function (props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' +
          propName +
          '` supplied to' +
          ' `' +
          componentName +
          '`. Validation failed.'
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function (
    propValue,
    key,
    componentName,
    location,
    propFullName
  ) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' +
          propFullName +
          '` supplied to' +
          ' `' +
          componentName +
          '`. Validation failed.'
      );
    }
  }),
};
```
