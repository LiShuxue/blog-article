> 基于vue@3.3.4源码

## 示例代码

接着上一节，当我们创建一个 Vue3 实例时，最终会用 app.mount() 方法，将我们的 Vue3 实例挂载到 DOM 节点。其实 Vue 的整个应用，就是一个根组件 App，根组件中又包含了子元素和子组件，子组件中再往下包含层层的子元素和子组件，构成了整个应用。

那这个根组件是怎么创建并渲染到页面的，其中的子组件又是怎么渲染的呢？

```js
<!-- App组件的模板 -->
<script type="text/x-template" id="app">
  <div @click="add">{{name}}</div>
  <div>这是根组件</div>
  <children />
</script>
<!-- 子组件 -->
<script type="text/x-template" id="children">
  <div>这是子组件</div>
</script>

// 子组件
const Children = {
  template: '#children',
};
// 根组件
const App = {
  template: '#app',
  components: {
    Children,
  },
  setup() {
    const name = ref(0);
    const add = () => {
      name.value++;
    };

    return {
      name,
      add,
    };
  },
};
const app = createApp(App);
app.mount('#demo');
```

## 实例挂载

### mount

首先是 mount 方法，mount 方法本来定义在 app 对象中，但是在 createApp 后又被重写了一下。

主要就是获取下 mount 方法中传入的 DOM 对象，再调用 app 内部的 mount 方法。

```js
export const createApp = (...args) => {
  const app = ensureRenderer().createApp(...args);
  const { mount } = app;
  app.mount = (containerOrSelector) => {
    // 获取DOM节点：document.querySelector()
    const container = normalizeContainer(containerOrSelector);
    if (!container) return;

    // clear content before mounting
    container.innerHTML = '';
    // 重新调用app对象内部定义的mount方法
    const proxy = mount(container);
    return proxy;
  };

  return app;
};
```

那我们接着看 app 内部的 mount 方法实现，这个定义在 createAppAPI 方法内部，声明在 app 对象中。

它主要就是将根组件变成一个虚拟 DOM 对象 vnode，再将这个 vnode，通过 render 方法渲染到 DOM 节点。

```js
function createAppAPI(render) {
  return function createApp(rootComponent, rootProps = null) {
    // ...
    const context = createAppContext();
    const app = {
      // ...
      mount(rootContainer) {
        // 将根组件变成虚拟DOM
        const vnode = createVNode(rootComponent, rootProps);
        // 将全局context挂在根组件上
        vnode.appContext = context;
        // 渲染根组件的虚拟DOM
        render(vnode, rootContainer);
        app._container = rootContainer;
        return getExposeProxy(vnode.component) || vnode.component.proxy;
      },
    };

    return app;
  };
}
```

### createVNode

createVNode 内部调用 createBaseVNode，来创建 vnode 对象。也就是虚拟 DOM 对象。

```js
const createVNode = _createVNode;
function _createVNode(
  type,
  props = null,
  children = null,
  patchFlag = 0,
  dynamicProps = null,
  isBlockNode = false
) {
  // 根据组件的类型确定一个标志，后续会通过这个标志做一些优化之类的处理。
  const shapeFlag = isString(type)
    ? 1
    : isSuspense(type)
    ? 128
    : isTeleport(type)
    ? 64
    : isObject(type)
    ? 4
    : isFunction(type)
    ? 2
    : 0;

  return createBaseVNode(
    type,
    props,
    children,
    patchFlag,
    dynamicProps,
    shapeFlag,
    isBlockNode,
    true
  );
}
```

### createBaseVNode

createBaseVNode，是实际生成 vnode 对象的地方。vnode 其实就是一个 js 对象，里面包括 type，props，children 等属性。这个 vnode 只是最基础，最上层的的 vnode，里面只挂载根组件的信息。并不是根据根组件模板所生成的 vnode。

对于本示例，props 和 childre 都是 null，而 type 属性里面其实还是包含的那个根组件对象，比如 setup，components，template 等。

![debug2](https://cdn.lishuxue.site/blog/image/Vue/debug2.png)

```js
function createBaseVNode(
  type,
  props = null,
  children = null,
  patchFlag = 0,
  dynamicProps = null,
  shapeFlag = type === Fragment ? 0 : 1,
  isBlockNode = false,
  needFullChildrenNormalization = false
) {
  const vnode = {
    __v_isVNode: true,
    __v_skip: true,
    type,
    props,
    key: props && normalizeKey(props),
    ref: props && normalizeRef(props),
    scopeId: currentScopeId,
    slotScopeIds: null,
    children,
    component: null,
    suspense: null,
    ssContent: null,
    ssFallback: null,
    dirs: null,
    transition: null,
    el: null,
    anchor: null,
    target: null,
    targetAnchor: null,
    staticCount: 0,
    shapeFlag,
    patchFlag,
    dynamicProps,
    dynamicChildren: null,
    appContext: null,
    ctx: currentRenderingInstance,
  };

  return vnode;
}
```

## 实例渲染

从上面可以看到，mount 最终是调用了 `render(vnode, rootContainer)` 将根组件渲染到了 id 为 demo 的 DOM 节点上。而这个 render 方法，就是 `createAppAPI(render)` 传进去的，它定义在 baseCreateRenderer 中。

### render

下面我们就来分析下，render 的时候都干了什么。

根据渲染的 vnode，来判断是卸载还是更新。如果是 null，表示要卸载容器中的内容，否则表示要进行正常的渲染操作。最后会将这个 vnode 挂在 DOM 节点的 \_vnode 属性上，表示渲染过了。

```js
function baseCreateRenderer(options) {
  // ...
  const render = (vnode, container) => {
    if (vnode == null) {
      // 如果存在_vnode，说明容器之前已经有内容渲染过
      if (container._vnode) {
        unmount(container._vnode, null, null, true);
      }
    } else {
      // 对比和更新之前的虚拟节点（container._vnode）和当前的虚拟节点（vnode），如果之前的虚拟节点不存在（即容器之前没有内容渲染过），则传入null作为之前的虚拟节点。
      patch(container._vnode || null, vnode, container);
    }
    container._vnode = vnode;
  };

  return {
    render,
    createApp: createAppAPI(render),
  };
}
```

### patch

接下来看看 render 方法里调用的 patch 方法，patch 方法主要接受几个重要参数：

- 虚拟 DOM 节点 n1 和 n2，n1 代表旧的 vnode，n2 代表新的 vnode。
- container 代表需要渲染的 DOM 节点容器。
- anchor 这个参数表示一个参考节点，用于确定新的虚拟 DOM 节点在容器中的位置。通常情况下，新节点会插入到 anchor 之前。

然后根据 n1 和 n2 之间的差异执行相应的操作，例如创建、更新、销毁 DOM 元素和组件。

创建时，对新的 vnode 不同的类型会走不同的方法，如果新的 vnode 是普通的元素 Element，走 processElement，如果是组件，走 processComponent。

此时上面生成的 vnode，是代表的根组件，是组件类型，所以会走 processComponent 来处理组件的渲染。

```js
function baseCreateRenderer(options) {
  //...
  const patch = (
    n1,
    n2,
    container,
    anchor = null,
    parentComponent = null,
    parentSuspense = null,
    isSVG = false,
    slotScopeIds = null,
    optimized = !!n2.dynamicChildren
  ) => {
    // 如果相等，表示没有需要更新的内容，直接返回
    if (n1 === n2) {
      return;
    }

    // 如果 n1 存在但类型不同于 n2，则表示需要卸载n1旧的虚拟节点，并在相同位置插入新的虚拟节点
    if (n1 && !isSameVNodeType(n1, n2)) {
      unmount(n1, parentComponent, parentSuspense, true);
      n1 = null;
    }

    // 根据 n2 的类型进行不同的处理。
    // 对于文本节点、注释节点、静态节点等特殊类型的节点，会调用相应的处理函数进行处理。
    // 对于片段节点、元素节点、组件节点、传送门节点等其他类型的节点，也会调用相应的处理函数进行处理。
    // processText 处理文本节点，processElement 处理元素节点，processComponent 处理组件等
    const { type, ref, shapeFlag } = n2;
    switch (type) {
      case Text:
        processText(n1, n2, container, anchor);
        break;
      case Comment:
        processCommentNode(n1, n2, container, anchor);
        break;
      case Static:
        if (n1 == null) {
          mountStaticNode(n2, container, anchor, isSVG);
        }
        break;
      case Fragment:
        processFragment(
          n1,
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        );
        break;
      default:
        if (shapeFlag & ShapeFlags.ELEMENT) {
          processElement(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          );
        } else if (shapeFlag & ShapeFlags.COMPONENT) {
          processComponent(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          );
        } else if (shapeFlag & ShapeFlags.TELEPORT) {
          type.process(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized,
            internals
          );
        } else if (shapeFlag & ShapeFlags.SUSPENSE) {
          type.process(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized,
            internals
          );
        }
    }
  };
}
```

### processComponent

当 app.mount() 的时候，会生成一个最基础的 vnode（这个 vnode 的 type 里面，其实就是 App 这个根组件对象），对这个 vnode 先调用 render 方法，render 里再调用 patch 方法，最终会调用 processComponent 来将根组件的 vnode 进行创建和渲染。

processComponent 中会根据旧的 vnode 是否存在，来判断是初始渲染还是更新。如果旧的 VNode 不存在，那么调用挂载方法 mountComponent，创建这个组件，并渲染到页面上。如果新旧都存在，调用 updateComponent 来更新组件。

```js
function baseCreateRenderer(options) {
  // ...

  const processComponent = (n1, n2, container, anchor, parentComponent, parentSuspense) => {
    if (n1 == null) {
      mountComponent(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized);
    } else {
      updateComponent(n1, n2, optimized);
    }
  };
}
```

### mountComponent

mountComponent 也是定义在 baseCreateRenderer 方法中。其中主要是依次调用了 3 个方法，分别是 createComponentInstance，setupComponent，setupRenderEffect。

下面我们挨个分析每个方法。

```js
function baseCreateRenderer(options) {
  const mountComponent = (
    initialVNode,
    container,
    anchor,
    parentComponent,
    parentSuspense,
    isSVG,
    optimized
  ) => {
    const instance = (initialVNode.component = createComponentInstance(
      initialVNode,
      parentComponent,
      parentSuspense
    ));
    setupComponent(instance);
    setupRenderEffect(instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized);
  };
}
```

### createComponentInstance

首先是通过 createComponentInstance 创建了组件的实例。它接收三个参数：vnode（虚拟节点）、parent（父级组件实例或根组件实例）、suspense（悬挂边界或 null）。

创建组件的实例，其实就是返回一个定义好格式的一个 js 对象，声明了组件所需要的全部数据，比如他的 props，他的 data，他的 setup 相关信息。

```js
function createComponentInstance(vnode, parent, suspense) {
  const type = vnode.type;

  // 每个子组件都重新拿到全局的数据
  const appContext = (parent ? parent.appContext : vnode.appContext) || emptyAppContext;

  const instance = {
    vnode, // 当前组件实例对应的虚拟 DOM 节点
    type, // 当前组件实例的类型，通常为组件的构造函数
    parent, // 当前组件实例的父级组件实例，如果没有父级，则为 null
    appContext, // 表示应用程序全局上下文
    root: null, // 根组件实例
    subTree: null, // 组件的子树
    effect: null, // Vue 3 内部用于追踪副作用的对象。
    update: null, // 更新函数，用于更新组件状态。
    render: null, // 渲染函数，用于生成组件的虚拟 DOM。
    proxy: null, // 组件实例的代理对象
    exposed: null, // 暴露给父级组件的属性和方法
    exposeProxy: null, // 暴露属性的代理对象
    provides: parent ? parent.provides : Object.create(appContext.provides), // 提供给子组件的属性和方法。
    components: null, // 子组件
    directives: null, // 指令
    propsOptions: normalizePropsOptions(type, appContext), // 组件的属性选项
    emitsOptions: normalizeEmitsOptions(type, appContext), // 组件的事件选项
    emit: null, // 组件的emit方法
    propsDefaults: {}, // 属性默认值

    // state
    ctx: {}, // 组件的上下文数据
    data: {}, // 组件的data数据
    props: {}, // 组件接收的属性
    attrs: {}, // 存储组件的非特定属性。这是一个对象，包含了组件从父组件继承的非特定属性。
    slots: {}, // 插槽内容
    refs: {}, // 组件的引用（ref）信息
    setupState: {}, // 存储组件 setup 函数的状态
    setupContext: null, // 存储组件 setup 函数的上下文

    // lifecycle hooks
    isMounted: false,
    isUnmounted: false,
    isDeactivated: false,
  };

  return instance;
}
```

### setupComponent

然后调用了 setupComponent 方法，来对这个实例进行设置。比如初始化 props，初始化 slots，以及根据 setup 函数来设置组件，还有实例的 render 方法。

```js
function setupComponent(instance) {
  const { props, children } = instance.vnode;
  // 如果是instance.vnode.shapeFlag是4，也就是vnode的type是一个对象的形式，就是stateful component
  const isStateful = isStatefulComponent(instance);
  initProps(instance, props, isStateful);
  initSlots(instance, children);
  const setupResult = isStateful ? setupStatefulComponent(instance) : undefined;
  return setupResult;
}
```

#### setupStatefulComponent

setupStatefulComponent 主要是为了处理组件选项的 setup 函数。

获取 setup 函数的返回值，如果 setup 返回的是一个函数，那么这个函数会直接被作为渲染函数。否则如果返回的是一个对象，会使用 proxyRefs 将这个对象转为 Proxy 代理的响应式对象。

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

#### finishComponentSetup

最后又调用了 finishComponentSetup 方法，这个函数主要是给组件实例，添加上 render 方法。方便将组件的模版渲染成 vnode。

它先判断组件是否存在渲染函数 render，如果不存在则判断是否存在 template 选项，将 template 编译成渲染函数。

我们根组件传的模板 template，所以会使用 compile 方法来将模板编译成渲染函数，渲染函数会返回一个虚拟 DOM 结构，用于描述组件的视图，也会绑定事件。他可以访问组件的属性（props）、状态（data）、计算属性（computed）、方法（methods）等数据，并将这些数据动态地渲染到视图中。这使得组件能够根据数据的变化自动更新视图，实现响应式的用户界面。

编译后的渲染函数如下：

![debug3](https://cdn.lishuxue.site/blog/image/Vue/debug3.png)

```js
function finishComponentSetup(instance) {
  const Component = instance.type;

  if (!instance.render) {
    if (compile && !Component.render) {
      const template = Component.template || resolveMergedOptions(instance).template;
      if (template) {
        const { isCustomElement, compilerOptions } = instance.appContext.config;
        const { delimiters, compilerOptions } = Component;
        const finalCompilerOptions = extend(
          extend(
            {
              isCustomElement,
              delimiters,
            },
            compilerOptions
          ),
          componentCompilerOptions
        );
        Component.render = compile(template, finalCompilerOptions);
      }
    }

    instance.render = Component.render || NOOP;
  }
}
```

### setupRenderEffect

接下来就是比较重要的 setupRenderEffect 方法，它主要是设置组件的挂载与更新逻辑，以及设置组件渲染的副作用函数。

这个组件渲染的副作用函数 effect，就跟响应式原理有关了，后面的文章我们继续分析响应式原理部分。

```js
const setupRenderEffect: SetupRenderEffectFn = (
  instance,
  initialVNode,
  container,
  anchor,
  parentSuspense,
  isSVG,
  optimized
) => {
  // 组件更新方法
  const componentUpdateFn = () => {};

  // 创建一个响应式的effect
  const effect = (instance.effect = new ReactiveEffect(
    componentUpdateFn,
    () => queueJob(update),
    instance.scope
  ));

  // 调用effect的run方法执行componentUpdateFn方法
  const update = (instance.update = () => effect.run());
  update.id = instance.uid;
  update();
};
```

#### componentUpdateFn

componentUpdateFn 就是组件挂载与更新的钩子函数，同时也会针对的调用生命周期函数。

组件无论是首次挂载，还是更新，做的事情核心是一样的，先调用 renderComponentRoot 方法生成组件模板的虚拟 DOM，然后调用 patch 对比两个虚拟 DOM，去做更新或者挂载。

```js
const componentUpdateFn = () => {
  if (!instance.isMounted) {
    const { bm, m, parent } = instance;

    // beforeMount hook
    if (bm) {
      invokeArrayFns(bm);
    }

    // 将组件的模板编译成vnode
    const subTree = (instance.subTree = renderComponentRoot(instance));
    // vnode对比并渲染
    patch(null, subTree, container, anchor, instance, parentSuspense, isSVG);

    // mounted hook
    if (m) {
      queuePostRenderEffect(m, parentSuspense);
    }

    initialVNode.el = subTree.el;
    instance.isMounted = true;
  } else {
    let { next, bu, u, parent, vnode } = instance;

    // beforeUpdate hook
    if (bu) {
      invokeArrayFns(bu);
    }

    // 更新组件
    const nextTree = renderComponentRoot(instance);
    const prevTree = instance.subTree;
    instance.subTree = nextTree;
    patch(
      prevTree,
      nextTree,
      hostParentNode(prevTree.el),
      getNextHostNode(prevTree),
      instance,
      parentSuspense,
      isSVG
    );
    next.el = nextTree.el;

    // updated hook
    if (u) {
      queuePostRenderEffect(u, parentSuspense);
    }
  }
};
```

#### renderComponentRoot

renderComponentRoot 就是为了将组件的模板编译成 vnode 对象。内部就是调用 render 方法，也就是我上面说的模版编译后的渲染函数。

生成的那个 subTree，其实就是上面我们的示例代码中的结构。就是两个 div，一个 children 组件。这个才是组件模板对应的 vnode，跟最开始创建的 vnode 区分开。

![debug4](https://cdn.lishuxue.site/blog/image/Vue/debug4.png)

```js
function renderComponentRoot(instance) {
  const {
    type: Component,
    vnode,
    proxy,
    withProxy,
    props,
    propsOptions: [propsOptions],
    slots,
    attrs,
    emit,
    render,
    renderCache,
    data,
    setupState,
    ctx,
    inheritAttrs,
  } = instance;

  let result;
  try {
    // 普通的对象式组件
    if (vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
      const proxyToUse = withProxy || proxy;
      result = normalizeVNode(
        render.call(proxyToUse, proxyToUse, renderCache, props, setupState, data, ctx)
      );
    } else {
      // 函数式组件，自己写的render方法
      const render = Component;
      result = normalizeVNode(render(props, { attrs, slots, emit }));
    }
  } catch (err) {
    handleError(err, instance, ErrorCodes.RENDER_FUNCTION);
    result = createVNode(Comment);
  }

  return result;
}
```

## 子元素/子组件渲染

在根组件的 componentUpdateFn 方法中，根据生成的根组件的 vnode，继续调用 patch 方法去挂载组件。上面我们已经分析了 patch，在 patch 里面会根据 vnode 的类型继续调用 processElement，processComponent，或者 processFragment 等。回到了我们本篇中最开头介绍的地方。所以也就是开始了 patch 方法对 vnode 的递归。

当在组件的模板中使用 `<template>` 元素来包裹多个子元素时，Vue 3 会将这些子元素作为一个 Fragment 虚拟节点，所以 App 组件模版编译成的 vnode 被视为一个 Fragment 节点，开始进入 processFragment 处理。

### processFragment

processFragment 中首次渲染会调用 mountChildren 来渲染子节点。

```js
const processFragment = (
  n1,
  n2,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  slotScopeIds,
  optimized
) => {
  // 首次渲染子节点
  if (n1 == null) {
    // 渲染子元素，fragment的子元素都是包含在数组中
    mountChildren(
      n2.children,
      container,
      fragmentEndAnchor,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    );
  } else {
    // 更新子节点
    patchChildren(
      n1,
      n2,
      container,
      fragmentEndAnchor,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    );
  }
};
```

#### mountChildren

mountChildren 里面遍历 children，再分别调用 patch，开始递归 vnode。

此时 vnode 还是根组件模板编译成的那个 vnode，children 里面也只是 div，div 和一个子组件，共三个 vnode，他们会分别调用 patch 方法。

![debug4](https://cdn.lishuxue.site/blog/image/Vue/debug4.png)

```js
const mountChildren: MountChildrenFn = (
  children,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  slotScopeIds,
  optimized,
  start = 0
) => {
  // 遍历children
  for (let i = start; i < children.length; i++) {
    const child = (children[i] = optimized
      ? cloneIfMounted(children[i])
      : normalizeVNode(children[i]));
    // 分别调用patch
    patch(
      null,
      child,
      container,
      anchor,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    );
  }
};
```

### processElement

patch 方法中会根据 vnode 的不同类型，走不同的逻辑。

如果是 html 元素的，直接调用 processElement，然后调用 mountElement，来将 html 标签渲染到页面。

如果是子组件的，会调用 processComponent，然后调用 mountComponent。就像我们上面分析的一样，子组件也会走一遍这所有的逻辑，开始了子组件的递归渲染。

```js
const processElement = (n1, n2, container, anchor, parentComponent, parentSuspense) => {
  if (n1 == null) {
    mountElement(
      n2,
      container,
      anchor,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    );
  } else {
    patchElement(n1, n2, parentComponent, parentSuspense, isSVG, slotScopeIds, optimized);
  }
};

const processComponent = (n1, n2, container, anchor, parentComponent, parentSuspense) => {
  if (n1 == null) {
    mountComponent(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized);
  } else {
    updateComponent(n1, n2, optimized);
  }
};
```

#### mountElement

mountElement 会根据 vnode 的 type 类型来创建真实的 DOM 元素，如果这个 vnode 是普通文本节点，直接设置 html 文本内容，如果 children 是数组，说明还有子元素，继续调用 mountChildren，开始递归 children。

所以，页面的呈现就是根组件递归子组件，子组件递归子元素，直到不再能递归的时候，在页面上创建 DOM 节点。

```js
const mountElement = (
  vnode,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  slotScopeIds,
  optimized
) => {
  let el;
  const { type, props, shapeFlag, transition, dirs } = vnode;

  // 创建DOM元素
  el = vnode.el = hostCreateElement(vnode.type, isSVG, props && props.is, props);

  // 设置文本内容
  if (shapeFlag & ShapeFlags.TEXT_CHILDREN) {
    hostSetElementText(el, vnode.children);
  } else if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    // 递归子元素
    mountChildren(
      vnode.children,
      el,
      null,
      parentComponent,
      parentSuspense,
      isSVG && type !== 'foreignObject',
      slotScopeIds,
      optimized
    );
  }
};
```

## 总结

当调用 app.mount() 方法时，会调用 createVnode，将根组件 App 转成 vnode，注意，此时并不是根组件模板的 vnode，而是一个最基础的最上层的 vnode。type 中只包含 App 对象。

再调用 render 方法，render 方法里会调用 patch 方法，将旧的 vnode 和新的 vnode 做对比，来选择不同的方法去处理。对于我们 app 初始化 mount 来说，此时没有旧的 vnode，只有新的 vnode 就是这个根组件，所以最终会调用 processComponent 来初始化这个组件以及渲染在 DOM 节点上。

processComponent 会调用 mountComponent。

而 mountComponent 中依次调用了 3 个方法，分别是 createComponentInstance 创建组件实例，setupComponent 设置组件的 props/slot/render/setup 等，setupRenderEffect 设置组件的挂载与更新逻辑。

setupRenderEffect 中最后调用了 componentUpdateFn 开始了组件挂载到页面的动作。

首先通过 renderComponentRoot 将组件模板编译成 vnode，然后调用 patch 方法挂载。此时的 vnode 是包含三个 children 的一个类型为 fragment 的 vnode，所以开始走 processFragment。

因为此时的 vnode 是包含三个 children 的，所以 processFragment 中调用 mountChildren 来分别对 children 中的 vnode 做 patch。

此时 patch 的时候，如果 vnode 是 html 元素类型，直接调用 processElement，然后调用 mountElement，来将 html 标签渲染到页面。如果 vnode 是组件类型的，表示是子组件，会调用 processComponent，然后调用 mountComponent。就像我们上面分析的一样，子组件也会走一遍这所有的逻辑，开始了子组件的递归渲染。

## 单文件组件编译

上面我们分析的 App 组件的写法，都是常规的对象，里面有 setup 函数。但是通常开发中其实都是单文件组件，使用 `<script setup>` 语法糖，那这种是怎么处理的呢？

这其实就涉及到单文件组件的编译，`packages/compiler-sfc` 模块，就是专门编译单文件组件的。

### 手动编译

当我们在项目根目录使用 `pnpm build` 后，会打包所有模块，包括这个 `packages/compiler-sfc` 模块，下面可以找到 `packages/compiler-sfc/dist/compiler-sfc.cjs.js` 文件，接着我们在 `packages/compiler-sfc/dist` 下创建 test.js 文件，模拟对单文件组件的编译。

```js
// packages/compiler-sfc/dist/test.js
const { parse, compileScript, compileTemplate } = require('./compiler-sfc.cjs');

const parseValue = parse(`
    <template>
        <div>
            <div>{{ name }}</div>
        </div>
    </template>

    <script setup>
    import { ref } from 'vue'
    const name = ref(0);
    </script>

    <style>
    </style>
    `);

const { content } = compileScript(parseValue.descriptor, {});
const { code } = compileTemplate(parseValue.descriptor, {});

console.log(content);
console.log(code);
```

### 输出

对于 script 部分的编译如下：

```js
import { ref } from 'vue';

export default {
  setup(__props, { expose: __expose }) {
    __expose();

    const name = ref(0);

    const __returned__ = { name, ref };
    Object.defineProperty(__returned__, '__isScriptSetup', {
      enumerable: false,
      value: true,
    });
    return __returned__;
  },
};
```

对于 template 部分的输出如下：

```js
import {
  toDisplayString as _toDisplayString,
  createElementVNode as _createElementVNode,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock,
} from 'vue';

export function render(_ctx, _cache) {
  return (
    _openBlock(),
    _createElementBlock('template', null, [
      _createElementVNode('div', null, [
        _createElementVNode('div', null, _toDisplayString(_ctx.name), 1 /* TEXT */),
      ]),
    ])
  );
}
```

### 总结

可以看到，`<script setup>` 语法糖被编译成了 setup 函数，template 模板被编译成了 render 渲染函数。在上面的 setupComponent 方法中，我们已经详细介绍过了。
