# 一、Babel是什么

## 1）概述

1. babel可以将ES6及以上代码，或者typescript代码，jsx代码等，转换为JS代码。也可以给我们的代码针对特定环境添加polyfill。
1. 如果不对babel做任何配置，一段ES6的代码经过babel处理，还是原来那段代码。babel的功能都是通过plugin来完成的。

# 二、Babel名词解释

## 1）配置文件

我们常见的配置文件，一般在项目根目录下：babel.config.js，babel.config.json，.babelrc，.babelrc.js
```js
// babal.config.js
module.exports = {
  sourceType: 'unambiguous',
  targets: [],
  presets: [...],
  plugins: [...]
};
```

### 1.1）sourceType

告诉babel怎么去编译当前文件，是否按照 ESM 语法来处理文件。
* script：使用脚本语法解析文件。 不允许 import / export 语句。
* module：使用模块语法解析文件。允许 import / export 语句。
* unambiguous：如果存在 import / export 语句，则将文件视为“模块”，否则将其视为“脚本”。

### 1.2）targets

指定我们编译完后的js需要支持的浏览器版本。没有指定targets会寻找项目的.browserslistrc中的配置代替。

如果都没有，默认转换为所有的ES5兼容的代码。

### 1.3）插件 plugins

默认的 babel 不做任何事情。解析代码，然后再次生成相同的代码。所以做任何事你都需要添加 babel 插件来实现。

官方提供了很多[插件](https://babeljs.io/docs/en/plugins-list)，大多是将 ES6+ 的语法转换为 js 语法。如：

* @babel/plugin-transform-arrow-functions：转换箭头函数

### 1.4）预设 presets
由于插件太多，babel提供了预设功能，用于简化配置，预设是一系列插件及其配置的集合。常见的预设：
* @babel/preset-env：基于你的目标浏览器及运行环境，自动的确定babel插件及polyfill，编译最新的ES6+语法。
    1. 默认只转语法，不转API。如果需要转换API的话，需要引入polyfill。
        * 语法：let，const，解构，箭头函数，async/await，class等
        * API：Promise，Set，Map等
    2. 如果需要 polyfill，可以直接全局import core-js 导入所有或使用 @babel/preset-env 的 useBuiltIns 选项自动加载core-js。
        * useBuiltIns: entry，根据target中浏览器版本的支持情况，仅引入有浏览器不支持的polyfill  
        * useBuiltIns: usage，根据当前代码中ES6/7/8等的使用情况，仅加载代码中用到的polyfill
        * corejs: { version: "3.25", proposals: true }，指定core-js版本和是否支持提案语法。
        * module: false, 是否把ES6的模块化语法改成其它模块化语法，默认import都被转码成require，设置成false不转换
        * ★注意：使用useBuiltIns选项需要主动安装core-js包：yarn add core-js@3
* @babel/preset-typescript：移除TypeScript语法，转换为标准的ES代码。没有ts类型校验。
* @babel/preset-react：转换react的jsx语法等。

### 1.5）★注意
* @babel/preset-env 可以简写为 @babel/env 或 env, 可省略 preset- 和 @babel
* @babel/plugin-transform-arrow-functions 可以简写为 @babel/transform-arrow-functions，可省略 plugin- 和 @babel
* 插件比预设先执行
* 插件执行顺序：从前向后执行
* 预设执行顺序：从后向前执行

## 2）core-js
[core-js](https://github.com/zloirock/core-js) 是关于 ES 标准最出名的 polyfill，包含所有ES6+的新特性。新版babel已经默认集成了core-js-compat。
* core-js：定义全局的polyfill
* core-js-pure：提供了不污染全局环境的polyfill
* core-js-compat：维护了按照 browserslist 规范的垫片 polyfill 数据，来帮助我们找到符合目标环境的 polyfills 集合
* core-js-builder：可以结合 core-js-compact 以及 core-js，并利用 webpack 能力，根据需求打包出 core-js 代码

## 3）polyfill垫片
polyfill 指当浏览器不支持某一最新 API 时，它将帮你实现，中文叫做垫片。

在babel7之前，babel专门提供了一个库叫@babel/polyfill来做这件事情，在babel7之后，这个库被废弃了，因为polyfill有了新的使用方式。比如：通过配置@babel/preset-env 的 useBuiltIns: usage 选项导入对应的polyfill。

转换之后，对于浏览器不支持的API及方法，会在文件上方导入corejs的polyfill。对于新的语法如class, async/await等，也会在该文件上方声明一大堆辅助函数，比如创建class的，创建generator的，来模拟ES6的操作。这些辅助函数在每个文件中都会出现，这就造成了代码冗余。

## 4）@babel/plugin-transform-runtime 和 @babel/runtime，@babel/runtime-corejs3
* @babel/runtime中包含了所有语法层面转义需要的辅助函数。（包需要主动引入）
* @babel/plugin-transform-runtime, 这个插件会在babel编译过程中，将文件中声明的辅助函数，替换为@babel/runtime中的已经写好的辅助函数。

这样就解决了代码冗余，所有代码提前定义好并采用导入的方式。

但是我们会发现一个问题，@babel/preset-env useBuiltIns 转译时，将core-js中的垫片直接导入进来，他们会挂载到全局，造成全局污染。

所以除了解决代码冗余，@babel/plugin-transform-runtime还有另外一个重要的能力，解决全局污染。我们可以将core-js交给transform-runtime来自动处理。

在该插件下配置corejs选项：
```js
"plugins": [
  [
    "@babel/plugin-transform-runtime",
    {
      "corejs": { version: 3, proposals: true }
    }
  ]
]
```

### ★注意：
* @babel/preset-env中的useBuiltIns和corejs选项就可以删除了
* 这个corejs选项需要主动安装 @babel/runtime-corejs3包：yarn add @babel/runtime-corejs3
* @babel/runtime-corejs3 就是 @babel/runtime 和 core-js3 的结合

# 三、Babel基础使用

## 1）babel基础库

<b>初始化项目：</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

<b>安装</b>

`yarn add @babel/core @babel/cli`

<b>使用命令行工具@babel/cli编译</b>

`./node_modules/.bin/babel ./es6.js --config-file ./babel.config.js --out-file ./dist/es6-babel-compiled.js`

## 2）preset及其配置

<b>初始化项目：</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

<b>安装</b>

`yarn add @babel/core @babel/cli @babel/preset-env`

<b>使用命令行工具@babel/cli编译</b>

`./node_modules/.bin/babel ./es6.js --config-file ./babel.config.js --out-file ./dist/es6-babel-compiled.js`

<b>配置preset-env</b>
```js
presets: [
  [
    '@babel/preset-env', 
    {
      modules: false, // 是否把ES6的模块化语法改成其它模块化语法，默认import都被转码成require，设置成false不转换
      useBuiltIns: 'usage', // 检测代码中ES6/7/8等的使用情况，仅仅加载代码中用到的polyfills
      corejs: '3.25',
    }
  ]
]
```

<b>再次编译，观察结果</b>

增加了core-js中的polyfill的导入，但是文件上的一大堆辅助函数，造成了代码冗余。

## 3）@babel/plugin-transform-runtime和@babel/runtime

<b>初始化项目：</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

<b>安装</b>

`yarn add @babel/core @babel/cli @babel/preset-env @babel/plugin-transform-runtime @babel/runtime`

配置plugins
```js
plugins: ['@babel/plugin-transform-runtime']
```

<b>使用命令行工具@babel/cli编译</b>

`./node_modules/.bin/babel ./es6.js --config-file ./babel.config.js --out-file ./dist/es6-babel-compiled.js`

<b>是否发现了什么问题？</b>

将core-js中的polyfill直接导入进来，他们会挂载到全局，造成全局污染。

## 4）优化corejs

<b>初始化项目：</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

<b>安装</b>

`yarn add @babel/core @babel/cli @babel/preset-env @babel/plugin-transform-runtime @babel/runtime-corejs3`

<b>配置plugins，preset配置可删除</b>
```js
plugins: [
  [
    '@babel/plugin-transform-runtime', // 导入@babel/runtime中的辅助函数，避免corejs全局污染
    {
      corejs: { version: 3, proposals: true }
    }
  ]
],
presets: ['@babel/preset-env']
```
<b>使用命令行工具@babel/cli编译</b>

`./node_modules/.bin/babel ./es6.js --config-file ./babel.config.js --out-file ./dist/es6-babel-compiled.js`

<b>观察结果</b>

此时所有的polyfill和辅助函数全是通过 @babel/runtime-corejs3 引入，解决了全局污染的问题，同时，由babel控制了所有的polyfill加载。

## 5）babel api编译

<b>初始化项目：</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

<b>使用api编译代码</b>

index.js
```js
const babel = require("@babel/core");

const sourceCode = `
const fn = () => {
  console.log('fn')
}
`;
const options = {
  configFile: "./babel.config.js"
};

const { code } = babel.transformSync(sourceCode, options);

console.log("compiled code: ");
console.log(code);
```

## 6）webpack加载babel

<b>初始化项目：</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

<b>用babel-loader处理js文件，他会默认加载根目录的babel.config.js</b>

webpack.config.js
```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: "./es6.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist")
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```

# 四、Babel进阶

## 1）babel编译流程

Babel 的编译流程主要分为三个部分：解析（parse），转换（transform），生成（generate）。

1. 解析 (parse)：这个过程由编译器实现，会经过词法分析过程和语法分析过程，从而生成 AST。
1. 转换 (transform)：在遍历的过程中可对节点信息进行修改，生成新的 AST。
1. 生成 (generate)：将AST转译成新的代码块。

![babel-parse](https://cdn.lishuxue.site/blog/image/前端笔记/babel-parse.png)

## 2）babel包解析

babel 仓库是monorepo形式，采用yarn workspaces管理，所有的核心包和官方插件，全在[packages](https://github.com/babel/babel/tree/main/packages)目录下。
* @babel/core： babel的核心，主要起串联的作用，功能包括加载配置、调用@babel/parser将代码编译成AST、调用@babel/traverse遍历并操作AST进行转换transform、调用@babel/generator生成代码。
* @babel/cli： babel 内置的命令行工具，可用于从命令行编译文件
* @babel/parser： babel解析器的实现。通过词法分析，生成tokens，根据tokens进行语法分析生成AST抽象语法树。对应parse阶段
* @babel/traverse： 自动遍历抽象语法树的工具，它会访问树中的所有节点，开发者借用该函数可以对 AST 进行访问和修改。对应transform阶段
* @babel/types： 它包含了构造、验证以及变换 AST 节点的方法
* @babel/generate： 读取AST并将其转换为代码和source map，对应 generate 阶段
* @babel/template： 能直接将字符串代码片段（可在字符串代码中嵌入变量）转换为 AST 节点
* @babel/runtime： 包含了所有运行时需要的辅助函数
* @babel/runtime-corejs3： 就是 @babel/runtime 和 core-js3 的结合

## 3）编译原理

### 3.1）概述

18年尤大曾说过，懂编译原理真的可以为所欲为。作为前端开发，我们没必要对这些编译器或者底层的编译原理了如指掌，但是如果能对编译原理有一些基本的认识，也能够对今后的日常开发很有帮助。

![为所欲为](https://cdn.lishuxue.site/blog/image/前端笔记/youda.png)

一般高级语言的编译可以分为五个阶段：

1. 词法分析阶段，该阶段会对构成源程序的字符串进行扫描和分解，识别出一个个的单词

1. 语法分析阶段，从词法分析器输出的token序列中识别出各类短语，并构造抽象语法树(ast)

1. 语义分析阶段，按照语法分析器识别的语法进行语义检查和处理，产生相应的中间代码

1. 中间代码生成和代码优化阶段。

1. 目标代码生成程序阶段。

Babel 是没有语义分析和中间代码生成/优化阶段的。但是我们熟知的JavaScript 的编译器 v8 引擎是有的。

<b>V8原理图</b>
1. 输入全局的js代码，解析器(Parser)通过词法分析，生成tokens，语法分析根据tokens生成AST抽象语法树。
1. 解释器(Ignition) 会将 AST 转换为字节码，一边解释一边执行。（解释执行）
1. 在解释执行字节码的过程中，如果发现一段代码被多次重复执行，就会将其标记为热点（Hot）代码。V8 会将这段热点代码丢给优化编译器 TurboFan 编译为机器码（二进制代码）。如果下次再执行时，就会直接执行机器码，提高执行速度。（编译执行）

![v8](https://cdn.lishuxue.site/blog/image/前端笔记/v8.png)

接下来我们主要介绍babel词法分析和语法分析。

### 3.2）词法分析
[在线分词工具](https://esprima.org/demo/parse.html#) 

词法分析也称为 分词 ，此阶段编译器从左向右扫描源文件，将源代码字符流分割成一个个的 词 （ token 、 记号  ）。所谓 token ，就是源文件中不可再进一步分割的一串字符，类似于英语中单词，或汉语中的词。进行词法分析的程序或者函数叫作词法分析器或分词器。

babel分词器的具体实现在[babel-parser/src/tokenizer/index.ts](https://github.com/babel/babel/blob/main/packages/babel-parser/src/tokenizer/index.ts)中822行。每个 case 就是一条分词规则。
1. 点，单点，三个点
1. 标点符号，{、}、;、,、[、]、(、)、:、?
1. 数字，0，1-9，0X，0O，0B
1. 变量标识符/关键字，$、a-zA-Z、_
1. 字符串，' 或 " 开头的字符，\n\t\等转义字符
1. 算术运算，+、++、+=、-、--、-=、*、/、%、>、<、=、==、 === 等运算符
1. 逻辑运算，&、&&、|、||、!、!! 等运算符
1. 注释，//、/*
1. ...

#### 3.2.1）源代码
```js
function square (n) {
  return n * n;
}
```
#### 3.2.2）tokens
```js
[
    {
        "type": "Keyword", // 关键字
        "value": "function"
    },
    {
        "type": "Identifier", // 标识符
        "value": "square"
    },
    {
        "type": "Punctuator", // 标点符号
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
```

### 3.3）语法分析
简单来说，语法分析就是读取词法分析产生的 token 序列，看是否满足该语言的语法，组合成各类语法短语，如“程序”，“语句”，“表达式”等等。然后通过自底向上或自顶向下分析法生成抽象语法树。

* 自底向上：自叶开始逐级向上归约，直到构造出表示句子结构的整个推导树为止的一种语言形式分析算法.
* 自顶向下：由根开始自顶向下地构造推导树，一直到得出语言中的合格句子为止的一种语言形式分析算法.

babel采用了自顶向下的分析法：[why-babel-uses-a-top-down-parser](https://stackoverflow.com/questions/46780413/why-babel-uses-a-top-down-parser)

babel的语法分析器具体实现在[babel-parser/src/parser/statement.ts](https://github.com/babel/babel/blob/main/packages/babel-parser/src/parser/statement.ts)中337行，每个 case 就是一条语句开头。大多数类型的语句都可以通过它们开头的关键字来识别。 

#### 3.3.1）抽象语法树AST
[在线AST转换](https://astexplorer.net/)

<b>抽象语法树</b>（abstract syntax code，AST）是源代码抽象语法结构的树状表示，树上的每个节点都表示源代码中的一种结构，之所以说语法是 "抽象" 的，是因为这里的语法不会表示出真实语法中出现的每个细节。比如说，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现。

<b>具象语法树</b>（Concret Syntax Tree，CST）是包含代码所有语法信息的树形结构，它是代码的直接翻译，所以解析树也被称为具象语法树。抽象语法树实际只是解析树的一个精简版。

AST节点数据结构：
```js
{
	type: 'Program' | 'FunctionDeclaration' | 'Identifier' // 节点的类型
  
 	// 不同节点类型的一些独有的描述字段
  kind: "const" // 变量，函数，表达式等的类型
  name: '' // 变量名
  ...
  
 	// 位置属性
  start: 0,
  end: 54
}
```

<b>常见的节点类型：</b>

* Programs（根节点，表示所有源程序开始）

* Declarations（声明）：FunctionDeclaration，VariableDeclaration，ImportDeclaration，ClassDeclaration等

* Statement（语句）：BlockStatement，ForStatement，IfStatement，ReturnStatement，SwitchStatement等

* Expression（表达式）：BinaryExpression，LogicalExpression，MemberExpression，NewExpression，CallExpression，FunctionExpression等

* Identifier（标识符）：比如变量名，函数名

* Literal（字面量）：StringLiteral，BooleanLiteral，NullLiteral，RegExpLiteral。（如：123，false，null）

所有节点的类型可以在[ESTree](https://github.com/estree/estree)这个仓库中找到。

源码转换成的整个AST数据结构：
```js
{
  "type": "Program",                    // 根节点，程序开始
  "body": [
    {
      "type": "FunctionDeclaration",    // 节点类型：函数声明
      "id": {
        "type": "Identifier",           // 函数标识符
        "name": "square"
      },
      "params": [                       // 函数参数
        {
          "type": "Identifier",
          "name": "n"
        }
      ],
      "body": {                        // 函数体
        "type": "BlockStatement",      // 语句块
        "body": [
          {
            "type": "ReturnStatement", // return 语句
            "argument": {
              "type": "BinaryExpression", // 二叉树表达式（算术运算表达式）
              "operator": "*",
              "left": {
                "type": "Identifier",
                "name": "n"
              },
              "right": {
                "type": "Identifier",
                "name": "n"
              }
            }
          }
        ]
      }
    }
  ],
  "sourceType": "script"
}
```

#### 3.3.2）树状图

上面的AST即为以下树状结构：

* FunctionDeclaration

  * Identifier (id)

  * Identifier (params[0])

  * BlockStatement (body)

    * ReturnStatement (body)

      * BinaryExpression (argument)

        * Identifier (left)

        * Identifier (right)

树状图：

![ast](https://cdn.lishuxue.site/blog/image/前端笔记/ast.png)

## 4）遍历AST

### 4.1）AST遍历思路

AST就是树，树的遍历可以是深度优先遍历或广度优先遍历。

* 深度优先算法将会从第一个指定的顶点开始遍历图，每次遍历当前访问顶点的临界点，一直到访问的顶点没有未被访问过的临界点为止。然后采用依次回退的方式，查看来的路上每一个顶点是否有其它未被访问的临界点。访问完成后，判断图中的顶点是否已经全部遍历完成，如果没有，以未访问的顶点为起始点，重复上述过程。

  ![DFS](https://cdn.lishuxue.site/blog/image/前端笔记/DFS-Ex.gif)

* 广度优先算法会从指定的第一个顶点开始遍历图，遍历每一个顶点时，依次遍历其所有的邻接点，然后再从这些邻接点出发，同样依次访问它们的邻接点。按照此过程，直到图中所有被访问过的顶点的邻接点都被访问到。最后还需要做的操作就是查看图中是否存在尚未被访问的顶点，若有，则以该顶点为起始点，重复上述遍历的过程。

  ![BFS](https://cdn.lishuxue.site/blog/image/前端笔记/BFS-Ex.gif)

前序、中序、后序遍历是对二叉树进行深度优先遍历的几种方式。

babel中采用深度优先遍历AST。

### 4.2）深度优先遍历

当我们遍历这颗树的每一个分支时我们最终会走到尽头，于是我们需要往上遍历回去从而获取到下一个节点。 

向下遍历这棵树我们进入每个节点，向上遍历回去时我们退出每个节点。

步骤

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

但是对于每个 AST 节点怎么处理呢，我们怎么知道该访问他的哪个子节点呢？

比如 " a + b " 这个 BinaryExpression，需要遍历 left、right 属性：

![a+b](https://cdn.lishuxue.site/blog/image/前端笔记/a+b.png)

比如 " if (a === 1) " {} 这个 IfStatement，需要遍历 test、consequece 属性：

![if](https://cdn.lishuxue.site/blog/image/前端笔记/if.png)

所以我们记录下每种 AST 怎么遍历，然后从根结点开始递归的遍历就可以了。

@babel/traverse库用于对ast进行遍历和修改。一般用在babel plugin中。在[babel-traverse/src/traverse-node.ts](https://github.com/babel/babel/blob/main/packages/babel-traverse/src/traverse-node.ts)中第20行，定义了如何遍历一个节点。

所有节点的需要遍历的属性，是VISITOR_KEYS，定义在babel中定义在[babel-types/src/definitions/utils.ts](https://github.com/babel/babel/blob/main/packages/babel-types/src/definitions/utils.ts)中的第5行，由第363行动态生成

生成后的 VISITOR_KEYS 对象：
```js
{
  ArrayExpression: [ 'elements' ],
  AssignmentExpression: [ 'left', 'right' ],
  BinaryExpression: [ 'left', 'right' ],
  InterpreterDirective: [],
  Directive: [ 'value' ],
  DirectiveLiteral: [],
  BlockStatement: [ 'directives', 'body' ],
  BreakStatement: [ 'label' ],
  CallExpression: [ 'callee', 'arguments', 'typeParameters', 'typeArguments' ],
  CatchClause: [ 'param', 'body' ],
  ConditionalExpression: [ 'test', 'consequent', 'alternate' ],
  ContinueStatement: [ 'label' ],
  DebuggerStatement: [],
  DoWhileStatement: [ 'test', 'body' ],
  EmptyStatement: [],
  ExpressionStatement: [ 'expression' ],
  File: [ 'program' ],
  ForInStatement: [ 'left', 'right', 'body' ],
  ForStatement: [ 'init', 'test', 'update', 'body' ],
  FunctionDeclaration: [ 'id', 'params', 'body', 'returnType', 'typeParameters' ],
  FunctionExpression: [ 'id', 'params', 'body', 'returnType', 'typeParameters' ],
  Identifier: [ 'typeAnnotation', 'decorators' ],
  IfStatement: [ 'test', 'consequent', 'alternate' ],
  LabeledStatement: [ 'label', 'body' ],
  StringLiteral: [],
  NumericLiteral: [],
  NullLiteral: [],
  BooleanLiteral: [],
  RegExpLiteral: [],
  LogicalExpression: [ 'left', 'right' ],
  MemberExpression: [ 'object', 'property' ],
  NewExpression: [ 'callee', 'arguments', 'typeParameters', 'typeArguments' ],
  Program: [ 'directives', 'body' ],
  ObjectExpression: [ 'properties' ],
  ObjectMethod: [
    'key',
    'params',
    'body',
    'decorators',
    'returnType',
    'typeParameters'
  ],
  ObjectProperty: [ 'key', 'value', 'decorators' ],
  RestElement: [ 'argument', 'typeAnnotation' ],
  ReturnStatement: [ 'argument' ],
  SequenceExpression: [ 'expressions' ],
  ParenthesizedExpression: [ 'expression' ],
  SwitchCase: [ 'test', 'consequent' ],
  SwitchStatement: [ 'discriminant', 'cases' ],
  ThisExpression: [],
  ThrowStatement: [ 'argument' ],
  TryStatement: [ 'block', 'handler', 'finalizer' ],
  UnaryExpression: [ 'argument' ],
  UpdateExpression: [ 'argument' ],
  VariableDeclaration: [ 'declarations' ],
  VariableDeclarator: [ 'id', 'init' ],
  WhileStatement: [ 'test', 'body' ],
  WithStatement: [ 'object', 'body' ],
  AssignmentPattern: [ 'left', 'right', 'decorators' ],
  ArrayPattern: [ 'elements', 'typeAnnotation' ],
  ArrowFunctionExpression: [ 'params', 'body', 'returnType', 'typeParameters' ],
  ClassBody: [ 'body' ],
  ClassExpression: [
    'id',
    'body',
    'superClass',
    'mixins',
    'typeParameters',
    'superTypeParameters',
    'implements',
    'decorators'
  ],
  ClassDeclaration: [
    'id',
    'body',
    'superClass',
    'mixins',
    'typeParameters',
    'superTypeParameters',
    'implements',
    'decorators'
  ],
  ExportAllDeclaration: [ 'source' ],
  ExportDefaultDeclaration: [ 'declaration' ],
  ExportNamedDeclaration: [ 'declaration', 'specifiers', 'source' ],
  ExportSpecifier: [ 'local', 'exported' ],
  ForOfStatement: [ 'left', 'right', 'body' ],
  ImportDeclaration: [ 'specifiers', 'source' ],
  ImportDefaultSpecifier: [ 'local' ],
  ImportNamespaceSpecifier: [ 'local' ],
  ImportSpecifier: [ 'local', 'imported' ],
  MetaProperty: [ 'meta', 'property' ],
  ClassMethod: [
    'key',
    'params',
    'body',
    'decorators',
    'returnType',
    'typeParameters'
  ],
  ObjectPattern: [ 'properties', 'typeAnnotation', 'decorators' ],
  SpreadElement: [ 'argument' ],
  Super: [],
  TaggedTemplateExpression: [ 'tag', 'quasi', 'typeParameters' ],
  TemplateElement: [],
  TemplateLiteral: [ 'quasis', 'expressions' ],
  YieldExpression: [ 'argument' ],
  AwaitExpression: [ 'argument' ],
  Import: [],
  BigIntLiteral: [],
  ExportNamespaceSpecifier: [ 'exported' ],
  OptionalMemberExpression: [ 'object', 'property' ],
  OptionalCallExpression: [ 'callee', 'arguments', 'typeParameters', 'typeArguments' ],
  ClassProperty: [ 'key', 'value', 'typeAnnotation', 'decorators' ],
  ClassAccessorProperty: [ 'key', 'value', 'typeAnnotation', 'decorators' ],
  ClassPrivateProperty: [ 'key', 'value', 'decorators', 'typeAnnotation' ],
  ClassPrivateMethod: [
    'key',
    'params',
    'body',
    'decorators',
    'returnType',
    'typeParameters'
  ],
  PrivateName: [ 'id' ],
  StaticBlock: [ 'body' ],
  AnyTypeAnnotation: [],
  ArrayTypeAnnotation: [ 'elementType' ],
  BooleanTypeAnnotation: [],
  BooleanLiteralTypeAnnotation: [],
  NullLiteralTypeAnnotation: [],
  ClassImplements: [ 'id', 'typeParameters' ],
  DeclareClass: [ 'id', 'typeParameters', 'extends', 'mixins', 'implements', 'body' ],
  DeclareFunction: [ 'id' ],
  DeclareInterface: [ 'id', 'typeParameters', 'extends', 'mixins', 'implements', 'body' ],
  DeclareModule: [ 'id', 'body' ],
  DeclareModuleExports: [ 'typeAnnotation' ],
  DeclareTypeAlias: [ 'id', 'typeParameters', 'right' ],
  DeclareOpaqueType: [ 'id', 'typeParameters', 'supertype' ],
  DeclareVariable: [ 'id' ],
  DeclareExportDeclaration: [ 'declaration', 'specifiers', 'source' ],
  DeclareExportAllDeclaration: [ 'source' ],
  DeclaredPredicate: [ 'value' ],
  ExistsTypeAnnotation: [],
  FunctionTypeAnnotation: [ 'typeParameters', 'params', 'rest', 'returnType' ],
  FunctionTypeParam: [ 'name', 'typeAnnotation' ],
  GenericTypeAnnotation: [ 'id', 'typeParameters' ],
  InferredPredicate: [],
  InterfaceExtends: [ 'id', 'typeParameters' ],
  InterfaceDeclaration: [ 'id', 'typeParameters', 'extends', 'mixins', 'implements', 'body' ],
  InterfaceTypeAnnotation: [ 'extends', 'body' ],
  IntersectionTypeAnnotation: [ 'types' ],
  MixedTypeAnnotation: [],
  EmptyTypeAnnotation: [],
  NullableTypeAnnotation: [ 'typeAnnotation' ],
  NumberLiteralTypeAnnotation: [],
  NumberTypeAnnotation: [],
  ObjectTypeAnnotation: [ 'properties', 'indexers', 'callProperties', 'internalSlots' ],
  ObjectTypeInternalSlot: [ 'id', 'value', 'optional', 'static', 'method' ],
  ObjectTypeCallProperty: [ 'value' ],
  ObjectTypeIndexer: [ 'id', 'key', 'value', 'variance' ],
  ObjectTypeProperty: [ 'key', 'value', 'variance' ],
  ObjectTypeSpreadProperty: [ 'argument' ],
  OpaqueType: [ 'id', 'typeParameters', 'supertype', 'impltype' ],
  QualifiedTypeIdentifier: [ 'id', 'qualification' ],
  StringLiteralTypeAnnotation: [],
  StringTypeAnnotation: [],
  SymbolTypeAnnotation: [],
  ThisTypeAnnotation: [],
  TupleTypeAnnotation: [ 'types' ],
  TypeofTypeAnnotation: [ 'argument' ],
  TypeAlias: [ 'id', 'typeParameters', 'right' ],
  TypeAnnotation: [ 'typeAnnotation' ],
  TypeCastExpression: [ 'expression', 'typeAnnotation' ],
  TypeParameter: [ 'bound', 'default', 'variance' ],
  TypeParameterDeclaration: [ 'params' ],
  TypeParameterInstantiation: [ 'params' ],
  UnionTypeAnnotation: [ 'types' ],
  Variance: [],
  VoidTypeAnnotation: [],
  EnumDeclaration: [ 'id', 'body' ],
  EnumBooleanBody: [ 'members' ],
  EnumNumberBody: [ 'members' ],
  EnumStringBody: [ 'members' ],
  EnumSymbolBody: [ 'members' ],
  EnumBooleanMember: [ 'id' ],
  EnumNumberMember: [ 'id', 'init' ],
  EnumStringMember: [ 'id', 'init' ],
  EnumDefaultedMember: [ 'id' ],
  IndexedAccessType: [ 'objectType', 'indexType' ],
  OptionalIndexedAccessType: [ 'objectType', 'indexType' ],
  JSXAttribute: [ 'name', 'value' ],
  JSXClosingElement: [ 'name' ],
  JSXElement: [ 'openingElement', 'children', 'closingElement' ],
  JSXEmptyExpression: [],
  JSXExpressionContainer: [ 'expression' ],
  JSXSpreadChild: [ 'expression' ],
  JSXIdentifier: [],
  JSXMemberExpression: [ 'object', 'property' ],
  JSXNamespacedName: [ 'namespace', 'name' ],
  JSXOpeningElement: [ 'name', 'attributes' ],
  JSXSpreadAttribute: [ 'argument' ],
  JSXText: [],
  JSXFragment: [ 'openingFragment', 'children', 'closingFragment' ],
  JSXOpeningFragment: [],
  JSXClosingFragment: [],
  Noop: [],
  Placeholder: [],
  V8IntrinsicIdentifier: [],
  ArgumentPlaceholder: [],
  BindExpression: [ 'object', 'callee' ],
  ImportAttribute: [ 'key', 'value' ],
  Decorator: [ 'expression' ],
  DoExpression: [ 'body' ],
  ExportDefaultSpecifier: [ 'exported' ],
  RecordExpression: [ 'properties' ],
  TupleExpression: [ 'elements' ],
  DecimalLiteral: [],
  ModuleExpression: [ 'body' ],
  TopicReference: [],
  PipelineTopicExpression: [ 'expression' ],
  PipelineBareFunction: [ 'callee' ],
  PipelinePrimaryTopicReference: [],
  TSParameterProperty: [ 'parameter' ],
  TSDeclareFunction: [ 'id', 'typeParameters', 'params', 'returnType' ],
  TSDeclareMethod: [ 'decorators', 'key', 'typeParameters', 'params', 'returnType' ],
  TSQualifiedName: [ 'left', 'right' ],
  TSCallSignatureDeclaration: [ 'typeParameters', 'parameters', 'typeAnnotation' ],
  TSConstructSignatureDeclaration: [ 'typeParameters', 'parameters', 'typeAnnotation' ],
  TSPropertySignature: [ 'key', 'typeAnnotation', 'initializer' ],
  TSMethodSignature: [ 'key', 'typeParameters', 'parameters', 'typeAnnotation' ],
  TSIndexSignature: [ 'parameters', 'typeAnnotation' ],
  TSAnyKeyword: [],
  TSBooleanKeyword: [],
  TSBigIntKeyword: [],
  TSIntrinsicKeyword: [],
  TSNeverKeyword: [],
  TSNullKeyword: [],
  TSNumberKeyword: [],
  TSObjectKeyword: [],
  TSStringKeyword: [],
  TSSymbolKeyword: [],
  TSUndefinedKeyword: [],
  TSUnknownKeyword: [],
  TSVoidKeyword: [],
  TSThisType: [],
  TSFunctionType: [ 'typeParameters', 'parameters', 'typeAnnotation' ],
  TSConstructorType: [ 'typeParameters', 'parameters', 'typeAnnotation' ],
  TSTypeReference: [ 'typeName', 'typeParameters' ],
  TSTypePredicate: [ 'parameterName', 'typeAnnotation' ],
  TSTypeQuery: [ 'exprName', 'typeParameters' ],
  TSTypeLiteral: [ 'members' ],
  TSArrayType: [ 'elementType' ],
  TSTupleType: [ 'elementTypes' ],
  TSOptionalType: [ 'typeAnnotation' ],
  TSRestType: [ 'typeAnnotation' ],
  TSNamedTupleMember: [ 'label', 'elementType' ],
  TSUnionType: [ 'types' ],
  TSIntersectionType: [ 'types' ],
  TSConditionalType: [ 'checkType', 'extendsType', 'trueType', 'falseType' ],
  TSInferType: [ 'typeParameter' ],
  TSParenthesizedType: [ 'typeAnnotation' ],
  TSTypeOperator: [ 'typeAnnotation' ],
  TSIndexedAccessType: [ 'objectType', 'indexType' ],
  TSMappedType: [ 'typeParameter', 'typeAnnotation', 'nameType' ],
  TSLiteralType: [ 'literal' ],
  TSExpressionWithTypeArguments: [ 'expression', 'typeParameters' ],
  TSInterfaceDeclaration: [ 'id', 'typeParameters', 'extends', 'body' ],
  TSInterfaceBody: [ 'body' ],
  TSTypeAliasDeclaration: [ 'id', 'typeParameters', 'typeAnnotation' ],
  TSInstantiationExpression: [ 'expression', 'typeParameters' ],
  TSAsExpression: [ 'expression', 'typeAnnotation' ],
  TSTypeAssertion: [ 'typeAnnotation', 'expression' ],
  TSEnumDeclaration: [ 'id', 'members' ],
  TSEnumMember: [ 'id', 'initializer' ],
  TSModuleDeclaration: [ 'id', 'body' ],
  TSModuleBlock: [ 'body' ],
  TSImportType: [ 'argument', 'qualifier', 'typeParameters' ],
  TSImportEqualsDeclaration: [ 'id', 'moduleReference' ],
  TSExternalModuleReference: [ 'expression' ],
  TSNonNullExpression: [ 'expression' ],
  TSExportAssignment: [ 'expression' ],
  TSNamespaceExportDeclaration: [ 'id' ],
  TSTypeAnnotation: [ 'typeAnnotation' ],
  TSTypeParameterInstantiation: [ 'params' ],
  TSTypeParameterDeclaration: [ 'params' ],
  TSTypeParameter: [ 'constraint', 'default' ]
}
```

### 4.3）访问者模式

访问者模式Visitor 对于某个对象或者一组对象，不同的访问者，产生的结果不同，执行操作也不同

Babel 处理一个节点时，就是以访问者的形式进行相关操作，这种方式是通过一个 visitor 对象来完成的。在 visitor 对象中定义了对于各种类型节点的访问函数，这样就可以针对不同的节点做出不同的处理。

实际在遍历节点的过程中，我们有两次机会访问每个节点，定义为enter和exit。

```js
import * as t from "@babel/types";

const myVisitor = {
  // 进入节点
  enter(path) {
    if (t.isIdentifier(path.node, { name: "n" })) {
      path.node.name = "x";
    }
  },
  
  // 退出节点
  exit(path) {
    console.log(`exit ${path.type}(${path.key})`)
  },
  
  // 对导入语句进行操作
  ImportDeclaration(path) {
    console.log(path.node.source.value);
  },
  
  // 对所有的变量标识符进行操作
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

## 5）自定义Babel插件修改AST

我们编写的 Babel 插件其实也是通过定义一个 visitor 对象处理一系列的 AST 节点，来完成我们对代码的修改操作。

<b>需求：给所有的方法添加try catch</b>
```js
// 原方法
function add(a, b) {
  console.log('1')
  throw new Error('Error')
  return a + b;
}

// 添加之后
function add(a, b) {
  try {    
    console.log('1')
    throw new Error('Error')
    return a + b;
  } catch (error) {
      report(error) // 自定义处理（eg：比如上报）
  }
}
```
<b>思路：</b>
1. 先看AST语法树的区别 [在线AST转换](https://astexplorer.net/)

2. 判断当前函数体是不是已经有try catch模块

3. 构造 try catch 模块，catch节点内也是函数的执行

4. 将当前函数体内的代码块放在try语句代码块中

<b>babel插件</b>

[codesandbox.io](https://codesandbox.io/dashboard/drafts?workspace=9374287f-1efe-4fb4-8e50-4e117b63c113)

```js
// babel-plugin-add-try-catch
const addTryCatchPlugin = (babel) => {
  const { types } = babel;

	return {
    name: 'add-try-catch',
    visitor: {
      FunctionDeclaration(path) {
        const node = path.node;
        const { id, params, body: blockStatement } = node;

        // 已经有 try catch 什么也不做
        if (blockStatement.body && types.isTryStatement(blockStatement.body[0])) {
            return
        }

        /**
         *  构造 try catch 语句分三步：
         *  1. 构造catch块里面的函数调用
         *  2. 构造catch语句
         *  3. 构造try 语句
         * */ 
        
        // 构造函数调用表达式：types.callExpression(callee, arguments);
        const callee = types.identifier('report');
        const args = types.identifier('error');
        const callExpression = types.callExpression(callee, [args]);
        const expressionStatement = types.expressionStatement(callExpression);

        // 构造 catch 语句 types.catchClause(param, body)
        const param = types.identifier('error');
        const catchBlockStatement = types.blockStatement([expressionStatement]);
        const handler = types.catchClause(param, catchBlockStatement)

        // 构造 try 语句 types.tryStatement(block, handler, finalizer);
        const tryStatement = types.tryStatement(blockStatement, handler);

        // 重新构造该函数声明
        const body = types.blockStatement([tryStatement]);
        const tryCatchFunctionDeclare = types.FunctionDeclaration(id, params, body);

        path.replaceWith(tryCatchFunctionDeclare);
      }
    }
  }
}

module.exports =  addTryCatchPlugin;
```

## 6）手写简易babel

手写plugin有点太简单了，今天我们搞点高端的，手写babel。

需求：实现一个简易的babel来编译：let a = 1;

思路：

1. 先看Tokens 和 AST语法树在线AST转换

1. 编写一个生成Tokens的分词器tokenizer

1. 编写一个生成AST的编译器parser

1. 编写一个生成AST转换器transformer

1. 编写一个代码生成器codeGenerator

```js
// 简易编译器，编译 let a = 1;
const compiler = (input) => {
  const tokens = tokenizer(input);
  const ast = parser(tokens);
  const newAst = transformer(ast);
  const output = codeGenerator(newAst);
  console.log(output);
  return output;
};

const tokenizer = (input) => {
  let current = 0;
  let tokens = [];

  while (current < input.length) {
    let char = input[current];
    if (char === "=" || char === ";") {
      tokens.push({
        type: "Punctuator",
        value: char,
      });
      current++;
      continue;
    }

    let WHITESPACE = /\s/;
    if (WHITESPACE.test(char)) {
      current++;
      continue;
    }

    let LETTERS = /[a-z]/i;
    if (LETTERS.test(char)) {
      let value = "";

      while (LETTERS.test(char)) {
        value += char;
        char = input[++current];
      }

      if (value === "let") {
        tokens.push({ type: "Keyword", value });
      } else {
        tokens.push({ type: "Identifier", value });
      }

      continue;
    }

    let NUMBERS = /[0-9]/;
    if (NUMBERS.test(char)) {
      let value = "";

      while (NUMBERS.test(char)) {
        value += char;
        char = input[++current];
      }
      tokens.push({ type: "Number", value });

      continue;
    }
  }
  console.log(tokens);
  return tokens;
};

const parser = (tokens) => {
  let current = 0;
  let ast = {
    type: "Program",
    body: [],
  };

  const walk = () => {
    let token = tokens[current];

    if (token.type === "Number") {
      current++;
      return {
        type: "Literal",
        value: token.value,
      };
    }

    if (token.type === "Identifier") {
      current++;
      return {
        type: "Identifier",
        name: token.value,
      };
    }

    if (token.type === "Keyword" && token.value === "let") {
      let node = {
        type: "VariableDeclaration",
        kind: token.value,
        declarations: [
          {
            type: "VariableDeclarator",
            id: {},
            init: {},
          },
        ],
      };

      current++;
      const node1 = walk();
      if (node1 && node1.type === "Identifier") {
        node.declarations[0].id = node1;
      }
      current++;
      const node2 = walk();
      if (node2 && node2.type === "Literal") {
        node.declarations[0].init = node2;
      }

      current++;
      return node;
    }

    if (token.type === "Punctuator") {
      current++;
    }
  };

  while (current < tokens.length) {
    ast.body.push(walk());
  }

  console.log(JSON.stringify(ast));
  return ast;
};

const transformer = (ast) => {
  return ast;
};

const codeGenerator = (node) => {
  switch (node.type) {
    case "Program":
      return node.body.map(codeGenerator).join("\n");

    case "VariableDeclaration":
      if (node.kind === "let") {
        let key = "var";
        return key + " " + node.declarations.map(codeGenerator);
      }

    case "VariableDeclarator":
      return codeGenerator(node.id) + " = " + codeGenerator(node.init) + ";";

    case "Identifier":
      return node.name;

    case "Literal":
      return node.value;
    default:
      throw new TypeError(node.type);
  }
};

module.exports = compiler;

```

## 7）参考

https://babel.dev/docs/en/

https://github.com/zloirock/core-js

https://webpack.js.org/guides/

https://baike.baidu.com/item/%E7%BC%96%E8%AF%91%E5%8E%9F%E7%90%86/4194

https://github.com/estree/estree

https://github.com/jamiebuilds/the-super-tiny-compiler

https://zhuanlan.zhihu.com/p/419252425