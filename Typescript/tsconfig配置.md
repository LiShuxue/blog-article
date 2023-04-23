## tsconfig.json

tsconfig.json 是用 typescript 的 `tsc` 命令 来编译 ts 项目所需的配置文件。

在调用 tsc 命令并且没有其它输入文件参数时，编译器将由当前目录开始向父级目录寻找包含 tsconfig 文件的目录。

可以在 [github.com/tsconfig/bases](https://github.com/tsconfig/bases/) 上寻找一个合适的基本配置

## 配置项说明

```js
{
  "compilerOptions": {
    "target": "es5",                               // 编译后的目标代码版本
    "module": "commonjs",                          // 编译后的文件的模块类型
    "lib": [                                       // 声明开发阶段能使用的所有语法，否则ts报错。比如，document, Map
      "dom",
      "es2022"
    ],
    "allowJs": true,                               // 允许引入js文件，而不是仅仅允许 .ts 和 .tsx 文件
    "checkJs": true,                               // 检测js语法
    "typeRoots": [                                 // 指定声明文件types的路径，当我们项目中定义了一些声明文件的时候使用
        "./node_modules/@types/",
        "./src/types/"
    ],
    "strict": true,                                // 严格模式
    "noImplicitAny": true,                         // 不允许变量或函数参数具有隐式any类型
    "noUnusedLocals": true,                        // 检查只声明、未使用的局部变量
    "noUnusedParameters": true,                    // 检查只声明、未使用的函数参数
    "forceConsistentCasingInFileNames": true,      // 强制文件名大小写一致
    "skipLibCheck": true,                          // 跳过声明文件的类型检查
    "esModuleInterop": true,                       // 正确处理ES6的import语法
    "allowSyntheticDefaultImports": true,          // 允许有没有默认导出的模块导入
    "moduleResolution": "node",                    // 模块解析策略：https://www.typescriptlang.org/docs/handbook/module-resolution.html
    "resolveJsonModule": true,                     // 允许导入.json格式的文件
    "isolatedModules": true,                       // 导入的文件必须是模块
    "noEmit": false,                               // 只做类型校验，不输出，一般用于有其他的编译工具，比如webpack，我们只在开发阶段校验ts类型
    "jsx": "react",                                // JSX 编译为js之后的输出方式
    "baseUrl": "./",                               // 解析非绝对路径模块名时的基准目录
    "paths": {                                     // 路径映射，相当于baseUrl
      "src/*": ["src/*"],
      "components/*": ["src/components/*"]
    },
    "rootDirs": ["src","out"],                     // 告诉编译器有许多“虚拟”的目录作为一个根目录
    "outDir": "./dist",                            // 指定输出目录
    "rootDir": "./",                               // 用于控制输出目录结构
  },
  "include": [                                     // 指定需要编译的文件
    "src"
  ],
  "exclude": [                                     // 解析include中的文件时应跳过的文件
    "test/**/*"
  ]
}
```

## 编译 ts

### 安装 typescript

`yarn add typescript`

### 编译

运行 `tsc` 命令，编译器将寻找 tsconfig 文件，根据 tsconfig 的配置项进行编译。
