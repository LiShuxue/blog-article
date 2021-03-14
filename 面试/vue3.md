## vue3都做了哪些改变
1. 重构响应式系统，使用Proxy替换Object.defineProperty，使用Proxy优势：
2. 新增Composition API，更好的逻辑复用和代码组织
3. 重构 Virtual DOM
4. 代码结构调整，更便于Tree shaking，使得体积更小
5. 使用Typescript替换Flow

## Vue3.0 里为什么要用 Proxy 替代 defineProperty
1. 可直接监听数组类型的数据变化
2. 监听的目标为对象本身，不需要像Object.defineProperty一样遍历每个属性，有一定的性能提升
3. 可拦截apply、ownKeys、has等13种方法，而Object.defineProperty不行

## Vue3.0 编译做了哪些优化？
1. 模板编译时的优化，将一些静态节点编译成常量，渲染时只用关心动态节点。
2. slot优化，将slot编译为lazy函数，将slot的渲染的决定权交给子组件
3. diff算法优化

## 为什么要新增Composition API，它能解决什么问题
1. 更好的按业务逻辑去组织代码
2. 解决mixins复用导致的问题

##  Composition API 与 React.js 中 Hooks的异同点
### hooks
* 每次组件重新渲染hooks都会执行
* 不能在循环、条件、嵌套函数中调用Hook
* 必须确保总是在你的React函数的顶层调用Hook
* useEffect、useMemo等函数必须手动确定依赖关系

### Composition API
* 声明在setup函数内，一次组件实例化只调用一次setup
* 可以在循环、条件、嵌套函数中使用
* 响应式系统自动实现了依赖收集，进而组件的部分的性能优化由Vue内部自己完成，而React Hook需要手动传入依赖，而且必须保证依赖的顺序，让useEffect、useMemo等函数正确的捕获依赖变量，否则会由于依赖不正确使得组件性能下降。