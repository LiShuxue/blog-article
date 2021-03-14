## Doctype 的作用
`<!DOCTYPE html>` 告诉浏览器用标准模式解析这个文档，没有声明的话默认使用怪异模式。
* 标准模式：按照W3C的标准去渲染
* 怪异模式：在W3C标准推出之前的，旧浏览器的渲染方式

## data- 属性
data-属性其实就是一个自定义的属性，用于一些自定义数据，所有html元素上都可以加，该属性值也能被JavaScript调用。  
```html
<li id="getId" data-id="122" data-vice-id="11">获取id</li>
```
```js
const getId = document.getElementById('getId');

// 取值
console.log(getId.getAttribute("data-id")); // 122
console.log(getId.dataset.id); // 112

// 赋值
getId.setAttribute("data-id","48");
getId.dataset.id = "113";
```
data-属性不应该包含任何大写字母，并且在data-后必须最少拥有一个字符，属性值可以是任意字符串。

## HTML 语义化
就是使用恰当语义的标签，为了开发者更好的阅读代码，也为了浏览器搜索引擎更好的搜索。  
如header, footer, article, section, aside, nav

## HTML5 新增了哪些内容
* <!doctype html>
* 画布canvas
* 媒体video audio
* 本地存储localStorage sessionStorage
* 语义化标签 header footer article section aside nav
* 多线程 web worker

## a标签禁止跳转
```html
<a href="https://www.baidu.com" onclick="return false">test</a>
```

## meta标签作用是什么，有哪些属性
meta标签用来描述HTML文档的属性信息。主要用于SEO，或者控制网页的一些显示。主要分为三种类型：  
1. charset 规定文档的字符集
    * `<meta charset="UTF-8">`  
2. http-equiv 主要用于模拟 HTTP 响应头，或者服务器可以将这些值添加在响应头里面（事实上没有服务器这么做，因为这样需要预先解析HTML并缓存所有的http-equiv）。所以都是浏览器根据这些值去模拟。
    * `<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">` 以当前IE浏览器所支持的最高版本模式来渲染页面，或者以chrome内核来渲染页面（前提是给当前IE浏览器安装了Google Chrome Frame插件）
    * `<meta http-equiv="content-security-policy" content="script-src 'self'">` CSP
    * `<meta http-equiv="refresh" content="5">` 每隔5秒刷新一次，或者重定向到某个URL
    * `<meta http-equiv="default-style">`指定默认样式表，实际很少使用
    * `<meta http-equiv="content-language" content="en-US">` 用来规定语言版本，已弃用。使用 `<html>` 元素上全局的 lang 属性来替代它.
    * `<meta http-equiv="content-type" content="text/html; charset=UTF-8">` 用来规定文本类型和字符集。已弃用。使用简写`<meta charset="UTF-8">`
    * `<meta http-equiv="set-cookie" content="cookievalue=xxx; expires=Wednesday,21-Oct-98 16:14:21 GMT; path=/">` 设置cookie。已弃用。请改用HTTP的Set-Cookie头部。  
3. name 描述文档相关信息
    * `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0">` 适用于移动端适配
    * `<meta name="author" content="Shawn">` 作者名称
    * `<meta name="description" content="test">` 页面内容的简短和精确的描述
    * `<meta name="keywords" content="test,test">` 以逗号分隔的页面内容相关的单词
    * `<meta name="robots" contect="all|none|index|noindex|follow|nofollow">` 设置页面哪些内容不希望被搜索引擎查到

## viewport视窗
viewport分为三种：
1. 浏览器默认的显示区域，默认980px。叫layout viewport。网页其实是渲染在这个viewport上
2. 显示在屏幕上的网页区域，没有固定值，其实就是你看见的区域，叫visual viewport。 它可能只是layout viewport的一部分。 移动，缩放，改变的是visual viewport
3. 理想区域，就是假设在一个区域，可以全部显示渲染的内容，而且用户可以看见。

通过`<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0">`来使浏览器的默认渲染区域等于device的宽度

## href和src有什么区别
href 主要在 link和 a 等元素上使用，用于在当前文档和引用资源之间确立联系，让当前标签能够链接到目标地址上。   
src 主要在 img、script、iframe 等元素上使用，引入一个资源，并将该元素的内容整体替换。如果不写src，script会不存在脚本代码，img会显示x，iframe会显示空白页。

## prefetch, preload
prefetch（这段资源将会在未来某个导航或者功能要用到，但是本资源的下载顺序权重比较低，prefetch通常用于加速下一次导航）

preload（preload将会把资源得下载顺序权重提高，使得关键数据提前下载好，优化页面打开速度）

```html
<link href="css/chunk-fc0b.93f4bd1e.css" rel="prefetch">
<link href="js/chunk-1308.c09665a1.js" rel="preload">
```

## 图片预加载，懒加载
预加载：在网页全部加载之前，提前加载图片，当用户需要查看时可直接从本地缓存中渲染。以提供给用户更好的体验，减少等待的时间  
懒加载：延迟加载图片，当浏览到相应区域时或符合某些条件时才加载某些图片。懒加载的主要目的是前端的优化，减少请求数或延迟请求数。  
两种技术的本质：两者的行为是相反的，一个是提前加载，一个是迟缓甚至不加载。
懒加载对服务器前端有一定的缓解压力作用，预加载则会增加服务器前端压力。

## img的srcset的作用是什么，sizes作用是什么
主要用来帮助浏览器选择适当的图片去显示。

sizes 定义了一组媒体条件（例如屏幕宽度）并且指明当某些媒体条件为真时，什么样的图片尺寸是最佳选择。如果没有设置srcset属性，或者没值，那么sizes属性也将不起作用。  
srcset 定义了我们允许浏览器选择的图像集，以及每个图像的大小。

```html
<img src="clock-demo-thumb-200.png" alt="Clock"
     srcset="clock-demo-thumb-200.png 200w,
             clock-demo-thumb-400.png 400w"
     sizes="(min-width: 600px) 200px, 50vw">
```
所以，有了这些属性，浏览器会：  
* 查看设备宽度  
* 检查 sizes 列表中哪个媒体条件是第一个为真  
* 查看给予该媒体查询的槽大小  
* 加载 srcset 列表中引用的最接近所选的槽大小的图像

## picture 元素的作用
用来做响应式图片。`<picture>` 元素通过包含零或多个 `<source>` 元素和一个 `<img>`元素来为不同的显示/设备场景提供图像版本。
```html
<picture>
    <source srcset="/media/examples/surfer-240-200.jpg"
            media="(min-width: 800px)">
    <img src="/media/examples/painted-hand-298-332.jpg" />
</picture>
```

## script标签中defer和async的区别
defer（延迟执行），async（立即执行）  

浏览器解析DOM 文档的时候，当浏览器解析到script标签的时候，如果不加defer或者async，浏览器会暂停DOM文档的解析，而去下载js脚本并执行，然后再继续解析DOM 文档  

加defer，浏览器解析到script标签的时候，不会停止DOM解析，而且并行执行js文件的下载，等DOM 解析完成（DOMContentLoaded）之后，再去执行js文件。而且它是按照加载顺序执行脚本的。 
 
加async，浏览器解析到sciprt标签的时候，不会停止DOM解析，而且并行执行js文件的下载，但是下载完之后就立即执行，仅执行过程中DOM解析会被暂停。它是乱序执行的，不管你声明的顺序如何，只要它加载完了就会立刻执行。

## click在ios上有300ms延迟，原因及如何解决
用户一次点击屏幕之后，浏览器并不能立刻判断用户是要进行双击缩放，还是想要进行单击操作。因此就等待 300 毫秒。 
1. 禁用缩放 user-scale=no  
2. fastclick. FastClick 在检测到 touchend 事件的时候，会通过 DOM 自定义事件立即触发一个模拟click 事件，并把浏览器在 300 毫秒之后真正触发的 click 事件阻止掉。

## seo怎么整
* 服务端渲染
* HTML语义化
* 正确设置网站的标题、关键字、描述
* 合理设置Robot.txt文件
* 网站结构布局尽量简单
* `<strong>`、`<em>`标签，需要强调时使用，突出关键词
* 少使用iframe框架

## canvas 相关
```js
// 使用canvas标签
<canvas id="myCanvas" width="200" height="100"></canvas>
// 获取canvas元素
var canvas = document.getElementById("myCanvas");
// 创建 context 对象
var ctx = canvas.getContext('2d');
// 绘制矩形
ctx.fillRect(0,0,150,75);
// 填充颜色
ctx.fillStyle="#FF0000";
// 绘制线条 定义开始坐标(0,0), 和结束坐标 (200,100)。然后使用 stroke() 方法来绘制线条
ctx.moveTo(0,0);
ctx.lineTo(200,100);
ctx.stroke();
// 绘制文本
ctx.font="30px Arial";
ctx.fillText("Hello World",10,50);
// 绘制图像
var img=document.getElementById("img");
ctx.drawImage(img,10,10);
```

## 各种宽高
* document.body.clientWidth;        //网页可见区域宽(body)   
* document.body.clientHeight;       //网页可见区域高(body)   
* document.body.offsetWidth;        //网页可见区域宽(body)，包括border、margin等  
* document.body.offsetHeight;       //网页可见区域宽(body)，包括border、margin等  
* document.body.scrollWidth;        //网页正文全文宽，包括有滚动条时的未见区域   
* document.body.scrollHeight;       //网页正文全文高，包括有滚动条时的未见区域  
* document.body.scrollTop;          //网页被卷去的Top(滚动条)  
* document.body.scrollLeft;         //网页被卷去的Left(滚动条)   
---
* window.screen.height;             //屏幕分辨率的高  
* window.screen.width;              //屏幕分辨率的宽  
* window.screen.availHeight;        //屏幕可用工作区的高  
* window.screen.availWidth;         //屏幕可用工作区的宽  
---
* window.screenTop;                 //浏览器窗口相对于整个屏幕Top的距离
* window.screenLeft;                //浏览器窗口相对于整个屏幕Left的距离  

## DOM渲染的过程中可能有哪些情况会阻塞渲染
1. 加载js脚本。解决方法，放在body之后，或者加defer
2. 加载CSS文件。解决方法，放在最前面，精简，压缩

## DOM的事件模型是什么
1. 脚本模型：  
`<button onclick="javascrpt:alert('Hello')">Hello1</button>`
2. 内联模型：  
`<button onclick="showHello()">Hello2</button>`
3. 动态绑定：
```html
<button id="btn3">Hello3</button>
```
```js
// DOM0:同一个元素，同类事件只能添加一个，如果添加多个，后面添加的会覆盖之前添加的
var btn3 = document.getElementById("btn3");
btn3.onclick = function () {
    alert("Hello");
}

// DOM2:可以给同一个元素添加多个同类事件
btn3.addEventListener("click",function () {
    alert("hello1");
});
btn3.addEventListener("click",function () {
    alert("hello2");
})
```

## DOM事件流，冒泡捕获
冒泡：子节点一层层冒泡到根节点。  
捕获：从document到触发事件的那个节点。  
W3C标准：先捕获再冒泡。  
element.addEventListener(event, function, useCapture)最后的参数，true代表捕获反之代表冒泡。  
参数默认值是false，表示在事件冒泡阶段调用事件处理函数。如果参数为true，则表示在事件捕获阶段调用处理函数。

## 什么是事件委托?
通俗地来讲，就是把一个元素响应事件（click、keydown......）的函数委托到另一个元素去处理，一般委托到它的父层或者更外层元素上。  
事件委托就是利用事件冒泡的原理。