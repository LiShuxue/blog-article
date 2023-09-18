## Rollup

Rollup 是一个用于 JavaScript 模块打包的工具，它允许您将多个 JavaScript 模块文件合并成一个或多个输出文件。它专注于支持 ES 模块（ECMAScript 模块规范），并提供了强大的树摇（tree shaking）功能，以删除未使用的代码，以减小最终输出文件的大小。

### 配置

一般用 rollup.config.js 配置文件来配置 rullup，打包时用命令 `rollup -c`，他会寻找根目录下的配置文件。

默认情况下，Rollup 会使用 Node 原生的模块机制来加载 rollup.config.js。如果配置文件是 ES6 模块 `export default { ... }`，需要在 package. js 中定义 "type": "module"，或者是将配置文件修改成 .mjs 后缀文件名。或者使用 `rollup -c --bundleConfigAsCjs` 打包。

如果是 `module.exports = { ... }` 的形式，则不需要。

```js
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'umd', // 输出格式
  },
};
```

#### input

input 指定打包入口文件，支持字符串、数组。对象三种配置方式。

input 是字符串时单入口打包，使用数组或者对象类型时，那么入口将被打包到独立的输出区块（chunks），简单来说，就是有几个入口文件，打包出来就有几个输出文件。

#### output

**output.file**

该选项用于指定打包后的 js 文件名。只有当生成的 chunk 不超过一个时，该选项才会生效。

**output.dir**

该选项用于指定所有生成 chunk 文件所在的目录。input 类型为对象或者数组时，该选项是必须的！！（即如果生成多个 chunk，则此选项是必须的。）

**output.format**

该选项用于指定生成 bundle 的格式。可以是以下之一：

- amd - 异步模块定义，适用于 RequireJS 等模块加载器

- cjs - CommonJS，适用于 Node 环境和其他打包工具

- es - 将 bundle 保留为 ES 模块文件，适用于其他打包工具以及支持

- iife - 自执行函数

- umd - 通用模块定义，生成的包同时支持 amd、cjs 和 iife 三种格式

**output.name**
当输出格式指定为 iife 或 umd，必须使用全局变量名来表示你的 bundle 。

例如，如果您将 output.name 设置为 "MyLibrary"，则其他脚本可以使用 window.MyLibrary 来访问您的模块。

**output.entryFileNames**

该选项用于指定 chunks 的入口文件名(意味着 input 是对象或数组类型)。默认值：`entryFileNames: "[name].js"`。支持以下形式：

- [format]：输出（output）选项中定义的 format 的值，例如：es 或 cjs。

- [hash]：哈希值，由入口文件本身的内容和所有它依赖的文件的内容共同组成。

- [name]：入口文件的文件名（不包含扩展名），当入口文件（entry）定义为对象时，它的值时对象的键。

**output.globals**

output.globals 用于告诉 Rollup 某些模块是外部依赖项，应该由外部环境提供。

当您将代码打包为 UMD 或 IIFE 格式时，这个选项通常与 external 配置选项一起使用，以指定哪些模块应该被视为外部依赖项。

#### external

指出哪些模块需要被视为外部引入，如 `external: ['lodash']`

### plugins

添加一些 rollup 插件

1. @rollup/plugin-node-resolve 插件可以让 Rollup 使用 node 的方式来查找文件或者外部模块。

2. 如果源码是用 ES6+，为了能够在目标浏览器和 nodejs 上使用一些不支持的特性，我们在打包时需要使用 Babel 来编译或者增加 polyfill。使用@rollup/plugin-babel 插件

3. rollup 默认都是使用 ES 模块处理，如果 import 的包是 commonjs 的会有问题，所以需要 commonjs 插件，将包转成 ES 模块的。

## 创建库

### 初始化

```sh
mkdir npm-test
cd npm-test
pnpm init
```

### 修改 package.json

首先在 package.json 中修改一些关键字段

- main: 对应 commonjs 引入方式的程序入口文件
- module: 对应 esmodule 引入方式的程序入口文件，这个字段并不是 package.json 的规范，是为了 rollup 或者 webpack 使用的。
- types: 描述了程序中所有组件以及变量的类型定义
- type: 在 node 支持 ES 模块后，要求 ES 模块采用 .mjs 后缀文件名。只要遇到 .mjs 文件，就认为它是 ES 模块。如果不想修改文件后缀，就可以在 package.json 文件中，指定 type 字段为 module。
- files: 项目在进行 npm 发布时，上传时需要包含的文件，默认会包括 package.json，license，README 和 main 字段里指定的文件。

当同时指定了 "module" 和 "main" 字段，但它们指定的文件路径不一样时，在代码中使用 `import NpmTest from "npm-test-lsx"`这样的导入语句时，具体行为取决于运行环境的模块系统支持：

- Node.js 环境：Node.js 会根据 "main" 字段指定的入口文件加载模块内容。

- 打包工具：现代打包工具通常会根据 "module" 字段指定的 ES6 模块入口加载模块内容。

如果 nodejs 想使用 import 和 require 同时导入

1. 分开写，require 的用在 .js 文件中，import 的写在 .mjs 中，node 会自己处理。

2. 使用 esm 库

```json
"repository": {
  "type": "git",
  "url": "https://github.com/LiShuxue/npm-test.git"
},
"main": "dist/umd/index.js",
"module": "dist/esm/index.js",
"files": [
  "dist/",
  "src/"
],
```

### 打包

增加 script build，并运行 `pnpm build`

```json
"scripts": {
  "build": "rollup -c"
}
```

## 上传到 npm

### 登录

如果没有账号，先注册：<https://www.npmjs.com/signup>

```sh
npm login --registry=https://registry.npmjs.org/
```

### 上传

```sh
npm publish --registry=https://registry.npmjs.org/
```

上传的时候遇到了问题：403 Forbidden - PUT <https://registry.npmjs.org/npm-test> - You do not have permission to publish "npm-test".

最终发现是 npm 上已经有这个包，跟别人重名了，所以没权限，修改了名字 npm-test-lsx 后成功。

## 附

demo 源码: <https://github.com/LiShuxue/npm-test>
