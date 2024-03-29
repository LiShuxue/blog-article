## vue3 都做了哪些改变

1. 重构响应式系统，使用 Proxy 替换 Object.defineProperty
2. 新增 Composition API，setup，更好的逻辑复用和代码组织
3. 重构了虚拟 DOM，模板编译优化，slot 优化，diff 算法优化
4. 代码结构调整，将一些全局代码抽出来，更便于 Tree shaking，使得体积更小，如 nextTick
5. 使用 Typescript
6. 支持在 `<style></style>`里使用 v-bind，给 CSS 绑定 JS 变量(color: v-bind(str))

## Vue3.0 里为什么要用 Proxy 替代 defineProperty

1. 可直接监听数组类型的数据变化，及对象新增属性
2. 监听的目标为对象本身，不需要像 Object.defineProperty 一样遍历每个属性，有一定的性能提升
3. 可拦截 apply、ownKeys、has 等 13 种方法，而 Object.defineProperty 不行

## Vue3.0 编译做了哪些优化？

1. 模板编译时的优化，将一些静态节点编译成常量，渲染时只用关心动态节点。（虚拟 dom 只比较动态变化的部分，以前是比较整个 template 树）
2. template 中不需要唯一根节点，可以直接放文本或者同级标签
3. slot 优化，将 slot 编译为 lazy 函数，将 slot 的渲染的决定权交给子组件。
4. diff 算法优化，使用 最长递增子序列 优化了对比流程
5. Vue2 当中在父组件渲染同时，子组件也会渲染。 Vue3 就可以单独渲染父组件、子组件。

## 为什么要新增 Composition API，它能解决什么问题

1. 更好的按业务逻辑去组织代码
2. 解决 mixins 复用导致的问题，方法属性容易覆盖，不能传参，调试麻烦。

## Composition API 与 React.js 中 Hooks 的异同点

### react hooks

- 每次组件重新渲染 hooks 都会执行
- 不能在循环、条件、嵌套函数中调用 Hook
- 必须确保总是在你的 React 函数的顶层调用 Hook
- useEffect、useMemo 等函数必须手动确定依赖关系

### Composition API

- 声明在 setup 函数内，一次组件实例化只调用一次 setup。
- 可以在循环、条件、嵌套函数中使用
- 响应式系统自动实现了依赖收集，进而组件的部分的性能优化由 Vue 内部自己完成，而 React Hook 需要手动传入依赖，而且必须保证依赖的顺序，让 useEffect、useMemo 等函数正确的捕获依赖变量，否则会由于依赖不正确使得组件性能下降。

## vite 是啥

- vite 是一个开发构建工具，开发过程中它利用浏览器支持原生 ES 模块的特性导入组织代码，生产中利用 rollup 作为打包工具。
- 光速启动，没有打包步骤，不需要生成 bundle，esbuild 预构建
- 热模块替换
- 按需编译，只在需要某个模块的时候动态（借助 import() ）的引入它，不需要提前打包

### ES module

浏览器中可以直接使用 export import 的方式导入和导出模块。在 script 标签里设置 type="module"，然后使用模块内容。

```html
<script type="module">
  import { bar } from './bar.js‘
</script>
```

当 html 里嵌入上面的 script 标签时候，浏览器会发起 http 请求，请求 htttp server 托管的 bar.js，bar.js 中导出 bar 变量

```js
// bar.js
export const bar = 'bar';
```

### vite 如何处理 ESM

在浏览器里使用 ES module 是使用 http 请求拿到模块，所以 vite 必须提供一个 web server 去代理这些模块，vite 使用 koa 负责这个事情，通过对请求路径的劫持获取资源的内容返回给浏览器

### esbuild 预构建

1. Vite 的开发服务器将所有代码视为原生 ES 模块。因此，Vite 必须先将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM
2. Vite 将有许多内部模块的 ESM 依赖关系转换为单个模块，以提高后续页面加载性能。如 lodash
