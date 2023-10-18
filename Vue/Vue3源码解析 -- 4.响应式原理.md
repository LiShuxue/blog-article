> 基于vue@3.3.4源码

## Vue3 响应式代码示例

Vue3 中可以使用 ref()，reactive()，computed() 等定义响应式，也可以使用 Vue2 的写法，data，computed 等选项定义响应式。如下所示，接下来我们将一个个分析他们是如何实现的。

```html
<script src="../../dist/vue.global.js"></script>

<!-- App组件的模板 -->
<script type="text/x-template" id="app">
  <div @click="click1">vue3 ref: {{name}}</div>
  <div @click="click2">vue3 reactive: {{obj.test}}</div>
  <div>vue3 computed方法: {{comput}}</div>
  <child :test="name" />
</script>
<!-- 子组件 -->
<script type="text/x-template" id="children">
  <div>vue3 props: {{test}}</div>
  <div @click="click3">vue2 data: {{testData}}</div>
  <div>vue2 computed: {{testCom}}</div>
</script>

<!-- app渲染的根节点 -->
<div id="demo"></div>

<script>
  const { createApp, ref, reactive, computed, watch } = Vue;

  const Child = {
    template: '#children',
    props: {
      test: {
        type: Number,
      },
    },

    data() {
      return {
        testData: 1,
      };
    },

    computed: {
      testCom() {
        return this.testData + 1;
      },
    },

    watch: {
      testData(newValue, oldValue) {
        console.log('vue2 watch 已触发', `testData 由 ${oldValue} --> ${newValue}`);
      },
    },

    methods: {
      click3() {
        this.testData = this.testData + 1;
      },
    },
  };
  // 根组件
  const App = {
    template: '#app',
    components: {
      Child,
    },
    setup() {
      // ref
      const name = ref(0);
      const click1 = () => {
        name.value++;
      };

      // reactive
      const obj = reactive({ test: 0 });
      const click2 = () => {
        obj.test++;
      };

      // computed
      const comput = computed(() => {
        return name.value + 1;
      });

      // watch
      watch(name, (newValue, oldValue) => {
        console.log('watch 已触发', `name 由 ${oldValue} --> ${newValue}`);
      });

      return {
        name,
        obj,
        click1,
        click2,
        comput,
      };
    },
  };
  const app = createApp(App);
  app.mount('#demo');
</script>
```

## setup 函数

使用 ref()，reactive()，computed() 等定义响应式时，需要定义在 setup 函数内。

根据上一节的分析，我们知道 setup 函数会在 setupStatefulComponent 方法中被 callWithErrorHandling 执行，进而 setup 函数内定义的 ref，reactive，computed 等方法都会执行，而且最终 setup 函数还要将值返回。如果 setup 的返回值是函数，则认为是渲染函数，如果是对象，将对象变为响应式的。

```js
function setupStatefulComponent(instance) {
  const { setup } = instance.type;
  if (setup) {
    const setupContext = (instance.setupContext = createSetupContext(instance));

    const setupResult = callWithErrorHandling(setup, instance, ErrorCodes.SETUP_FUNCTION, [
      instance.props,
      setupContext,
    ]);

    handleSetupResult(instance, setupResult, isSSR);
  } else {
    finishComponentSetup(instance, isSSR);
  }
}

function handleSetupResult(instance, setupResult) {
  if (isFunction(setupResult)) {
    instance.render = setupResult;
  } else if (isObject(setupResult)) {
    instance.setupState = proxyRefs(setupResult);
  }
  finishComponentSetup(instance);
}
```

### ref() 函数

ref()函数用来将一个基本数据类型的变量变为响应式，接收的值为 number, string, boolean 等。使用的时候要加 value。

ref 接收的参数是 object 时，内部调用 reactive 处理。

```js
const name = ref(0);
const click1 = () => {
  name.value++;
};
```

ref 的值使用时为什么要加 value 呢，我们来分析 ref 的源码，他是定义在 `packages/reactivity/src/ref.ts` 中的。

ref 函数会创建一个 RefImpl 的实例，也就是一个对象，这个对象提供了 value 的 set 和 get 方法，可以操作 value 属性。

对 value 进行 get 时，会调用 trackRefValue 收集依赖，对 value set 时会调用 triggerRefValue 触发更新。

```js
export function ref(value) {
  return createRef(value, false)
}

function createRef(rawValue, shallow) {
  // 如果已经是ref，直接返回
  if (isRef(rawValue)) {
    return rawValue
  }
  // 创建一个RefImpl的实例
  return new RefImpl(rawValue, shallow)
}

class RefImpl {
  let _value // 存 xx.value 的值
  let _rawValue // 存原始值

  dep = undefined // ref的依赖
  readonly __v_isRef = true // 标识是ref

  constructor(value, __v_isShallow) {
    this._rawValue = __v_isShallow ? value : toRaw(value)
    this._value = __v_isShallow ? value : toReactive(value)
  }

  // 使用 xx.value 时
  get value() {
    trackRefValue(this) // 收集依赖
    return this._value
  }

  // 使用 xx.value = xxx 时
  set value(newVal) {
    // 判断值是否改变，改变了就更新值，并触发依赖更新
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = newVal
      this._value = useDirectValue ? newVal : toReactive(newVal)
      triggerRefValue(this, newVal) // 触发依赖更新
    }
  }
}

// ref接收的参数是object时，内部调用reactive处理
export const toReactive = (value) => isObject(value) ? reactive(value) : value
```

### reactive() 函数

reactive()函数用来将一个对象或数组类型的变量变为响应式的。使用的时候可以直接 obj.xxx 使用，不可重新赋值或者解构，不然会丢失响应性。

```js
const obj = reactive({ test: 0 });
const click2 = () => {
  obj.test++;
};
```

reactive() 函数做了什么，我们接着看源码。他是定义在 `packages/reactivity/src/reactive.ts` 中的。

reactive() 方法会调用 createReactiveObject 来对目标对象创建一个 proxy 代理对象。根据目标对象的类型，是 Object，Array 还是 Map，Set，采用不同的 handler，我们主要分析处理 Object 的 baseHandlers，baseHandlers 传进来时是 mutableHandlers。

```js
export function reactive(target) {
  if (isReadonly(target)) {
    return target;
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers,
    reactiveMap
  );
}

function createReactiveObject(target, isReadonly, baseHandlers, collectionHandlers, proxyMap) {
  const targetType = getTargetType(target);
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  );
  return proxy;
}
```

mutableHandlers 就是创建 proxy 时传入的 handler，各种拦截器。主要拦截 get，set，deleteProperty，has，ownKyes 等。

在 get，has，ownKeys 中会调用 track 方法进行依赖收集，在 set，deleteProperty 方法中调用 trigger 方法去触发更新。

```js
export const mutableHandlers = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys,
};

const get = createGetter();
function createGetter(isReadonly = false, shallow = false) {
  return function get(target, key, receiver) {
    // 判断target是不是Array，如果是的话，判断key是不是['push', 'pop', 'shift', 'unshift', 'splice','includes', 'indexOf', 'lastIndexOf']中的一个，如果是的话就返回arrayInstrumentations中对应重写过后的方法。
    // 重写就是为了暂停调用这些API过程中触发的track(target, ..., length)，防止在某些情况下死循环。
    const targetIsArray = isArray(target);
    if (targetIsArray && hasOwn(arrayInstrumentations, key)) {
      return Reflect.get(arrayInstrumentations, key, receiver);
    }

    const res = Reflect.get(target, key, receiver);

    if (!isReadonly) {
      track(target, TrackOpTypes.GET, key); // 收集依赖
    }
    return res;
  };
}

const set = createSetter();
function createSetter(shallow = false) {
  return function set(target, key, value, receiver) {
    let oldValue = target[key];
    oldValue = toRaw(oldValue);
    value = toRaw(value);
    const result = Reflect.set(target, key, value, receiver);
    if (hasChanged(value, oldValue)) {
      trigger(target, TriggerOpTypes.SET, key, value, oldValue); // 触发更新
    }
    return result;
  };
}

function deleteProperty(target, key) {
  const hadKey = hasOwn(target, key);
  const oldValue = target[key];
  const result = Reflect.deleteProperty(target, key);
  if (result && hadKey) {
    trigger(target, TriggerOpTypes.DELETE, key, undefined, oldValue); // 触发更新
  }
  return result;
}

function has(target, key) {
  const result = Reflect.has(target, key);
  if (!isSymbol(key) || !builtInSymbols.has(key)) {
    track(target, TrackOpTypes.HAS, key); // 收集依赖，适用于模版中的 prop in Obj 语法
  }
  return result;
}

function ownKeys(target: object): (string | symbol)[] {
  track(target, TrackOpTypes.ITERATE, isArray(target) ? 'length' : ITERATE_KEY); // 收集依赖，适用于模版中的 for item in array 语法
  return Reflect.ownKeys(target);
}
```

### computed() 函数

computed() 函数就是创建一个计算属性的，接受一个函数，或者是带有 get，set 方法的对象，返回一个 ComputedRefImpl 实例，他是一个只读的响应式 ref 对象。可以通过 `.value` 获取值。

如果参数只是一个函数，这个函数就是 getter，如果是带有 get，set 方法的对象，get 方法就是 getter，set 方法就是 setter。

```js
export function computed(getterOrOptions) {
  let getter;
  let setter;

  const onlyGetter = isFunction(getterOrOptions);
  if (onlyGetter) {
    getter = getterOrOptions;
    setter = NOOP;
  } else {
    getter = getterOrOptions.get;
    setter = getterOrOptions.set;
  }

  const cRef = new ComputedRefImpl(getter, setter, onlyGetter);

  return cRef;
}
```

computed() 方法内部是调用 ComputedRefImpl 类创建的实例，ComputedRefImpl 和上面介绍的 RefImpl 差不多，他的实例也是一个包含 value 的对象。

计算属性被创建时，会调用 ReactiveEffect 创建一个副作用对象，ReactiveEffect 实例化时内部会自动调用计算属性的计算方法，这样就会访问到内部依赖的所有其他响应式属性，而这个副作用实例也会被其他响应式属性作为依赖收集起来。当被依赖的属性变化时，就会执行这个副作用实例的 scheduler，scheduler 里调用 triggerRefValue 来触发更新。

在计算属性被访问时，也就是 get 时，会调用 trackRefValue 来收集依赖项，也就是依赖这个计算属性的地方。当计算属性的依赖项变化时，会调用 triggerRefValue 来触发更新。

```js
export class ComputedRefImpl {
  let dep = undefined // 依赖

  let _value // 计算属性的当前值
  let effect // 副作用函数，用于执行计算属性的计算函数（getter）。

  let __v_isRef = true
  let [ReactiveFlags.IS_READONLY] = false

  let _dirty = true // 计算属性是否需要重新计算

  constructor(getter, _setter, isReadonly) {
    this.effect = new ReactiveEffect(getter, () => {
      if (!this._dirty) { // computed的依赖项的值变化后，副作用函数中再将_dirty设置回true
        this._dirty = true
        triggerRefValue(this) // 触发更新
      }
    })
    this.effect.computed = this
    this.effect.active = true
    this[ReactiveFlags.IS_READONLY] = isReadonly
  }

  get value() {
    const self = toRaw(this)
    trackRefValue(self) // 收集依赖
    // 首次获取计算属性值的时候，_dirty是true，表示需要重新计算。然后将_dirty设置为false，并重新计算。
    // 如果 computed 中的依赖没有发生变化，则副作用函数不会再次执行
    if (self._dirty) {
      self._dirty = false
      self._value = self.effect.run() // 重新计算
    }
    return self._value
  }

  set value(newValue) {
    this._setter(newValue)
  }
}
```

### watch() 函数

watch 函数是为了监听某个数据，当数据变化时，执行 callback 函数。

在 watch 中会先确定 watch 真正观察的属性，然后也会创建 ReactiveEffect 副作用对象，这个 ReactiveEffect 实例化时内部会调用 watch 观察的属性的 getter，这样被观察的属性就会将这个副作用实例作为依赖收集起来。当被观察的属性变化时，就会执行 scheduler 来执行 callback 动作。

```js
export function watch(source, cb, options) {
  return doWatch(source, cb, options);
}

function doWatch(source, cb, { immediate, deep, flush, onTrack, onTrigger }) {
  let getter = () => {};

  // 确定getter，即确定watch所依赖的响应式数据
  if (isRef(source)) {
    getter = () => source.value;
  } else if (isReactive(source)) {
    getter = () => source;
  } else if (isArray(source)) {
    getter = () =>
      source.map((s) => {
        if (isRef(s)) {
          return s.value;
        } else if (isReactive(s)) {
          return traverse(s);
        } else if (isFunction(s)) {
          return callWithErrorHandling(s, instance, ErrorCodes.WATCH_GETTER);
        }
      });
  } else if (isFunction(source)) {
    if (cb) {
      getter = () => callWithErrorHandling(source, instance, ErrorCodes.WATCH_GETTER);
    }
  } else {
    getter = NOOP;
  }

  let oldValue = isMultiSource
    ? new Array(source.length).fill(INITIAL_WATCHER_VALUE)
    : INITIAL_WATCHER_VALUE;
  // 观察到变化后执行的动作
  const job = () => {
    if (cb) {
      const newValue = effect.run();
      if (hasChanged(newValue, oldValue)) {
        // 调用callback
        callWithAsyncErrorHandling(cb, instance, ErrorCodes.WATCH_CALLBACK, [
          newValue,
          oldValue,
          onCleanup,
        ]);
        oldValue = newValue;
      }
    }
  };

  let scheduler = () => queueJob(job);
  // 创建ReactiveEffect，来让watch的属性进行依赖收集，将自己收集起来，当人家变化的时候，就会执行scheduler来通知更新。
  const effect = new ReactiveEffect(getter, scheduler);
  if (cb) {
    oldValue = effect.run();
  }

  const unwatch = () => {
    effect.stop();
  };

  return unwatch;
}
```

## 依赖收集和触发更新

### 依赖收集

在 ref()方法介绍时，我们提到了 trackRefValue 去收集依赖。依赖就是 activeEffect 对象，也就是 ReactiveEffect 实例。activeEffect 是在 ReactiveEffect 的 run 方法中被赋值的。

上面代码示例中 name 属性，依赖收集了四次，一个是 watch 方法，一个是模版中 name 渲染，一个是 computed 计算属性，还有一个是作为属性传给 child 子组件时。

- 在 watch 方法调用时，创建了一个 ReactiveEffect 实例 `new ReactiveEffect(getter, scheduler)`。

- 模版渲染时，创建了一个 ReactiveEffect 实例 `new ReactiveEffect(componentUpdateFn, () => queueJob(update))`。

- 在计算属性中，创建了一个 ReactiveEffect 实例 `new ReactiveEffect(getter, () => { if (!this._dirty) { this._dirty = true; triggerRefValue(this); } })`。

- 传给 child 子组件时，也相当于是模版渲染时，所以跟第二个一样 `new ReactiveEffect(componentUpdateFn, () => queueJob(update))`。

在计算属性 computed()方法介绍时，也是调用 trackRefValue 来收集依赖项，也就是收集依赖这个计算属性的地方。

上面代码示例中 comput 属性，依赖收集就一次，模版中渲染 comput 时。所以他的依赖就是 `new ReactiveEffect(componentUpdateFn, () => queueJob(update))`。

```js
export function trackRefValue(ref) {
  trackEffects(ref.dep || (ref.dep = createDep()));
}

export function trackEffects(dep) {
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}
```

在 reactive() 方法介绍中，是调用 track 进行依赖收集，跟上面的原理是一样的。

上面代码示例中 obj 属性，依赖收集就一次，模版中渲染 obj.test 时。所以他的依赖就是 `new ReactiveEffect(componentUpdateFn, () => queueJob(update))`。

```js
export function track(target, type, key) {
  if (shouldTrack && activeEffect) {
    trackEffects(dep, eventInfo);
  }
}

export function trackEffects(dep) {
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}
```

### 触发更新

在介绍 ref 方法和 computed 方法时，我们都提到了他们是用 triggerRefValue 来触发更新。不同的时，普通 ref 属性是 set 时触发更新，而计算属性是计算属性的依赖项的值改变时，触发更新。

triggerEffects 时会将这组副作用函数分为两类：计算属性（computed）和普通副作用（普通响应式）。主要是为了确保计算属性的副作用函数优先执行，因为其他的副作用函数可能依赖 computed 的 value。

triggerEffect 就是负责调用副作用 ReactiveEffect 实例的 run 方法的，不过调用的方式有所不同，如果有调度器，就会通过调度器来调用，没有就直接调用 run。当然有可能调度器中没有调用 run 方法。

就像上面依赖收集中提到的那几个 ReactiveEffect 实例，watch 中创建的实例，scheduler 是执行 callback 函数。computed 中创建的实例，scheduler 是触发更新，让使用这个计算属性的地方重新获取值。模版渲染时创建的实例，scheduler 就是执行 effect.run，run 里面再执行模版的更新方法 componentUpdateFn。

```js
export function triggerRefValue(ref) {
  const dep = ref.dep;
  if (dep) {
    triggerEffects(dep);
  }
}

export function triggerEffects(dep) {
  const effects = isArray(dep) ? dep : [...dep];
  for (const effect of effects) {
    if (effect.computed) {
      triggerEffect(effect);
    }
  }

  for (const effect of effects) {
    if (!effect.computed) {
      triggerEffect(effect);
    }
  }
}

function triggerEffect(effect) {
  if (effect.scheduler) {
    effect.scheduler();
  } else {
    effect.run();
  }
}
```

### ReactiveEffect

ReactiveEffect 这个类用于创建一个包含响应式的副作用函数的对象，上面我们好多次提到 ReactiveEffect 这个副作用对象。其实就是依赖对象，就是为了让响应式数据收集依赖的时候收集的。每次需要收集依赖的时机，我们就会创建这个实例。比如上面提到的：

- 在组件即将第一次渲染时，我们创建了这个实例，那渲染时，模版生成的 render 函数中会调用所有的 ref，reactive 或者 computed 数据的 getter 方法，就会执行依赖收集，将这个实例收集起来。

- 在计算属性被创建时，我们创建了这个实例，然后调用计算属性的 getter，就会调用计算属性自己的依赖项的 getter，让计算属性的依赖项执行依赖收集，将这个实例收集起来。

- 在创建 watch 时，我们创建了这个实例，然后调用被 watch 的属性的 getter，这些被 watch 的属性会执行依赖收集，将这个实例收集起来。

一般创建完这个 ReactiveEffect 实例后，会立马调用 run 方法，让其执行传进去的副作用函数，就会进行依赖收集。

```js
export class ReactiveEffect<T = any> {
  let active = true
  let deps = []
  let parentndefined
  let computed
  let deferStop

  constructor(fn, scheduler) {}

  run() {
    try {
      activeEffect = this
      return this.fn() // 执行副作用函数，这个函数会产生副作用，进行依赖收集。
    } finally {
      if (this.deferStop) {
        this.stop()
      }
    }
  }

  stop() {
    if (activeEffect === this) {
      this.deferStop = true
    } else if (this.active) {
      cleanupEffect(this)
      if (this.onStop) {
        this.onStop()
      }
      this.active = false
    }
  }
}
```

## Vue3 中选项式语法

Vue3 中不仅支持 setup 函数，还继续支持了 data，computed，methods 这些 2.x 的选项式语法。他们是在哪初始化的呢，为什么有响应式呢？

上节中我们提到了在 finishComponentSetup，会调用 applyOptions 方法去处理 2.x 的写法。我们看下 applyOptions 的实现：

它内部会调用 reactive 方法处理 data，也会调用 computed 方法处理 computed，也会处理各种生命周期，provide，inject 等等。

```js
export function applyOptions(instance) {
  const options = resolveMergedOptions(instance);
  const publicThis = instance.proxy;
  const ctx = instance.ctx;

  // beforeCreate生命周期
  if (options.beforeCreate) {
    callHook(options.beforeCreate, instance, LifecycleHooks.BEFORE_CREATE);
  }

  // 获取所有的options
  const {
    // state
    data: dataOptions,
    computed: computedOptions,
    methods,
    watch: watchOptions,
    provide: provideOptions,
    inject: injectOptions,
    // lifecycle
    created,
    beforeMount,
    mounted,
    beforeUpdate,
    updated,
    activated,
    deactivated,
    beforeDestroy,
    beforeUnmount,
    destroyed,
    unmounted,
    render,
    renderTracked,
    renderTriggered,
    errorCaptured,
    serverPrefetch,
    // public API
    expose,
    inheritAttrs,
    // assets
    components,
    directives,
    filters,
  } = options;

  // 处理inject
  if (injectOptions) {
    resolveInjections(injectOptions, ctx, checkDuplicateProperties);
  }

  // 处理methods
  if (methods) {
    for (const key in methods) {
      const methodHandler = methods[key];
      ctx[key] = methodHandler.bind(publicThis);
    }
  }

  // 处理data，使用reactive方法
  if (dataOptions) {
    const data = dataOptions.call(publicThis, publicThis);
    instance.data = reactive(data);
  }

  // 处理computed options，使用computed方法
  if (computedOptions) {
    for (const key in computedOptions) {
      const opt = computedOptions[key];
      const get = isFunction(opt)
        ? opt.bind(publicThis, publicThis)
        : isFunction(opt.get)
        ? opt.get.bind(publicThis, publicThis)
        : NOOP;

      const set = !isFunction(opt) && isFunction(opt.set) ? opt.set.bind(publicThis) : NOOP;
      const c = computed({
        get,
        set,
      });
      Object.defineProperty(ctx, key, {
        enumerable: true,
        configurable: true,
        get: () => c.value,
        set: (v) => (c.value = v),
      });
    }
  }

  // 处理watch
  if (watchOptions) {
    for (const key in watchOptions) {
      createWatcher(watchOptions[key], ctx, publicThis, key);
    }
  }

  // 处理provide
  if (provideOptions) {
    const provides = isFunction(provideOptions) ? provideOptions.call(publicThis) : provideOptions;
    Reflect.ownKeys(provides).forEach((key) => {
      provide(key, provides[key]);
    });
  }

  // 处理create生命周期
  if (created) {
    callHook(created, instance, LifecycleHooks.CREATED);
  }

  // 注册各种生命周期
  registerLifecycleHook(onBeforeMount, beforeMount);
  registerLifecycleHook(onMounted, mounted);
  registerLifecycleHook(onBeforeUpdate, beforeUpdate);
  registerLifecycleHook(onUpdated, updated);
  registerLifecycleHook(onActivated, activated);
  registerLifecycleHook(onDeactivated, deactivated);
  registerLifecycleHook(onErrorCaptured, errorCaptured);
  registerLifecycleHook(onRenderTracked, renderTracked);
  registerLifecycleHook(onRenderTriggered, renderTriggered);
  registerLifecycleHook(onBeforeUnmount, beforeUnmount);
  registerLifecycleHook(onUnmounted, unmounted);
  registerLifecycleHook(onServerPrefetch, serverPrefetch);

  // 处理expose
  if (isArray(expose)) {
    if (expose.length) {
      const exposed = instance.exposed || (instance.exposed = {});
      expose.forEach((key) => {
        Object.defineProperty(exposed, key, {
          get: () => publicThis[key],
          set: (val) => (publicThis[key] = val),
        });
      });
    } else if (!instance.exposed) {
      instance.exposed = {};
    }
  }

  // options that are handled when creating the instance but also need to be
  // applied from mixins
  if (render && instance.render === NOOP) {
    instance.render = render;
  }
  if (inheritAttrs != null) {
    instance.inheritAttrs = inheritAttrs;
  }

  // asset options.
  if (components) instance.components = components;
  if (directives) instance.directives = directives;
}
```

## 总结

### Vue3 响应式原理

Vue3 实例挂载的过程中，首先进行根组件虚拟 DOM vnode 的渲染，过程中会对根组件的 setup 函数做处理，执行 setup 函数。

执行 setup 函数时，函数内定义的 ref，reactive，computed，watch 等方法会执行。

- ref 方法会创建一个 RefImpl 的实例，包含 value 属性，对 value 进行 get 时，会收集依赖，对 value set 时会触发更新。

- reactive 方法会使用 Proxy 对目标对象创建一个 proxy 代理，主要拦截 get，set，deleteProperty，has，ownKyes 方法。在 get，has，ownKeys 中会进行依赖收集，在 set，deleteProperty 方法中去触发更新。

- computed 方法会创建一个 ComputedRefImpl 对象，包含 value 属性，对 value 进行 get 时，会收集依赖。当计算属性的依赖项变化时，会触发更新。

- watch 方法在创建时就会收集依赖，当被观察的属性变化时，就会执行 scheduler 来执行 callback 动作。

当实例被渲染时，模版生成的 render 函数中会调用这些属性，访问他们的 getter，触发依赖收集。收集到的就是这个更新函数，当这些数据更新时，会调用这个更新函数，重新渲染。

### 回顾 Vue2 的响应式原理

Vue2 实初始化时，会遍历 data 所有属性，通过 defineReactive 方法让每一个属性变成响应式的。defineReactive 中调用 Object.defineProperty 方法，给目标对象的所有属性设置 get 和 set 方法，在 get 的时候去收集依赖，在 set 的时候去通知更新。对象的每一个属性都有一个 dep 实例。

调用 get 的阶段（依赖收集的阶段）：

- 初始化 watch 的时候，会 new Watcher，在 Watcher 的构造函数中调用 watch 的属性的 getter，触发该属性 get。

- 渲染时，模板被转化成 render 函数后，在 render 函数中会调用该属性，会触发该属性 get。

- 模板中访问计算属性的时候，调用计算属性的 getter，计算属性中主动调用 depend 方法，完成依赖收集。

初始化时对状态数据做了劫持，在执行组件的 render 函数时，会访问一些状态数据，就会触发这些状态数据的 getter，然后 render 函数对应的 render watcher 就会被这个状态收集为依赖，当状态变更触发 setter，setter 中通知 render watcher 更新，然后 render 函数重新执行以更新组件。就这样完成了响应式的过程。

## Proxy & Reflect

Vue3 中使用了 Proxy 和 Reflect 作为他的响应式支撑，那我们再来了解下这两个东西。

### Proxy

Proxy，他是 ES6 中的新增内置模块，用于创建一个对象的代理。它内置了一系列方法，从而实现对对象进行基本操作时的拦截和自定义（如属性查找、取值、赋值、函数调用等）。

```js
const target = {};
const handler = {
  // ...
};
const proxy = new Proxy(target, handler);
```

handler 一共支持 13 种拦截操作：

1. get(target, propKey, receiver)：拦截对象属性的读取，比如 proxy.foo 和 proxy['foo']。

2. set(target, propKey, value, receiver)：拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v，返回一个布尔值。

3. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。

4. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)。

5. has(target, propKey)：拦截 propKey in proxy 的操作，返回一个布尔值。

6. deleteProperty(target, propKey)：拦截 delete proxy[propKey]的操作，返回一个布尔值。

7. ownKeys(target)：拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。

8. defineProperty(target, propKey, propDesc)：拦截 Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。

9. getOwnPropertyDescriptor(target, propKey)：拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。

10. getPrototypeOf(target)：拦截 Object.getPrototypeOf(proxy)，返回一个对象。

11. setPrototypeOf(target, proto)：拦截 Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

12. preventExtensions(target)：拦截 Object.preventExtensions(proxy)，返回一个布尔值。

13. isExtensible(target)：拦截 Object.isExtensible(proxy)，返回一个布尔值。

注意，要使得 Proxy 起作用，必须针对 Proxy 实例（上例是 proxy 对象）进行操作，而不是针对目标对象（上例是 target 空对象）进行操作。

虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 Proxy 代理的情况下，目标对象内部的 this 关键字会指向 Proxy 代理。

```js
const target = {
  m: function () {
    console.log(this === proxy);
  },
};
const handler = {};
const proxy = new Proxy(target, handler);
target.m(); // false
proxy.m(); // true
```

有些原生对象的内部属性，只有通过正确的 this 才能拿到，所以 Proxy 也无法代理这些原生对象的属性。这时需要，this 绑定原始对象，就可以解决这个问题。

```js
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);
proxy.getDate();
// TypeError: this is not a Date object.
```

```js
const target = new Date('2015-01-01');
const handler = {
  get(target, prop) {
    if (prop === 'getDate') {
      return target.getDate.bind(target);
    }
    return Reflect.get(target, prop);
  },
};
const proxy = new Proxy(target, handler);
proxy.getDate(); // 1
```

### Reflect

Reflect 对象与 Proxy 对象一样，也是 ES6 为了操作对象而提供的新 API。设计目的主要是：

1. 让 Object 操作都变成函数行为，比如 name in obj 和 delete obj[name]，都用 Reflect.has(obj, name)和 Reflect.deleteProperty(obj, name)替代。

2. 将 Object 对象的一些明显属于语言内部的方法（比如 Object.defineProperty），放到 Reflect 对象上。虽然现阶段，某些方法同时在 Object 和 Reflect 对象上部署，但是未来的新方法将只部署在 Reflect 对象上。

3. 修改某些 Object 方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而 Reflect.defineProperty(obj, name, desc)则会返回 false。

4. Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法。这就让 Proxy 对象可以方便地调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础。也就是说，不管 Proxy 怎么修改默认行为，你总可以在 Reflect 上获取默认行为。

### 为什么 Proxy 和 Reflect 要一起使用

Proxy 中的 receiver 是在 Proxy 内部的 get 和 set 处理程序中使用的，用于指示执行操作的上下文。它代表的是代理对象本身或者继承于代理对象的对象，它表示触发拦截时的上下文。

Relfect 中的 receiver 可以修改属性访问中的 this 指向为传入的 receiver 对象。

假设我们有一个对象 parent，通过 value 属性，可以获取到他的 name。并且对他创建了代理对象 proxy。

Reflect.get 没有加 receiver，此时通过 proxy.value 可以正确获取到值。

```js
const parent = {
  name: 'parent',
  get value() {
    return this.name;
  },
};
const handler = {
  get(target, key, receiver) {
    console.log(receiver === proxy); // true
    return Reflect.get(target, key);
  },
};
const proxy = new Proxy(parent, handler);

console.log(proxy.value); // parent
```

当我们有一个 child 对象，继承了 parent 的代理时，也就是 child 的原型链上有了 parent 对象的代理对象。

Reflect.get 没有加 receiver，此时 child.value 获取到的是 parent 的值，是不正确的

```js
const parent = {
  name: 'parent',
  get value() {
    return this.name;
  },
};
const handler = {
  get(target, key, receiver) {
    console.log(receiver === proxy); // false
    console.log(receiver === child); // true
    return Reflect.get(target, key);
  },
};
const proxy = new Proxy(parent, handler);

// 增加的代码
const child = {
  name: 'child',
};
Object.setPrototypeOf(child, proxy);
console.log(child.value); // parent
```

Reflect.get 加上 receiver，此时 child.value 获取到的是 child 的值，是正确的。

```js
const parent = {
  name: 'parent',
  get value() {
    return this.name;
  },
};
const handler = {
  get(target, key, receiver) {
    console.log(receiver === proxy); // false
    console.log(receiver === child); // true
    return Reflect.get(target, key, receiver); // 加上receiver，保证此时是正确的上下文
  },
};
const proxy = new Proxy(parent, handler);

const child = {
  name: 'child',
};
Object.setPrototypeOf(child, proxy);
console.log(child.value); // child
```

综上所述，Proxy 和 Reflect 一起使用，是为了在复杂的使用场景保持正确的上下文和 this。可以将 `Reflect.get(target, key, receiver)` 理解成为 `target[key].call(receiver)`。
