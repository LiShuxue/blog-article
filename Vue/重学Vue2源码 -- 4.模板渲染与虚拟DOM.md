> 基于vue@2.6.14源码，最后一个纯 vue2 版本源码，2.7 开始集成了 v3 相关 Composition API

## Vue 中 UI 的编写方式

组件或者实例，对于页面 html 渲染有两种写法：

1. 最常见的是基于 template 写，类似写 html，可以使用 Vue 提供的指令等。
2. 用 render 函数，通过 createElement 来创建虚拟节点 VNode。createElement 的参数分别是 tag，data，children
   - tag 可以是 html 标签名，也可以是组件对象。
   - data 是一个与模板中 attribute 对应的数据对象，包含 class，style，attrs，props，domProps，on，nativeOn，directives，slot，scopedSlots，key，ref 等。
   - children 是子级虚拟节点 (VNode)，也可以直接是一个文本。
3. 也是 render 函数，函数内部直接 return jsx 代码，类似 react，但是需要额外的插件支持：babel-plugin-transform-vue-jsx。 或者预设集 preset： [babel-preset-jsx](https://github.com/vuejs/jsx-vue2/tree/master)。如果是 Vue3 支持 jsx，见 [babel-plugin-jsx](https://github.com/vuejs/babel-plugin-jsx)

```js
// 使用template的写法
new Vue({
  el: '#app',
  components: { App },
  template: '<app/>',
});

// 使用render函数的写法
new Vue({
  el: '#app',
  render: (h) => h(App), // h 就是 createElement
});
```

## 模板渲染

### 挂载 $mount

Vue 实例或者组件实例，在初始化的时候，都会调用 mount 方法，先去判断是否有写 render 函数，如果没有定义 render 方法，则会把 template 字符串转换成 render 方法，最终调用 mountComponent 方法。如果写了 render，直接调用 mountComponent 方法。

对于加了 v-once 的节点，或者一个模板中没有数据，只有 html 的，这些会生成 staticRenderFns，静态渲染函数，将只在最开始计算一次 VNode 并缓存起来，后续不会再更新。

```js
// src/core/instance/init.js
Vue.prototype._init = function (options) {
  const vm = this;
  // 只有是vue实例的时候，才需要mount挂载，当是子组件init的时候，没有el，不需要mount挂载，但是会在组件内部主动调mount
  if (vm.$options.el) {
    vm.$mount(vm.$options.el);
  }
};

// src/platforms/web/entry-runtime-with-compiler.js
Vue.prototype.$mount = function (el) {
  // 找到需要挂载的DOM节点，将el这个DOM id变为DOM节点
  el = el && query(el);

  const options = this.$options;
  // 如果初始化时候没有自己写render函数
  if (!options.render) {
    let template = options.template; // 获取对应的template选项
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          // 如果template是一个dom id，找到这个id对应的模板
          template = idToTemplate(template);
        }
      } else if (template.nodeType) {
        template = template.innerHTML;
      } else {
        return this;
      }
    }

    if (template) {
      // 将模板编译成render函数和staticRenderFns函数。
      // 参数是模板，编译的配置，vue或者组件实例
      const { render, staticRenderFns } = compileToFunctions(
        template,
        {
          outputSourceRange: process.env.NODE_ENV !== 'production',
          shouldDecodeNewlines,
          shouldDecodeNewlinesForHref,
          delimiters: options.delimiters,
          comments: options.comments,
        },
        this
      );
      // 挂在options上面
      options.render = render;
      options.staticRenderFns = staticRenderFns;
    }
  }
  // 这里我将两个mount的代码合并了
  return mountComponent(this, el);
};
```

### mountComponent

这个方法才是真正的渲染，将组件或者 vue 实例，渲染在浏览器上。内部主要将模板的更新作为数据变化的观察者 Watcher 实例。

其中首先会创建 watcher 实例，进行初次渲染。如果数据更新之后，会重新触发渲染。渲染的时候先生成 vnode，再通过 vnode 生成真的 DOM，渲染在浏览器上。

```js
// src/core/instance/lifecycle.js
export function mountComponent(vm, el) {
  // vm.$el中最终存储的是真实的DOM节点
  vm.$el = el;
  // 如果没有render方法，创建一个生成空VNode节点的方法
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode;
  }
  // 触发beforeMount生命周期
  callHook(vm, 'beforeMount');

  // 调用vm._render()方法生成该组件的vnode
  // 再调用vm._update()方法将vnode转化成DOM渲染在浏览器上
  let updateComponent = () => {
    vm._update(vm._render());
  };

  // 创建一个Watcher实例，在Watcher构造方法中会调用getter，即updateComponent方法，就会完成初始化的渲染。
  new Watcher(
    vm,
    updateComponent,
    noop,
    {
      before() {
        if (vm._isMounted && !vm._isDestroyed) {
          callHook(vm, 'beforeUpdate');
        }
      },
    },
    true /* isRenderWatcher */
  );

  // src/core/instance/render.js 的 initRender 方法中，会声明几个变量
  // vm.$vnode === options._parentVnode，表示该组件在父组件中的VNode
  // vm._vnode 表示子组件所有的template编译成的VNode
  // 当$vnode是null时，表示没有父组件了，即是Vue实例
  if (vm.$vnode == null) {
    vm._isMounted = true;
    callHook(vm, 'mounted');
  }

  return vm;
}
```

### vm.\_render

该方法主要是调用组件或者 Vue 实例中自己写的 render 函数，或者是模板编译后生成的的 render 函数，来创建虚拟 DOM 节点 VNode.

```js
// src/core/instance/render.js
Vue.prototype._render = function () {
  const vm = this;
  // _parentVnode 是创建组件实例的时候，将父组件VNode节点中的该子组件传进来
  const { render, _parentVnode } = vm.$options;
  // 整理插槽和作用域插槽的内容
  if (_parentVnode) {
    vm.$scopedSlots = normalizeScopedSlots(
      _parentVnode.data.scopedSlots,
      vm.$slots,
      vm.$scopedSlots
    );
  }

  vm.$vnode = _parentVnode;
  let vnode;
  try {
    // 调用render方法，生成虚拟节点VNode
    vnode = render.call(vm._renderProxy, vm.$createElement);
  } catch (e) {
    handleError(e, vm, `render`);
    // 报错后还用上次的
    vnode = vm._vnode;
  }

  // 如果vnode是个数组，会取第一个。手写render方法时，vnode可能会返回数组。如：
  /**
      render(h) {
        return [
          h('div', 'first')
        ]
      }
     */
  if (Array.isArray(vnode) && vnode.length === 1) {
    vnode = vnode[0];
  }
  // 如果vnode是不是虚拟DOM节点，就创建一个空的
  if (!(vnode instanceof VNode)) {
    vnode = createEmptyVNode();
  }
  // 设置vnode节点中的parent属性为该组件在父组件中的VNode节点
  vnode.parent = _parentVnode;
  return vnode;
};
```

### vm.\_update

该方法主要是根据最新的 vnode，生成真实的 DOM，用最新的 DOM 替换当前浏览器 DOM 树中旧的 DOM。

其中调用`vm.__patch__`方法进行 DOM 节点生成，diff 算法啥的，都是在`vm.__patch__`这个里面的，就是对比新旧节点，最后转化成新的 DOM 对象。然后把 vm.$el 这个旧的 DOM 对象替换掉，浏览器就变了。

这才是真正的 DOM patch，没有任何的算法，就是利用 js 的引用，把整个 DOM 树对象中的某一部分节点替换掉。

```js
// src/core/instance/lifecycle.js
Vue.prototype._update = function (vnode) {
  const vm = this;
  // 存储更新前的真实DOM，始终是整个vue实例或者组件实例的真实DOM的引用
  const prevEl = vm.$el;
  // 存储更新前的虚拟DOM
  const prevVnode = vm._vnode;
  // 将更新后最新的虚拟DOM VNode赋值给实例的_vnode属性
  vm._vnode = vnode;
  if (!prevVnode) {
    // 初始化渲染，__patch__将vnode转换为真实的DOM
    // 因为vm.$el始终是整个vue实例或者组件实例的真实DOM的引用，所以更新这个引用，会使得浏览器页面变化。
    vm.$el = vm.__patch__(vm.$el, vnode);
  } else {
    // 根据新旧虚拟VNode来渲染，__patch__会有diff，获取最新的真实DOM
    // 将最新的DOM更新到vm.$el这个引用上，真实DOM数会改变，浏览器页面改变
    vm.$el = vm.__patch__(prevVnode, vnode);
  }
  // 因为旧的DOM已被更新，所以将引用也删除
  if (prevEl) {
    prevEl.__vue__ = null;
  }
  // 将最新的DOM上挂载本实例
  if (vm.$el) {
    vm.$el.__vue__ = vm;
  }
};
```

## 虚拟 DOM

### VNode

真正的 DOM 元素是非常庞大的，因为浏览器的标准就把 DOM 设计的非常复杂。当我们频繁的去做 DOM 更新，会产生一定的性能问题。而 Virtual DOM 就是用一个原生的 JS 对象去描述一个 DOM 节点，所以它比创建一个真实 DOM 的代价要小很多。

在 Vue 中，用 VNode 去描述它，VNode 是对真实 DOM 的一种抽象描述，它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等。

```js
// src/core/vdom/vnode.js
export default class VNode {
  tag;
  data;
  children;
  text;
  elm; // 这个vnode创建的真正的DOM对象
  context;
  key;
  componentInstance; // 组件实例
  parent; // 这个组件在父级中的vnode节点

  constructor(tag, data, children, text, elm, context) {
    this.tag = tag;
    this.data = data;
    this.children = children;
    this.text = text;
    this.elm = elm;
    this.context = context;
    this.key = data && data.key;
    this.componentInstance = undefined;
    this.parent = undefined;
  }

  get child() {
    return this.componentInstance;
  }
}
```

### render

上面我们看到，调用 render 函数，会生成 vnode，render 方法可以是手写的（平时用的少），也可以是模板编译成的。

```js
// 手写render函数
const render = (h) => h(App); // render函数可以直接传入组件
const render = (h) => {
  return h('h1', 'test msg'); // render函数也可以传入html标签名
};

// 模板编译后的render匿名函数
const render = function anonymous() {
  with (this) {
    return _c('div', [_c('my-other-comp')], 1);
  }
};
```

vue 实例或者组件手写的 render 函数，通过调用 vm.$createElement 来创建虚拟节点 VNode。

对于模板编译后的匿名函数，并没有接收参数，而是用\_c，\_c 也被赋值为 createElement。

vm.$createElement 和 _c的区别就是最后的那个参数不同，vm.$createElement 调用的，也就是手写的 render，需要做正常规范化，而如果是模板编译的，可能只需要简单规范化或者无需规范化。

```js
// src/core/instance/render.js
vnode = render.call(vm._renderProxy, vm.$createElement);
```

```js
// src/core/instance/render.js
export function initRender(vm) {
  // ...
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false);
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true);
  // ...
}
```

### createElement

createElement 内部会调整下参数，将参数正确赋值，内部再调用\_createElement 去创建 vnode。

createElement 参数是 context，tag，data，children，normalizationType 和 alwaysNormalize

- context 就是组件实例或者 Vue 实例
- tag 可以是 html 标签名，也可以是组件对象。
- data 是一个与模板中 attribute 对应的数据对象，包含 class，style，attrs，props，domProps，on，nativeOn，directives，slot，scopedSlots，key，ref 等。
- children 是子级虚拟节点 (VNode)，也可以直接是一个文本。
- normalizationType 表示子节点规范的类型，类型不同，规范的方法也就不一样。
- 如果是手写的 render，调用的 vm.$createElement，需要做正常规范化。而如果是模板编译的，可能只需要简单规范化或者无需规范化。

```js
// src/core/vdom/create-element.js
export function createElement(context, tag, data, children, normalizationType, alwaysNormalize) {
  // 如果data是数组，或者string，number，boolean等进入if，如果data是对象，表示是手写的render，不进入if
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children; // 手写的一般没有这个值，但是模板编译后调用的可能会传这个值。
    children = data; // 将数据传给children，后面再将children规范化。
    data = undefined;
  }

  // 如果是手写的render，始终做children的规范化
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE;
  }
  return _createElement(context, tag, data, children, normalizationType);
}
```

在\_createElement 中会根据 tag 去真正的生成 VNode 实例。首先将 children 参数规范化，为什么需要规范化，因为 Virtual DOM 实际上是一个树状结构，每一个 VNode 可能会有若干个子节点，这些子节点应该也是 VNode 的类型。而 \_createElement 接收的第 4 个参数 children 是任意类型的，因此我们需要把它们转变成 VNode 类型。

其次通过判断 tag 标签，如果是普通的 html 标签，直接创建对应的 vnode，如果是对应已经注册的组件，就创建组件，创建组件时也会对应创建 vnode，将 vnode 返回。

```js
// src/core/vdom/create-element.js
export function _createElement(context, tag, data, children, normalizationType) {
  // 动态组件
  if (isDef(data) && isDef(data.is)) {
    tag = data.is;
  }

  // children规范化
  // 当手写render时，children 可能会写成基础类型用来创建单个简单的文本节点，这种情况会调用 createTextVNode 创建一个文本节点的 VNode
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children);
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    // simpleNormalizeChildren 方法调用场景是 render 函数是编译生成的。
    // 理论上编译生成的 children 都已经是 VNode 类型的，但这里有一个例外，就是函数式组件返回的是一个数组而不是一个根节点，所以会通过 Array.prototype.concat 方法把整个 children 数组打平，让它的深度只有一层。
    children = simpleNormalizeChildren(children);
  }

  // 下面就是创建一个vnode实例
  let vnode;
  // 如果tag参数是字符串，需要根据这个tag创建vnode
  if (typeof tag === 'string') {
    let Ctor;
    if (config.isReservedTag(tag)) {
      // 内置的标签，就创建普通的vnode
      vnode = new VNode(tag, data, children, undefined, undefined, context);
    } else if (
      (!data || !data.pre) &&
      isDef((Ctor = resolveAsset(context.$options, 'components', tag)))
    ) {
      // 如果tag名是已经注册的组件
      // 创建这个组件的vnode
      vnode = createComponent(Ctor, data, context, children, tag);
    } else {
      // 直接创建普通的vnode
      vnode = new VNode(tag, data, children, undefined, undefined, context);
    }
  } else {
    // 如果tag不是字符串，那就是组件对象，直接创建这个组件的vnode
    vnode = createComponent(tag, data, context, children);
  }
}
```

### createComponent

createComponent 中首先会获取到 VueComponent 这个子类，将 componentVNodeHooks 中的几个钩子函数 init，prepatch，insert，destroy 合并到 data.hook 中，然后生成一个 vnode 节点，将这个组件的相关信息传进去。

data.hook 中的那些钩子函数，在 VNode 执行 patch 的过程中会执行相关的钩子函数，具体的执行我们稍后在介绍 patch 过程中会详细介绍。

```js
// src/core/vdom/create-component.js
export function createComponent(Ctor, data, context, children, tag) {
  // 获取Vue类
  const baseCtor = context.$options._base;

  // Ctor参数一般会是个js对象，里面包含data，methods等等，将它转变成VueComponent类
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor);
  }

  data = data || {};
  // ...
  const listeners = data.on;

  // 将 componentVNodeHooks 中的几个钩子函数init，prepatch，insert，destroy合并到data.hook中。在 VNode 执行 patch 的过程中会执行相关的钩子函数
  installComponentHooks(data);

  // 创建vnode节点，注意：组件的 vnode 是没有 children 的
  const name = Ctor.options.name || tag;
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data,
    undefined,
    undefined,
    undefined,
    context,
    {
      Ctor,
      propsData,
      listeners,
      tag,
      children,
    } /* 这些参数会存在vnode的componentOptions中 */,
    asyncFactory
  );

  return vnode;
}
```

## patch 和 diff 算法

### patch

上面我们介绍了通过调用 vm.\_update 方法，将浏览器中的 DOM 进行更新。其中 vm.\_update 的核心就是调用 `vm.__patch__` 方法，将 vnode 转换为真实的 DOM。这个方法实际上在不同的平台，比如 web 和 weex 上的定义是不一样的，因此在 web 平台中它的定义在 src/platforms/web/runtime/index.js 中。

该 patch 方法的定义是调用 createPatchFunction 方法的返回值，这里传入了一个对象，包含 nodeOps 参数和 modules 参数。

- nodeOps 参数是一个对象，包含了一些基本的 DOM 操作方法，比如 createElement 用来创建 DOM 元素，appendChild 用来向父元素中添加子元素等等。

- modules 参数是一个数组，包含了不同阶段的一些工具函数的实现。比如 create 阶段的 updateClass 可以给 DOM 添加 class，updateAttrs 可以给 DOM 添加一些属性，updateDOMListeners 给 DOM 添加点击等事件。

createPatchFunction 内部定义了一系列的辅助方法，最终返回了一个 patch 方法，这个方法就赋值给了 `vm.__patch__`。

```js
const patch = createPatchFunction({ nodeOps, modules });
// src/platforms/web/runtime/index.js
Vue.prototype.__patch__ = patch;
```

回到 patch 方法本身，它将 vnode 转换为真实的 DOM。它的参数，oldVnode 表示旧的 VNode 节点，也可以不存在或者是一个 已存在的 DOM 对象（比如 Vue 实例初始化渲染时），vnode 表示执行 \_render 后返回的最新的 VNode 的节点。

- 如果旧的 VNode 不存在，就将新的 VNode 渲染成真实的 DOM，并插入到父元素中。
- 如果新的 VNode 不存在，就将旧的 VNode 对应的真实 DOM 元素从父元素中移除。
- 如果旧的 VNode 和新的 VNode 是相同的节点，就对比它们的属性和子节点，如果有变化就更新对应的 DOM 元素。
- 如果旧的 VNode 和新的 VNode 不是相同的节点，就将旧的 VNode 对应的真实 DOM 元素替换成新的 VNode 对应的真实 DOM 元素。

在进行对比和更新的过程中，patch 方法会调用一系列的钩子函数，比如 create、update、destroy 等等。这些钩子函数会在特定的时机被调用，用来执行一些特殊的操作，比如创建新的 DOM 元素、更新 DOM 元素的属性、监听事件等等

```js
// src/core/vdom/patch.js
export function createPatchFunction(backend) {
  // ...
  // 一些列辅助工具函数
  // ...

  return function patch(oldVnode, vnode) {
    // 新节点不存在，老节点存在，调用 destroy，销毁老节点
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode);
      return;
    }
    let isInitialPatch = false;
    const insertedVnodeQueue = [];

    // 组件第一次渲染时，oldVnode还是undefined
    if (isUndef(oldVnode)) {
      isInitialPatch = true;
      // 创建组件的DOM树
      createElm(vnode, insertedVnodeQueue);
    } else {
      const isRealElement = isDef(oldVnode.nodeType);
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // 不是真实元素，但是老节点和新节点是同一个节点，则是更新阶段，执行 patch 更新节点
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null);
      } else {
        // vue实例第一次渲染时，oldVnode 实际上是一个 DOM对象
        if (isRealElement) {
          // 创建一个空的vnode节点
          oldVnode = emptyNodeAt(oldVnode);
        }

        // 获取旧的节点
        const oldElm = oldVnode.elm;
        const parentElm = nodeOps.parentNode(oldElm);

        // 基于新 vnode 创建整棵 DOM 树并插入到 body 元素下
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        );

        // ...

        // 移除老的节点
        if (isDef(parentElm)) {
          removeVnodes([oldVnode], 0, 0);
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode);
        }
      }
    }

    // 调用组件的insert hook
    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch);
    return vnode.elm;
  };
}
```

### createElm

createElm 通过虚拟节点创建真实的 DOM 并插入到它的父节点中

```js
function createElm(vnode, insertedVnodeQueue, parentElm, refElm, nested, ownerArray, index) {
  // 尝试创建子组件，如果不是子组件它的返回值为 false
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    return;
  }

  const data = vnode.data;
  const children = vnode.children;
  const tag = vnode.tag;
  if (isDef(tag)) {
    // 创建HTML元素
    vnode.elm = nodeOps.createElement(tag, vnode);

    // 创建子元素，逻辑很简单，实际上是遍历子虚拟节点，递归调用 createElm，这是一种常用的深度优先的遍历算法，这里要注意的一点是在遍历过程中会把 vnode.elm 作为父容器的 DOM 节点占位符传入。
    createChildren(vnode, children, insertedVnodeQueue);
    if (isDef(data)) {
      // 调用create hook钩子
      invokeCreateHooks(vnode, insertedVnodeQueue);
    }
    // 把 DOM 插入到父节点中
    insert(parentElm, vnode.elm, refElm);
  }
}
```

### patchVnode

patchVnode 的作用就是把新的 vnode patch 到旧的 vnode 上

- 首先会判断是否是文本节点，如果是直接将文本内容替换就好，如果不是则开始对比 children。
- 如果只有新节点有 children，则执行 addVnodes，添加新子节点
- 如果只有旧节点有 children，则直接 removeVnode，删除旧节点
- 如果新旧节点都有 children，则执行 updateChildren 方法。

```js
function patchVnode(oldVnode, vnode, insertedVnodeQueue, ownerArray, index, removeOnly) {
  // 如果新旧node一样，直接返回
  if (oldVnode === vnode) {
    return;
  }

  let i;
  const data = vnode.data;
  // 执行prepatch 钩子
  if (isDef(data) && isDef((i = data.hook)) && isDef((i = i.prepatch))) {
    i(oldVnode, vnode);
  }

  // 获取两个vnode的子节点
  const oldCh = oldVnode.children;
  const ch = vnode.children;
  // 执行 update 钩子函数
  if (isDef(data) && isPatchable(vnode)) {
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode);
    if (isDef((i = data.hook)) && isDef((i = i.update))) i(oldVnode, vnode);
  }

  // 真正的patch过程
  if (isUndef(vnode.text)) {
    if (isDef(oldCh) && isDef(ch)) {
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly);
    } else if (isDef(ch)) {
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '');
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
    } else if (isDef(oldCh)) {
      removeVnodes(oldCh, 0, oldCh.length - 1);
    } else if (isDef(oldVnode.text)) {
      nodeOps.setTextContent(elm, '');
    }
  } else if (oldVnode.text !== vnode.text) {
    nodeOps.setTextContent(elm, vnode.text);
  }

  // 执行 postpatch 钩子函数
  if (isDef(data)) {
    if (isDef((i = data.hook)) && isDef((i = i.postpatch))) i(oldVnode, vnode);
  }
}
```

### diff 算法：updateChildren

diff 过程的实现主要是在 updateChildren 函数中。diff 过程：

- 对新老 VNode 的开始结束位置进行标记，4 个指针，分别指向新旧两个 vnode 数组的头尾。

- 标记好节点后，进入到 while 循环中

- 循环过程中首先对新老节点的首尾节点进行比较。每次对比下两个头指针指向的节点、两个尾指针指向的节点，头和尾指向的节点，是不是 key 是一样的，也就是可复用的。如果是可复用的话就直接用，调用 patch 更新一下，如果是头尾这种，还要移动下位置。这样就完成了节点的复用和移动的逻辑。

  - 头头比较
  - 尾尾比较
  - 头尾，需要移动
  - 尾头，需要移动

- 如果双端都没有可复用节点，那就在旧节点数组中找，找到了就把它移动过来，并且原位置置为 undefined。没找到的话就插入一个新的节点。

- 经过前面的移动之后，剩下的节点都被移动到了中间，如果新 vnode 有剩余，那就批量的新增，如果旧 vnode 有剩余那就批量的删除。

```js
function updateChildren(parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
  let oldStartIdx = 0;
  let newStartIdx = 0;
  let oldEndIdx = oldCh.length - 1;
  let oldStartVnode = oldCh[0];
  let oldEndVnode = oldCh[oldEndIdx];
  let newEndIdx = newCh.length - 1;
  let newStartVnode = newCh[0];
  let newEndVnode = newCh[newEndIdx];
  let oldKeyToIdx, idxInOld, vnodeToMove, refElm;

  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    // 空节点的处理逻辑
    if (isUndef(oldStartVnode)) {
      oldStartVnode = oldCh[++oldStartIdx];
    } else if (isUndef(oldEndVnode)) {
      oldEndVnode = oldCh[--oldEndIdx];
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      // 头头
      patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
      oldStartVnode = oldCh[++oldStartIdx];
      newStartVnode = newCh[++newStartIdx];
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      // 尾尾
      patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
      oldEndVnode = oldCh[--oldEndIdx];
      newEndVnode = newCh[--newEndIdx];
    } else if (sameVnode(oldStartVnode, newEndVnode)) {
      // 头尾
      patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
      nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
      oldStartVnode = oldCh[++oldStartIdx];
      newEndVnode = newCh[--newEndIdx];
    } else if (sameVnode(oldEndVnode, newStartVnode)) {
      // 尾头
      patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
      nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
      oldEndVnode = oldCh[--oldEndIdx];
      newStartVnode = newCh[++newStartIdx];
    } else {
      // 如果双端都没有可复用节点，那就在旧节点数组中找，找到了就把它移动过来
      if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx);
      idxInOld = isDef(newStartVnode.key)
        ? oldKeyToIdx[newStartVnode.key]
        : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
      if (isUndef(idxInOld)) {
        // New element
        createElm(
          newStartVnode,
          insertedVnodeQueue,
          parentElm,
          oldStartVnode.elm,
          false,
          newCh,
          newStartIdx
        );
      } else {
        vnodeToMove = oldCh[idxInOld];
        if (sameVnode(vnodeToMove, newStartVnode)) {
          patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
          oldCh[idxInOld] = undefined;
          nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
        } else {
          // same key but different element. treat as new element
          createElm(
            newStartVnode,
            insertedVnodeQueue,
            parentElm,
            oldStartVnode.elm,
            false,
            newCh,
            newStartIdx
          );
        }
      }
      newStartVnode = newCh[++newStartIdx];
    }
  }
  // 如果新 vnode 有剩余，那就批量的新增，如果旧 vnode 有剩余那就批量的删除。
  if (oldStartIdx > oldEndIdx) {
    refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
    addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
  } else if (newStartIdx > newEndIdx) {
    removeVnodes(oldCh, oldStartIdx, oldEndIdx);
  }
}
```

## 参考

https://ustbhuangyi.github.io/vue-analysis/
