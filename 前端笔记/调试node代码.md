## 浏览器调试node程序
```js
// 以调式模式运行
node --inspect test.js

// 以调式模式运行，并停在第一行断点
node --inspect-brk test.js
```
![terminal](https://raw.githubusercontent.com/LiShuxue/blog-article/master/前端笔记/terminal.png)

可以看到，node 启动了一个 web socket 的 server，地址是：ws://127.0.0.1:9229/f9519489-5363-47f8-bdf6-f194e6173102，这是一个websocket server，这时候只要启动一个 websocket client 连接上这个 server 就可以调试 nodejs 代码了。

在 chrome 地址栏输入 chrome://inspect，然后点击 configure 来配置目标端口，就可以看到Remote Target中自己需要调试的文件。

## vscode调试node程序
### Attach模式
1. 先运行程序，并使断点停在第一行
    ```js
    // 以调式模式运行，并停在第一行断点
    node --inspect-brk test.js
    ```
2. 创建一个.vscode/launch.json配置文件，点击右下角的“Add Configuration”按钮添加配置。
    ```js
    {
      "name": "Attach",
      "port": 9229,
      "request": "attach",
      "type": "node"
    },
    ```

    因为已经通过 node --inspect-brk 启动了 websocket 的 debugger server，那么只需要启动 websocket client，然后 attach 上 9229 端口就行。

    在vscode中调试面板运行Attach脚本即可，会看见vscode调式面板且自动停在第一行。

    ![vscode](https://raw.githubusercontent.com/LiShuxue/blog-article/master/前端笔记/vscode.png)

### Launch模式
1. 创建一个.vscode/launch.json配置文件，点击右下角的“Add Configuration”按钮添加配置。

    ```js
    {
      "name": "Launch",
      "program": "${workspaceFolder}/test.js",
      "request": "launch",
      "type": "node",
      "stopOnEntry": true,
    },
    ```
2. 在vscode中，在需要调试的文件中，左侧打上断点，还可以设置 stopOnEntry 来在首行断住。在vscode中调试面板运行Launch脚本即可。

## vscode 调试 npm scripts
上面的调试都是node来执行具体的文件，有时候我们是通过npm script运行起来的一些程序，比如eslint等。

### 直接 Launch 模式
这些命令行工具的 package.json 里都会有个 bin 字段，来声明有哪些命令，npm install 这个包以后，就会放到 node_modules/.bin 目录下，这样我们就可以通过 node ./node_modules/.bin/xx 来跑不同的工具了，但是需要传一些参数。

1. 创建一个.vscode/launch.json配置文件，点击右下角的“Add Configuration”按钮添加配置。

    ```js
    {
      "name": "Launch Program",
      "program": "${workspaceFolder}/node_modules/.bin/vue-cli-service",
      "args": ["build"], // 需要run的命令
      "request": "launch",
      "skipFiles": ["<node_internals>/**"],
      "type": "node",
    },
    ```
2. 在命令行工具中打上断点，在vscode中调试面板运行Launch Program脚本即可

### Launch via NPM 模式
其实还有更简单的方式，VSCode Debugger 对 npm scripts 调试的场景做了封装，可以直接选择 npm 类型的调试配置。

1. “Add Configuration”添加配置：

    ```js
    {
      "name": "Launch via NPM",
      "request": "launch",
      "runtimeArgs": [
          "run-script",
          "serve" // 自己需要run的命令
      ],
      "runtimeExecutable": "npm",
      "console": "integratedTerminal", // 输出在terminal，跟正常启动没啥两样，且可以调试
      "skipFiles": [
          "<node_internals>/**"
      ],
      "type": "node"
    },
    ```

2. 在命令行工具中打上断点，在vscode中调试面板运行Launch via NPM脚本即可

把 console 配置为 integratedTerminal 之后，日志会输出到 terminal，和平时直接跑 npm run xx 就没区别了。而且还可以断点看看执行逻辑。
