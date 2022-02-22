## entry output 
* entry: 入口文件， 可以是数组或者对象形式，可以配置多入口
    ```js
    entry: 'index.js'

    entry: ['index.js']

    entry : { main: 'index.js' }

    // 多页面，多入口
    entry: {
        a: a.js,
        b: b.js
    }
    ```
* output: 输出文件，配置路径，文件名，chunk文件名等。
    ```js
    output: {
        path: xxx绝对路径,
        filename: [name].[id].[hash].[chunkhash].[contenthash].js
        chunkFilename: [name].[id].js, // 这个name并不是ntry入口的名字， 而是魔法注释 /* webpackChunkName: 'myChunk' */
        publicPath: xxx, // 指定app运行在哪个路径， 如根路径 “ / ” 或者子路径 “ /subpath/ ”
    }
    ```

## loader和plugins的区别
* loader: webpack自身只支持js和json这两种格式的文件，对于其他文件需要通过loader将其转换为commonJS规范的文件后，webpack才能解析到。Loader是在打包构建过程中用来处理源文件的（JSX，Scss，Less..），一次处理一个。
* plugin: 插件用于扩展webpack的功能，作用于webpack的整个生命周期，比loader更强大。plugin可以监听webpack的生命周期事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。


## 有哪些常见的Loader？
```js
css-loader, style-loader, sass-loader, less-loader, url-loader, file-loader, babel-loader, eslint-loader
```

## 有哪些常见的Plugin？
* HtmlWebpackPlugin： 生成HTML文件，并将打包的bundle和chunk引入到HTML中
* webpack.DefinePlugin：在js代码中使用自定义变量
* InterpolateHtmlPlugin：在index.html中使用自定义变量
* MiniCssExtractPlugin: 该插件将CSS提取到单独的文件中，可以指定输出文件名
* CleanWebpackPlugin： 清空上次打包的文件夹
* CopyPlugin： Copy指定文件夹下的内容到指定文件夹
* SplitChunksPlugin： 用来提取第三方库和公共模块
* TerserPlugin： 压缩js代码, 一般用于optimization选项
* OptimizeCSSAssetsPlugin： 压缩css代码，一般用于optimization选项

## 如何编写一个plugin
1. 创建一个类，包含apply方法
2. 逻辑在apply中实现，可以监听webpack的各种钩子函数， 比如beforeRun, run, compile, emit, done等。

    ```js
    class webpackPlugin {
    　　constructor(options){
    　　　　this.options = options;
    　　}
    　　apply(compiler) {
    　　// compiler 很重要，是webpack的一个实例，这个实例存储了webpack各种信息，所有打包信息
            compiler.hooks.run.tap('', () => {})
            compiler.hooks.compile.tap('', () => {})
            compiler.hooks.emit.tap('', () => {})
            compiler.hooks.done.tap('', () => {})
    　　}
    }
    module.exports = webpackPlugin;
    ```

## webpack的构建流程是什么?
Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

* 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
* 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
* 确定入口：根据配置中的 entry 找出所有的入口文件；
* 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
* 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
* 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
* 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

## webpack与grunt、gulp的不同？
三者都是前端构建工具，grunt和gulp在早期比较流行，现在webpack相对来说比较主流，不过一些轻量化的任务还是会用gulp来处理，比如单独打包CSS文件等。

grunt和gulp是基于任务和流（Task、Stream）的。类似jQuery，找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据， 整条链式操作构成了一个任务，多个任务就构成了整个web的构建流程。

webpack是基于入口的。webpack会自动地递归解析入口所需要加载的所有资源文件，然后用不同的Loader来处理不同的文件，用Plugin来扩展webpack功能。

## webpack、rollup、parcel优劣？
* webpack适用于大型复杂的前端站点构建
* rollup适用于基础库的打包，如vue、react
* parcel适用于简单的实验性项目，他可以满足低门槛的快速看到效果

## 分别介绍module, chunk, bundle是什么
* module: 简单来说，你通过import语句引入的代码块，都叫module。css，图片等也都叫module
* chunk: 是webpack根据功能拆分出来的，包含三种情况：
    * 你的项目入口（entry）chunk，也就是main chunk，一般这个chunk包含了你大部分的代码。
    * 通过import()动态引入的模块，会被打包为子chunk
    * 通过splitChunks拆分出来的代码, 也是子chunk
* bundle: 是webpack打包之后的各个文件，一般就是和chunk是一对一的关系。bundle就是对chunk进行编译压缩打包等处理之后的产出。

## 热更新
不用刷新浏览器而将新变更的模块替换掉旧的模块。

## 如何提高webpack构建速度
1. 使用cache-loader缓存结果
2. 使用DLLPlugin 和 DLLReferencePlugin

    * DLLPlugin 这个插件是在一个额外独立的webpack设置中创建一个只有dll的bundle，也就是说我们在项目根目录下除了有webpack.config.js，还会新建一个webpack.dll.config.js文件。webpack.dll.config.js作用是把所有的第三方库依赖打包到一个bundle的dll文件里面，还会生成一个名为 manifest.json文件。
该manifest.json的作用是用来让 DllReferencePlugin 映射到相关的依赖上去的。

    * DllReferencePlugin 这个插件是在webpack.config.js中使用的，该插件的作用是把刚刚在webpack.dll.config.js中打包生成的dll文件引用到需要的预编译的依赖上来。什么意思呢？就是说在webpack.dll.config.js中打包后比如会生成 vendor.dll.js文件和vendor-manifest.json文件，vendor.dll.js文件包含所有的第三方库文件，vendor-manifest.json文件会包含所有库代码的一个索引，当在使用webpack.config.js文件打包DllReferencePlugin插件的时候，会使用该DllReferencePlugin插件读取vendor-manifest.json文件，看看是否有该第三方库。vendor-manifest.json文件就是有一个第三方库的一个映射而已。

    所以说 第一次使用 webpack.dll.config.js 文件会对第三方库打包，打包完成后就不会再打包它了，然后每次运行 webpack.config.js文件的时候，都会打包项目中本身的文件代码，当需要使用第三方依赖的时候，会使用 DllReferencePlugin插件去读取第三方依赖库。所以说它的打包速度会得到一个很大的提升。

## 如何用webpack来优化前端性能？
1. 压缩代码。js, css, image
2. 利用CDN 加速， 所以文件需要有hash值
3. gzip
4. tree shaking
5. 提取公共代码
6. 代码分割， 避免太大

## tree shaking 原理
Tree shaking是基于ES6模板语法（import与export），主要是借助ES6模块的静态编译思想，在编译时就能确定模块的依赖关系，以及输入和输出的变量

Tree shaking无非就是做了两件事：

编译阶段利用ES6 Module判断哪些模块已经加载  

判断那些模块和变量未被使用或者引用，进而删除对应代码