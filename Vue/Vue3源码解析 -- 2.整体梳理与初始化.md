> 基于vue@3.3.4源码

## 整体梳理

### Vue 导出了什么

上一节我们知道了，Vue3 打包时入口文件就是 `packages/vue/src/runtime.ts` 或者 `packages/vue/src/index.ts`，那我们就看下这两个文件的源码。

先看 `packages/vue/src/index.ts`，相当于导出了 compile 方法，又导出了 `@vue/runtime-dom` 包中的所有内容。

```js
function compileToFunction(template, options) {
  // ...
}

export { compileToFunction as compile };
export * from '@vue/runtime-dom';
```

再看 `packages/vue/src/runtime.ts` 文件，同样是导出了 `@vue/runtime-dom` 包的所有内容和 compile 方法，但是 compile 方法实际上没有任何作用。这也证明，其实整个的 Vue，相比 runtime 的，就是多了 compile 的内容。

```js
export * from '@vue/runtime-dom';

export const compile = () => {
  // ...
};
```

### @vue/runtime-dom 导出了什么

我们再看 `@vue/runtime-dom` 都导出了什么，这个顾名思义，在 `packages/runtime-dom/src/index.ts` 文件中。

可以看到，导出了很多全局方法，比如 createApp，还导出了一些全局组件和全局指令。然后是导出了 `@vue/runtime-core` 包中的所有内容，和 jsx 文件中的所有内容，jsx 中全是针对 jsx 写法的 ts 类型。

```js
export const createApp = (...args) => {};

// DOM-only components
export { Transition } from './components/Transition';
export { TransitionGroup } from './components/TransitionGroup';

// DOM-only runtime directive helpers
export { withModifiers } from './directives/vOn';
export { vShow } from './directives/vShow';

// re-export everything from core
// h, Component, reactivity API, nextTick, flags & types
export * from '@vue/runtime-core';

export * from './jsx';
```

### @vue/runtime-core 导出了什么

然后我们接着分析 `@vue/runtime-core` 包中都导出了什么，在 `packages/runtime-core/src/index.ts` 文件中

可以看到，这个把 Vue3 的所有核心和高级的 api 全部导出来了，同时也导出了很多定义好的 ts 类型。具体 api 作用可看官网中的介绍：<https://cn.vuejs.org/api/>。

```js
// Core API ------------------------------------------------------------------
export {
  // 核心
  reactive,
  ref,
  readonly,
  // 工具
  unref,
  isRef,
  toRef,
  toValue,
  toRefs,
  isProxy,
  isReactive,
  isReadonly,
  // 进阶
  customRef,
  triggerRef,
  shallowRef,
  shallowReactive,
  shallowReadonly,
  markRaw,
  toRaw,
  effectScope,
  getCurrentScope,
  onScopeDispose,
} from '@vue/reactivity';
// 计算属性
export { computed } from './apiComputed';
// 观察者
export { watch, watchEffect, watchPostEffect, watchSyncEffect } from './apiWatch';
// 生命周期
export {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onActivated,
  onDeactivated,
  onErrorCaptured,
} from './apiLifecycle';
// 注入
export { provide, inject } from './apiInject';
export { nextTick } from './scheduler';
export { defineComponent } from './apiDefineComponent';
export { defineAsyncComponent } from './apiAsyncComponent';

// <script setup> API ----------------------------------------------------------
export {
  // 只能在 <script setup> 中使用的编译器宏，用于类型检查和警告
  defineProps,
  defineEmits,
  defineExpose,
  defineOptions,
  defineSlots,
  useAttrs,
  useSlots,
} from './apiSetupHelpers';

// Advanced API ----------------------------------------------------------------
// setup 函数中不能使用 this。如果确实需要使用，可以用 const { proxy } = getCurrentInstance();获取实例
export { getCurrentInstance } from './component';
// 创建 vnodes
export { h } from './h';
// vnodes的工具方法
export { cloneVNode, mergeProps, isVNode } from './vnode';
// Built-in components
export { Teleport } from './components/Teleport';
export { Suspense } from './components/Suspense';
export { KeepAlive } from './components/KeepAlive';
// For using custom directives
export { withDirectives } from './directives';

// Custom Renderer API ---------------------------------------------------------
export { createRenderer } from './renderer';

// Types -------------------------------------------------------------------------
// ...
```

### 总结

综上所述，从上面的分析中，可以看到在入口文件中把 Vue3 的所有的 api 方法全部导出来了。

跟 Vue2 不同，Vue2 是导出一个 Vue 类，Vue3 是导出方法，使用时要 import 这些方法：

```js
import { createApp, nextTick, ref, computed, watch, provide, inject, onMounted } from 'vue';
```

## Vue 初始化

Vue3 中通过 createApp() 来创建整个 app 实例，app 实例上可以使用其他插件，比如 router 和其他组件库等。app.config 是对这个 app 实例的配置，比如全局错误处理等。

最后，通过 mount() 方法将 app 实例挂载在一个容器元素中。

看下面的例子。

```js
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import App from './App.vue';
import handleError from './utils/handleError';

const app = createApp(App);
app.use(createPinia());
app.config.errorHandler = handleError;

app.mount('#app');
```

### createApp

接下来我们一步步分析这些源码是怎么实现的，先来看 createApp，它内部调用了 ensureRenderer().createApp()，也就是 ensureRenderer 最终会返回一个 createApp 方法。

之后重写了 mount 方法。

```js
const createApp = (...args) => {
  const app = ensureRenderer().createApp(...args);
  const { mount } = app;
  app.mount = (containerOrSelector) => {
    // ...
  };

  return app;
};
```

### baseCreateRenderer

再看 ensureRenderer 方法，它最终调用了 baseCreateRenderer 方法。

baseCreateRenderer 里面很多方法我们先忽略，最终就是创建一个对象并返回，对象中包含 createApp 方法。这个方法会调用 createAppAPI。

```js
function ensureRenderer() {
  return createRenderer(rendererOptions);
}

function createRenderer(options) {
  return baseCreateRenderer(options);
}

function baseCreateRenderer(options) {
  // ...
  return {
    render,
    createApp: createAppAPI(render),
  };
}
```

### createAppAPI

接着看 createAppAPI，它直接返回了一个函数，名字也是 createApp，这个方法才是真正创建实例的地方。它创建了一个 context 对象和 app 对象，最后返回 app 对象。

app 对象是一个应用程序实例，用于组织和配置应用程序的各个部分，包括注册组件、指令、挂载根组件等。它是开发者与应用程序进行交互的接口。

context 对象是一个内部的上下文对象，它用于在 Vue3 内部管理应用程序的全局状态、配置和插件。可以用于存储全局状态，这些状态可能会被多个组件或指令共享。

```js
function createAppAPI(render) {
  // 这里才是真正的创建app实例的地方
  return function createApp(rootComponent, rootProps = null) {
    // 全局的app上下文，里面包含全局组件，全局指令，mixins，config，以及app实例等
    const context = createAppContext();
    const installedPlugins = new Set();
    const app = {
      _uid: uid++,
      _component: rootComponent,
      _props: rootProps,
      _container: null,
      _context: context,
      _instance: null,

      get config() {
        return context.config;
      },

      // 注册插件
      use(plugin, ...options) {
        installedPlugins.add(plugin);
        plugin.install(app, ...options);
        return app;
      },

      // 这些方法将mixin，component，directive都塞到context的属性中
      mixin(mixin) {
        if (!context.mixins.includes(mixin)) {
          context.mixins.push(mixin);
        }
        return app;
      },
      component(name, component) {
        if (!component) {
          return context.components[name];
        }
        context.components[name] = component;
        return app;
      },
      directive(name, directive) {
        if (!directive) {
          return context.directives[name];
        }
        context.directives[name] = directive;
        return app;
      },
      provide(key, value) {
        context.provides[key] = value;
        return app;
      },

      // 挂载和卸载
      mount() {},
      unmount() {},
    };

    return app;
  };
}

function createAppContext() {
  return {
    app: null,
    config: {
      isNativeTag: NO,
      performance: false,
      globalProperties: {},
      optionMergeStrategies: {},
      errorHandler: undefined,
      warnHandler: undefined,
      compilerOptions: {},
    },
    mixins: [],
    components: {},
    directives: {},
    provides: Object.create(null),
    optionsCache: new WeakMap(),
    propsCache: new WeakMap(),
    emitsCache: new WeakMap(),
  };
}
```

### 总结

我们通过 `const app = createApp(App);` 创建了一些 app 对象，这个 app 对象提供了一些全局方法，比如注册组件、指令、挂载根组件等，也可以设置 config 对象，app 中的 config 对象就是 context 中的 config。
