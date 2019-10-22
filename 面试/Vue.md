## vue生命周期
创建前/后, 载入前/后, 更新前/后, 销毁前/后

## 第一次页面加载会触发哪几个钩子？
第一次页面加载时会触发 beforeCreate, created, beforeMount, mounted 这四个钩子

## DOM渲染在哪个周期中就已经完成？
在 mounted 中就已经完成了

## 每个生命周期适合哪些场景？
* beforecreate : 可以在这加个loading事件，在加载实例时触发，在created时进行移除。
* created : 初始化完成时的事件写在这里，需要异步请求数据的方法可以在此时执行，完成数据的初始化。
* mounted : 当需要操作dom的时候执行
* updated : 当数据更新要做统一业务处理的时候

## v-show与v-if区别
1. v-show是css切换,display:none。
2. v-if是完整的销毁和重新创建

频繁切换时用v-show，运行时较少改变时用v-if

## computed watch
computed和watch都起到监听/依赖一个数据，并执行相应操作

1. computed：常用于比较消耗性能的计算场景，具有缓存性，只有它依赖的属性值变化后，下一次获取的computed值才会重新计算
2. 常用于某些数据的监听，无缓存性，页面重新渲染时值不变化也会执行。

## 常用指令有哪些
`v-if v-else v-for v-show v-modal`

## 绑定class的方法
```html
<!-- 对象语法，直接传给class一个对象 -->
<p v-bind:class="{ active: isActive, test1: isActive}">通过对象语法绑定class</p>
<!-- 对象语法，传给class一个计算属性，这个计算属性会返回一个对象 -->
<p v-bind:class="classObject">通过对象语法绑定class</p>

<!-- 数组语法，直接传给class一个数组，数组里的每个值都是data值，代表一个class名 -->
<p v-bind:class="[className1, className2]">通过数组语法绑定class</p>
<!-- 数组语法，三元表达式 -->
<p v-bind:class="[isActive ? className1 : className2]">通过数组语法绑定class</p>
```

## 组件通信
父子通讯，props down, events up。父组件通过 props 向下传递数据给子组件，子组件通过 events 给父组件发送消息
1. 父传子，用props，可以传数据和函数
2. 子传父，子组件emit，父组件on
    ```js
    // 子组件，在方法中调用emit
    this.$emit('listenToChildEvent', 'this message is from child')
    
    // 父组件上监听，在html代码中
    <child @listenToChildEvent="showMsgFromChild"></child>
    ```
兄弟组件通信
1. eventBus，`bus.$emit(...) bus.$on(...)`
2. vuex 定义全局的状态

## 路由跳转
1. 声明式，`<router-link to='home'>` router-link标签会渲染为`<a>`标签

2. 编程式导航，比如`this.$router.push('/home')`

## 路由导航守卫
1. 全局前置守卫 `router.beforeEach((to, from, next) => {...})`
2. 全局解析守卫 `router.beforeResolve((to, from, next) => {...})`，和router.beforeEach 类似，区别是在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用。
3. 全局后置钩子 `router.afterEach((to, from) => {...})`，没有 next 函数也不会改变导航本身
4. 路由独享守卫 `beforeEnter((to, from, next) => {...})`，在路由配置上直接定义
5. 组件内的守卫
    * beforeRouteEnter (to, from, next) {...}在渲染该组件的对应路由被confirm
    * beforeRouteUpdate (to, from, next) {...}在当前路由改变，但是该组件被复
    * beforeRouteLeave (to, from, next) {...}导航离开该组件的对应路由时调用

## 完整的导航解析流程
1. 导航被触发。
2. 在失活的组件里调用离开守卫 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。

## MVC MVP MVVM
### MVC 
* View：显示数据，检测用户的行为，调用Controller执行应用逻辑。
* Controller：应用逻辑或业务逻辑处理，并更新modal
* Model：Model变更后，通过观察者模式通知View更新视图。
### MVP
* View：对Presenter提供接口。不再依赖Modal。
* Presenter（Supervising Controller）：和经典MVC的Controller相比，任务更加繁重，不仅要处理应用业务逻辑，还要处理同步逻辑(高层次复杂的UI操作)。
* Model：Model变更后，通过观察者模式通知Presenter，如果有视图更新，Presenter又可能调用View的接口更新视图。
### MVVM
* ViewModel：内部集成了Binder(Data-binding Engine，数据绑定引擎)，将View和Model双向绑定，从而实现View或Model的自动更新。
* View：显示数据，View的变化会通过Binder自动更新相应的Model。
* Model：Model的变化也会被Binder监听(仍然是通过观察者模式)，一旦监听到变化，Binder就会自动实现View的更新。

## 单向数据流
1. 数据只能从父组件流向子组件
2. vuex的单向数据流，状态只能通过提交mutation去更改。

## v-for循环为什么要加key
使用v-for更新已渲染的元素列表时,默认用”就地复用“策略;列表数据修改的时候,他会根据key值去判断某个值是否修改,如果修改,则重新渲染这一项,否则复用之前的元素。  
从原理上来说，使用key来给每个节点做一个唯一标识，Diff算法就可以正确的识别此节点，key的作用主要是为了高效的更新虚拟DOM。

## v-for为什么不能跟v-if一起使用
v-for优先级高于v-if，这意味着 v-if 将分别重复运行于每个 v-for 循环中，效率比较低。  
推荐将v-if移到父元素

## 组件中的data为什么是函数而不是对象
一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝。  
组件是可复用的vue实例，一个组件被创建好之后，就可能被用在各个地方，而组件不管被复用了多少次，组件中的data数据都应该是相互隔离，互不影响的。

## v-model 的原理
v-modal只是一个语法糖，相当于执行了两步：  
1. 将组件的value绑定为一个值
2. 当组件的原生change事件发生时，将新的值赋给绑定的这个值。所以组件的value会发生变化。
```html
<input type="text" v-model="msg">

<input v-bind:value="msg" v-on:input="msg=$event.target.value"/>
```
input 元素本身有个 oninput 事件，这是 HTML5 新增加的，类似     onchange ，每当输入框内容发生变化，就会触发 oninput。

## keep-alive
`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。它是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。当组件在 `<keep-alive>` 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。

## 自定义指令
一般自定义指令解决的问题或者说使用场景是对普通 DOM 元素进行底层操作。

全局自定义指令  `Vue.directive('test', hookOptions)`  
局部自定义指令  
在组件的directives选项中进行声明。
```js
directives: {
    test: hookOptions
}
```

上面的hookOptions用来定义指令的行为。有bind, inserted, update, componentUpdated, unbind 共5个hook函数。

## 自定义过滤器
过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式中。被添加在 JavaScript 表达式的尾部，由“管道”符号指示：
```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```
全局过滤器： `Vue.filter('test', value => { ... })`
局部过滤器，定义在组件的filters选项中。
```js
filters: {
    test() { ... }
}
```

## nextTick()
nextTick一般用在，当我们想对视图更新后的DOM进行操作。  
因为数据更新时，并不会立即更新 DOM。如果在更新数据之后的代码执行DOM操作，有可能达不到预想效果。
```js
this.msg = 'hello'
this.$nextTick(()=>{
    console.log(this.$refs.msg.innerHTML)
})
```

## 路由鉴权
```js
meta: {
    requireAuth: true
}

router.beforeEach((to, from, next) => {
    if (to.matched.some(record => record.meta.requireAuth)) { ... }
}
```
为什么不直接用to.mate判断，而要用to.matched来判断：  
to.matched能够拿到父级的组件的路由对象，用to.matched则只需要给较高一级的路由添加requiresAuth即可，其下的所有子路由不必添加。

## 路由动态加载与动态删除
动态添加用: router.addRoutes([...])
动态删除：
1. 刷新页面
2. 替换 router.matcher 为一个新的matcher

```js
// 创建静态的路由表
const createRouter = () => new Router({
  mode: 'hash',
  scrollBehavior: () => ({ y: 0 }),
  isAddDynamicRoutes: false, // 是否已经动态添加路由
  routes: [...]
})
const router = createRouter()

// 动态添加路由
const addDynamicRoute = () => {
  let routes = dynamicRoutes() // 获得动态添加的路由表
  router.addRoutes(routes)
  router.options.isAddDynamicRoutes = true
}

// 重置路由，即动态删除添加的路由
const resetRoute = () => {
  const newRouter = createRouter()
  router.matcher = newRouter.matcher // reset router
  router.options.isAddDynamicRoutes = false
}
```

## Vue的运行机制
### 初始化流程
1. 创建Vue实例对象
2. init过程会初始化生命周期，初始化事件中心，初始化渲染、执行beforeCreate周期函数、初始化 data、props、computed、watcher、执行created周期函数等。
3. 初始化后，调用$mount方法对Vue实例进行挂载（挂载的核心过程包括模板编译、渲染以及更新三个过程）。
4. 如果没有在Vue实例上定义render方法而是定义了template，那么需要经历编译阶段。需要先将template 字符串编译成 render function
5. 调用render方法将render function渲染成虚拟的Node
6. 

## Vue响应式原理(数据绑定原理)
Vue 采用数据劫持结合发布—订阅模式的方法，通过 Object.defineProperty() 来劫持各个属性的 setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调

## defineProperty

## watch实现原理

## computed实现原理

## EventBus的实现原理

## Vue的数据为什么频繁变化但只会更新一次

## Proxy与Object.defineProperty的优劣势对比

## 虚拟DOM（Virtual Dom）原理

## vue-router实现原理
前端路由：  
路由模块的本质 就是建立起url和页面之间的映射关系，在单页面应用程序中，url变化时，只更新某个指定的容器中内容。

hash模式：  
使用 URL 的 hash 来模拟一个完整的 URL，只改变hash#后的部分。通过监听hashchange事件，监测hash值变化，实现更新页面部分内容

history模式：
利用HTML5 API, pushState 和 replaceState，通过这两个 API 可以改变 url 地址且不会发送请求，同时还有popstate 事件。

## vuex原理

## vuex state、getter、mutation、action、module

## vue项目优化
### 代码层面的优化
* v-if 和 v-show 区分使用场景
* computed 和 watch  区分使用场景
* v-for 遍历必须为 item 添加 key，且避免同时使用 v-if
* 事件的销毁
* 图片资源懒加载
* 路由懒加载
* 第三方插件的按需引入
* 服务端渲染 SSR or 预渲染

### Webpack 层面的优化
* Webpack 对图片进行压缩
* 减少 ES6 转为 ES5 的冗余代码
* 提取公共代码
* 模板预编译
* 提取组件的 CSS
* 优化 SourceMap
* 构建结果输出分析
* Vue 项目的编译优化

### 基础的 Web 技术的优化
* 开启 gzip 压缩
* 浏览器缓存
* CDN 的使用
* 使用 Chrome Performance 查找性能瓶颈

