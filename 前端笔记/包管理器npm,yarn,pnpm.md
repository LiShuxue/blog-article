## 包管理器历史

### 远古时代

npm 没出现之前，前端都是通过下载一些压缩过的 js 包，或者使用部署在 cdn 等线上的 js 压缩包，这样的方式不管是自己贡献 js 包，还是引用别人的 js 包都极其麻烦。

### npm 时代

2011 年，[npm1.0 发布](https://nodejs.org/es/blog/npm/npm-1-0-released)，开启了前端时代的新篇章，加快了模块化编程的发展。用户可以从中央软件包存储库 npm registry 上下载和共享软件包，可以在全局或者本地安装。当时提供了一些简单的安装，卸载，更新，搜索等命令。

2014 年，[npm2 发布](https://blog.npmjs.org/post/98131109725/npm-2-0-0.html)，对于依赖管理和版本控制方面进行了重大改进，提高了 npm 的可靠性和性能。

npm2 安装依赖是嵌套安装的，这种方式会导致目录层级较深，甚至超出操作系统文件路径长度限制，导致安装失败的问题。依赖包的多个版本也可能会被安装在不同的目录下，导致冗余以及 node_modules 非常大。

2015 年，[npm3 发布](https://github.com/npm/npm/releases/tag/v3.0.0)，开始采用扁平化的依赖安装，将所有依赖包放在项目的 node_modules 目录的同一层级下。顶层依赖如果已经存在一个包 A，后续其他包再依赖这个包 A 的不同版本时，会下载在这个包内部而不是顶层。

[npm3 具有不确定性](https://npm.github.io/how-npm-works-docs/npm3/non-determinism.html)，也就是不同的人本地的 node_modules 中的依赖树可能是不一样的。会根据安装顺序不同而不同，如果想保持每个人或服务器一致，需要在每次更新 package.json 之后删除 node_moduels 重新安装。这个致命问题是催生出新的包管理器 Yarn 的直接因素

另一个致命问题是，npm3 安装依赖时是基于语义版本控制 (semver) 进行版本控制和安装的，也就是得依赖于包的维护人员知道自己升级的不是破坏性变更。这显然是不合理的。

- ~ 版本：只更新到最新的小版本，例如 ~1.2.3 只会更新到 1.2.x 的最新版本，不会更新到 1.3.x 甚至更高版本的包。
- ^ 版本：只更新到最新的兼容版本，例如 ^1.2.3 只会更新到 1.x.x 的最新版本，不会更新到 2.x.x 的包。

### yarn 时代

2016 年，[Yarn0.15 发布](https://engineering.fb.com/2016/10/11/web/yarn-a-new-package-manager-for-javascript/)，Yarn 采用了 lock 文件来解决这些版本控制和不确定性的问题。同时速度更快，下载包时会到一个全局缓存目录寻找，如果没有就下载并放入缓存，最后将所有需要的文件从全局缓存复制到本地 node_modules 目录。

2016 年，[npm4 也发布了](https://github.com/npm/npm/releases/tag/v4.0.0)，并没有什么突破性的重大改变，做了一些优化。

2017 年 5 月，[npm5 发布](https://github.com/npm/npm/releases/tag/v5.0.0)，跟 yarn 一样，使用了 lock 文件和全局缓存。

2017 年 9 月，[Yarn1.0 发布](https://engineering.fb.com/2017/09/07/web/announcing-yarn-1-0/)，增加了 workspace 功能，可以结合 lerna 来管理 monorepo。

后续 npm6,npm7，npm8 也都发布，大都是一些优化，npm7 中也实现了 workspace 的功能。

### pnpm 时代

2016 年 发布，[项目初衷](https://pnpm.io/zh/motivation)是为了安装更快，更节省磁盘空间。

2017 年，[v1.0.0 版本发布](https://github.com/pnpm/pnpm/releases/tag/v1.0.0)，也解释了[为什么用这个而不是 yarn](https://www.kochan.io/nodejs/why-should-we-use-pnpm.html)

- 本地有一个全局仓库，项目安装时，安装在全局仓库，并硬链接到项目虚拟仓库。如果用到了某依赖项的不同版本，只会将不同版本间有差异的文件添加到仓库。
- 非扁平的 node_modules 目录。并且不会有路径过长问题。

随着 pnpm 历代的版本更新，其刚开始的核心思想与初衷并没有发生变化。只是不断的进行 bug 的修复、性能的优化、新特性/命令的增加以及解决包生态的兼容性问题。比如支持类似 yarn 的 workspace, pnp 等。

### 后记

2020 年，[Yarn2.0 发布](https://dev.to/arcanis/introducing-yarn-2-4eh1)，采用了 Plug'n'Play (即插即用 Pnp)的默认安装策略，来解决 node_modules 太大的问题，生成一个.pnp.cjs 文件（最初是.png.js），而不是通常 node_modules 包含各种包副本的文件夹。该.pnp.cjs 文件包含各种映射：一个将包名称和版本链接到它们在磁盘上的位置，另一个将包名称和版本链接到它们的依赖项列表。此外，每个包都以 zip 文件的形式存储在.yarn/cache/文件夹内，占用的磁盘空间也比 node_modules 少，需要提交到 git。

默认安装的 yarn 都是 1.x 的稳定版本。2.x/3.x 由于改动太大，现在大部分都是观望，并不建议升级。

从 node16.10 开始，新增了[corepack 工具](https://nodejs.org/api/corepack.html)，来管理 yarn 和 pnpm，无需再手动安装 yarn 和 pnpm。`corepack enable` 来启用 corepack，会自动将 yarn 和 pnpm 添加到 bin 中，就可以使用他们了。

## npm 用法

```sh
npm init

npm i xxx （-S，-D，--save，--save-dev）
npm i -g xxxx

npm uninstall xxx
npm uninstall -g xxx

npm config get cache

npm list -g --depth 0

npm config get registry
npm config set registry https://registry.npmmirror.com
npm config delete registry
```

### npm init / npm create / npm exec / npm x / npx

- `npm init` 用来初始化一个项目，后面可以跟一个包名 `npm init xxx`，包名必须是 `create-xxx` 形式的。`npm init foo@latest` 用最新的 foo 包初始化项目。这个包会被 `npm exec` 执行。

- `npm create` 跟上面一样。

  ```sh
  npm create foo -> npm init foo -> npm exec create-foo
  ```

- `npm exec` 从本地或远程 npm 包中运行命令的。别名`npm x`。`npm exec create-foo` 跟 `npm exec -- create-foo` 结果是一样的。

  ```sh
  npm exec -- <pkg>[@<version>] [args...]
  npm exec --package=<pkg>[@<version>] -- <cmd> [args...]
  npm exec -c '<cmd> [args...]'
  npm exec --package=foo -c '<cmd> [args...]'
  ```

- `npx` 跟上面差不多，也是通过远程运行命令。

## yarn1.x 用法

```sh
yarn init

yarn add xxx (-S，-D，--save，--dev)
yarn global add xxx

yarn remove
yarn global remove

yarn cache dir
yarn config set cache-folder <path>

yarn global list --depth=0

yarn config get registry
yarn config set registry https://registry.npmmirror.com
yarn config delete registry
```

yarn workspace 和 lerna。使用 yarn workspace 来管理依赖，使用 lerna 来管理 npm 包的版本发布。

启用 workspace：在项目根目录的 package.json 中增加 workspace 字段。

安装项目依赖时，可以在项目根目录执行 yarn install，这样就会安装整个 Workspace 的依赖。

```js
{
  "private": true,
  "workspaces": ["app1", "app2"] // 具名Workspace，名字任意，但需跟子项目中package.json中name属性值一致
}

{
  "private": true,
  "workspaces": ["projects/*"] // 当Workspace很多时，也可以采用全目录引用的方式。假设项目代码在projects目录下
}
```

```sh
yarn workspace <workspace_name> <command> # 在指定的package中运行指定的命令
yarn workspaces run <command> # 在所有package中运行指定的命令
yarn <add|remove> <package> -W # 在根目录安装依赖，尽量都用这样安装，不要单独在某一个下面安装
```

## pnpm 用法

全局 store：/Users/lishuxue/Library/pnpm/store/v3。

虚拟 store：项目目录/.pnpm。项目中所有直接和间接的依赖项都链接到此目录中，平铺展示。

硬链接：同一个文件的不同引用（只能用于文件）。所有依赖包的文件是从全局 store 硬连接到虚拟 store 的。

软链接：实际上是一个文本文件，其中包含的有另一文件或者目录的位置信息。包和包之间的依赖关系是通过软链接组织的。

workspace：在项目根目录中新建 pnpm-workspace.yaml 文件，并声明对应的工作区就好。工作区协议"workspace:"使用此协议时，pnpm 将拒绝解析除本地工作区 package 之外的任何内容。

```yaml
packages:
  # 所有在 packages/ 子目录下的 package
  - 'packages/**'
```

```sh
pnpm init

pnpm install # 安装项目中所有依赖

pnpm add （-P，-D，--save-prod，--save-dev）# 添加依赖
pnpm add -w # workspace 根目录安装
pnpm add xxx --global

pnpm remove xxx

pnpm list # 查看项目依赖
pnpm list -g --depth 0

pnpm store path # 查看store路径

pnpm config get registry
pnpm config set registry https://registry.npmmirror.com
pnpm config delete registry
```

pnpm 依赖文件通过三次寻址方式找到下载资源：

- 第一次通过 package.json 中安装依赖找到 node_modules 目录下依赖包，即 node_modules/antd；

- 第二次通过 node_modules/antd 软链接 node_modules/.pnpm/antd@4.23.1/node_modules/antd 解决了代码重复引用的问题；

- 第三次通过 node_modules/.pnpm/antd@4.23.1/node_modules/antd 硬链接 ~/.pnpm-store/v3/files/00/xxx 已经脱离当前项目路径，指向一个全局统一管理路径，这种方式符合 node_modules 默认寻址方式，节省了磁盘空间，也解决了幽灵依赖

## 幻影依赖 & 依赖分身

幽灵依赖或幻影依赖，解释起来很简单，即某个包没有在 package.json 被依赖，但是用户却能够引用到这个包。

依赖分身：因为只有一个被提升到最外层，所以内部还是有一些嵌套，有大量的依赖的被重复安装。

## nodejs 模块解析

在 Node.js 中引入一个模块时，Node.js 会按照特定的规则来解析模块的路径，并找到正确的模块文件。下面是 Node.js 模块解析的一般规则：

- 核心模块（Built-in Modules）：首先，Node.js 会检查模块是否是核心模块，即 Node.js 提供的内置模块。如果是核心模块，则直接加载并返回该模块。

- 文件模块（File Modules）：如果模块不是核心模块，则 Node.js 会尝试从文件系统中加载模块。Node.js 根据模块路径的相对性进行解析。

  - 如果模块路径以 '/'、'./' 或 '../' 开头，它会被视为相对路径，并相对于当前模块的位置进行解析。

  - 如果模块路径不是相对路径，则 Node.js 会在 node_modules 目录中搜索该模块。它会从当前模块所在的目录开始，逐级向上搜索，直到找到匹配的模块或达到文件系统的根目录。

- 文件夹模块（Folder Modules）：如果模块路径指向一个目录而非文件，则 Node.js 会尝试查找该目录下的 package.json 文件。

  - 如果存在 package.json 文件，并且其中有指定 "main" 字段，Node.js 将加载指定的主文件作为模块。

  - 如果没有 package.json 文件或者没有指定 "main" 字段，Node.js 会尝试加载该目录下的 index.js 或 index.json 文件作为模块。

- 模块路径解析：如果以上步骤仍未找到模块，则 Node.js 会按照模块路径的解析规则，尝试查找全局安装的模块或其他自定义路径中的模块。

## nvm

安装：https://github.com/nvm-sh/nvm#installing-and-updating

使用

```sh
nvm list

nvm install stable
nvm install 14/16/18

nvm uninstall 14/16/18

nvm use <version> // 当前窗口使用

nvm alias default <version> // 所有窗口使用
```
