## 引入 `typescript` 和 `@types`

```js
"devDependencies": {
    "@types/jsonwebtoken": "^8.3.3",
    "@types/koa": "^2.0.49",
    "@types/koa-bodyparser": "^4.3.0",
    "@types/koa-router": "^7.0.42",
    "@types/koa2-cors": "^2.0.1",
    "@types/mongoose": "^5.5.12",
    "typescript": "^3.5.3"
}
```

## 配置 `tsconfig.json`

当使用 tsc 编译 typescript 的时候，会寻找这些配置

```js
{
  "compilerOptions": { // 编译器选项 详见：https://www.tslang.cn/docs/handbook/compiler-options.html
    "module": "commonjs",
    "esModuleInterop": true,
    "target": "es6",
    "noImplicitAny": true,
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "dist",
    "baseUrl": ".",
    "paths": {
      "*": [
        "node_modules/*",
        "src/types/*"
      ]
    }
  },
  "include": [ // 使用"include"引入的文件可以使用"exclude"属性过滤
    "src/**/*"
  ]
}
```

## 重构 `.js` 文件为 `.ts` 文件

如 `server.js` 修改为 `server.ts`

将所有的文件都修改后缀名

## 重构代码

- `require` 语法改为 `import` 语法

```ts
const Koa = require('koa');
// 改为
import Koa from 'koa';
```

- `module.exports` 语法改为 `export` 语法

```ts
module.exports = {
  publishNewBlog,
  getAllBlog,
};
// 改为
export default {
  publishNewBlog,
  getAllBlog,
};
```

- 定义变量的时候加上类型

```ts
let str = '';
// 改为
let str: string = '';
```

- `ctx` 和 `next` 对象的类型

```ts
import { Context } from 'koa';
(ctx: Context, next: Function)
```

- 改写 `middleware` 或者方法

```ts
const tokenMiddleware = async (ctx, next) => {};
// 改为
async function tokenMiddleware(ctx: Context, next: Function): Promise<any> {}
// 或
const tokenMiddleware = async (ctx: Context, next: Function): Promise<any> => {};
```

- 用 `interface` 定义对象的形状

```ts
const payload = {
  iss: 'Journey',
  sub: 'www.lishuxue.site',
  aud: 'www.lishuxue.site',
};
// 改为
interface Payload {
  iss: string;
  sub: string;
  aud: string;
}
const payload: Payload = {
  iss: 'Journey',
  sub: 'www.lishuxue.site',
  aud: 'www.lishuxue.site',
};
```

- 用 `interface` 定义 `Model` 的形状

```ts
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const blogSchema = new Schema(
  {
    title: String,
    subTitle: String,
    htmlContent: String,
    markdownContent: String,
    image: {
      name: String,
      url: String,
    },
    isOriginal: Boolean,
    publishTime: Date,
    updateTime: Date,
    see: Number,
    like: Number,
    category: String,
    tags: [String],
    comments: [Schema.Types.Mixed],
  },
  {
    collection: 'Blog',
  }
);
const Blog = mongoose.model('Blog', blogSchema);

// 改为
import { Schema, Document, Model, model } from 'mongoose';
interface IImage {
  name: string;
  url: string;
}
interface ITag {
  [index: number]: string;
}
interface IComment {
  arthur: string;
  date: Date;
  content: string;
}
export interface IBlog extends Document {
  title: string;
  subTitle: string;
  htmlContent: string;
  markdownContent: string;
  image: IImage;
  isOriginal: boolean;
  publishTime: number;
  updateTime: number;
  see: number;
  like: number;
  category: string;
  tags: ITag;
  comments: IComment[];
}
const blogSchema: Schema = new Schema(
  {
    title: String,
    subTitle: String,
    htmlContent: String,
    markdownContent: String,
    image: {
      name: String,
      url: String,
    },
    isOriginal: Boolean,
    publishTime: Date,
    updateTime: Date,
    see: Number,
    like: Number,
    category: String,
    tags: [String],
    comments: [Schema.Types.Mixed],
  },
  {
    collection: 'Blog',
  }
);
const Blog: Model<IBlog, {}> = model<IBlog>('Blog', blogSchema);
```

- 编译项目

在项目根目录执行 `tsc` , 它会默认寻找根目录的 `tsconfig.json` , 根据这个配置去编译文件

```sh
tsc
```

- 运行项目

编译完成之后，会把编译出来的 js 文件输出到指定目录，如 dist，这是在 `tsconfig.json` 配置的。

之后我们只需要运行编译好的 js 文件即可。

```sh
node dist/server.js
```
