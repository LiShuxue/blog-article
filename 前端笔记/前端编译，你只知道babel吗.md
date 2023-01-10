大部分前端可能都或多或少的知道一些AST相关的东西，用过Babel，ESLint，Prettier等，听过
@babel/parser，@babel/eslint-parser，acorn，@typescript-eslint/parser等，但并不知道他们的区别。

本文为大家介绍了AST规范是怎么来的，最初的js编译器是谁，现在都有哪些编译器等。

## ESTree
最初 Mozilla 一个工程师将 SpiderMonkey 引擎的 JavaScript 解析器公开，包含输出AST的规范文档，后来逐渐形成了现在大家通用的AST格式规范 [ESTree](https://github.com/estree/estree)，一般js解析器输出的AST都会遵循此规范中的格式，或者是基于此规范再细化。

## Esprima 
[Esprima](https://github.com/jquery/esprima) 是最早的一个js解析器，输出的AST基于ESTree规范，只支持js，不支持ts，flow等。

## Acorn
[Acorn](https://github.com/acornjs/acorn) 也是一个js解析器，在 Esprima 之后诞生，按照 Acorn 作者的说法，当时造这个轮子更多只是好玩，速度可以和 Esprima 媲美，但是实现代码更少，且输出的AST也是基于ESTree规范。

现在是社区主流，它速度更快，支持新语法，而且支持插件扩展，全面超过了Esprima，但是文档不太完善。

## Espree
[ESLint](https://eslint.org/) 最开始是基于Esprima做的，es6之前都很好，es6之后由于js更新较快，但是esprima更新跟不上，所以 ESLint folk 了 Esprima，取名 [Espree](https://github.com/eslint/espree)。

后来 Acorn 火了之后，Espree 也开始基于 Acorn。这里有一些[解释](https://github.com/eslint/espree#why-another-parser)。

## @babel/parser
[@babel/parser](https://babeljs.io/docs/en/babel-parser) 是babel中使用的js解析器，前身是 Babylon，基于 acorn 和 acorn-jsx 。支持解析最新的ECMAScript，包括一些实验性提案，以及TypeScript，JSX等，也支持解析注释。

Babel 解析器根据 [Babel AST](https://github.com/babel/babel/blob/main/packages/babel-parser/ast/spec.md) 格式生成 AST，它基于 [ESTree 规范](https://github.com/estree/estree)，但相比原规范更[细化一些](https://babeljs.io/docs/en/babel-parser#output)。JSX 代码的AST基于 [Fackbook JSX AST](https://github.com/facebook/jsx/blob/main/AST.md)。

## @babel/eslint-parser
[ESLint](https://eslint.org/) 默认使用Espree作为解析器，也支持使用[自定义的parser](https://eslint.org/docs/latest/user-guide/configuring/plugins#configure-a-parser)来解析代码,但是需要遵循[ESLint指定的parser规范](https://eslint.org/docs/latest/developer-guide/working-with-custom-parsers)，比如parser需要接收什么样的参数，输出什么样的结果等。

[@babel/eslint-parser](https://github.com/babel/babel/tree/main/eslint/babel-eslint-parser) 就是除 Espree 之外专门用于ESLint的一个自定义解析器，内部使用@babel/parser来解析代码（使用你项目中的babel配置），返回ESLint需要的结果，输出的AST也是基于ESTree规范的。相比于Espree只支持最终的标准，babel解析器支持实验性的语法和typescript的语法。

## @typescript-eslint/parser
[@typescript-eslint/parser](https://typescript-eslint.io/architecture/parser) 也是用于eslint的一个自定义解析器，主要用来解析ts代码。

内部调用[@typescript-eslint/typescript-estree](https://typescript-eslint.io/architecture/typescript-estree)，该库会调用typescript的原生方法，生成Typescript AST节点，并且将生成的AST转换为ESTree规范兼容的AST格式，从而可以使ESLint、Prettier等使用。

[@typescript-eslint/eslint-plugin](https://typescript-eslint.io/architecture/eslint-plugin) 提供了typescript语法中特有的一些rule

## vue-eslint-parser
[vue-eslint-parser](https://github.com/vuejs/vue-eslint-parser) 也是用于eslint的一个自定义解析器，用来解析.vue文件中`<template>`中的语法。同样也会生成一些vue特有的AST节点，基于[ast.md](https://github.com/vuejs/vue-eslint-parser/blob/master/docs/ast.md)。比如VElement，VDirectiveKey等

对于`<script>`中的代码，默认调用ESLint处理，即使用ESLint默认的解析器Espree。如果想使用自定义的解析器，如 @babel/eslint-parser 或 @typescript-eslint/parser，也可以在parserOptions.parser指定。

[eslint-plugin-vue](https://eslint.vuejs.org/user-guide/#installation) 中定义了一些vue中特有的rule

## vue-template-compiler
[vue-template-compiler](https://www.npmjs.com/package/vue-template-compiler) 可将 Vue 2.0 `<template>`预编译为渲染函数（template => ast => render）。vue-template-compiler 的代码是从 vue 源码中抽离的。

## prettier中的parser 
prettier的原理

通过 [prettier](https://prettier.io/docs/en/index.html) 的 [playground](https://prettier.io/playground/)，我们可以看到prettier是先将代码转换成AST，再将AST转成自己风格的代码。

所以第一步就是将源码解析成AST，prettier中对于不同的文件已经预定义了一些parser，详细参考[官方文档](https://prettier.io/docs/en/options.html#parser)

## terser
[terser](https://www.npmjs.com/package/terser) 是用来做ES6+代码混淆压缩的，最初是由 uglify-es fork过来的。最开始的uglify-js不支持ES6的语法，所以有了uglify-es，后来又重写了下，就是现在的terser。

压缩的原理是：JS 源代码 -> AST -> 混淆、压缩 -> 新的 AST -> 压缩后的代码。terser是有自己的一套ast标准，并不遵循ESTree规范。

terser-webpack-plugin 内部封装了 terser 库。

## postcss && stylelint && autoprefixer
[postcss](https://postcss.org/docs/postcss-architecture) 是用来解析css代码的，是一款使用插件去转换CSS的工具。不同于css预处理器sass或less。它将 CSS 代码转换为抽象语法树 (AST)，然后提供 API（应用程序编程接口）用于使用 JavaScript 插件对其进行分析和修改。

Autoprefixer, Stylelint, CSSnano都是基于 PostCSS 之上的。一般配合webpack等打包工具使用。在项目中会有postcss.config.js文件，里面配置一些插件及其参数。

预处理器是基于 Css，在其上做了一层属于自己的DSL（Domain specific language），用来解决 Css 遇到的问题。

## 参考
https://zhuanlan.zhihu.com/p/295291463

https://zhuanlan.zhihu.com/p/358518402

https://blog.csdn.net/mouday/article/details/126336560

https://juejin.cn/post/7084882650233569317
