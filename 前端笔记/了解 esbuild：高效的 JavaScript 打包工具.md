## 介绍

### 什么是 ESbuild

ESbuild 是一个快速、高效的 JavaScript 打包工具，它使用 Go 语言编写，旨在提供极快的打包和转换速度。

ESbuild 主要用于打包和构建 JavaScript 代码，它的目标是将多个 JavaScript 模块合并成一个或多个输出文件，并可选地进行代码压缩和优化

与其他流行的打包工具（如 Webpack 和 Rollup）不同，ESbuild 的目标是尽可能快速地构建 JavaScript 代码。

### 优势

- ESbuild 的最大优势之一是其惊人的构建速度。它通过并行处理文件和使用高度优化的 Go 代码实现了卓越的构建速度。
- ESbuild 提供了强大的优化功能，包括代码压缩、去除未使用的代码、摇树优化（tree shaking）等。这些功能可以显著减小生成的代码文件大小，提高应用程序的性能。
- ESbuild 易于集成到现有的构建流程中。它可以作为命令行工具使用，也可以通过 Node.js API 进行配置和集成。

## 与其他工具的对比

### 与 Webpack 对比

与 Webpack 相比，ESbuild 通常更快速。它不需要大量的配置文件，因为默认设置已经足够好。但与 Webpack 不同，ESbuild 不支持加载资源（如图像或样式表），因此在处理资源时可能需要额外的工具。

### 与 Rollup 对比

Rollup 与 ESbuild 一样专注于速度和性能，ESbuild 的性能可能更优。但 Rollup 拥有丰富的插件生态系统，提供了更灵活的配置选项，支持许多不同的任务，如处理 CSS、图片、HTML 等。ESbuild 也有一些插件，但它的生态系统相对较小。

ESbuild 适用于需要极快构建速度的项目，尤其是在开发过程中需要频繁构建的情况下。Rollup 适用于需要更高度定制的项目，以及需要处理各种资源类型的项目。

### 与 TypeScript Compiler (tsc) 对比

ESbuild 可以用于构建 TypeScript 项目，打包成一个文件，但不提供类型检查功能。而 tsc 的主要是将 TypeScript 代码转换为 JavaScript 代码，并进行类型检查。

ESbuild 在编译 TypeScript 代码方面比 TypeScript 本身的编译器更快。ESbuild 使用了增量编译和并行处理的技术，使得 TypeScript 代码的编译速度得到显著提升。

当你同时在 ESbuild 配置和 tsconfig.json 文件中设置了相同的选项，例如 target 和 outDir 时，ESbuild 通常会采用以下规则：ESbuild 配置优先，如果没有就使用 tsconfig.json 的。

### 与 Babel 对比

ESbuild 默认 target 输出的代码是 esnext 格式的，因为它专注于速度和性能，所以它会尽可能地保留原始代码的语法和特性。你可以将 target 设置为其他值，但是最低仅支持 es6，如果将代码转换为更低的 JavaScript 版本可能会导致某些高级语法和特性无法转换或无法正常工作。

如果你需要将 ES6 语法转换为 ES5 语法以提供更广泛的兼容性，可以使用其他工具，如 Babel，来对 ESbuild 生成的代码进行后处理。

在实际项目中，通常会使用 Babel 与 ESbuild 结合，先使用 ESbuild 进行快速打包和构建，然后使用 Babel 进行必要的语法转换，以确保兼容性。这种组合可以充分利用 ESbuild 的速度和性能，同时保证最终生成的代码能够在不同浏览器和环境中正常运行。

### 与 Terser 比较

ESbuild 配置简单，有基本的压缩功能，压缩和优化代码的速度比 Terser 稍快一些，但是社区相对较小。

Terser 有一个庞大的生态系统，而且提供了更多的配置选项，允许你微调压缩的行为。它更适合需要特殊定制的项目。

## 缺点

前面说了那么多，好像 ESbuild 很完美，可以统一前端构建工具了，但它没有缺点吗，并不是。

- 相对于其他前端构建工具（如 webpack 和 Rollup），ESbuild 的生态系统和插件支持相对较新和有限，社区支持和文档资源相对较少。
- ESbuild 的设计目标是提供一个快速、简单易用的打包工具，因此它在功能上相对较少。与其他构建工具相比，ESbuild 可能缺少一些高级功能，如代码分割、动态导入、自定义插件等。
- ESbuild 的配置选项相对较少，可能不支持某些特定的构建需求。如果你的项目需要更复杂的配置选项，如自定义 loaders、解析规则、优化选项等，那么 ESbuild 可能无法满足你的需求。

## 使用介绍

ESbuild 可以将高级的语法转换为 ES6 语法，也可以将多个 ES6 模块文件打包成一个单独的文件，并进行一些优化，例如删除未使用的代码、压缩代码、提取公共模块等。

### 命令行使用

可以在命令行中直接使用 ESbuild 运行构建任务。例如，以下命令将一个入口文件（例如 main.js）打包成一个输出文件（例如 bundle.js）：

```sh
npx ESbuild main.js --bundle --outfile=bundle.js
```

### 常用配置选项

下面是一些常用的 ESbuild 配置选项：

- entryPoints: 指定入口文件或文件数组。
- outfile: 指定输出文件的名称。
- bundle：是否将所有依赖打包为单个文件，默认为 false。
- format：指定输出的模块格式，如 iife、umd、cjs、esm 等。
- target：指定目标浏览器或 Node.js 版本，以进行代码转换和优化。
- minify：是否压缩代码，默认为 false。
- sourcemap：是否生成源映射文件，默认为 false。
- splitting：是否启用代码拆分，默认为 false。
- external：指定需要排除的外部依赖模块，以便在打包时不包含它们。
- plugins：使用插件来扩展 ESbuild 的功能，如处理 CSS、TypeScript 等。
- jsxFactory 和 jsxFragment：指定 JSX 的工厂函数和片段标识符。
- define：预定义全局变量或常量，在代码中会替换。
- loader：自定义加载器，用于处理特定类型的文件。
- tsconfig: 指定 tsconfig.json 的路径

除了这些以外，还有更多的配置项可以使用，参考：<https://esbuild.github.io/api/>
![esbuildapi](https://cdn.lishuxue.site/blog/image/前端笔记/esbuildapi.png)

### JavaScript API

#### build

对于更复杂的配置需求，你可以在 JavaScript 代码中使用 ESbuild 的 `build()` 进行配置。配置完之后，使用 node 运行该 js 文件即可。

还有个 `buildSync()` 是 build 的同步版本。

```js
const { build } = require('ESbuild');

build({
  entryPoints: ['src/index.js'],
  outfile: 'dist/bundle.js',
  bundle: true,
  minify: true,
  sourcemap: true,
  target: 'es2015',
}).catch(() => process.exit(1));
```

#### transform

这个方法允许你单独转换一串 js 代码字符串，而不需要构建整个项目。它返回一个包含转换结果的对象，包括转换后的代码串和映射。

也有个同步版本：`transformSync()`

```js
const { transformSync } = require('esbuild');
const code = `
// JavaScript 代码
const a = () => {
  let a = 1;
  console.log(a);
};
a();
`;
const result = transformSync(code, {
  loader: 'js',
  target: 'es2015',
  sourcemap: true,
});
console.log(result);
```

## Loader

ESbuild 的 loader 选项用于指定不同类型文件的加载器。加载器负责将特定类型的文件转换为 JavaScript 导出的模块，以便 ESbuild 可以对其进行处理和打包。

ESbuild 内置提供了很多 loader，可以自动根据后缀名，自动开启，一般不需要自己配置。然而，如果你需要进行一些特殊的操作，或者希望自定义 loader 的行为，那么你可以手动配置 loader，以满足项目的特定需求。

### 内置 loader

- js：用于加载和转换 JavaScript 文件。这个 loader 默认对以下后缀的文件开启 .js，.cjs，.mjs。所以他可以将 commonjs 的文件转成 es6 的。
- jsx：用于加载和转换 JSX 文件。这个 loader 默认对 .jsx 文件开启。
- ts：用于加载和转换 TypeScript 文件。这个 loader 默认对以下后缀的文件开启 .ts, .mts, .cts。
- tsx：用于加载和转换 TypeScript + JSX 文件。这个 loader 默认对以 .tsx 文件开启。
- json：用于加载和转换 JSON 文件。这个 loader 默认对 .json 文件开启。
- css: 用于加载和转换 CSS 文件。对 .css 文件开启。
- local-css: 用于加载和转换 CSS 模块文件。对 .module.css 文件开启。
- text: 默认情况下为 .txt 文件启用此加载程序。它在构建时将文件加载为字符串，并将该字符串导出为默认导出
- binary: 处理文件，并使用 Base64 编码将其嵌入到 bundle 中。在运行时，原始文件的字节会从 Base64 解码，然后作为 Uint8Array 导出。
- base64: 处理文件，并使用 Base64 编码将其嵌入到 bundle 中，作为字符串导出。
- dataurl: 处理文件，并使用 Base64 编码的 data URL 将其嵌入到 bundle 中。这个 data URL 会以字符串形式导出，可以在运行时使用。
- file: 处理文件，该加载程序会将文件复制到输出目录，并将文件名作为字符串嵌入到包中。该字符串使用默认导出导出。

还有 copy 和 empty 等 loader，对 loader 的更多详细了解可以参考：<https://esbuild.github.io/content-types/>

如果需要自定义配置 loader，可以按后缀名指定

```js
loader: {
    '.txt': 'text',
    '.jpg': 'binary',
},
```

## 插件

ESbuild 插件是一种扩展 ESbuild 功能的方式，它们可以在构建过程中对代码进行转换、优化和处理。自定义插件也可以用于添加自定义功能、创建自定义加载器等。

一些社区的插件列表：<https://github.com/esbuild/community-plugins>

### 自定义插件

ESbuild 插件是一个具有 name 和 setup 函数的对象。它们以数组形式传递给 build API 调用。每次 build API 调用都会运行一次 setup 函数。

更多详细内容可参考：<https://esbuild.github.io/plugins/>
