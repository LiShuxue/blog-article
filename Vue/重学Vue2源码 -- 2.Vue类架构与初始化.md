> 基于vue@2.7.14源码

## Vue 类

从打包入口来，我们一步步分析 Vue 类都做了什么

1. src/platforms/web/entry-runtime-with-compiler.ts 是整个 Vue 打包的入口，导出了整个 Vue 类

2. 入口中导出的 Vue 类是从 src/platforms/web/runtime-with-compiler.ts 导入的。这个文件中对 Vue 类简单的修改，重写实例方法 `$mount` 和增加静态方法 `compile`。

3. runtime-with-compiler.ts 中修改的 Vue 是从 src/platforms/web/runtime/index.ts 导入的。这个文件中对 Vue 进行了一系列的修改，增加实例方法 `__patch__` 和 `$mount`，修改静态属性 `Vue.options` 和 `Vue.config`

4. src/platforms/web/runtime/index.ts 中修改的 Vue 是从 src/core/index.ts 中导入的。这个文件中又对 Vue 进行了一系列修改。比如 `initGlobalAPI(Vue)` ，会给 Vue 类添加大量的静态方法和属性。

5. src/core/index.ts 中的 Vue 又是从 src/core/instance/index.ts 中导入的。这个文件中定义了 Vue 类，并且给 Vue 类的原型链上挂载了一系列方法和属性，通过五个方法，逐步对 Vue 类的 prototype 进行修改，`initMixin(Vue)`，`stateMixin(Vue)`，`eventsMixin(Vue)`，`lifecycleMixin(Vue)`，`renderMixin(Vue)`。

但是 Vue 真正加载的时候，不是按上面的顺序，而是上面反过来的顺序。也就是先创建了 Vue 类，对原型 prototype 进行一系列修改，然后增加一系列静态方法和属性，接着又增加或者修改一些属性。最终导出整个 Vue 类。

## Vue 构造方法

创建 Vue 类，在实例化的时候，调用 `_init` 方法。

```js
// src/core/instance/index.ts
function Vue(options) {
  this._init(options);
}
```

## Vue 实例方法 和 实例属性

Vue 类的实例方法和属性，通过实例或者 this 来调用，在 src/core/instance/index.ts 文件中，通过多个方法来添加实例方法或者属性，`initMixin(Vue)`，`stateMixin(Vue)`，`eventsMixin(Vue)`，`lifecycleMixin(Vue)`，`renderMixin(Vue)`

```js
// initMixin(Vue) src/core/instance/init.ts
Vue.prototype._init = function(options) {}

// stateMixin(Vue) src/core/instance/state.ts
Vue.prototype.$data
Vue.prototype.$props
Vue.prototype.$set = function() {}
Vue.prototype.$delete = function() {}
Vue.prototype.$watch = function() {}

// eventsMixin(Vue) src/core/instance/events.ts
Vue.prototype.$on = function() {}
Vue.prototype.$once = function() {}
Vue.prototype.$off = function() {}
Vue.prototype.$emit = function() {}

// lifecycleMixin(Vue) src/core/instance/lifecycle.ts
Vue.prototype._update = function() {}
Vue.prototype.$forceUpdate = function() {}
Vue.prototype.$destroy = function() {}

// renderMixin(Vue) src/core/instance/render.ts
Vue.prototype.$nextTick = function() {}
Vue.prototype._render = function() {}

// src/platforms/web/runtime/index.ts
Vue.prototype.__patch__ = = function() {}

// src/platforms/web/runtime-with-compiler.ts 和 src/platforms/web/runtime/index.ts
Vue.prototype.$mount = function() {}
```

## Vue 静态方法 和 静态属性

Vue 类的静态方法和属性，通过 Vue 类直接调用，无需实例化。其中大部分是由 `initGlobalAPI` 方法挂载

```js
// initGlobalAPI(Vue)  src/core/global-api/index.ts
Vue.config;
Vue.util;
Vue.set = function () {};
Vue.delete = function () {};
Vue.nextTick = function () {};
Vue.observable = function () {};
Vue.options;
Vue.use = function () {};
Vue.mixin = function () {};
Vue.cid;
Vue.extend = function () {};
Vue.component = function () {};
Vue.directive = function () {};
Vue.filter = function () {};

// src/core/index.ts
Vue.version;

// src/platforms/web/runtime-with-compiler.ts
Vue.compile = function () {};
```

## Vue 的初始化

通过上面的构造方法，可以看到，当初始化 Vue 实例的时候，会去执行 init 方法。

init 方法中会给该实例逐步添加很多属性，比如$options, $events, $attrs, $listeners, \_vnode, \_data, \_computedWatchers 等等，同时会把所有 data 或者 compute 中定义的变量挂在实例上，把 methods 中定义的方法也挂在实例上。

最后渲染整个 DOM 树到 el 节点。

```js
let uid = 0;
Vue.prototype._init = function (options) {
  const vm = this;
  // 每一个实例是一个不同的uid
  vm._uid = uid++;
  // 合并options，挂在实例上$options
  if (options && options._isComponent) {
    // 对于vue组件的options，增加parent相关的信息
    initInternalComponent(vm, options);
  } else {
    // 对于vue实例的，会将我们直接挂在vue类上的一些东西合并进来
    vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    );
  }

  // 创建用于判断生命周期的一些标志位和一些实例属性，方便后面的代码使用
  initLifecycle(vm);
  // 创建用于存储事件的实例属性，用于this.on或者组件模板上的on事件监听
  initEvents(vm);
  // 创建用于存储渲染后的节点的实例属性，给实例添加$attrs和$listeners
  initRender(vm);
  // 执行options中的beforeCreate方法
  callHook(vm, "beforeCreate");
  // 将inject的数据挂在实例上，且变为响应式的
  initInjections(vm); // resolve injections before data/props
  // 初始化methods, data, computed 和 watch
  // 初始化方法：将方法挂在实例上
  // 初始化data：将data中的数据，全部挂到实例上，并且observe整个data
  // 初始化computed：将所有计算属性，挂在实例_computedWatchers属性上，并且给计算属性创建订阅者watcher
  // 初始化watch：给所有被观察的属性，调用Vue.prototype.$watch方法，创建一个订阅者watcher
  initState(vm);
  // 将父级的provide和本身的provide结合
  initProvide(vm); // resolve provide after data/props
  // 执行options中的created方法
  callHook(vm, "created");

  // 只有整个vue实例的时候，才需要渲染，子组件其实都在虚拟dom树中了
  if (vm.$options.el) {
    vm.$mount(vm.$options.el);
  }
};
```
