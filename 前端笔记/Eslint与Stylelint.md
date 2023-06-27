## 安装与使用

```js
// 安装
yarn add eslint --dev

// 生成配置文件，会回答一些问题
npx eslint --init

// 校验文件
eslint file1.js file2.js
// 校验某个目录
eslint "src/**"
// 校验根目录所有的
eslint "." // 或者 eslint "./"
// 校验某类型的文件
eslint --ext .js --ext .jsx ./

// 自动修复错误
eslint --ext --fix .js --ext .jsx ./
```

## Parser

Parser 用来解析你的源码。ESLint 默认使用 Espree 作为解析器。

一般我们不需要使用其他解析器，除非我们使用了 ESLint 本身不支持的语法，如 TypeScript, Flow 或者是一些实验性的语法。

babel-eslint: 解析 Flow 或者一些高级的实验性的语法  
@typescript-eslint/parser：解析 typescript

```js
{
  "parser": "@typescript-eslint/parser"
  // 或
  "parser": "babel-eslint",
}
```

## Parser Options

指定需要 lint 的源代码的语法，如 ES7, JSX 等

```js
"parserOptions": {
  "ecmaVersion": 9, // 使用的ES版本，ES9 = ES2018
  "sourceType": "module", // 是否使用了模块语法，import/export
  "ecmaFeatures": { // 是否使用了额外的语言特性，如JSX
    "jsx": true
  }
}
```

一些其他的 parser 如 babel-eslint，也会有自己单独的 parserOptions 选项，详细的参考 parser 文档。

## ENV

使用一些 ESLint 预定义的全局变量，如为浏览器环境定义的，和为 node 环境定义的等。

```js
env: {
  browser: true,
  commonjs: true,
  es6: true,
  jest: true,
  node: true
}
```

## 自定义全局变量

如果不配置这些预定义的，或者自定义的全局变量，则在使用的时候 Eslint 会报 no-undef 警告。

```js
"globals": {
  "var1": true, // 可读可写
  "var2": false, // 只读
  "var3": "off" // 禁用
}

// 或者在文件内部
/* global var1, var2 */
```

## Plugins

Plugins 是干什么用的？

1. 暴露一些自定义的 rules
2. 暴露一些自定义的全局变量
3. 暴露一些自定义的配置
4. Plugin 中的 Processors 也可以告诉 EsLint 如何处理非 js 的文件

书写配置的时候，`eslint-plugin-`可以被省略。

```js
"plugins": [
  "plugin1", // eslint-plugin-plugin1
  "eslint-plugin-plugin2",
  "@jquery/jquery", // @jquery/eslint-plugin-jquery
  "@foobar" // @foobar/eslint-plugin
]
"extends": [
  "plugin:@foo/foo/recommended", // 使用plugin中的推荐配置
  "plugin:@bar/recommended"
],
"rules": {
  "jquery/a-rule": "error",
  "@foo/foo/some-rule": "error", // 使用plugin中提供的rule
  "@bar/another-rule": "error"
},
"env": {
  "jquery/jquery": true,
  "@foo/foo/env-foo": true, // 使用plugin中提供的预定义全局变量
  "@bar/env-bar": true,
}
```

常见的插件：

- eslint-plugin-prettier: 校验你的代码是否格式化，否则报 error。可以定制.prettierrc 文件。
- eslint-plugin-import：校验你的 import/export 语法错误，包括路径或名字错误。
- eslint-plugin-jsx-a11y: 检验 jsx 代码的 a11y 功能。可访问性（a11y）是衡量所有人（包括残疾或残障人士）对计算机系统的可访问性的度量。
- eslint-plugin-react: 校验 react 语法
- eslint-plugin-react-hooks: 校验 react hooks 语法
- @typescript-eslint/eslint-plugin：用来提供一些额外的 typescript 的 rule 供你使用。

## rules 级别

1. "off" or 0 ：关闭这个规则
2. "warn" or 1 ：将这个规则设成 warning 级别
3. "error" or 2 ：将这个规则设成 error 级别

## 文件内关闭 rule

1. 添加 `/* eslint-disable */` 在文件顶部关闭所有的 rule
2. 添加 `/* eslint-disable no-console */` 在文件顶部，关闭 no-console 的 rule，指定具体的 rule 关闭
3. 在某一行代码上面，添加 `// eslint-disable-next-line` 可以关闭下面的那一行的报错

### .eslintignore

.eslintignore 文件可以配置哪些文件无需 lint

## overrides 选项可以单独配置某一类文件的 rule

需要配合 files 字段使用，来为此类文件单独制定 rule

如项目中使用了 typescript，ESLint 本身不足以支持他的校验。则需要配置如下。

```js
overrides: [
  {
    files: ['**/*.ts?(x)'],
    parser: '@typescript-eslint/parser',
    parserOptions: {
      ecmaVersion: 2018,
      sourceType: 'module',
      ecmaFeatures: {
        jsx: true,
      },
      // typescript-eslint specific options
      warnOnUnsupportedTypeScriptVersion: true,
    },
    plugins: ['@typescript-eslint'],
    // make sure to disable the ESLint rule here.
    rules: {
      'default-case': 'off',
      ....
    }
  }
]
```

## 继承某一个配置文件

使用 extends 选项，可以继承一个现有的配置。

可以是一个配置文件的路径，也可以是一个可分享的配置文件名。

```js
extends: ['eslint:recommended', 'plugin:prettier/recommended']
```

数组中后面的项会继承前面的配置，所有的 rule 会被后面的配置修改。

常见的配置文件库：

1. eslint-config-airbnb
2. eslint-config-react-app

## 其他配置

假如我们在根目录有一个 eslintrc.js（项目级配置） 和 src/.eslintrc.js（目录级配置），当进行 Eslint 的时候，这两个配置文件会进行合并，且 src/.eslintrc.js 具有更高的优先级。

但是，我们只要在 src/.eslintrc.js 中配置 "root": true，那么 ESLint 就会认为 src 目录为根目录，不再向上查找配置。

如果我们整个项目根目录有一个.eslintrc.js 文件，就不需要设置 root: true

## stylelint

stylelint 用法跟 eslint 非常相似，具体参考: https://stylelint.io/
