> 基于vue@2.6.14源码，最后一个纯 vue2 版本源码，2.7 开始集成了 v3 相关 Composition API

## 响应式

当 data, props 或者 computed 中的数据改变时, Vue 会做出对应的响应，比如执行事件，更新页面等。具体干什么取决于订阅这个变化的订阅者，也就是 Watcher (后面会介绍)。

具体的实现主要是 Vue 实例初始化时的 \_init 方法中会调用 initState 方法，这个方法会调用其他不同的方法，依次将 props，data，computed 属性全部变为响应式的，且会初始化所有的 methods 和 watch。

initState 精简以后：

```js
// src/core/instance/state.js
export function initState(vm) {
  vm._watchers = [];
  const opts = vm.$options;
  if (opts.props) initProps(vm, opts.props);
  if (opts.methods) initMethods(vm, opts.methods);
  if (opts.data) initData(vm);
  if (opts.computed) initComputed(vm, opts.computed);
  if (opts.watch) initWatch(vm, opts.watch);
}
```

## props 的初始化

实例在初始化后就有$props属性，这是因为源码中在定义Vue类的时候，就给Vue.prototype上增加了该属性。$data 也一样。默认没有 set 只有 get。在非生产环境下，不允许修改 props。

```js
// src/core/instance/state.js
const dataDef = {};
dataDef.get = function () {
  return this._data;
};
const propsDef = {};
propsDef.get = function () {
  return this._props;
};
Object.defineProperty(Vue.prototype, "$data", dataDef);
Object.defineProperty(Vue.prototype, "$props", propsDef);
```

在\_init 方法中合并 options 的时候，会将 props 中的数据，挂在实例的$options.propsData 中，值是传进来的值。

在 initProps 方法中，我们会遍历组件的整个 props 对象，将每个属性挂在实例的\_props 属性上，并且变成响应式的。如果实例上不存在（在创建组件类的时候，就会将 props 挂在组件类的 prototype 上），也会将属性和值再直接挂在实例上，方便代码中通过 this.xxx 调用。

```js
// src/core/instance/state.js
function initProps(vm, propsOptions) {
  // propsData是当前props传进来的值
  const propsData = vm.$options.propsData || {};
  // 创建实例的_props属性
  const props = (vm._props = {});
  // propsOptions是组件中定义的props，包含类型。遍历所有的propsOptions
  for (const key in propsOptions) {
    // 获取真正应该的值，通过外面传的和本身的默认值等计算出来。
    const value = validateProp(key, propsOptions, propsData, vm);
    // 将属性和值，挂在props对象上，同时也会挂在vm._props上，因为引用的。并且变成响应式的。
    defineReactive(props, key, value);
    // 如果实例上不存在，将属性和值再挂在实例上
    if (!(key in vm)) {
      proxy(vm, `_props`, key);
    }
  }
}
```

## methods 的初始化

将所有的方法，全部直接挂在实例上，方便代码中直接用 this.xxfn() 来调用。

```js
// src/core/instance/state.js
function initMethods(vm, methods) {
  for (const key in methods) {
    vm[key] =
      typeof methods[key] !== "function" ? noop : bind(methods[key], vm);
  }
}
```

## data 的初始化

### initData

将 data 所有的属性直接挂在实例上，方便代码中直接用 this.xxx 使用。并且通过 observe 方法，将 data 全部变为响应式的。

```js
// src/core/instance/state.js
function initData(vm) {
  let data = vm.$options.data;
  data = vm._data = typeof data === "function" ? getData(data, vm) : data || {};
  const keys = Object.keys(data);
  let i = keys.length;
  while (i--) {
    const key = keys[i];
    // 将data所有属性全部挂在实例上，方便代码中通过this.xxx调用
    proxy(vm, `_data`, key);
  }
  // 将data变成响应式的
  observe(data);
}
```

### proxy

proxy 方法的作用就是将 options 中的不同模块的对象属性直接挂在实例上，比如 data, props

```js
// src/core/instance/state.js
export function proxy(target, sourceKey, key) {
  sharedPropertyDefinition.get = function proxyGetter() {
    // 返回源对象中的值
    return this[sourceKey][key];
  };
  sharedPropertyDefinition.set = function proxySetter(val) {
    // 给源对象赋值
    this[sourceKey][key] = val;
  };
  // 给目标对象，也就是组件实例，添加对应的属性
  Object.defineProperty(target, key, sharedPropertyDefinition);
}
```

### observe

observe 方法主要是通过 new 一个 Observer 对象，来观察目标对象 data 的所有属性。Observer 类在实例化的时候，首先会 new 一个 Dep 实例，主要是监听这个数组或者对象的本身，后面如果主动通过 set 等方法修改对象或者数组的时候可以做出反应。然后遍历目标对象所有属性，通过 defineReactive 方法让每一个属性变成响应式的。

```js
// src/core/observer/index.js
export function observe(value) {
  let ob;
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if ((Array.isArray(value) || isPlainObject(value)) && Object.isExtensible(value) &&) { // 只有对象或者数组才可以，基本数据类型的不生效
    ob = new Observer(value)
  }
  return ob
}

export class Observer {
  // 实例属性，可以直接定义在class中，不需要像 let、const 和 var 这样的关键字。
  // 以前都是定义在constructor中，通过 this.xxx 来定义
  value;
  dep;
  constructor(value) {
    this.value = value; // 存储被观察的对象
    // 创建一个任务调度类的实例
    this.dep = new Dep();
    // 通过 Object.defineProperty，给要观察的对象添加__ob__属性，值是自己这个observer实例。
    def(value, "__ob__", this);

    // 如果是数组，给数组的原型链上或者数组本身添加Vue内部重写后的那些方法，比如pop/push/shift等
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }

    // 将数据的每一项变成响应式的
    walk (obj) {
      const keys = Object.keys(obj)
      for (let i = 0; i < keys.length; i++) {
        defineReactive(obj, keys[i])
      }
    }

    // 递归数组的每一项
    observeArray (items) {
      for (let i = 0, l = items.length; i < l; i++) {
        observe(items[i])
      }
    }
  }
}
```

### defineReactive

通过 Object.defineProperty，给目标对象的所有属性设置 get 和 set 方法，在 get 的时候去收集依赖，在 set 的时候去通知更新。对象的每一个属性都有一个 dep 实例。

调用 get 的阶段（依赖收集的阶段）：

- 初始化 watch 的时候，会 new Watcher，在 Watcher 的构造函数中调用 watch 的属性的 getter，触发该属性 get。
- 渲染时，模板被转化成 render 函数后，在 render 函数中会调用该属性，会触发该属性 get。
- 模板中访问计算属性的时候，调用计算属性的 getter，计算属性中主动调用 depend 方法，完成依赖收集。

如果只是在纯 js 中使用，比如 this.xx = xx，但是这个属性没有在视图或者 watch 或者 computed 中，不会依赖收集。依赖收集在初始化阶段就完成了。

调用 set 的阶段（通知更新的阶段）：

- 主要就是直接修改数据时。
- 或者通过 set，del 方法修改数据时。
- 还有调用被重写的数组的七个方法时。

```js
// src/core/observer/index.js
export function defineReactive(obj, key, val) {
  // 针对每一个属性，创建一个dep实例
  const dep = new Dep();
  // 获取该属性原本的getter和setter
  const property = Object.getOwnPropertyDescriptor(obj, key);
  const getter = property && property.get;
  const setter = property && property.set;
  if ((!getter || setter) && arguments.length === 2) {
    // 获取这个属性的值
    val = obj[key];
  }
  // 递归这个值，如果值是对象或者数组的话，会被观察且返回一个Observer实例
  let childOb = observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      const value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        dep.depend(); // 依赖收集，当模板中调用该属性的时候，会被收集依赖
        if (childOb) {
          childOb.dep.depend(); // 也是依赖收集，这个属性值（数组或者对象）本身的来收集依赖，主要用来主动通过set等方法修改对象或者数组的监听
          if (Array.isArray(value)) {
            dependArray(value); // 递归数组的每一项，让他本身收集依赖，方便后面通过set等方法改变时做出监听
          }
        }
      }
      return value;
    },
    set: function reactiveSetter(newVal) {
      val = newVal;
      // 如果新的值是对象或者数组的话，也会被重新观察，赋予响应式
      childOb = observe(newVal);
      dep.notify(); // 属性被改变的时候发布通知，但是set方法的通知不在这，在set方法里面
    },
  });

  return dep;
}
```

### Dep

Dep 作为任务调度中心，会收集依赖，巧妙的添加订阅者，还有删除订阅者，以及通知订阅者更新等。

源码中有两处地方生成 Dep 实例，new Observer() 的时候，dep 主要管理这个对象本身增删属性或者数组本身增删的依赖收集和触发更新。还有将属性转变为响应式的时候，每个属性的 getter，setter 都依赖一个 dep，管理这个属性值依赖收集和触发更新。

```js
// src/core/observer/dep.js
export default class Dep {
  static target; // 订阅者在订阅那一刻，订阅者本身，用完会清除
  subs; // 存所有的订阅者

  constructor() {
    this.subs = [];
  }

  addSub(sub) {
    this.subs.push(sub);
  }

  // 当某些属性值不再被视图使用的时候，就应该取消掉对这些属性的订阅。
  removeSub(sub) {
    // 这里方便理解，我用了以前的写法，最新写法不易理解
    remove(this.subs, sub);
  }

  // 依赖收集的巧妙实现，通过订阅者本身的addDep方法，将调度中心的this传进去，里面调用调度中心的addSub来收集依赖
  depend() {
    if (Dep.target) {
      Dep.target.addDep(this);
    }
  }

  // 通知订阅者做出响应
  notify() {
    const subs = this.subs.slice();
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update(); // 订阅者做出动作
    }
  }
}
Dep.target = null;

// 对于父子组件而言，渲染函数存在一个嵌套的关系。也就是说父组件渲染到一半的时候，会去渲染子组建，如果此时不更换全局的watcher，那么依赖收集必然发生错误，因此使用了栈结构去保存渲染watcher。
const targetStack = [];
export function pushTarget(target) {
  targetStack.push(target);
  Dep.target = target; // 给target赋值
}
export function popTarget() {
  targetStack.pop();
  Dep.target = targetStack[targetStack.length - 1];
}
```

## computed 的初始化

遍历 computed options 中的每一个属性，获取到其 get 方法，并对应生成一个 watcher 实例。watcher 实例生成的时候，会调用 get 方法进行依赖收集。对于 vue 实例的计算属性，重新定义在实例上，且是带有缓存性的。

将这些计算属性的 watcher 用\_computedWatchers 来存储。

缓存的实现主要通过dirty属性，当数据被访问时，如果dirty是true，就重新计算，否则就直接返回值。当数据被更新时，dirty会被赋值为true，当数据计算完后，又被赋值为false。
```js
// src/core/instance/state.js
function initComputed (vm, computed)
  // 将计算属性的watcher用_computedWatchers来存储
  const watchers = vm._computedWatchers = Object.create(null)

  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    const computedWatcherOptions = { lazy: true }

    // 创建watcher，在watcher实例化的时候，会
    watchers[key] = new Watcher(
      vm,
      getter || noop,
      noop,
      computedWatcherOptions
    )

    // 将计算属性都直接挂在实例上，方便this.xxx调用，而子组件的计算属性，在创建子组件类的时候就挂在了Sub.prototype上
    if (!(key in vm)) {
      defineComputed(vm, key, userDef) // 只有vue实例初始化才会执行这个
    }
  }
}

// 只有vue实例初始化才会执行这个
export function defineComputed (target, key, userDef) {
  sharedPropertyDefinition.get = createComputedGetter(key) // 创建一个getter
  sharedPropertyDefinition.set = userDef.set || noop
  Object.defineProperty(target, key, sharedPropertyDefinition)
}

function createComputedGetter (key) {
  return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      if (watcher.dirty) { // 当watcher执行过一次之后，下次再获取值的时候，不再重新执行，这就是compute缓存
        watcher.evaluate() // 执行获取value
      }
      if (Dep.target) { // 如果当前有任务调度中心，也会收集依赖，也就是计算属性第一次访问getter时的依赖收集
        watcher.depend()
      }
      return watcher.value
    }
  }
}
```

## watch 的初始化

遍历 watch options 中的每一个属性，对 watch 中的每一个属性创建一个 watcher，expOrFn 就是监听的属性，cb 就是 watch 的回调

```js
function initWatch(vm, watch) {
  for (const key in watch) {
    const handler = watch[key];
    createWatcher(vm, key, handler);
  }
}

function createWatcher(vm, expOrFn, handler, options) {
  // watch的两种写法
  /*
  watch: {
    message: 'handler'
  },
  methods: {
    handler (newVal, oldVal) { ... }
  }
  */
  /*
  watch: {
    message: function (newVal, oldVal) { ... }
  }
  */
  // 如果handler是对象，直接取对象中的
  if (isPlainObject(handler)) {
    options = handler;
    handler = handler.handler;
  }
  // 如果是字符串，取实例上的，methods中的方法都会挂在实例上
  if (typeof handler === "string") {
    handler = vm[handler];
  }
  return vm.$watch(expOrFn, handler, options);
}

Vue.prototype.$watch = function (expOrFn, cb, options) {
  const vm = this;
  options = options || {};
  options.user = true; // watch 的 watcher 设置user是true
  const watcher = new Watcher(vm, expOrFn, cb, options); // 创建watcher，依赖收集
  if (options.immediate) {
    pushTarget();
    invokeWithErrorHandling(cb, vm, [watcher.value], vm, info);
    popTarget();
  }
};
```

## 其他

### Watcher

Watcher 就是订阅者，new Watcher 的构造方法中，调用 get，来收集依赖，并且移除旧的 watcher。当数据被改变后，触发 watcher 的 update 方法，使订阅者做出相应的动作。

订阅者基本就是三个：

- mount 的时候，mountComponent 方法中会 new Watcher 实例，用来更新页面
- watch 的属性会创建 watcher 实例
- 计算属性会创建 watcher 实例

```js
// src/core/observer/watcher.js
export default class Watcher {
  vm; // 组件实例
  cb; // 观察到变更后的回调，只有watch的属性创建的watcher实例才有
  id; // 用于批量处理
  deep; // 深度监听，用于watch
  user; // user是true表示这是一个watch侦听器
  deps; // 2 个 Dep 实例数组，newDeps 表示新添加的 Dep 实例数组，而 deps 表示上一次添加的 Dep 实例数组。
  newDeps;
  depIds;
  newDepIds;
  before; // 渲染前执行beforeUpdated
  lazy; // lazy是true表示是计算属性
  dirty; // 用于计算属性，表示是否已经获取过一次值了，如果是，下次获取的时候，直接取值
  getter; // getter方法用来获取当前时刻下 expOrFn 获取到的值
  value; // 存储这个 expOrFn 获取到的值，主要用于计算属性

  constructor(vm, expOrFn, cb, options, isRenderWatcher) {
    this.vm = vm;
    if (isRenderWatcher) {
      vm._watcher = this; // 模板渲染的watcher，挂在实例上
    }
    vm._watchers.push(this); // 将所有watcher放在_watchers中

    // 初始化各种数据
    this.deep = !!options.deep;
    this.user = !!options.user;
    this.dirty = this.lazy;
    this.cb = cb;
    this.id = ++uid; // uid for batching
    this.deps = [];
    this.newDeps = [];
    this.depIds = new Set();
    this.newDepIds = new Set();

    // expOrFn 就是真正的watcher的表达式, 比如计算属性的getter方法，或者创建watch监听器时所监听的表达式或者属性，如msg，obj.msg等
    // 还有就是渲染组件的函数
    // updateComponent = () => {
    //   vm._update(vm._render())
    // }
    if (typeof expOrFn === "function") {
      this.getter = expOrFn;
    } else {
      // parsePath会将表达式转换为目标对象调用该属性的方法，返回这个方法
      this.getter = parsePath(expOrFn);
    }

    // 初始化Watcher的时候，非计算属性，就会调用get方法，来收集依赖
    this.value = this.lazy ? undefined : this.get();
  }

  // 主动触发getter方法，方便后面进行依赖收集
  get() {
    pushTarget(this); // Dep.target = this
    const vm = this.vm;
    let value = this.getter.call(vm, vm);
    if (this.deep) {
      traverse(value); // 递归访问每一个属性，则这些属性都会被收集依赖
    }
    popTarget();
    this.cleanupDeps();
    return value;
  }

  // 在Object.defineProperty定义的getter中，进行依赖收集，调用Dep.target.addDep(this)，就是下面的方法
  addDep(dep) {
    const id = dep.id; // 获取任务调度中心dep的id，因为每一个属性都有唯一一个dep
    if (!this.newDepIds.has(id)) {
      // 这个数据可能在页面中，或者在计算属性中好几个地方都使用到，所以保证同一数据不会被添加多次
      this.newDepIds.add(id); // 将id放到newDepIds
      this.newDeps.push(dep); // 将依赖放到newDeps
      if (!this.depIds.has(id)) {
        // depIds也是放的任务调度中心的id
        dep.addSub(this); // 将这个watcher添加到这个属性的任务调度中心dep的subs中，一个属性可能对应很多watcher
      }
    }
  }

  // 每次数据变化都会重新 render，那么 vm._render() 方法又会再次执行，并再次触发数据的 getters，所以 Watcher 在构造函数中会初始化 2 个 Dep 实例数组，newDeps 表示新添加的 Dep 实例数组，而 deps 表示上一次添加的 Dep 实例数组。
  // 所以每次调用get后，执行cleanupDeps方法，先清空旧的deps，newDepIds里永远存的每次渲染时最新的依赖
  cleanupDeps() {
    let i = this.deps.length;
    while (i--) {
      const dep = this.deps[i];
      if (!this.newDepIds.has(dep.id)) {
        // 如果第二次渲染时，没有旧的调度中心，就是说模板中不再依赖这个属性，就将这个dep中的subs移除
        dep.removeSub(this); // 将这个dep中的subs移除，防止浪费，因为我们已经没有地方对这个属性做监听了，所以即使属性变化，也不需要通知任何watcher了
      }
    }
    // 数据交换，将newDeps赋给deps，并且清空newDeps
    let tmp = this.depIds;
    this.depIds = this.newDepIds;
    this.newDepIds = tmp;
    this.newDepIds.clear();
    tmp = this.deps;
    this.deps = this.newDeps;
    this.newDeps = tmp;
    this.newDeps.length = 0;
  }

  // 当数据被赋值的时候，执行dep.notify，notify中遍历所有的watcher，执行update方法
  update() {
    if (this.lazy) {
      this.dirty = true; // 数据变化之后，对于计算属性的，将dirty置为true，下次访问这个计算属性的值时，再重新计算
    } else {
      queueWatcher(this); // 并不是每次update时都要立即更新模板，我们会先将这个watcher推入到队列，然后在 nextTick 后执行 flushSchedulerQueue。
    }
  }

  // 执行真正的更新
  run() {
    const value = this.get(); // 调用get方法，来执行这个watcher的getter，也就是expOrFn真正的方法，如果是更新渲染，就更新了页面，如果是watch，获取当前最新的值，并执行下面的回调
    if (value !== this.value) {
      const oldValue = this.value; // 设置新值和旧值
      this.value = value;
      if (this.user) {
        // 如果是watch的，执行watch后的回调
        invokeWithErrorHandling(this.cb, this.vm, [value, oldValue], this.vm);
      } else {
        this.cb.call(this.vm, value, oldValue);
      }
    }
  }

  // 计算属性的getter执行的时候，会调用该方法
  evaluate() {
    this.value = this.get();
    this.dirty = false;
  }

  // 访问计算属性时，完成依赖收集
  depend() {
    let i = this.deps.length;
    while (i--) {
      this.deps[i].depend();
    }
  }
}
```

### queueWatcher

当数据改变时，通知 watcher 更新，update 的时候并不是每次都立即执行真正的更新，比如 DOM 渲染，或者 watch 的回调，而是先推到一个队列中

```js
export function queueWatcher(watcher) {
  const id = watcher.id;
  // has是一个对象 {}，如果没有当前的watcher，再将watcher放到queue队列
  if (has[id] == null) {
    has[id] = true;
    if (!flushing) {
      // 如果当前watcher队列没有在执行，就放进队列
      queue.push(watcher);
    } else {
      // 如果当前watcher队列正在执行
      let i = queue.length - 1;
      while (i > index && queue[i].id > watcher.id) {
        // index代表正在执行的队列中的watcher的位置，如果准备插入的位置的id大于当前watcher id，就往前移一位。
        i--;
      }
      queue.splice(i + 1, 0, watcher); // 将watcher插入到合适的位置，就是队列中正在执行的watcher的后面，如果没有找到合适位置，就是插入到队列的尾部。方便一会儿执行。
    }
    // waiting变量是为了保证当前只执行一次flushSchedulerQueue逻辑。
    if (!waiting) {
      waiting = true;
      nextTick(flushSchedulerQueue);
    }
  }
}
```

### nextTick

nextTick 会接收回调函数，先放入一个数组中，然后定义一个批次清空数组的函数。这个批次清空的函数会被放入浏览器的微任务队列中。

将这个函数放入微任务或则宏任务队列，主要借助 Promise.resolve().then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0)。

当微任务执行到这个方法时，就会执行所有的 nextTick 接收的回调函数。

```js
const callbacks = [];
let pending = false;
let timerFunc;

if (typeof Promise === "function" && /native code/.test(Promise.toString())) {
  // 原生有Promise，用then的微任务执行
  timerFunc = () => {
    Promise.resolve().then(flushCallbacks); // Promise.resolve()会立即执行，将then的回调放入微任务队列
  };
} else if (
  typeof MutationObserver === "function" &&
  /native code/.test(MutationObserver.toString())
) {
  // 没有Promise就用MutationObserver，也是微任务
  const observer = new MutationObserver(flushCallbacks); // 监控DOM节点的变化，监听到变化后的将回调放入微任务队列
  const textNode = document.createTextNode(String(counter)); // 创建一个需要监听的节点
  observer.observe(textNode, {
    // 监控这个节点
    characterData: true,
  });
  let counter = 0;
  timerFunc = () => {
    counter = (counter + 1) % 2; // 主动触发变化，然后会将回调的flushCallbacks放入微任务队列
    textNode.data = String(counter);
  };
} else {
  // 实在不行就用setTimeout，宏任务
  timerFunc = setTimeout(flushCallbacks, 0);
}

function flushCallbacks() {
  pending = false;
  const copies = callbacks.slice(0);
  callbacks.length = 0; // 清空
  for (let i = 0; i < copies.length; i++) {
    copies[i]();
  }
}

export function nextTick(cb, ctx) {
  // push一个函数，函数中再去执行这个真正的回调cb
  callbacks.push(() => {
    cb.call(ctx);
  });
  if (!pending) {
    pending = true;
    timerFunc();
  }
}
```

### flushSchedulerQueue

在事件循环中，一次性执行队列中所有的 watcher，如果再执行 watcher 队列的过程中增加了新的 watcher，就添加到后面

```js
function flushSchedulerQueue() {
  flushing = true;
  let watcher, id;

  // 将所有watcher按从小到大排序（先创建的watcher id最小），可以保证
  // 1. 父组件先更新，子组件后更新
  // 2. watch 的执行先于 render方法
  // 4. 如果组件watcher 执行的时候，子组件销毁了的话，子组件的watcher可以跳过
  queue.sort((a, b) => a.id - b.id);

  for (index = 0; index < queue.length; index++) {
    watcher = queue[index];
    if (watcher.before) {
      watcher.before(); // 执行beforeUpdate方法
    }
    id = watcher.id;
    has[id] = null; // 清掉watcher
    watcher.run(); // 执行更新或者watch回调
  }

  // 清空一些状态和存储
  index = queue.length = 0;
  has = {};
  waiting = flushing = false;

  // 触发一些hoos
  callActivatedHooks(activatedQueue);
  callUpdatedHooks(updatedQueue);
}
```

### $set

通过 set，去改变数组的话，内部通过调用重写的数组方法来触发更新，改变对象的话，内部主要通过主动触发 notify，来通知更新。

```js
// src/core/observer/index.js
export function set(target, key, val) {
  // 数组通过重写后的数组方法
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1, val);
    return val;
  }
  // 对象如果本身含有这个key，直接更新
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val;
  }
  // 获取Observer实例，没有Observer实例说明不是响应式的，直接更新值
  const ob = target.__ob__;
  if (!ob) {
    target[key] = val;
    return val;
  }

  // ob.value是原本被观察的对象，添加一个响应式的新的key
  defineReactive(ob.value, key, val);
  // 主动触发通知，通过observer内部的dep对象
  ob.dep.notify();
  return val;
}
```
