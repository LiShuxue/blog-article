
## 1. 文件操作
写node程序，可能经常用到的文件的操作，或者文件夹的操作。

不需要用原生的fs模块，用更好的fs-extra包：https://www.npmjs.com/package/fs-extra

```js
/**
 * 文件操作: fs，fs-extra
 */

const fse = require("fs-extra");
const fs = require("fs");

// 读一个文件夹，返回文件夹下的内容 files
fs.readdirSync(path1);

// 判断文件或者目录是否存在
fse.pathExistsSync(path1);
// 确保文件存在(文件目录结构没有会新建)
fse.ensureFile(path1);
// 确保文件夹存在(文件夹目录结构没有会新建)
fse.ensureDir(path1);

// 清空文件夹（文件夹目录不删，内容清空）
fse.emptyDir(path1) 
// 删除空目录，不是空目录会报错
fs.rmdirSync(path1);
// 删除文件或文件夹，类似rm -rf
fse.remove(path1);
// 删除文件
fs.unlinkSync(path2);

// 判断路径是否是目录
fs.statSync(path1).isDirectory();
// 判断路径是否是文件
fs.statSync(path1).isFile();

// 写文件(目录结构没有会新建)
fse.outputFileSync(path1, 'test');
// 写文件，文件不存在自动创建
fs.writeFileSync(path2, "测试数据");

// 读取文件，返回buffer
fs.readFileSync(path2, "utf8");
// 读取JSON文件，将其解析为对象
fse.readJson()
// 将对象写入JSON文件
fse.writeJson()

// 文件末尾添加
fs.appendFileSync(path2, "拼接数据");

// 重命名目录 或者 文件, test1目录改为test2
fs.renameSync("./test1", "./test2");

// 移动目录或者文件
fs.renameSync("./test1", "./test/test2");
// 移动文件或文件夹
fse.move()

// 复制文件
fs.copyFileSync("./1.js", "./2.js");
// 复制文件或文件夹
fse.copy() 
```

## 2. 路径path操作
写node程序，可能经常用到的路径的操作或者判断。
```js
/**
 * 路径操作: path
 */

const path = require("path");

// 当前模块的目录名，不受node执行命令的所属路径影响
console.log(__dirname);

// 当前模块的文件名，不受node执行命令的所属路径影响
console.log(__filename);

// 拼接路径，根据参数返回一个规范的路径
console.log(path.join(__dirname, "test.js")); // /Users/lishuxue/Documents/study/journey/test.js
console.log(path.join("a", "b")); // a/b
console.log(path.join("a", "/b")); // a/b
console.log(path.join("a", "/b", "c")); // a/b/c
console.log(path.join("/a", "b")); // /a/b
console.log(path.join("/a", "/b")); // /a/b

// 把一个路径或路径片段的序列解析为一个绝对路径，表示依次要进入的目录，如果根据参数无法得到绝对路径，就以当前所在路径__dirname作为基准
// 相当于进行了一系列的cd操作，
console.log(path.resolve("a", "b")); // /Users/lishuxue/Documents/study/journey/a/b
console.log(path.resolve("a", "/b")); // /b
console.log(path.resolve("a", "/b", "c")); // /b/c
console.log(path.resolve("/a", "b")); // /a/b
console.log(path.resolve("/a", "/b")); // /b

// 获取路径的文件名
console.log(path.basename("/Users/lishuxue/Documents/study/journey/test.js"));

// 获取路径的目录名
console.log(path.dirname("/Users/lishuxue/Documents/study/journey/test.js"));

// 获取路径的后缀名
console.log(path.extname("/Users/lishuxue/Documents/study/journey/test.js"));

// 判断一个路径是否是绝对路径
console.log(path.isAbsolute("/Users/lishuxue/Documents/study/journey/test.js"));

// 将路径转成js对象
// {
//   root: '/',
//   dir: '/Users/lishuxue/Documents/study/journey',
//   base: 'test.js',
//   ext: '.js',
//   name: 'test'
// }
console.log(path.parse("/Users/lishuxue/Documents/study/journey/test.js"));
```

## 3. 进程操作
进程：进程是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础，进程是线程的容器。比如 node app.js 开启一个node进程

线程：一个线程只能隶属于一个进程，但是一个进程是可以拥有多个线程的。

单线程：就是一个进程只开一个线程。

浏览器中js执行是单线程的：https://www.ruanyifeng.com/blog/2014/10/event-loop.html

nodejs中js执行也是单线程的。

单线程的指的是 JavaScript 的执行是单线程的，但 Javascript 的宿主环境并非单线程。也就是浏览器或者node本身并不是单线程。

nodejs特点：异步非阻塞，事件驱动，I/O密集型

```js
/**
 * node进程程操作。process：当前 Node.js 进程的信息，child_process：子进程
 */


const child_process = require("child_process");
// 系统环境变量
process.env;

// 启动 Node.js 进程时传入的命令行参数
process.argv;

// 在 JavaScript 堆栈上的当前操作运行完成之后，且在下一个事件循环开始之前执行
process.nextTick(() => {
  //do something
});

// 获取进程id和父进程的id
console.log(process.pid);
console.log(process.ppid);

// 获取当前进程工作目录
console.log(process.cwd());

// 更改进程的当前工作目录
process.chdir("/");

// 当前进程运行的操作系统平台
console.log(process.platform);

// process.on监听事件
process.on("exit", (code) => {
  console.error("process exit, code: " + code);
});
process.on("uncaughtException", (err) => {
  console.error("Uncaught exception:", err);
});
process.on("unhandledRejection", (reason) => {
  console.error("Unhandled Promise Rejection: " + reason);
});

// 返回描述 Node.js 进程的内存使用量（以字节为单位）的对象
process.memoryUsage();

// stdout, stdin 和 stderr 是标准的流，当进程执行时，都会初始化这三个开放的文件描述符，跟外界建立联系。
// 当前进程的输入流，可以用来监听
process.stdin.on("data", (data) => {
  process.stdout.write(data + "\n"); // 当前进程的输入流，可以输出到控制台
});

// 给某进程发信号 'SIGUSR1' 启动调试器，'SIGINT' 终止 Node.js
process.kill(process.pid, "SIGUSR1");

// 开启一个子进程来执行其他程序
child_process.spawn();

// 开启一个shell子进程来直接执行命令，对返回的数据大小有限制
child_process.exec();

// 开启一个子进程来执行其他程序文件
child_process.execFile();

// 开启一个新的 Node.js 进程
child_process.fork();
```

## 4. 编写cli交互常用工具库
```js
/**
 * nodejs交互工具库
 */

// minimist 将命令行参数解析为 kv 键值对，方便代码操作
const minimist = require("minimist");
// commander 命令行参数解析，用于创建cli工具
const program = require("commander");
// inquirer terminal终端用户交互
const inquirer = require("inquirer");
// 字体颜色
const chalk = require("chalk");
// 终端的 loading 图标
const ora = require("ora");
// 终端中显示的进度条
const ProgressBar = require("progress");
// execa 对Node.js内置模块child_process的增强，用来执行命令或者文件
const execa = require("execa");
// shelljs 也是重新包装了 child_process，调用系统命令更加方便
const shelljs = require("shelljs");
// log-symbols 用来显示一个状态标记 √ x 等。
const logSymbols = require("log-symbols");
// listr 顺序执行或者同时执行一系列任务
const listr = require("listr");
// get-port 获取一个可用的端口
const getPort = require("get-port");
// fs-extra 对fs的增强
const fsExtra = require("fs-extra");
// axios http请求
const axios = require("axios");
// http-server 开启一个静态web服务器
const httpServer = require("http-server");
// glob 对文件进行模式匹配，查找不同名字或者格式等等的文件
const glob = require("glob");
// adm-zip 文件压缩和解压
const admZip = require("adm-zip");
```