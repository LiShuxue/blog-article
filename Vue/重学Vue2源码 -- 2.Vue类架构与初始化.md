> 基于vue@2.6.14源码，最后一个纯 vue2 版本源码，2.7 开始集成了 v3 相关 Composition API

## Vue 类

从打包入口来，我们一步步分析 Vue 类都做了什么

1. src/platforms/web/entry-runtime-with-compiler.js 是整个 Vue 打包的入口，导出了整个 Vue 类。这个文件中对 Vue 类也做了简单的修改，重写实例方法 `$mount` 和增加静态方法 `compile`。

2. 入口中导出的 Vue 类是从 src/platforms/web/runtime/index.js 导入的。这个文件中对 Vue 进行了一系列的修改，增加实例方法 `__patch__` 和 `$mount`，修改静态属性 `Vue.options` 和 `Vue.config`

3. src/platforms/web/runtime/index.js 中修改的 Vue 是从 src/core/index.js 中导入的。这个文件中又对 Vue 进行了一系列修改。比如 `initGlobalAPI(Vue)` ，会给 Vue 类添加大量的静态方法和属性。

4. src/core/index.js 中的 Vue 又是从 src/core/instance/index.js 中导入的。这个文件中定义了 Vue 类，并且给 Vue 类的原型链上挂载了一系列方法和属性，通过五个方法，逐步对 Vue 类的 prototype 进行修改，`initMixin(Vue)`，`stateMixin(Vue)`，`eventsMixin(Vue)`，`lifecycleMixin(Vue)`，`renderMixin(Vue)`。

但是 Vue 真正加载的时候，不是按上面的顺序，而是上面反过来的顺序。也就是先创建了 Vue 类，对原型 prototype 进行一系列修改，然后增加一系列静态方法和属性，接着又增加或者修改一些属性。最终导出整个 Vue 类。

## Vue 构造方法

创建 Vue 类，在实例化的时候，调用 `_init` 方法。

```js
// src/core/instance/index.js
function Vue(options) {
  this._init(options);
}
```

## Vue 实例方法 和 实例属性

Vue 类的实例方法和属性，通过实例或者 this 来调用，在 src/core/instance/index.js 文件中，通过多个方法来添加实例方法或者属性，`initMixin(Vue)`，`stateMixin(Vue)`，`eventsMixin(Vue)`，`lifecycleMixin(Vue)`，`renderMixin(Vue)`

```js
// initMixin(Vue) src/core/instance/init.js
Vue.prototype._init = function(options) {}

// stateMixin(Vue) src/core/instance/state.js
Vue.prototype.$data
Vue.prototype.$props
Vue.prototype.$set = function() {}
Vue.prototype.$delete = function() {}
Vue.prototype.$watch = function() {}

// eventsMixin(Vue) src/core/instance/events.js
Vue.prototype.$on = function() {}
Vue.prototype.$once = function() {}
Vue.prototype.$off = function() {}
Vue.prototype.$emit = function() {}

// lifecycleMixin(Vue) src/core/instance/lifecycle.js
Vue.prototype._update = function() {}
Vue.prototype.$forceUpdate = function() {}
Vue.prototype.$destroy = function() {}

// renderMixin(Vue) src/core/instance/render.js
installRenderHelpers() // 将render方法中使用到的工具方法全部挂在Vue.prototype上
Vue.prototype.$nextTick = function() {}
Vue.prototype._render = function() {}

// src/platforms/web/runtime/index.js
Vue.prototype.__patch__ = = function() {}

// src/platforms/web/entry-runtime-with-compiler.js 和 src/platforms/web/runtime/index.js
Vue.prototype.$mount = function() {}
```

## Vue 静态方法 和 静态属性

Vue 类的静态方法和属性，通过 Vue 类直接调用，无需实例化。其中大部分是由 `initGlobalAPI` 方法挂载

```js
// initGlobalAPI(Vue)  src/core/global-api/index.js
Vue.config;
Vue.util;
Vue.set = function () {};
Vue.delete = function () {};
Vue.nextTick = function () {};
Vue.observable = function () {};
Vue.options;
Vue.options._base = Vue;
Vue.use = function () {};
Vue.mixin = function () {};
Vue.extend = function () {};
Vue.component = function () {};
Vue.directive = function () {};
Vue.filter = function () {};

// src/core/index.js
Vue.version;

// src/platforms/web/entry-runtime-with-compiler.js
Vue.compile = function () {};
```

## Vue 的初始化

通过上面的构造方法，可以看到，当初始化 Vue 实例的时候，会去执行 init 方法。

init 方法中会给该实例逐步添加很多属性，比如$options, $events, $attrs, $listeners, \_vnode, \_data, \_computedWatchers 等等，同时会把所有 data 或者 compute 中定义的变量挂在实例上，把 methods 中定义的方法也挂在实例上。

最后渲染整个 DOM 树到 el 节点。

```js
// src/core/instance/init.js
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

  vm._renderProxy = vm;
  vm._self = vm;
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

## 组件的初始化

Vue 定义组件的方式有两种，全局注册组件，或者局部注册组件。

单文件组件是一种常见的局部组件实现方式，但是局部组件并不一定要使用单文件组件的方式来实现，可以直接定义对象。

```js
// 全局注册
Vue.component("my-component-name", {
  /* ... */
});

// 局部注册
const ComponentA = {
  /* ... */
};

new Vue({
  el: "#app", // el 只在用 new 创建实例时生效，局部组件中不能有 el，只能是 template。
  components: {
    "component-a": ComponentA,
  },
});
```

### 全局组件初始化

所以组件的初始化入口也会有两个。我们先看下全局的 Vue.component 方法。

在 initGlobalAPI 函数中，有下面两个代码。

```js
// src/core/global-api/index.js
Vue.options._base = Vue;
initExtend(Vue);
initAssetRegisters(Vue);
```

initAssetRegisters 方法就是给 Vue 类添加注册全局组件，指令，过滤器的方法。

```js
// src/core/global-api/assets.js
export function initAssetRegisters(Vue) {
  ["component", "directive", "filter"].forEach((type) => {
    Vue[type] = function (id, definition) {
      if (type === "component" && isPlainObject(definition)) {
        definition.name = definition.name || id;
        // this.options._base.extend 即为 Vue.extend
        definition = this.options._base.extend(definition);
      }
      if (type === "directive" && isFunction(definition)) {
        definition = { bind: definition, update: definition };
      }
      this.options[type + "s"][id] = definition;
      return definition;
    };
  });
}
```

可以看到 Vue.component 方法就是调用的 Vue.extend 方法。在这个方法中声明了一个 Vue 子类 VueComponent，继承自 Vue。以后子类实例化的时候，也会调\_init 方法，这个方法会从父级 Vue 中继承过来。所以后面的逻辑跟我们上面分析的 Vue 初始化时的基本一致了。

```js
// src/core/global-api/extend.js
Vue.extend = function (extendOptions) {
  extendOptions = extendOptions || {};
  // Vue调用的extend，所以当前this指向Vue
  const Super = this;

  // 创建Vue的子类Sub
  const Sub = function VueComponent(options) {
    this._init(options);
  };
  // 子类Sub继承Vue原型链上的方法
  Sub.prototype = Object.create(Super.prototype);
  Sub.prototype.constructor = Sub;

  // 子类添加静态属性options 和 super。 合并子类接收的options和Vue的全局方法。
  Sub.options = mergeOptions(Super.options, extendOptions);
  Sub["super"] = Super;

  // 将子类的props 和 computed属性全部挂在Sub.prototype上，为了以后实例化子类的时候，可以从实例上取出来这些属性。
  if (Sub.options.props) {
    initProps(Sub);
  }
  // 计算属性initComputed时会通过defineComputed来声明计算属性的getter，带有缓存性，详见：响应式原理计算属性分析那篇。
  if (Sub.options.computed) {
    initComputed(Sub);
  }

  // 继承Vue的方法
  Sub.extend = Super.extend;
  Sub.mixin = Super.mixin;
  Sub.use = Super.use;

  // 子类的compnents，filter, directive方法也继承父类
  ASSET_TYPES.forEach(function (type) {
    Sub[type] = Super[type];
  });

  return Sub;
};
```

### 局部组件初始化

当我们写了局部组件并在 template 中使用的时候

```html
<div>
  <my-other-comp />
</div>
```

渲染的时候 template 最会被转换成 render 函数，render 函数执行后会生成虚拟 DOM。render 函数精简以后

```js
function anonymous() {
  with (this) {
    return _c("div", [_c("my-other-comp")], 1);
  }
}
```

\_c 其实就是 createElement 方法，这被定义在 initRender 方法中

```js
// src/core/instance/render.js
export function initRender(vm) {
  // ...
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false);
  // ...
}
```

createElement 的时候，会判断这个是否是自定义组件（不是 html 的 tag），如果是的话，获取到自定义组件（就是注册的那个对象），然后调用创建组件的方法 createComponent 去创建该组件。

```js
// src/core/vdom/create-element.js
export function createElement(
  context,
  tag,
  data,
  children,
  normalizationType,
  alwaysNormalize
) {
  // ...
  return _createElement(context, tag, data, children, normalizationType);
  // ...
}

export function _createElement(
  context,
  tag,
  data,
  children,
  normalizationType
) {
  // ...
  // 获取到自定义组件那个大对象 { /* ... */ }
  Ctor = resolveAsset(context.$options, "components", tag);
  vnode = createComponent(Ctor, data, context, children, tag);
  // ...
}
```

createComponent 方法内部最终也是调用 Vue.extend 方法来初始化这个局部组件。Vue.extend 方法上面已经介绍过了。

```js
// src/core/vdom/create-component.js
export function createComponent(Ctor, data, context, children, tag) {
  // ...
  // 获取到Vue类
  const baseCtor = context.$options._base;
  // 调用Vue.extend方法
  Ctor = baseCtor.extend(Ctor);
  // ...
}
```
