# react-router的使用

> 该文章基于react-router v4/v5 版本

## 安装
```js
npm install --save react-router-dom
```
不应该直接安装 `react-router`，上面这个包会依赖 `react-router`

## Router
每个React router应用程序的核心都是Router组件。是route渲染和跳转的容器。

对于Web项目，react-router-dom库提供了两种Router，`<BrowserRouter>` 和 `<HashRouter>`，这些 router 初始化的时候都会创建一个 history 对象，通过 props 传递给 子组件，用于页面跳转，提供 push、replace、go、back、forward 等方法。

`BrowserRouter` 使用普通的URL  
`HashRouter` 使用hash URL

Route组件必须位于Router组件中，一般我们可以将Router组件写在最顶层。
```js
ReactDOM.render((
  <BrowserRouter>
    <App />
  </BrowserRouter>
), document.getElementById('root'))
```

## Route
Route 用来匹配地址栏的路径，并渲染对应的组件，没有匹配到的路由会被替换为null。

Route可以传以下参数：`path`、`exact`、`component`、`render`、`children`

### Route 渲染的三种方式
1. component
```html
<BrowserRouter>
    <!-- 没有指定 path 属性，这时只要你打开项目，无论你访问什么路径，这个Route都会匹配到。 -->
    <Route component={ Test } />

    <!-- 指定 path="/" ，和上面配置的效果一样，只要你打开项目，无论你访问什么路径，这个Route都会匹配到。这种配置方式我们成为 "非严格匹配" -->
    <Route path="/" component={ Test } />

    <!-- 指定 exact 属性，说明这个配置方式是 "严格匹配" ，只有当我们访问项目的根路径'/'的时候，才会匹配到这个Route -->
    <Route exact path="/" component={ Test } />

    <!-- 常规的Route配置方式，当我们访问"/help"路径的时候，匹配到这个Route，进而渲染出对应的component -->
    <Route path="/test" component={ Test } />
</BrowserRouter>
```
2. render
```html
<BrowserRouter>
    <!-- render属性是一个函数，当匹配到这个Route的时候，页面将会渲染出这个函数返回的内容。 -->
    <Route path="/render" render={ 
        () => { return <h1>我是Test</h1> } 
    } />
</BrowserRouter>  
```
3. children
```html
<BrowserRouter>
    <!-- 与前面 component 或者 render 不同的是，这种方法总是会被渲染，无论路由与当前的路径是否匹配。 -->
    <Route path="/children" children={
        () => { return <h1>我总会显示</h1> } 
    } />
    <!-- 或者 -->
    <Route path="/children">
    { () => { return <h1>我总会显示</h1> }
    </Route>
</BrowserRouter>
```

上面三种渲染方式，在渲染过程中都会将父组件Router传递的 `history` 对象 、 `location` 对象 以及 匹配对象 `match` 通过 props 传递给页面组件。 这样在页面组件中即可通过 props 访问 history、location、match

### history 的方法如下
1. `push(path, [state])` - Pushes a new entry onto the history stack
2. `replace(path, [state])` - Replaces the current entry on the history stack
3. `go(n)` - Moves the pointer in the history stack by n entries(可以前进或者后退N个)
4. `goBack()` - Equivalent to go(-1)
5. `goForward()` - Equivalent to go(1)

### location 的属性如下
1. `location.pathname` - 当前 url 的路径；
2. `location.hash` - 当前 url 的 hash 值；
3. `location.search` - 当前 url 的 查询部分；
4. `location.state` - 用户传递的 自定义信息；

我们可以在 展示组件 中通过 props.location 或者 prop.history.location 访问 当前页面的 url 信息。但是由于history对象是可变的，所以推荐使用location

### match 的属性如下
1. url - 页面跳转传递的 pathname；
2. path - Route 组件 的 path 配置项；
3. params - 动态路径参数；

match.url 是浏览器 URL 中的实际路径，而 match.path 是为路由编写的路径。假如我们在Route上写的path="/users/:userId", 在浏览器中访问 /users/5，那么 match.url 将是 "/users/5" 而 match.path 将是 "/users/:userId"。

## Switch
按照上面的写法，任何path被匹配到，Route都会渲染。

如果使用Switch，那只会渲染匹配到的第一个路由，后续所有的路由不会继续渲染。如果没有匹配到任何路由，我们可以在最后加一个默认路由。
```html
<BrowserRouter>
    <Switch>
        <Route path="/test" component={ Test } />
        <Route path="/mock" component={ Mock } />
        <!-- 如果上面都没匹配到，会渲染下面的 -->
        <Route component={ NotFound } />
    </Switch>
</BrowserRouter>
```

## 路由跳转
React Router提供了 `<Link>` `<NavLink>` 和 `<Redirect>` 组件来进行路由跳转。
```html
<Link to="/">Home</Link>

<NavLink to="/react" activeClassName="active"> React </NavLink>

<Redirect to="/login" />
```
Link 会被渲染成< a >标签，可以用来做页面跳转  

NavLink 是一种特殊的Link组件，当她的to属性匹配地址栏的路径时，她渲染成的< a >标签会带有'active'的样式。所以可以用来做Tab页，或者NavBar  

如果遇到 Redirect 组件将会始终执行浏览器重定向，它会导航到其to属性匹配的路径。但是当它位于 Switch 语句中时，只有在其他路由不匹配的情况下，才会渲染重定向组件

## 编程式导航
1. `history.push(path, [state])` - Pushes a new entry onto the history stack
2. `history.replace(path, [state])` - Replaces the current entry on the history stack
3. `history.go(n)` - Moves the pointer in the history stack by n entries(可以前进或者后退N个)
4. `history.goBack()` - Equivalent to go(-1)
5. `history.goForward()` - Equivalent to go(1)

## 路由嵌套
直接在子组件的某个位置添加Route组件即可，但是此时要注意path属性也要加上父级的path。使用 `this.props.match.path`
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
我们可以使用 `this.props.match.params` 来获取URL上传过来的参数
```js
<Route path='/test/:number' component={ Test }/>

// 在Test组件中
this.props.match.params // { number: '6' }
```

## Hooks
上面我们介绍了Route渲染的三种方法，component, render, children。这都是router v4中常用的，但是在router v5中，我们更推荐像下面这样去渲染。
```html
<BrowserRouter>
    <Route path="/about">
        <About />
    </Route>
</BrowserRouter>
```
这个其实也是children的写法，这样可以少写一些code，路由嵌套也变得更简单了。但是这样我们无法在页面组件中通过props获取Router传过来的history, location, match 等对象。

这个时候hooks就派上用场了
### useHistory
获取history对象 `let history = useHistory();`

### useLocation
获取location对象 `let location = useLocation();`

### useParams
将URL参数返回为一个key/value的对象 `let { key } = useParams();`

### useRoutematch
获取match对象数据。

尝试以与`<Route>`相同的方式匹配当前URL。一般用在没有真正的渲染Route，但是我们需要访问match对象数据的时候。
```js
function BlogPost() {
  let match = useRouteMatch("/blog/:slug");

  // Do whatever you want with the match...
  return <div />;
}
```

## withRouter
1. 目的就是让被修饰的组件可以从属性中获取history,location,match

    路由组件可以直接获取这些属性，而非路由组件就必须通过withRouter修饰后才能获取这些属性

    所以如果在非路由组件中执行push跳转，必须有withRouter

2. withRouter是专门用来处理数据更新问题的。

    在使用一些redux的的connect()或者mobx的inject()的组件中，如果依赖于路由的更新要重新渲染，会出现路由更新了但是组件没有重新渲染的情况。这是因为redux和mobx的这些连接方法会修改组件的shouldComponentUpdate。

    在使用withRouter解决更新问题的时候，一定要保证withRouter在最外层，比如withRouter(connect(Component))