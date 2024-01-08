## webpack 是什么

webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

## entry

入口文件，webpack 从入口文件开始分析项目的依赖，创建依赖关系图，处理每一个依赖，最后输出到 bundle 文件中。

entry 入口文件的书写形式，可以是字符串，对象或者数组，name 默认是 main

- 字符串形式

```js
entry: './app.js'; // 等价于 entry: { main: './app.js' }
```

- 对象形式：每一对属性对，都代表着一个入口文件，因此多页面配置时，肯定是要用这种形式的 entry 配置。webpack 会分别分析其中的依赖，并打包成对应的 bundle.

```js
entry: { main: './app.js' }

// 多页面配置
entry: {
  main: './app.js',
  main2: './app2.js'
}
```

- 数组形式，webpack 会把数组里所有文件打包成一个 js 文件

```js
entry: ['./app.js', 'lodash'];
// 等价于 entry: { main: ['./app.js', 'lodash'] }
```

## output

output 属性告诉 webpack 在哪个文件夹输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist 文件夹。

- path：输出文件到哪个路径，绝对路径
- filename：main chunk 的名字，根据入口文件打包后的输出文件的名字
  - `[name]`: 用 entry name， 默认 name 就是 main
  - `[id]`: 内部生成的 chunk id
  - `[hash]`: 每次 webpack build，都会有一个唯一的 hash。对于热加载，每次修改任何一个文件，都会重新 build，hash 都会不一致。如果指定了长度，如`[hash:8]` ，那么它只会获取哈希值的前 8 位。devServer 建议用 hash
  - `[chunkhash]`: chunk 表示一个文件，每一个输出的文件都是一个 chunk，由入口文件和其依赖所构成。根据每个 chunk 或者叫每个入口文件，所对应的 hash，打包后可能一系列文件的 hash 是一样的，他们属于同一个 chunk
  - `[contenthash]`: 由原始文件内容产生的 hash 值，内容不同产生的 contenthash 值也不一样，打包后每个文件都是单独的 hash 值。
- chunkFilename：子 chunk 的名字。按需加载，动态导入的文件会被打包成单独的 chunk
- publicPath：用来指定你的 app 运行在哪个路径，如根路径 “ / ” 或者子路径 “ /subpath/ ”

chunkFilename 不能像 filename 中的 name 那样，可以在 entry 中定义。也就是说对于 chunkFilename，默认[id]和[name]是一样的。那么如何自定义 name 呢，就需要通过魔法注释`/* webpackChunkName: 'myChunk' */`

```js
import(/* webpackChunkName: 'myChunk' */ './myModule.js');
```

## 三个概念 module、chunk 和 bundle

module: 简单来说，你通过 import 语句引入的代码块，都叫 module。css，图片等也都叫 module

chunk: 是 webpack 根据功能拆分出来的，包含三种情况：

- 你的项目入口（entry）chunk，也就是 main chunk，一般这个 chunk 包含了你大部分的代码。
- 通过 import()动态引入的模块，会被打包为子 chunk
- 通过 splitChunks 拆分出来的代码, 也是子 chunk

bundle: 是 webpack 打包之后的各个文件，一般就是和 chunk 是一对一的关系。bundle 就是对 chunk 进行编译压缩打包等处理之后的产出。

## devtool

确定 source map 格式。不同的值会明显影响到构建(build)和重新构建(rebuild)的速度。

一般我们常用的有 source-map 或者 cheap-module-source-map。

source-map: 生成完整的.map 文件，该文件保存有原始代码与运行代码的映射关系，浏览器可以通过它找到原始代码的位置。

cheap-module-source-map: 也会生成.map 文件，但是不包含列信息。也就是说当你在浏览器中点击该代码的位置时， 光标只定位到行数，不定位到具体字符位置。而不包含 cheap 关键字时， 点击控制台 log 将会定位到字符位置。

- source-map： 产生.map 文件
- cheap： 不包含列信息，也不包含 loader 的 sourcemap
- module： 包含 loader 的 sourcemap（比如 jsx to js ，babel 的 sourcemap）

详细的解释参考：<https://segmentfault.com/a/1190000008315937>

## loader

webpack 自身只理解 javascript 文件，对于非 javascript 的文件，如 css, 图片等，就需要用 loader 去处理，它可以将这些文件转为 webpack 可以处理的模块。

常用的 loader 有：

- eslint-loader, 用于对代码进行 eslint 校验

- babel-loader, 将 jsx 或者 ES6 代码转换为 ES5
- url-loader，处理图片资源或字体文件或者其他媒体资源，可以配置将小图片转换为 base64 的加在代码中
- file-loader, 也是处理图片资源或者字体或者其他媒体资源。
- @svgr/webpack, 处理 svg 文件，把 svg 文件当做 React Component 使用
- style-loader, 把 js 中 import 导入的样式文件打包到 js 文件中，运行 js 文件时，将样式自动插入到`<style>`标签中。
- css-loader, 解释 @import 和 url() ，会 import/require() 后再解析它们。
- postcss-loader, 为 css 属性加上浏览器厂商前缀
- sass-loader, 加载 Sass/SCSS 文件且编译成 css
- less-loader, 加载 less 文件并编译成 css
- resolve-url-loader, 如过一个.sass 文件 import 了另一个文件，另一个文件中通过 url()引用了一个相对路径的资源，这样如果被其他 sass 文件引入以后，就会有路径问题，这个 loader 就是处理这个问题，主要是配合 sass 使用。less 提供了自己的方法去解决这个问题
- MiniCssExtractPlugin.loader， 把 js 中 import 导入的样式文件代码，打包成一个实际的 css 文件，这点跟 style-loader 不同，他们俩只需要其中一个。

## module 选项和 rules

module 选项决定了如何处理项目中的不同类型的模块，所以 loader 一般也是配在 module 选项中。

rules 制定规则来匹配不同的文件，然后 use 不同的 loader 去处理。

loader 可以通过 options 有自己的配置。

```js
rules: [
  {
    test: /\.(js|jsx|ts|tsx)$/,
    loader: 'babel-loader',
    exclude: /node_modules/,
    include: path.resolve(__dirname, '../src'),
  },
  {
    test: /\.(scss|sass)$/,
    use: [
      'style-loader',
      {
        loader: 'css-loader',
        options: {
          importLoaders: 2, // importLoaders用于配置「css-loader 作用于 @import 的资源之前」有多少个 loader。
        },
      },
      'postcss-loader',
      'sass-loader',
      'resolve-url-loader',
    ],
  },
];
```

**从上到下顺序执行每一个 rule**

**每个 rule 中如果有多个 loader，会从后往前执行。**

`use: ["style-loader"]` 是 loader 属性的简写方式，如：`use: [{ loader: "style-loader"}]`

`Rule.loader` 是 `Rule.use: [{ loader }]` 的简写，而且 `Rule.options` 又是 `Rule.use: [{ options }]` 的简写。

所以，下面两种写法是等价的

```js
{
  test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
  loader: "url-loader",
  options: {
    limit: 10 * 1024, // 单位 byte， 1 kb = 1024 byte
    name: "static/media/[name].[hash:8].[ext]"
  }
}
// 等价于
{
  test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
  use: [{
    loader: "url-loader",
    options: {
      limit: 10 * 1024, // 单位 byte， 1 kb = 1024 byte
      name: "static/media/[name].[hash:8].[ext]"
    }
  }]
}
```

## externals

引入第三方库， 但是不希望这个库被 webpack 打包。

## plugins

loader 被用于转换某些类型的模块，而 plugin 可以用于以多种方式自定义 webpack 构建过程，从打包优化和压缩，一直到重新定义环境中的变量。

插件目的在于解决 loader 无法实现的其他事。可以用来处理各种各样的任务，比 loader 范围更广，功能更强大。

使用一个插件，只需要 require() 它，然后 new 一个实例，添加到 plugins 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件。

从上到下加载数组中的插件，插件一般会定义一些 hook,作用于 webpack 编译的具体生命周期。

常见的插件有：

- TerserPlugin： 压缩 js 代码, 一般用于 optimization 选项

- OptimizeCSSAssetsPlugin： 压缩 css 代码，一般用于 optimization 选项
- HtmlWebpackPlugin： 生成 HTML 文件，并将打包的 bundle 和 chunk 引入到 HTML 中
- InlineChunkHtmlPlugin： 将 webpack 运行时脚本内联到 html 中，即把 runtime-main.xxxx.js 脚本内容放在 html 的 script 标签中
- InterpolateHtmlPlugin：在`index.html`中使用自定义变量
- webpack.DefinePlugin：在 js 代码中使用自定义变量
- webpack.HotModuleReplacementPlugin：CSS 更改热替换，js 更改刷新页面
- webpack.IgnorePlugin: 忽略第三方库的指定目录，这些目录不会被打包
- MiniCssExtractPlugin: 该插件将 CSS 提取到单独的文件中，可以指定输出文件名
- ManifestPlugin: 生成 asset-manifest.json 文件，即资源清单。webpack runtime 会通过 manifest 来解析和加载模块
- ForkTsCheckerWebpackPlugin：执行 typescript 的类型检查
- CleanWebpackPlugin： 清空上次打包的文件夹
- CopyPlugin： Copy 指定文件夹下的内容到指定文件夹
- WatchMissingNodeModulesPlugin：安装新的依赖无需重启
- ModuleNotFoundPlugin： 为 "module not found" 的错误提供了一些必要的上下文
- SplitChunksPlugin： 用来提取第三方库和公共模块

## resolve 选项 和 模块解析(module resolution)

resolve 选项用来设置模块的解析方式，如何寻找这些模块。

webpack 使用 enhanced-resolve 来解析文件路径。

### 解析规则

1. 绝对路径

```js
import '/home/me/file';
import 'C:\\Users\\me\\file';
```

无需额外解析

2. 相对路径

```js
import '../src/file1';
import './file2';
```

当前文件所在的目录被认为是上下文路径。在 import/require 中给定的相对路径，会添加此上下文路径，以产生模块的绝对路径。

3. 模块路径

```js
import 'module';
import 'module/lib/file';
```

模块将在 resolve.modules 中指定的所有目录内搜索。 你可以替换初始模块路径，此替换路径通过使用 resolve.alias 配置选项来创建一个别名。

一旦根据上述规则解析路径后，解析器(resolver)将检查路径是指向文件还是目录。如果路径指向一个文件（看是否有 resolve.extensions 中包含的文件扩展名），直接打包。如果指向一个文件夹，就默认用文件夹下面的 index 文件。

```js
  resolve: {
    // 告诉 webpack 解析模块时应该搜索的目录
    modules: ["node_modules"],
    // 如果没有加扩展名，默认找这些扩展名的文件
    extensions: [".js", ".ts", ".jsx", ".tsx", ".json"],
    // 创建 import 或 require 文件时的别名，来确保模块引入变得更简单。末尾添加 $，以表示精准匹配
    alias: {
      src$: path.resolve(__dirname, "../src"),
      components$: path.resolve(__dirname, "../src/components")
    },
    // 使用一些额外的插件列表来解析模块
    plugins: [PnpWebpackPlugin]
  },
```

## Runtime && Manifest

webpack 的 runtime 和 manifest，管理所有模块的交互。

### 什么是 runtime

简单来说，runtime 指的是 webpack 的运行环境(具体作用就是模块解析, 加载)

详细来说 ，webpack 的 runtime 主要是指：在浏览器运行过程中，webpack 用来连接模块化应用程序所需的所有代码。它包含：在模块交互时，连接模块所需的加载和解析逻辑。已经加载到浏览器中的连接模块逻辑，以及尚未加载模块的延迟加载逻辑。

一般我们会将 runtime 代码抽出来放到一个文件中。

### 什么是 manifest

当 webpack compiler 开始执行、解析和映射应用程序时，它会保留所有模块的详细要点。这个数据集合称为"manifest"。

### 为什么需要 manifest

在你的应用程序中，形如 index.html 文件、js 文件和各种资源，在经过打包、压缩、为延迟加载而拆分为细小的 chunk 等这些 webpack 优化之后，你的 /src 目录的文件结构都已经不再存在。所以 webpack 如何管理所有所需模块之间的交互呢？这就是 manifest 数据用途的由来

### 如何寻找模块

当完成打包并发送到浏览器时，runtime 会通过 manifest 来解析和加载模块。无论你选择哪种 模块语法，那些 import 或 require 语句现在都已经转换为 **webpack_require** 方法，此方法指向模块标识符(module identifier)。通过使用 manifest 中的数据，runtime 将能够检索这些标识符，找出每个标识符背后对应的模块。

## 依赖图 dependency graph

任何时候，一个文件依赖于另一个文件，webpack 就把此视为文件之间有 依赖关系 。这使得 webpack 可以接收非代码资源(non-code asset)（例如图像或 web 字体），并且可以把它们作为 _依赖_ 提供给你的应用程序。

webpack 从命令行或配置文件中定义的一个模块列表开始，处理你的应用程序。 从这些 入口起点 开始，webpack 递归地构建一个 依赖图 ，这个依赖图包含着应用程序所需的每个模块，然后将所有这些模块打包为一个或者多个 bundle.

`webpack-bundle-analyzer`可以用来分析 bundle 的大小占比

## HMR && live reloading

HMR, hot module replacement 模块热替换。
live reloading, 实时重新加载

HMR 功能会在应用程序运行过程中，替换、添加或删除 模块，而无需重新加载整个页面。

CSS 变化会热更新，js 变化会触发浏览器刷新。

## Tree Shaking

tree shaking 是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。它依赖于 ES2015 模块语法的 静态结构 特性，例如 import 和 export。这个术语和概念实际上是由 ES2015 模块打包工具 rollup 普及起来的。

可以通过这个来优化代码，详细配置：<https://webpack.docschina.org/guides/tree-shaking/>

## webpack 简单配置

```js
const webpackConfig = function(isDev) {
    return {
        mode: isDev ? 'development' : 'production',
        devtool: isDev ? 'cheap-module-source-map' : 'source-map',
        devServer: {

        },
        entry: {
            main: 'index.js'
        },
        output: {
            path: 'xx/xx/dist',
            publicPath:
            filename:
            chunkFilename:
        },
        optimization: {
            minimize: !isDev,
            minimizer: [],
            splitChunks: {

            }
        },
        resolve: {
            modules: [],
            extensions: []
            alias: {

            }
        },
        module: {
            rules: [
                {
                    test: /\.(js|jsx|ts|tsx)$/,
                    loader: "babel-loader",
                    exclude: /node_modules/,
                    include: path.resolve(__dirname, "../src")
                }
            ]
        },
        externals: {
            // 引入第三方库， 但是不希望这个库被webpack打包。
        },
        plugins: [
            new CopyWebpackPlugin({...}),
            new CompressionWebpackPlugin({...})
        ],
        node: {
            module: 'empty',
            dgram: 'empty',
            dns: 'mock',
            fs: 'empty',
            http2: 'empty',
            net: 'empty',
            tls: 'empty',
            child_process: 'empty',
        },
    }
}
```
