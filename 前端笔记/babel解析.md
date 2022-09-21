## babel
babel可以将ES6及以上代码，或者typescript代码，jsx代码等，转换为JS代码。也可以给我们的代码针对特定环境添加polyfill。

如果不对babel做任何配置，一段ES6的代码经过babel处理，还是原来那段代码。babel的功能都是通过plugin来完成的。

@babel/preset-env 默认只转语法，不转API。比如Iterator、Generator、Set、Map、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Array.from()）都不会转码。如果需要转换的话，需要引入polyfill。

如果需要 polyfill，可以直接导入 "core-js" 或使用 @babel/preset-env 的 useBuiltIns 选项。

## 配置文件
babel.config.js，babel.config.json
```js
// js
module.exports = {
  sourceType: 'unambiguous',
  targets: [],
  presets: [...],
  plugins: [...]
};

// json
{
  "sourceType": "unambiguous",
  "targets": [],
  "presets": [...],
  "plugins": [...]
}
```

## 插件 plugins
将ES6+语法转换为ES5的语法。如：
* plugin-transform-arrow-functions：转换箭头函数
* plugin-transform-async-to-generator：将 async/await 转换为 generator 函数

## 预设 presets
由于插件太多，Babel提供了预设功能，用于简化配置，预设是一系列插件的集合。常见的预设：

* @babel/preset-env：基于你的目标浏览器及运行环境，自动的确定babel插件及polyfill，编译最新的ES6+语法。
  * useBuiltIns: entry: 根据target中浏览器版本的支持，将polyfills拆分引入，仅引入有浏览器不支持的polyfill
  * useBuiltIns: usage(新)：检测代码中ES6/7/8等的使用情况，仅仅加载代码中用到的polyfills
  * corejs: `{ version: "3.25", proposals: true }`，指定core-js版本和是否支持提案语法。
  * module: false, 是否把ES6的模块化语法改成其它模块化语法，默认import都被转码成require，设置成false不转换
  > 使用useBuiltIns选项需要主动安装core-js包：`yarn add core-js@3`
* @babel/preset-typescript：移除TypeScript语法，转换为标准的ES代码。没有ts类型校验。
* @babel/preset-react：转换react的jsx语法等。

### 注意：
> * @babel/preset-env 简写 @babel/env 或 env, 可省略 preset-  
> * @babel/plugin-transform-arrow-functions 简写 @babel/transform-arrow-functions，可省略 plugin-  
> * 插件比预设先执行  
> * 插件执行顺序：从前向后执行  
> * 预设执行顺序：从后向前执行  

## core-js
core-js 是关于 ES 标准最出名的 polyfill，包含所有ES6+的新特性，新版babel已经默认集成了core-js-compat。

* core-js：定义全局的polyfill
* core-js-pure：提供了不污染全局环境的polyfill
* core-js-compat：维护了按照'browserslist'规范的垫片需求数据，来帮助我们找到'符合目标环境'的 'polyfills' 需求集合
* core-js-builder：可以结合 'core-js-compact' 以及 'core-js'，并利用 'webpack '能力，根据需求打包出 core-js 代码

## polyfill
polyfill 意指当浏览器不支持某一最新 API 时，它将帮你实现，中文叫做垫片。

在babel7之前，babel专门提供了一个库叫@babel/polyfill来做这件事情，在babel7之后，这个库被废弃了，因为polyfill有了新的使用方式。

@babel/preset-env 通过 useBuiltIns 选项导入core-js。通过 corejs 选项来指定corejs的版本。

这种方式转换之后，会在每个文件上方导入corejs的polyfill。对于语法层面的转换，如class, async/await，该文件上方也会声明一大堆辅助函数，比如创建class的，创建generator的，来模拟ES6的操作。这些辅助函数在每个文件中都会出现，这就造成了代码冗余。

## @babel/plugin-transform-runtime 和 @babel/runtime，@babel/runtime-corejs3
@babel/runtime中包含了所有语法层面转义需要的辅助函数。

@babel/plugin-transform-runtime, 这个插件会在babel编译过程中，将文件中声明的辅助函数，替换为@babel/runtime中的已经写好的辅助函数。

除了解决代码冗余，@babel/plugin-transform-runtime还有另外一个重要的能力——解决全局污染。@babel/preset-env useBuiltIns 转译api层时，将core-js中的垫片直接导入进来，他们会挂载到全局，造成全局污染。我们可以将core-js交给transform-runtime来处理。

在该插件下配置：（@babel/preset-env中的useBuiltIns和corejs选项就可以删除）
```
"plugins": [
  [
    "@babel/plugin-transform-runtime",
    {
      "corejs": { version: 3, proposals: true }
    }
  ]
]
```
> @babel/plugin-transform-runtime 的corejs选项需要主动安装 @babel/runtime-corejs3包：`yarn add @babel/runtime-corejs3`  
> @babel/runtime-corejs3 就是 @babel/runtime 和 core-js3 的结合

## targets
指定我们编译完后的js需要支持的浏览器版本。没有指定targets会寻找项目的.browserslistrc中的配置代替。

如果都没有，默认转换为所有的ES5兼容的代码。

## 安装
`yarn add @babel/core @babel/cli @babel/preset-env`

## 使用

### 使用babel api编译代码
```js
const babel = require('@babel/core');
const esFilePath = '/xxx/xxxx/xx/script.js';
const esCode = fs.readFileSync(esFilePath, 'utf8');
const options = {
    ast: true
}
const { ast, code } = babel.transformSync(esCode, options);
```

### 使用babel cli编译代码
`./node_modules/.bin/babel src --out-dir build`

## 编译ES6+
### 安装依赖
`yarn add @babel/core @babel/cli @babel/preset-env @babel/plugin-transform-runtime @babel/runtime-corejs3`
### 配置文件
```js
// babel.config.js
module.exports = {
  sourceType: 'unambiguous',
  plugins: [
    [
      '@babel/plugin-transform-runtime',
      {
        corejs: { version: 3, proposals: true }
      }
    ] 
  ],
  presets: ['@babel/preset-env'],
};
```
### 源代码
[仓库](https://github.com/LiShuxue/babel-transform/blob/main/es6.js)

## 编译TypeScript
### 安装依赖
`yarn add @babel/core @babel/cli @babel/preset-typescript @babel/preset-env @babel/plugin-transform-runtime @babel/runtime-corejs3`
### 配置文件
```js
// babel.config.js
module.exports = {
  sourceType: 'unambiguous',
  plugins: [
    [
      '@babel/plugin-transform-runtime',
      {
        corejs: { version: 3, proposals: true }
      }
    ] 
  ],
  presets: ['@babel/preset-env', '@babel/preset-typescript'],
};
```
### 源代码
[仓库](https://github.com/LiShuxue/babel-transform/blob/main/es6.js)

## babel包解析
babel 的主要编译流程是 parse、traverse, transform、generate。
1. 解析 (Parse)：这个过程由编译器实现，会经过词法分析过程和语法分析过程，从而生成 AST。
2. 转换 (Transform)：在遍历的过程中可对节点信息进行修改，生成新的 AST。
3. 生成 (Generate)：对初始 AST 进行转换后，根据不同的场景，既可以直接输出新的 AST，也可以转译成新的代码块。

* @babel/core babel的核心，主要起串联的作用，功能包括加载配置、调用@babel/parser解析AST、调用@babel/traverse遍历并操作AST进行转换transform、调用@babel/generator生成代码。
* @babel/parser babel解析器的实现。通过词法分析，生成tokens，语法分析根据tokens生成AST抽象语法树。
* @babel/traverse 自动遍历抽象语法树的工具，它会访问树中的所有节点，开发者借用该函数可以对 AST 进行访问和修改。
* @babel/types 它包含了构造、验证以及变换 AST 节点的方法。
* @babel/generate 读取AST并将其转换为代码和源码映射，对应 generate 阶段
* @babel/template 能直接将字符串代码片段（可在字符串代码中嵌入变量）转换为 AST 节点
* @babel/runtime 包含了所有运行时需要的辅助函数。

## 源码转换
https://astexplorer.net/

https://esprima.org/demo/parse.html
```js
// 源码
function square(n) {
  return n * n;
}

// Tokens
[
    {
        "type": "Keyword",
        "value": "function"
    },
    {
        "type": "Identifier",
        "value": "square"
    },
    {
        "type": "Punctuator",
        "value": "("
    },
    {
        "type": "Identifier",
        "value": "n"
    },
    {
        "type": "Punctuator",
        "value": ")"
    },
    {
        "type": "Punctuator",
        "value": "{"
    },
    {
        "type": "Keyword",
        "value": "return"
    },
    {
        "type": "Identifier",
        "value": "n"
    },
    {
        "type": "Punctuator",
        "value": "*"
    },
    {
        "type": "Identifier",
        "value": "n"
    },
    {
        "type": "Punctuator",
        "value": ";"
    },
    {
        "type": "Punctuator",
        "value": "}"
    }
]

// AST 抽象语法树
{
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  params: [{
    type: "Identifier",
    name: "n"
  }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "*",
        left: {
          type: "Identifier",
          name: "n"
        },
        right: {
          type: "Identifier",
          name: "n"
        }
      }
    }]
  }
}
```

## 遍历或修改AST
### 树状图
上面的代码转换成AST后，即为以下树状结构：

- FunctionDeclaration
  - Identifier (id)
  - Identifier (params[0])
  - BlockStatement (body)
    - ReturnStatement (body)
      - BinaryExpression (argument)
        - Identifier (left)
        - Identifier (right)

![树](https://raw.githubusercontent.com/LiShuxue/blog-article/master/前端笔记/babel-tree.png)

### 深度优先遍历：
当我们遍历这颗树的每一个分支时我们最终会走到尽头，于是我们需要往上遍历回去从而获取到下一个节点。 向下遍历这棵树我们进入每个节点，向上遍历回去时我们退出每个节点。

* 进入 FunctionDeclaration
  * 进入 Identifier (id)
  * 走到尽头
  * 退出 Identifier (id)
  * 进入 Identifier (params[0])
  * 走到尽头
  * 退出 Identifier (params[0])
  * 进入 BlockStatement (body)
  * 进入 ReturnStatement (body)
    * 进入 BinaryExpression (argument)
    * 进入 Identifier (left)
      * 走到尽头
    * 退出 Identifier (left)
    * 进入 Identifier (right)
      * 走到尽头
    * 退出 Identifier (right)
    * 退出 BinaryExpression (argument)
  * 退出 ReturnStatement (body)
  * 退出 BlockStatement (body)
* 退出 FunctionDeclaration

所以当创建访问者时你实际上有两次机会来访问一个节点。enter 和 exist

babel-traverse是一个对ast进行遍历的工具。基于访问者模式。babel 会采用深度优先搜索来访问每个 AST 的节点（node），你可以在visitor里上指定一个回调方法（可以通过节点名称指定回调方法），当遍历到当前节点时会调用该回调方法。

### 访问者visitor
简单的说它就是一个对象，定义了用于在一个树状结构中获取具体节点的方法。如下：
```js
import * as t from "@babel/types";
const myVisitor = {
  // 对所有的节点操作
  enter(path) {
    // 进入节点
    if (t.isIdentifier(path.node, { name: "n" })) {
      path.node.name = "x";
    }
  },
  exit(path) {
    // 退出节点
    console.log(`exit ${path.type}(${path.key})`)
  },
  // 对导入语句进行操作
  ImportDeclaration(path) {
    console.log(path.node.source.value);
  },
  // 对所有的变量进行操作
  Identifier(path) {
    console.log(path.node.name);
  },
  // 对所有的function操作
  FunctionDeclaration(path) {
    console.log(path);
  }
};

// 你也可以先创建一个访问者对象，并在稍后给它添加方法。
let visitor = {};
visitor.MemberExpression = function() {};
visitor.FunctionDeclaration = function() {}
```

babel-traverse一般用于在babel plugin中对ast进行遍历和修改。当然我们也可以单独使用来做额外的处理。
```js
import * as parser from '@babel/parser';
import traverse from '@babel/traverse';
import * as t from "@babel/types";

const code = `
function square(n) {
  return n * n;
}
`;

const ast = parser.parse(code);

const myVisitor = {
  // 同上 ...
}
traverse(ast, myVisitor);
```

## 自定义babel插件
将源码 `foo === bar` 替换左右变量名

1. 默认导出一个接收babel对象的方法
2. 方法中return一个对象，包含访问者对象
3. 访问者中针对特定节点执行回调方法
```js
export default function (babel) {
  const t = babel.types;
  return {
    visitor: {
      BinaryExpression(path) {
        path.node.left = t.identifier("sebmck");
        path.node.right = t.identifier("dork");
      }
    }
  }
} 
```
