## vue生命周期
创建前/后, 载入前/后, 更新前/后, 销毁前/后  
beforeCreate/created, beforeMount/mounted, beforeUpdate/updated, beforeDestroy/destroyed  
keep-alive 组件激活时 activated，keep-alive 组件停用时 deactivated

## 第一次页面加载会触发哪几个钩子？
第一次页面加载时会触发 beforeCreate, created, beforeMount, mounted 这四个钩子

## DOM渲染在哪个周期中就已经完成？
在 mounted 中就已经完成了

## 每个生命周期适合哪些场景？
* beforecreate : 此时data和method都还没初始化，可以在这加个loading事件，在加载实例时触发，在created时进行移除。
* created : 初始化完成时的事件写在这里，需要异步请求数据的方法可以在此时执行，完成数据的初始化。
* mounted : 当需要操作dom的时候执行
* updated : 当数据更新要做统一业务处理的时候

## Vue 的父组件和子组件生命周期钩子函数执行顺序
加载渲染：  
父beforeCreate --> 父created --> 父beforeMounted --> 子beforeCreate --> 子created --> 子beforeMounted --> 子mounted -->父mounted  
父组件更新：  
父 beforeUpdate --> 父 updated  
子组件更新：  
父beforeUpdate --> 子beforeUpdate --> 子updated --> 父updated  
销毁：  
父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

## beforeCreate 的时候能拿到 Vue 实例么
可以拿到组件实例，也就是可以用this。但是不能获取到data和methods。如果想强行获取data，可以用this.$options.data()
## 什么情况下会触发组件销毁，销毁的时候会卸载自定义事件和原生事件么
页面关闭，路由切换（没有keep-alive时），v-if

Vue本身的一些自定义事件监听，比如@click, @blur等会自动销毁，但是原生的document.addEventListener事件比如scroll，keydown，keyup，vue监测不到，无法移除监听，你可以自己销毁。

其实组件销毁后DOM元素也被移除，根据JS的垃圾回收机制，节点的销毁，会顺带把该节点所有的监听事件置空，所以绑定在DOM元素上的事件自然也就被移除了。但是绑定的是document的话，因为document对象没有被清楚，所以事件没有被卸载。
## 哪个生命周期调用异步请求
created、beforeMount、mounted 中都可以，因为此时data已经创建。一般在created中。

## 父组件怎么监听子组件的生命周期
1. 子组件生命周期中emit，在子组件上监听，并触发父组件的方法。
    ```js
    // Parent.vue
    <Child @mounted="doSomething"/>
        
    // Child.vue
    mounted() {
        this.$emit("mounted");
    }
    ```
2. 使用@hook监听
    ```js
    //  Parent.vue
    <Child @hook:mounted="doSomething" ></Child>
        
    //  Child.vue
    mounted(){
        console.log('子组件触发 mounted 钩子函数 ...');
    }
    ```

## v-show与v-if区别
1. v-show是css切换,display:none。
2. v-if是完整的销毁和重新创建

频繁切换时用v-show，运行时较少改变时用v-if

## computed watch
computed和watch都起到监听/依赖一个数据，并执行相应操作

1. computed：常用于比较消耗性能的计算场景，具有缓存性，只有它依赖的属性值变化后，下一次获取的computed值才会重新计算
2. watch：常用于某些数据的监听，无缓存性，页面重新渲染时值不变化也会执行。

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

## 动态组件
```js
<component v-bind:is="currentTabComponent"></component>
```

## 直接给一个数组项赋值，Vue 能检测到变化吗？怎么解决？
不能检测到以下变化：  
当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`  
当你修改数组的长度时，例如：`vm.items.length = newLength`  

解决方法：  
解决第一种问题：  
`vm.$set(vm.items, indexOfItem, newValue)`  
`vm.items.splice(indexOfItem, 1, newValue)`  
解决第二种问题：  
`vm.items.splice(newLength)` 

## Vue 如何实现的数组的监听，为什么 Vue 没有对数组下标修改做劫持 
重写数组的方法。不对数组下标做劫持就是因为性能问题。

## 组件通信
1. props/$emit 父子通信  
    如果子组件需要修改父组件传进来的props，可以用.sync来减少父组件监听和修改的代码
    ```js
    // before
    // 父组件中
    <child :val="name" @update="modify">
    modify(newVal){
      this.name=newVal
    }
    // 子组件中
    <input :value=val @input="$emit('update', $event.target.value)"/>

    // after
    // 父组件中，省略了写监听函数
    <child :val.sync="name">
    // 子组件中
    <input :value=val @input="$emit('update:val', $event.target.value)"/>
    ```
2. 使用vuex 父子，兄弟，隔代都可以用
3. EventBus 父子，兄弟，隔代都可以用
    ```js
    const EventBus = new Vue()
    EventBus.$emit("aMsg", '来自A页面的消息');
    EventBus.$on("aMsg", (msg) => {
        this.msg = msg;
    });
    ```
4. provide/inject 隔代通信   
    祖先组件中通过 provide 来提供变量，然后在子孙组件中通过 inject 来注入变量  
    ```js 
    // 祖先
    provide: {
      test: "demo"
    }
    // 子孙
    inject: ['test']
    ```
5. `$attrs/$listeners` 隔代通信
    * $attrs：当子组件的props中没有声明父组件传下来的prop属性时，那么父组件传下来的prop属性会被保存在子组件的$attrs属性上( class 和 style 除外 )。  
    子组件加了inheritAttrs:false，DOM上就不会继承未声明的props。
        ```js
        // 父
        <child :foo="foo" :coo="coo"></child>

        // 子，继承了所有属性并传给下一代
        <p>attrs:{{$attrs}}</p>
        <grandChild v-bind="$attrs"></grandChild>

        // 孙子，只声明了coo，并设置inheritAttrs: false表示不继承
        props:["coo"],
        inheritAttrs:false
        <p>coo:{{coo}}</p>
        ```
    * $listeners：包含了父作用域中的 (不含 .native 修饰器的)  v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件
        ```js
        // 父
        <child @parentTest="parentTestMethod"></child>
        parentTestMethod(value){
            console.log(value)
        }

        // 子，相当于也监听了事件
        <grandChild v-on="$listeners"></grandChild>

        // 孙子
        <button @click="test">我要发射火箭</button>
        test(){
            this.$emit("parentTest",'test');
        }  
        ```
6. ref 与 $parent / $children 父子通信  
    * 使用 this.$parent 查找当前组件的父组件实例。
    * 使用 this.$children 查找当前组件的直接子组件，可以遍历全部子组件，需要注意 $children 并不保证顺序，也不是响应式的。
    * 使用 this.$refs 查找命名子组件。
    * 使用 this.$root 查找根组件，并可以配合$children遍历全部组件。


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

## Vue的数据为什么频繁变化但只会更新一次DOM
现在有这样的一种情况，mounted的时候test的值会被循环执行++1000次。 每次++时，都会根据响应式触发setter->Dep->Watcher->update->run。 如果这时候没有异步更新视图，那么每次++都会直接操作DOM更新视图，这是非常消耗性能的。 所以Vue实现了一个queue队列，在下一个tick（或者是当前tick的微任务阶段）统一执行queue中Watcher的run。同时，拥有相同id的Watcher不会被重复加入到该queue中去，所以不会执行1000次Watcher的run。最终更新视图只会直接将test对的DOM的0变成1000。 保证更新视图操作DOM的动作是在当前栈执行完以后下一个tick（或者是当前tick的微任务阶段）的时候调用，大大优化了性能。

## Vue 框架怎么实现对象和数组的监听？
* 通过递归遍历对象，利用 Object.defineProperty() 也能对对象进行监听
* 通过重写数组的方法， push, pop....实现对数组的监听。

## v-model 的原理
v-modal只是一个语法糖，相当于执行了两步：  
1. 将组件的value绑定为一个值
2. 组件内部数据的变化，触发input事件更新。
```html
<child v-model="msg">
// 相当于
<child :value="msg" @input="msg=$event.target.value"/>
watch: {
    value(val) {
        this.innerValue = val
    }
    innerValue(val) {
        this.emit('input', val)
    }
}
```

## keep-alive
`<keep-alive>` 是 Vue 内置的一个组件，可以使被包含的组件保留状态，避免重新渲染。  
`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。  
它是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。  
当组件在 `<keep-alive>` 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。

## 自定义指令
一般自定义指令解决的问题或者说使用场景是对普通 DOM 元素进行底层操作。

全局自定义指令  `Vue.directive('test', hookOptions)`  
局部自定义指令  
在组件的directives选项中进行声明。
```js
directives: {
    test: {
        bind() {

        },
        inserted() {

        },
        update() {
            
        }
    }
}
```

上面的hookOptions用来定义指令的行为。有bind, inserted, update, componentUpdated, unbind 共5个hook函数。

bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。  
inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。  
update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。  
componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。  
unbind：只调用一次，指令与元素解绑时调用。  

## 自定义过滤器
过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式中。被添加在 JavaScript 表达式的尾部，由“管道”符号指示：
```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```
全局过滤器： 

```
Vue.filter('test', value => { ... })
```

局部过滤器，定义在组件的filters选项中。
```js
filters: {
    test() { ... }
}
```

## nextTick()
nextTick一般用在，当我们想在更新数据后，获取被更新的DOM进行操作。  
因为数据更新时，并不会立即更新 DOM。如果在更新数据之后的代码执行DOM操作，有可能达不到预想效果。
```js
this.msg = 'hello'
// 此时dom还没有更新
this.$nextTick(()=>{
    // 此时dom已经更新
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
动态添加: router.addRoutes([...])  
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

## Vue的运行流程，new Vue都做了什么
1. initLifecycle(vm) 初始化生命周期
2. initEvents(vm) 初始化事件中心
3. initRender(vm) 初始化渲染
4. callHook(vm, 'beforeCreate') 触发beforeCreate生命周期
5. initInjections(vm) 初始化别处注入过来的inject对象
6. initState(vm) 初始化state， props, computed, watcher
7. initProvide(vm) 初始化给别处提供的Provide对象
8. callHook(vm, 'created') 触发created生命周期
9. vm.$mount()   
    1. compile 将el或者template编译成render方法
    2. render 通过执行 createElement 方法生成虚拟DOM节点
    3. vnode  // create diff patch
    4. patch 将虚拟DOM树插入到真实的DOM中

## Vue响应式原理(数据绑定原理)
Vue 采用数据劫持结合发布—订阅模式的方法，通过 Object.defineProperty() 来劫持各个属性的 setter，getter，在set时通知订阅者更新，get的时候收集订阅者。模板编译的时候，生成对应的订阅者，调用这个属性的get，主动去完成这个订阅者收集。所以一共分为4部分：

1. 对数据对象进行遍历，包括子属性对象的属性，利用 Object.defineProperty() 对属性都加上 setter 和 getter。调用getter的时候，进行依赖收集，将Observer存起来。调用setter的时候通知更新。每一个对象属性都有自己的任务中心。

2. 模板编译的时候，对模板上的每一个属性，生成对应的订阅者，这样数据变化时，可以通知模板视图的更新。

3. 实现一个任务调度中心：采用 发布-订阅 设计模式，用来收集订阅者 Observer，以及通知Observer去更新。

4. 实现一个Observer类，用来作为订阅者类。订阅者初始化的时候主动调用属性的getter，将自己作为订阅这个属性变化的，完成依赖收集。

```js
class Vue {
  constructor(options) {
    this.data = options.data;
    this.walkData(this.data);
    this.compile();
  }

  // 第一步
  walkData(data) {
    Object.keys(data).forEach((key) => {
      let val = data[key];
      let eventCenter = new EventCenter(); // 每一个属性都有一个任务中心
      Object.defineProperty(data, key, {
        set(value) {
          val = value;
          eventCenter.notify(); // 通知所有的observer更新
        },
        get(value) {
          eventCenter.collect(); // 最开始collect的时候， 因为EventCenter.target是null， 所以并没有依赖，只有编译页面的时候， 才将依赖收集上来
          return value;
        },
      });
    });
  }

  // 第二步
  compile() {
    new Observer(); // 指定target 和 调用get，完成依赖收集
  }
}

// 第三步
class EventCenter {
  constructor() {
    this.observers = []; // 存储所有的observer
  }
  addObserver(ob) {
    this.observers.push(ob);
  }
  notify() {
    this.observers.forEach((ob) => {
      ob.update();
    });
  }
  collect() {
    // 依赖收集
    if (EventCenter.target) { // 这里是重点，借用静态属性target存储当前的observer实例。
      EventCenter.target.addSelf(this);
    }
  }
}
EventCenter.target = null;

// 第四步
class Observer {
  constructor() {
    this.init();
  }
  init() {
    EventCenter.target = this;
    // 调用get， 完成依赖收集
  }
  update() {}
  addSelf(eventCenter) {
    eventCenter.addObserver(this); // 将自己添加到事件调度中心的订阅者队列
  }
}
```

## vue响应式原理的两大缺陷，解决办法
1. 无法监听到对象属性的动态添加和删除。用this.$set 或者this.$delete解决
2. 无法监听到数组下标和length长度的变化。用 arr.splice 解决

## Object.defineProperty(obj, prop, descriptor)
会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的。
1. obj  要在其上定义属性的对象。
2. prop  要定义或修改的属性的名称。
3. descriptor  将被定义或修改的属性描述符。属性描述符有两种主要形式：数据描述符和存取描述符。描述符必须是这两种形式之一，不能同时是两者。

    数据描述符
    * writable  
    当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为 false。
    * value  
    该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。

    存取描述符
    * get  
   当访问该属性时，该方法会被执行，默认为 undefined。
    * set  
    当属性值修改时，触发执行该方法，该方法将接受唯一参数，即该属性新的参数值。默认为 undefined  

    数据描述符和存取描述符均具有以下可选键值
    * configurable  
    当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变(也就是可以被重新defineProperty)，同时该属性也能从对应的对象上被删除。默认为 false。
    * enumerable  
    当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。默认为 false。

## Proxy与Object.defineProperty的优劣势对比
Proxy 的优势如下:
* Proxy 可以直接监听对象而非属性；
* Proxy 可以直接监听数组的变化；
* Proxy 有多达 13 种拦截方法, 很多是 Object.defineProperty 不具备的；
* Proxy 返回的是一个新对象,我们可以只操作新的对象达到目的,而 Object.defineProperty 只能遍历对象属性直接修改；

Object.defineProperty 的优势如下:

兼容性好，支持 IE9，而 Proxy 的存在浏览器兼容性问题,而且无法用 polyfill 磨平，因此 Vue 的作者才声明需要等到下个大版本( 3.0 )才能用 Proxy 重写。

## watch实现原理
initWatch的过程中其实就是实例化new Watcher完成观察者的依赖收集的过程。  
参考Vue响应式原理。

## computed实现原理
除了依赖收集，他还有个动态计算的过程，只有依赖的数据发生变化，他才重新计算。  
参考Vue响应式原理。

## EventBus的实现原理
其实就是发布订阅模式的事件调度中心的实现。
1. 有一个对象或者Map，来存储事件，以及对应的处理函数。
2. 一个on方法，来监听某事件，然后将对应的处理函数push进去。
3. 一个off方法，来删除某事件的监听。
4. 一个trigger方法， 来触发某事件的处理函数。

```js
class EventBus {
    constructor() {
        this.map = new Map();
    }
    on(topic, callback) {
        if (this.map.get(topic)) {
            this.map.get(topic).push(callback)
        } else {
            this.map.set(topic, [callback])
        }   
    }
    off(topic) {
        this.map.delete(topic)
    }
    trigger(topic){
        if (this.map.get(topic)){
            this.map.get(topic).forEach(fn => fn())
        } 
    }
}
```

## 虚拟DOM（Virtual Dom）原理
虚拟dom只是一层对真实DOM树的抽象，对这颗抽象树进行创建节点,删除节点以及修改节点的操作， 经过diff算法得出一些需要修改的最小单位,再更新视图，减少了dom操作，提高了性能。

虚拟 DOM 的实现原理主要包括以下 3 部分：
1. 用 JavaScript 对象模拟真实 DOM 树，对真实 DOM 进行抽象；一般虚拟DOM都有以下几个属性。
    * 节点名称 tag
    * 节点属性 props 对象
    * 子节点 children 数组
    ```js
    <div id="app">
        <p class="text">hello world!!!</p>
    </div>
    // 转化为虚拟dom
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
                },
                chidren: [
                    'hello world!!!'
                ]
            }
        ]
    }
    ```
2. diff 算法 — 比较两棵虚拟 DOM 树的差异；
    * 比较只会在同层级进行, 不会跨层级比较
    * 在diff比较的过程中，循环从两边向中间比较
3. pach 算法 — 将两个虚拟 DOM 对象的差异应用到真正的 DOM 树。

## vue scoped 是怎么实现的
1. 给HTML的DOM节点加一个不重复data属性(形如：data-v-2311c06a)来表示他的唯一性。
2. 在每句css选择器的末尾（编译后的生成的css语句）加一个当前组件的data属性选择器的哈希特征值（如[data-v-2311c06a]）来私有化样式。

## vue-router实现原理
前端路由：  
路由模块的本质 就是建立起url和页面之间的映射关系，在单页面应用程序中，动态替换DOM内容并同步修改url地址。

### hash模式：  
使用 URL 的 hash 来模拟一个完整的 URL，只改变hash#后的部分。通过监听hashchange事件，监测hash值变化，实现更新页面部分内容。

### history模式：
利用HTML5 History API, pushState 和 replaceState，通过这两个 API 可以改变 url 地址且不会发送请求，同时还有popstate事件。

### hash模式和history模式的区别
* 一般使用场景没啥区别，他们俩也都可以使用浏览器的前进后退按钮。
* hisotry 模式需要配置服务器， 负责刷新页面可能会导致404
* hash 模式带#号，一般不能用来做分享的url，因为有的app里面url是不允许带有#号的

## vuex state、getter、mutation、action、module
state对象可以包含本应用全部的状态。在组件中通过this.$store.state来访问 
```js
const state = {
    testState: 'this is a state'
}
```
getter可以认为是 store 的计算属性。在组件中通过this.$store.getters来访问 
```js
const getters = {
    testGetter: (state) => {
        return state.testState + ' testGetters'
    }
}
```
更改 store 中的状态的唯一方法是提交 mutation  
mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数
```js
const mutations = {
    // ES6中属性命名也可以用 []，里面是表达式
    [types.TEST_MUTATION_CHANGE_STATE] (state, payload) {
        state.testState = payload.newState
    }
}

this.$store.commit('TEST_MUTATION_CHANGE_STATE', {
    newState: 'this is new state'
})
```
Action 类似于 mutation，不同在于：Action 提交的是 mutation，而不是直接变更状态； Action 可以包含任意异步操作。  
Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters
```js
testActionChangeState ({ commit }, payload) {
    setTimeout(() => {
        commit(types.TEST_MUTATION_CHANGE_STATE, {
            newState: payload.newState
        })
    }, 1000)
}

this.$store.dispatch('testActionChangeState', {
    newState: 'this is new state'
})
```

module可以将store分割成小模块
```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

## mapState mapGetters mapMutations mapActions
mapState 和 mapGetters 可以将state和getter映射成组件中的计算属性  
mapMutations 和 mapActions 可以映射成组件中的方法
```js
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'

computed: {
    ...mapState([
        'testState'
    ]),

    ...mapGetters([
        'testGetter'
    ])
},

methods: {
    ...mapMutations([
        'TEST_MUTATION_CHANGE_STATE'
    ]),
    ...mapActions([
        'testActionChangeState'
    ])
}
```

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

