## HTML 语义化

就是使用恰当语义的标签，为了开发者更好的阅读代码，也为了浏览器搜索引擎更好的搜索。

如 header, footer, article, section, aside, nav

## HTML5 新增了哪些内容

- `<!doctype html>`
- 画布 canvas
- 媒体 video audio
- 本地存储 localStorage sessionStorage
- 语义化标签 header footer article section aside nav
- 多线程 web worker

## Doctype 的作用

`<!DOCTYPE html>` 告诉浏览器用标准模式解析这个文档，没有声明的话默认使用怪异模式。

- 标准模式：按照 W3C 的标准去渲染
- 怪异模式：在 W3C 标准推出之前的，旧浏览器的渲染方式

## canvas 相关

const ctx = canvasDom.getContext('2d');

```js
// 使用canvas标签
<canvas id="myCanvas" width="200" height="100"></canvas>;
// 获取canvas元素
var canvas = document.getElementById('myCanvas');
// 创建 context 对象
var ctx = canvas.getContext('2d');
// 绘制矩形
ctx.fillRect(0, 0, 150, 75);
// 填充颜色
ctx.fillStyle = '#FF0000';
// 绘制线条 定义开始坐标(0,0), 和结束坐标 (200,100)。然后使用 stroke() 方法来绘制线条
ctx.moveTo(0, 0);
ctx.lineTo(200, 100);
ctx.stroke();
// 绘制文本
ctx.font = '30px Arial';
ctx.fillText('Hello World', 10, 50);
// 绘制图像
var img = document.getElementById('img');
ctx.drawImage(img, 10, 10);
```

## 自定义属性，data-

data-属性用于一些自定义数据，所有 html 元素上都可以加，该属性值也能被 JavaScript 调用。

data-属性不应该包含任何大写字母，并且在 data-后必须最少拥有一个字符，属性值可以是任意字符串。

```html
<li id="getId" data-id="122" data-vice-id="11">获取id</li>
```

```js
const getId = document.getElementById('getId');

// 取值
console.log(getId.getAttribute('data-id')); // 122
console.log(getId.dataset.id); // 112

// 赋值
getId.setAttribute('data-id', '48');
getId.dataset.id = '113';
```

## meta 标签作用是什么，有哪些属性

meta 标签用来描述 HTML 文档的属性信息。主要用于 SEO，或者控制网页的一些显示。主要分为三种类型：  
1、charset 规定文档的字符集

- `<meta charset="UTF-8">`

2、http-equiv 主要用于模拟 HTTP 响应头，或者服务器可以将这些值添加在响应头里面（事实上没有服务器这么做，因为这样需要预先解析 HTML 并缓存所有的 http-equiv）。所以都是浏览器根据这些值去模拟。

- `<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">` 以当前 IE 浏览器所支持的最高版本模式来渲染页面，或者以 chrome 内核来渲染页面（前提是给当前 IE 浏览器安装了 Google Chrome Frame 插件）
- `<meta http-equiv="content-security-policy" content="script-src 'self'">` CSP
- `<meta http-equiv="refresh" content="5">` 每隔 5 秒刷新一次，或者重定向到某个 URL
- `<meta http-equiv="default-style">`指定默认样式表，实际很少使用
- `<meta http-equiv="content-language" content="en-US">` 用来规定语言版本，已弃用。使用 `<html>` 元素上全局的 lang 属性来替代它.
- `<meta http-equiv="content-type" content="text/html; charset=UTF-8">` 用来规定文本类型和字符集。已弃用。使用简写`<meta charset="UTF-8">`
- `<meta http-equiv="set-cookie" content="cookievalue=xxx; expires=Wednesday,21-Oct-98 16:14:21 GMT; path=/">` 设置 cookie。已弃用。请改用 HTTP 的 Set-Cookie 头部。

3、name 描述文档相关信息

- `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0">` 适用于移动端适配
- `<meta name="author" content="Shawn">` 作者名称
- `<meta name="description" content="test">` 页面内容的简短和精确的描述
- `<meta name="keywords" content="test,test">` 以逗号分隔的页面内容相关的单词
- `<meta name="robots" contect="all|none|index|noindex|follow|nofollow">` 设置页面哪些内容不希望被搜索引擎查到

## viewport 视窗

viewport 分为三种：

1. 浏览器默认的显示区域，默认 980px。叫 layout viewport。网页其实是渲染在这个 viewport 上
2. 显示在屏幕上的网页区域，没有固定值，其实就是你看见的区域，叫 visual viewport。 它可能只是 layout viewport 的一部分。 移动，缩放，改变的是 visual viewport
3. 理想区域，就是假设在一个区域，可以全部显示渲染的内容，而且用户可以看见。

通过`<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0">`来使浏览器的默认渲染区域等于 device 的宽度

## href 和 src 有什么区别

href 主要在 link 和 a 等元素上使用，用于在当前文档和引用资源之间确立联系，让当前标签能够链接到目标地址上。

src 主要在 img、script、iframe 等元素上使用，引入一个资源，并将该元素的内容整体替换。如果不写 src，script 会不存在脚本代码，img 会显示 x，iframe 会显示空白页。

## prefetch, preload

一般用在`<link>`元素的 rel 属性的属性值，可以指明哪些资源是在页面加载完成后即刻需要的。

提升下次将打开的页面速度， 用 prefetch。优化当前加载的页面，用 preload。

如果 prefetch 还没下载完之前，浏览器发现 script 标签也引用了同样的资源，浏览器会再次发起请求，所以不要在当前页面马上就要用的资源上用 prefetch，要用 preload。

prefetch 空闲时间下载该资源并缓存。

```html
<link href="js/chunk-1308.c09665a1.js" rel="prefetch" />
```

preload 立即下载该资源并缓存起来，但不执行。只有遇见这个 js 的时候，才开始执行，无需重新下载。

```html
<link href="js/chunk-1308.c09665a1.js" rel="preload" />
```

## 图片预加载，懒加载

预加载：在网页全部加载之前，提前加载图片，当用户需要查看时可直接从本地缓存中渲染。以提供给用户更好的体验，减少等待的时间

在当前页面的空闲时间，提前加载下一页的图片。

```js
// 这种写法会立即请求图片
let img = new Image()
img.src = 'xxx';
img.onload = function() { ... }
```

懒加载：延迟加载图片，当浏览到相应区域时或符合某些条件时才加载某些图片。懒加载的主要目的是前端的优化，减少请求数或延迟请求数。

在图片没有进入可视区域时，先不给 src 赋值（或者可以先给一个很小的 loading 图），等到图片进入可视区域再给 src 赋真正的值。图片的真实地址可以先存储在 data-src 中。

```js
// <img id="test" src="xxx/loading.png" data-src="xxx/real.png">

const dom = document.getElementById('test');
const distance = dom.getBoundingClientRect(); // 元素据视口的距离
const clientHeight = window.innerHeight; // 视口的高度
// distance.top <= clientHeight 时，图片是在可视区域内的。

window.onscroll = function () {}; // 在滚动条滚动的时候，判断图片的位置，如果在可视区域，替换src
```

两种技术的本质：两者的行为是相反的，一个是提前加载，一个是迟缓甚至不加载。
懒加载对服务器前端有一定的缓解压力作用，预加载则会增加服务器前端压力。

## img 的 srcset 的作用是什么，sizes 作用是什么

srcset 的值是一个字符串，用来定义一个或多个图像候选地址，以 , 分割，每个候选地址将在特定条件下得以使用。候选地址包含图片 URL 和一个可选的宽度描述符和像素密度描述符，该候选地址用来在特定条件下替代原始地址成为 src 的属性。

sizes 定义了一组媒体条件（例如屏幕宽度）并且指明当某些媒体条件为真时，什么样的图片尺寸是最佳选择。如果没有设置 srcset 属性，或者没值，那么 sizes 属性也将不起作用。

```html
<img
  src="clock-demo-thumb-200.png"
  alt="Clock"
  srcset="clock-demo-thumb-200.png 200w, clock-demo-thumb-400.png 400w"
  sizes="(min-width: 600px) 200px, 50vw"
/>
```

所以，有了这些属性，浏览器会：

- 查看设备宽度
- 检查 sizes 列表中哪个媒体条件是第一个为真
- 查看给予该媒体查询的槽大小
- 加载 srcset 列表中引用的最接近所选的槽大小的图像

## picture 元素的作用

用来做响应式图片。`<picture>` 元素通过包含零或多个 `<source>` 元素和一个 `<img>`元素来为不同的显示/设备场景提供图像版本。

```html
<picture>
  <source srcset="/media/examples/surfer-240-200.jpg" media="(min-width: 800px)" />
  <img src="/media/examples/painted-hand-298-332.jpg" />
</picture>
```

## 判断元素是否在视窗内

### 1. Element.getBoundingClientRect()

返回元素的大小及其相对于视口的位置。该函数返回一个 Object 对象，该对象有 6 个属性：
top,lef,right,bottom,width,height；
这里的 top、left 和 css 中的理解很相似，width、height 是元素自身的宽高，但是 right，bottom 和 css 中的理解有点不一样。right 是指元素右边界距窗口最左边的距离，bottom 是指元素下边界距窗口最上面的距离。

```js
const box = document.getElementById('box'); // 获取元素

box.getBoundingClientRect().top; // 元素上边距离页面上边的距离

box.getBoundingClientRect().right; // 元素右边距离页面左边的距离

box.getBoundingClientRect().bottom; // 元素下边距离页面上边的距离

box.getBoundingClientRect().left; // 元素左边距离页面左边的距离
```

### 2. IntersectionObserver API

自动"观察"元素是否可见，当元素变的可见时，会自动触发 callback

```js
const callback = (entries) => {
  console.log(entries);
};
const io = new IntersectionObserver(callback, option);
io.observe(document.getElementById('example'));
```

callback 一般会触发两次。一次是目标元素刚刚进入视口（开始可见），另一次是完全离开视口（开始不可见）。

entry 对象的属性：

- time：可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
- target：被观察的目标元素，是一个 DOM 节点对象
- rootBounds：根元素的矩形区域的信息，getBoundingClientRect()方法的返回值，如果没有根元素（即直\* 接相对于视口滚动），则返回 null
- boundingClientRect：目标元素的矩形区域的信息
- intersectionRect：目标元素与视口（或根元素）的交叉区域的信息
- intersectionRatio：目标元素的可见比例，即 intersectionRect 占 boundingClientRect 的比例，完全可见时为 1，完全不可见时小于等于 0

## script 标签中 defer 和 async 的区别

defer（延迟执行），async（立即执行）

浏览器解析 DOM 文档的时候，当浏览器解析到 script 标签的时候，如果不加 defer 或者 async，浏览器会暂停 DOM 文档的解析，而去下载 js 脚本并执行，然后再继续解析 DOM 文档

加 defer，浏览器解析到 script 标签的时候，不会停止 DOM 解析，而且并行执行 js 文件的下载，等 DOM 解析完成（DOMContentLoaded）之后，再去执行 js 文件。而且它是按照加载顺序执行脚本的。

加 async，浏览器解析到 sciprt 标签的时候，不会停止 DOM 解析，而且并行执行 js 文件的下载，但是下载完之后就立即执行，仅执行过程中 DOM 解析会被暂停。它是乱序执行的，不管你声明的顺序如何，只要它加载完了就会立刻执行。

## DOM 渲染的过程中可能有哪些情况会阻塞渲染

1. 加载 js 脚本。解决方法，放在 body 之后，或者加 defer

2. 加载 CSS 文件。解决方法，放在最前面，精简，压缩

## DOM 绑定事件

```js
// onclick: 同一个元素，同类事件只能添加一个，如果添加多个，后面添加的会覆盖之前添加的
dom.onclick = function () {
  alert('Hello');
};

// addEventListener: 可以给同一个元素添加多个同类事件
dom.addEventListener('click', function () {
  alert('hello1');
});
dom.addEventListener('click', function () {
  alert('hello2');
});
```

## DOM 事件流，冒泡捕获

冒泡：子节点一层层冒泡到根节点。

捕获：从 document 到触发事件的那个节点。

W3C 标准：先捕获再冒泡。

element.addEventListener(event, function, useCapture) useCapture 默认值是 false，表示在事件冒泡阶段调用事件处理函数。如果为 true，则表示在事件捕获阶段调用处理函数。

## 如何实现先冒泡后捕获

先捕获后冒泡这是机制问题，不可能改变的。可以改变的只有二者的回调函数的执行顺序。

1. 可以通过定时器才实现；

2. 可以用一个局部变量来实现，在捕获的回调中去修改这个局部变量，在冒泡的回调中判断这个局部变量是否被修改。冒泡的回调中可以是一个循环定时器，来不停地判断是否被修改，直到被修改为止。

## 什么是事件委托

事件委托就是利用事件冒泡的原理，把一个元素的事件委托到另一个元素去处理，一般委托到它的父层或者更外层元素上。

## seo 搜素引擎优化的方式

- 服务端渲染
- HTML 语义化
- 正确设置网站的标题、关键字、描述
- 合理设置 Robot.txt 文件
- 网站结构布局尽量简单
- `<strong>`、`<em>`标签，需要强调时使用，突出关键词
- 少使用 iframe 框架

## a 标签禁止跳转

```html
<a href="https://www.baidu.com" onclick="return false">test</a>
```

## 各种宽高

- document.body.clientWidth; // 宽度包含内边距（padding)
- document.body.offsetWidth; // 包括内边距（padding）和边框（border）和滚动条宽度
- document.body.scrollWidth; // 包括有滚动条时的未见区域
- document.body.scrollLeft; // 网页被卷去的 Left
- document.body.scrollTop; // 网页被卷去的 Top

---

- window.screen.height; // 屏幕分辨率的高
- window.screen.width; // 屏幕分辨率的宽
- window.screen.availHeight; // 屏幕可用工作区的高
- window.screen.availWidth; // 屏幕可用工作区的宽

---

- window.innerHeight; // 视口的高度
- window.screenTop; // 浏览器窗口相对于整个屏幕 Top 的距离
- window.screenLeft; // 浏览器窗口相对于整个屏幕 Left 的距离

![width-height](https://cdn.lishuxue.site/blog/image/面试/widthheight.png)
