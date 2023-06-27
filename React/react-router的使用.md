# react-router 的使用

> 该文章基于 react-router v4/v5 版本

## 安装

```js
npm install --save react-router-dom
```

不应该直接安装 `react-router`，上面这个包会依赖 `react-router`

## Router

每个 React router 应用程序的核心都是 Router 组件。是 route 渲染和跳转的容器。

对于 Web 项目，react-router-dom 库提供了两种 Router，`<BrowserRouter>` 和 `<HashRouter>`，这些 router 初始化的时候都会创建一个 history 对象，通过 props 传递给 子组件，用于页面跳转，提供 push、replace、go、back、forward 等方法。

`BrowserRouter` 使用普通的 URL  
`HashRouter` 使用 hash URL

Route 组件必须位于 Router 组件中，一般我们可以将 Router 组件写在最顶层。

```js
ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);
```

## Route

Route 用来匹配地址栏的路径，并渲染对应的组件，没有匹配到的路由会被替换为 null。

Route 可以传以下参数：`path`、`exact`、`component`、`render`、`children`

### Route 渲染的三种方式

1. component

```html
<BrowserRouter>
  <!-- 没有指定 path 属性，这时只要你打开项目，无论你访问什么路径，这个Route都会匹配到。 -->
  <Route component="{" Test } />

  <!-- 指定 path="/" ，和上面配置的效果一样，只要你打开项目，无论你访问什么路径，这个Route都会匹配到。这种配置方式我们成为 "非严格匹配" -->
  <Route path="/" component="{" Test } />

  <!-- 指定 exact 属性，说明这个配置方式是 "严格匹配" ，只有当我们访问项目的根路径'/'的时候，才会匹配到这个Route -->
  <Route exact path="/" component="{" Test } />

  <!-- 常规的Route配置方式，当我们访问"/help"路径的时候，匹配到这个Route，进而渲染出对应的component -->
  <Route path="/test" component="{" Test } />
</BrowserRouter>
```

2. render

```html
<BrowserRouter>
  <!-- render属性是一个函数，当匹配到这个Route的时候，页面将会渲染出这个函数返回的内容。 -->
  <Route path="/render" render={ () => { return
  <h1>我是Test</h1>
  } } />
</BrowserRouter>
```

3. children

```html
<BrowserRouter>
  <!-- 与前面 component 或者 render 不同的是，这种方法总是会被渲染，无论路由与当前的路径是否匹配。 -->
  <Route path="/children" children={ () => { return
  <h1>我总会显示</h1>
  } } />
  <!-- 或者 -->
  <Route path="/children">
    { () => { return
    <h1>我总会显示</h1>
    }
  </Route>
</BrowserRouter>
```

上面三种渲染方式，在渲染过程中都会将父组件 Router 传递的 `history` 对象 、 `location` 对象 以及 匹配对象 `match` 通过 props 传递给页面组件。 这样在页面组件中即可通过 props 访问 history、location、match

### history 的方法如下

1. `push(path, [state])` - Pushes a new entry onto the history stack
2. `replace(path, [state])` - Replaces the current entry on the history stack
3. `go(n)` - Moves the pointer in the history stack by n entries(可以前进或者后退 N 个)
4. `goBack()` - Equivalent to go(-1)
5. `goForward()` - Equivalent to go(1)

### location 的属性如下

1. `location.pathname` - 当前 url 的路径；
2. `location.hash` - 当前 url 的 hash 值；
3. `location.search` - 当前 url 的 查询部分；
4. `location.state` - 用户传递的 自定义信息；

我们可以在 展示组件 中通过 props.location 或者 prop.history.location 访问 当前页面的 url 信息。但是由于 history 对象是可变的，所以推荐使用 location

### match 的属性如下

1. url - 页面跳转传递的 pathname；
2. path - Route 组件 的 path 配置项；
3. params - 动态路径参数；

match.url 是浏览器 URL 中的实际路径，而 match.path 是为路由编写的路径。假如我们在 Route 上写的 path="/users/:userId", 在浏览器中访问 /users/5，那么 match.url 将是 "/users/5" 而 match.path 将是 "/users/:userId"。

## Switch

按照上面的写法，任何 path 被匹配到，Route 都会渲染。

如果使用 Switch，那只会渲染匹配到的第一个路由，后续所有的路由不会继续渲染。如果没有匹配到任何路由，我们可以在最后加一个默认路由。

```html
<BrowserRouter>
  <Switch>
    <Route path="/test" component="{" Test } />
    <Route path="/mock" component="{" Mock } />
    <!-- 如果上面都没匹配到，会渲染下面的 -->
    <Route component="{" NotFound } />
  </Switch>
</BrowserRouter>
```

## 路由跳转

React Router 提供了 `<Link>` `<NavLink>` 和 `<Redirect>` 组件来进行路由跳转。

```html
<Link to="/">Home</Link>

<NavLink to="/react" activeClassName="active"> React </NavLink>

<Redirect to="/login" />
```

Link 会被渲染成< a >标签，可以用来做页面跳转

NavLink 是一种特殊的 Link 组件，当她的 to 属性匹配地址栏的路径时，她渲染成的< a >标签会带有'active'的样式。所以可以用来做 Tab 页，或者 NavBar

如果遇到 Redirect 组件将会始终执行浏览器重定向，它会导航到其 to 属性匹配的路径。但是当它位于 Switch 语句中时，只有在其他路由不匹配的情况下，才会渲染重定向组件

## 编程式导航

1. `history.push(path, [state])` - Pushes a new entry onto the history stack
2. `history.replace(path, [state])` - Replaces the current entry on the history stack
3. `history.go(n)` - Moves the pointer in the history stack by n entries(可以前进或者后退 N 个)
4. `history.goBack()` - Equivalent to go(-1)
5. `history.goForward()` - Equivalent to go(1)

## 路由嵌套

直接在子组件的某个位置添加 Route 组件即可，但是此时要注意 path 属性也要加上父级的 path。使用 `this.props.match.path`

```html
// App.js
<div>
    父页面
    <NavLink activeClassName="selected" to="/test">test</NavLink >
    <NavLink activeClassName="selected" to="/nest">nest route</NavLink >

    <Route path="/test" component={ Test } />
    <Route path="/nest" component={ Nest } />
</div>

// Nest.js
<div >
    子页面
    <Link to={`${this.props.match.path}/child1`}>child1</Link>
    <Link to={`${this.props.match.path}/child2`}>child2</Link>

    <Route path={`${this.props.match.path}/child1`} component={ Child1 } />
    <Route path={`${this.props.match.path}/child2`} component={ Child2 } />
</div>
```

## 动态路由，路由参数

我们可以使用 `this.props.match.params` 来获取 URL 上传过来的参数

```js
<Route path="/test/:number" component={Test} />;

// 在Test组件中
this.props.match.params; // { number: '6' }
```

## Hooks

上面我们介绍了 Route 渲染的三种方法，component, render, children。这都是 router v4 中常用的，但是在 router v5 中，我们更推荐像下面这样去渲染。

```html
<BrowserRouter>
  <Route path="/about">
    <About />
  </Route>
</BrowserRouter>
```

这个其实也是 children 的写法，这样可以少写一些 code，路由嵌套也变得更简单了。但是这样我们无法在页面组件中通过 props 获取 Router 传过来的 history, location, match 等对象。

这个时候 hooks 就派上用场了

### useHistory

获取 history 对象 `let history = useHistory();`

### useLocation

获取 location 对象 `let location = useLocation();`

### useParams

将 URL 参数返回为一个 key/value 的对象 `let { key } = useParams();`

### useRoutematch

获取 match 对象数据。

尝试以与`<Route>`相同的方式匹配当前 URL。一般用在没有真正的渲染 Route，但是我们需要访问 match 对象数据的时候。

```js
function BlogPost() {
  let match = useRouteMatch('/blog/:slug');

  // Do whatever you want with the match...
  return <div />;
}
```

## withRouter

1. 目的就是让被修饰的组件可以从属性中获取 history,location,match

   路由组件可以直接获取这些属性，而非路由组件就必须通过 withRouter 修饰后才能获取这些属性

   所以如果在非路由组件中执行 push 跳转，必须有 withRouter

2. withRouter 是专门用来处理数据更新问题的。

   在使用一些 redux 的的 connect()或者 mobx 的 inject()的组件中，如果依赖于路由的更新要重新渲染，会出现路由更新了但是组件没有重新渲染的情况。这是因为 redux 和 mobx 的这些连接方法会修改组件的 shouldComponentUpdate。

   在使用 withRouter 解决更新问题的时候，一定要保证 withRouter 在最外层，比如 withRouter(connect(Component))
