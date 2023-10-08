> 基于vue@3.3.4源码

## Vue3 源码构建

1、下载 vue3.3.4 源码 `https://codeload.github.com/vuejs/core/zip/refs/tags/v3.3.4`

2、源码 .github/CONTRIBUTING.md 文件中有描述怎么安装依赖，使用 `pnpm i` 安装依赖

3、安装依赖的时候，遇到 puppeteer 下载问题，需要在项目根目录添加`.npmrc`，加上这一行代码`puppeteer-download-base-url="https://cdn.npmmirror.com/binaries/chrome-for-testing"`

4、在 package.json 中的 scripts 中添加如下命令 build-vue-only，主要是只打包全局的 vue 和输出 sourcemap 文件。package.json 中默认有 build 命令，是打包所有的 package，并且打包所有格式。

```bash
"build-vue-only": "node scripts/build.js vue --formats global --sourcemap true",
```

5、执行 `pnpm build-vue-only`，会在 packages/vue/dist 文件夹下生成 `vue.global.js` 和 `vue.global.js.map`，接下来就可以直接使用了。

## Vue3 源码目录

在项目目录运行命令 `tree -aI ".git*|.vscode|node_modules" -C -L 2`，会打印目录结构，我们只分析 packages 下面的目录。

```js
├── packages
│   ├── compiler-core        // 包含Vue编译器的核心功能，提供了编译器的基本逻辑和算法。比如编译成抽象语法树等。
│   ├── compiler-dom         // 包含Vue编译器针对浏览器环境的特定功能，例如处理DOM元素和属性。
│   ├── compiler-sfc         // 包含Vue单文件组件（SFC）编译器，用于将单文件组件转换为渲染函数。
│   ├── compiler-ssr         // 包含Vue服务器端渲染（SSR）编译器，用于将Vue组件编译为在服务器上渲染的函数。
│   ├── dts-test             // 包含用于测试Vue类型声明文件的相关工具和测试用例。
│   ├── global.d.ts          // 全局类型声明文件，用于定义全局变量和类型。
│   ├── reactivity           // 包含Vue响应式系统的实现，用于实现数据的响应式和依赖追踪。
│   ├── reactivity-transform // 包含用于转换Vue 2.x代码以适应Vue 3响应式系统的工具。
│   ├── runtime-core         // 包含Vue运行时的核心功能，提供了Vue实例的创建、组件的渲染等基本功能。
│   ├── runtime-dom          // 包含Vue运行时针对浏览器环境的特定功能，例如处理DOM操作和事件处理。
│   ├── runtime-test         // 包含用于测试Vue运行时的相关工具和测试用例。
│   ├── server-renderer      // 包含Vue服务器端渲染（SSR）的相关功能，用于在服务器上渲染Vue组件。
│   ├── sfc-playground       // 包含用于在浏览器中演示和测试Vue单文件组件的工具。
│   ├── shared               // 包含Vue内部共享的工具函数和常量。
│   ├── size-check           // 包含用于检查Vue库大小的工具和配置文件。
│   ├── template-explorer    // 包含用于在浏览器中探索和分析Vue模板的工具。
│   ├── vue                  // 包含面向用户的Vue的完整版本，包括编译器和运行时。
│   └── vue-compat           // 包含用于向后兼容Vue 2.x的功能和API的适配层。
```

其中最重要的 5 个模块，分别是：compiler-core，compiler-dom，compiler-sfc，runtime-core，runtime-dom，reactivity。

compile 可以理解为编译器，将 vue 的语法，代码，编译成 JS 的过程。而 runtime 则是程序运行时的一些方法，比如创建实例，生命周期，渲染等 vue 的核心功能。

## 打包生成的文件

package.json 中默认有 build 命令，是打包所有的 package，并且生成不同格式的文件。分别有 iife，cjs，es 格式。iife 表示立即执行函数，可以直接在 html 中使用；cjs 表示用在 Nodejs 环境，或其他支持 CommonJS 的环境；es 表示生成的文件遵循 ES6 的模块规范。

生成的文件名字分别是 `vue.global.js`，`vue.cjs.js`，`vue.esm-browser.js`，`vue.esm-bundler.js`，`vue.runtime.global.js`，`vue.runtime.esm-browser.js`，`vue.runtime.esm-bundler.js`。其中大致分为两种，一种是带 runtime 的，表示这个文件是纯 Vue 功能的，包含 Vue 运行时的各种方法，但是不支持模版语法等，也就是不支持编译。另外一种不带 runtime 的，表示是全部的代码，既包含编译部分，也包含 Vue 的运行时功能。

- cjs：CommonJS（CJS）模块格式的文件，适用于 Node.js 环境或其他支持 CommonJS 的环境。
- global：将 Vue 3 作为一个全局变量暴露给浏览器环境使用。
- global-runtime：仅包含运行时代码的全局变量模块文件，适用于浏览器环境。相比 global.js 就是少了 compile 方法。
- esm-bundler 和 esm-browser：这两个配置用于打包 ES 模块（ESM）格式的文件。'esm-bundler' 用于打包供 bundler（如 Rollup 或 Webpack）使用的 ES 模块文件，而 'esm-browser' 用于打包供浏览器环境使用的 ES 模块文件。

- esm-bundler-runtime 和 esm-browser-runtime：这两个配置用于打包仅包含运行时代码的 ES 模块文件。'esm-bundler-runtime' 用于 bundler 环境，如 Rollup 或 Webpack，而 'esm-browser-runtime' 用于浏览器环境。

## Vue3 使用和调试

1、在 packages/vue/examples 文件夹下创建 debug 文件夹，并且在 debug 文件夹下创建 index.html，App.js，index.js。

2、将下面的代码复制到对应文件中

3、浏览器运行 index.html，需要使用服务器启动这个 html

4、开始打断点调试，从 createApp 方法开始

```html
<script src="../../dist/vue.global.js"></script>

<!-- App组件的模板 -->
<script type="text/x-template" id="app">
  <div @click="add">{{name}}</div>
</script>

<!-- App根组件 -->
<script type="module" src="./App.js"></script>

<!-- 项目启动文件 -->
<script type="module" src="./index.js"></script>

<!-- 模拟渲染的根节点 -->
<div id="demo"></div>
```

```js
// App.js
const { ref } = Vue;
export default {
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
```

```js
// index.js
import App from './app.js';

const { createApp } = Vue;
const app = createApp(App);
app.mount('#demo');
```

## 其他

### 不能打断点的代码

打包的时候，是有 sourcemap 的，所以引入的 vue.global.js 在调试时会指向真正的未打包之前的源码。但是调试过程发现在源代码上，有的不能打断点，是灰色的，像下图。

![debug](https://cdn.lishuxue.site/blog/image/Vue/debug.png)

这些`__XXX__`类型的变量通常用于在开发环境和生产环境之间进行条件编译，以实现在开发阶段添加额外的调试信息或者去除一些不必要的代码。这些变量通常是在构建过程中通过构建工具的插件定义的。

vue3 打包时，用到了 rollup-plugin-esbuild 的 define 属性，设置了许多预定义常量。类似的功能还有 webpack.DefinePlugin，@rollup/plugin-replace 等插件。

所以那些灰色的代码，其实是因为在 vue.global.js 中不存在，所以源码调试时，相当于在源码中没有映射到该代码。

### 存粹的函数或表达式

源码中经常会看到类似于 `/*#__PURE__*/` 的注释，这是一种特殊的注释语法，用于告诉工具或编译器某个函数或表达式是纯粹的（pure），没有副作用的，可以安全的被替换或者删除。以便在编译过程中进行优化，或减小最终生成的代码的体积。

源码被打包成 vue.global.js 后，`/*#__PURE__*/` 注释会变为 `/* @__PURE__ */` 注释，这是 esbuild 做的，将这种注释统一了下。后续 Terser 压缩时可以识别，当然 Terser 中也支持 `#` 开头的注释。

### 打包时为什么 rollup 和 esbuild 同时使用

Vue3 在打包时，使用了 rollup-plugin-esbuild 插件，借助了 esbuild 在构建速度方面的强大优势。在 Rollup 构建过程中使用 esbuild 引擎进行快速代码转换，如 JSX 转换、TypeScript 转换等，而无需额外配置和使用 @rollup/plugin-babel 和 @rollup/plugin-typescript 插件。

同时由于 Rollup 的强大的生态，有大量的插件，且在处理模块化代码、代码拆分和库的构建方面有很强的优势。比如分模块打包，打包成不同格式的文件等。

因此，Vue 3 选择在 Rollup 中使用 rollup-plugin-esbuild 插件，以便充分利用 esbuild 引擎的性能优势，同时保留 Rollup 的灵活性和丰富的生态系统。这样可以在保证性能的同时，与现有的 Vue 生态系统和插件进行良好的集成。
