## 响应式

当 data, props 或者 computed 中的数据改变时, Vue 会做出对应的响应，比如执行事件，更新页面等。具体干什么取决于订阅这个变化的订阅者，也就是 Watcher (后面会介绍)。

具体的实现主要是 Vue 实例初始化时的 \_init 方法中会调用 initState 方法，这个方法会调用其他不同的方法，依次将 props，data，computed 属性全部变为响应式的，且会初始化所有的 methods 和 watch。

initState精简以后：

```js
// src/core/instance/state.ts 
export function initState(vm) {
  const opts = vm.$options;
  if (opts.props) initProps(vm, opts.props);
  if (opts.methods) initMethods(vm, opts.methods);
  if (opts.data) initData(vm);
  if (opts.computed) initComputed(vm, opts.computed);
  if (opts.watch ) initWatch(vm, opts.watch);
}
```

## props的实现原理
实例在初始化后就有$props属性，这是因为源码中在定义Vue类的时候，就给Vue.prototype上增加了该属性。$data也一样。默认没有set只有get。
```js
const dataDef = {}
dataDef.get = function () {
  return this._data
}
const propsDef = {}
propsDef.get = function () {
  return this._props
}
Object.defineProperty(Vue.prototype, '$data', dataDef)
Object.defineProperty(Vue.prototype, '$props', propsDef)
```

在\_init方法中合并options的时候，会将props中的数据，挂在实例的$options.propsData中，值是传进来的值。

在initProps方法中，我们会遍历组件的整个props对象，将每个属性挂在实例的_props属性上，并且变成响应式的。如果实例上不存在，也会将属性和值再挂在实例上（组件实例在生成的时候，实例上默认包含props那些属性）。
```js
function initProps(vm, propsOptions) {
  // propsData是当前props传进来的值
  const propsData = vm.$options.propsData || {}
  // 创建实例的_props属性
  const props = (vm._props = shallowReactive({}))
  // propsOptions是组件中定义的props，包含类型。遍历所有的propsOptions
  for (const key in propsOptions) {
    // 获取真正应该的值，通过外面传的和本身的默认值等计算出来。
    const value = validateProp(key, propsOptions, propsData, vm)
    // 将属性和值，挂在props对象上，同时也会挂在vm._props上，因为引用的。并且变成响应式的。
    defineReactive(props, key, value)
    // 如果实例上不存在，将属性和值再挂在实例上
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
}
```

## defineReactive & proxy 方法
