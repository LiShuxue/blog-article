> 基于vue@3.3.4源码

## 实例挂载

接着上一节，当我们创建一个 Vue3 实例时，最终会用 app.mount() 方法，将我们的 Vue3 实例挂载到 DOM 节点。这个中间具体发生了什么，我们接着看源码。

```js
// 根组件
const App = {
  template: '#app',
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

vnode 其实就是一个 js 对象，里面包括 type，props，children 等属性。对于本示例，props 和 childre 都是 null，而 type 属性里面其实还是包含的那个根组件对象，比如 setup，template 等。

![debug2](https://cdn.lishuxue.site/blog/image/Vue/debug2.png)

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

### processComponent 和 processElement

在 patch 方法中，对新的 vnode，不同的类型会走不同的方法，如果新的 vnode 是普通的元素 Element，走 processElement，如果是组件，走 processComponent。

如果旧的 VNode 不存在，那么调用挂载方法，创建这个组件或者元素，并渲染到页面上。如果新旧都存在，那么进行 diff 操作，更新 DOM 内容。

```js
const processComponent = (n1, n2, container, anchor, parentComponent, parentSuspense) => {
  if (n1 == null) {
    mountComponent(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized);
  } else {
    updateComponent(n1, n2, optimized);
  }
};

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
```

## 总结

当调用 app.mount() 方法时，会调用 createVnode，将根组件转成 vnode，再调用 render 方法，将 vnode 渲染到容器上。

render 方法里会调用 patch 方法，将旧的 vnode 和新的 vnode 做对比，来选择不同的方法去处理。

对于我们 app 初始化 mount 来说，此时没有旧的 vnode，只有新的 vnode 就是这个根组件，所以最终会调用 mountComponent 来初始化这个组件以及渲染在 DOM 节点上。
