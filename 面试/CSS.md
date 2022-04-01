## 盒模型，标准盒模型，怪异（IE）盒模型
浏览器将所有元素表示为一个个矩形的盒子，包括content, padding, border, margin  
* 标准盒模型：box-sizing: content-box; width = content  
* 怪异盒模型：box-sizing: border-box; width = content + padding + border  

## 行元素与块元素
* 行元素：不会独占一行，不能设置宽高，不能设置垂直方向的padding, margin  
* 块元素：独占一行，可以设置宽高，padding, margin, 如果不设置宽度，那么宽度将默认变为父级的100%  

行元素和块元素都可以设置border  

## margin: 0px 10px 10px 分别代表哪几个
* 四个值：上右下左
* 三个值：中间的值代表右左
* 两个值：上下，左右

## display: block, inline, inline-block的区别
* display: block; 将一个行元素设置为块元素，独占一行，并且可以给他设置宽, 高, padding, margin
* display: inline; 将一个块元素改变为行元素，不会独占一行，并且不能设置宽高，不能设置垂直方向上的padding和margin
* display: inline-block; 将一个元素设置为行内块。即既有行元素的特性，不会独占一行，跟其他行元素并列排一行，又有块元素的特性，可以设置宽高padding margin。

## CSS实现一个三角形
```css
.sanjiao{
    width: 0px;
    height: 0px;
    border: 50px solid transparent;
    border-top-color: black;
}
```

## CSS实现一个扇形
```css
.sanxing{
    width: 0px;
    height: 0px;
    border: 100px solid transparent;
    border-top-color: red;
    border-radius: 50%;
}
```

## CSS实现一个箭头，45度角
原理：需要实现一个45度的菱形，取border。通过正方形来调整边长和倾斜。
```css
.arrow {
    width: 42px; // 高的根号2倍
    height: 30px;
    background: red;
    border-right: 1px solid black;
    border-bottom: 1px solid black;
    transform: rotate(-22deg) skewX(45deg)
}
```

## 水平居中，垂直居中
### 水平居中  
行元素：  
首先看它的父元素是不是块级元素，如果是，则直接给父元素设置 text-align: center;   
如果不是，则先将其父元素设置为块级元素，再给父元素设置 text-align: center;

块元素：
* margin: 0 auto; 需要设置该元素的宽度
* justify-content: center;
* 绝对定位+translate，适用于不定宽  
    position: absolute;   
    left:50%;  
    transform:translateX(-50%);  
* 绝对定位+负margin，适用于定宽  
    position: absolute;  
    width: 100px;  
    left: 50%;  
    margin-left: -50px;

### 垂直居中
行元素：  
若元素是单行文本, 则可设置 line-height 等于父元素高度  
或者设置父元素 display: table-cell; vertical-align: middle;  
若元素是多行文本，也是设置父元素 display: table-cell; vertical-align: middle;  

块元素中：  
* align-items: center;  
* 绝对定位+translate，适用于不定高
* 绝对定位+负margin，适用于定高

## 浮动，清除浮动
浮动：float: left或者float: right. 脱离文档流，不能撑开父元素。  
清除浮动：
* 在父元素使用伪类：  
    ```css
    .parent::after {  
        content: '';  
        display: block;  
        clear: left;  
    }
    ```
* 父元素使用overflow: auto;

## 定位方式
static：正常文档流定位。此时 top, right, bottom, left 和 z-index 属性无效，块级元素从上往下纵向排布，行级元素从左向右排列。  
relative：相对定位，相对自身正常文档流的位置。  
absolute：绝对定位，相对于最近的非static的祖先元素定位。  
fixed：固定定位，相对于屏幕视口定位。  
sticky：粘性定位，初始时，元素在正常文档流位置，当屏幕滚动时，该元素会始终固定于顶部某位置。
```css
position: sticky;
top: 10px;
```

## transition，transform，translate，animation，@keyframes
transition：过渡。可以定义过渡属性，过渡效果时间，速度曲线，延迟执行。  
`transition: width 2s ease 2s;`  
transform：变换。平移translate，缩放scale，旋转rotate，倾斜skew。  
`transform: translate(10px) rotate(45deg) scale(1.5) skew(45deg)`
translate：transform的一个值。  
animation：动画。可以定义keyframe的名字，动画时间，速度曲线，延迟执行，播放次数，轮流反向播放。  
`animation: myKeyframe 4s infinite;`  
@keyframes定义具体的动画细节。注意keyframes， 加s  
```css
@keyframes myKeyframe
{
    0%   {top:0px;}
    25%  {top:200px;}
    50%  {top:100px;}
    75%  {top:200px;}
    100% {top:0px;}
}
```
速度曲线：
* linear	动画从头到尾的速度是相同的。
* ease	默认。动画以低速开始，然后加快，在结束前变慢。
* ease-in	动画以低速开始。
* ease-out	动画以低速结束。
* ease-in-out	动画以低速开始和结束。

## transition和animation的区别
* transition只能播放一次
* transition只有开始和结束状态，而animation可以定义很多中间状态
* transition需要有触发条件，如hover或者js事件，而animation写了之后会自动播放

## link标签和@import的区别
* 两者都是引入外部css文件
* link是html标签，@import是css语法
* link除了css，还可以引用其他资源文件。
* link引用的css会在页面加载的时候同时加载，而import引入的css需要等页面完全加载完后才去加载。

## flex布局
父元素： 
```css 
display: flex;  
flex-direction: row | column; /* 横竖 */
justify-content: center | flex-start | flex-end | space-between | space-around; /* 居中，左对齐，右对齐，两端对齐，中间间隔相等 */
flex-wrap: wrap | no-wrap | wrap-reverse; /* 换行， 不换行， 跟正常换行相反，如向上换行，第一行在下，新行在上 */
align-items: center | flex-start | flex-end | base-line | stretch; /* 交叉轴对齐方式：居中，上对齐，下对齐，第一行基线对齐，子元素占满整个容器高度 */
```
子元素：  
默认会缩小不会放大
```css
order: 0; /* 数值越小越靠前 */
flex-grow: 0; /* 有剩余空间时放大系数，0不放大 */
flex-shrink: 1; /* 空间不足时如何缩小，0不缩小 */
flex-basis: 100px; /* 项目的初始长度。若不设置伸缩性，则为固定大小 */
align-self: flex-start; /* 自己单独的对齐方式，可覆盖align-items属性 */
flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。
```

## grid布局
父元素：
```css
display: grid; /* 创建一个网格容器 */
grid-template-columns: 200px 100px 200px; /* 声明了三列，宽度分别为 200px 100px 200px */
grid-gap: 5px; /* 声明行间距和列间距 */
grid-template-rows: 50px 50px 50px; /* 声明了三行，行高分别为 50px 50px 50px  */
grid-auto-flow: row; /* row或者column， 默认row， "先行后列"，即先填满第一行，再开始放入第二行 */
/* 上面的布局是一个九宫格 */
justify-items: start | end | center | stretch; 单元格水平方向的对齐方式
align-items: start | end | center | stretch; 单元格垂直方向的对齐方式
justify-content: start | end | center | stretch | space-around | space-between | space-evenly;  是整个内容区域在容器里面的水平位置（左中右）
align-content 属性是整个内容区域的垂直位置（上中下）
```
grid-row-gap 属性、grid-column-gap 属性分别设置行间距和列间距。grid-gap 属性是两者的简写形式。

假如有多余的网格，那么它的行高和列宽可以根据 grid-auto-columns 属性和 grid-auto-rows 属性设置。

子元素：

grid-column-start 属性、grid-column-end 属性、grid-row-start 属性以及grid-row-end 属性，可以指定网格项目所在的四个边框，分别定位在哪根网格线，从而指定项目的位置。

justify-self 属性、align-self 属性

justify-self 属性设置单元格内容的水平位置（左中右），跟 justify-items 属性的用法完全一致，但只作用于单个项目   
align-self 属性设置单元格内容的垂直位置（上中下），跟align-items属性的用法完全一致，也是只作用于单个项目

## 隐藏页面元素 
* opacity=0：不会影响页面布局，还可以触发点击事件
* visibility=hidden：不会影响页面布局，不可以触发点击事件
* display=none：影响布局。可以理解为将那个元素删掉一样

## 选择器
1. id选择器
2. 元素选择器
3. class选择器
4. 属性选择器 `.div[data-v-60c8fc3f] | img[src] | img[src="./img/test.jpg"] | img[src~="./img"]`
5. 相邻(兄弟)元素选择器，用 ‘+’
6. 子元素选择器，用 ‘>’
7. 后代元素选择器， 用空格 ‘ ’
8. 伪类选择器 a:hover  a:visited
9. 伪元素选择器 div::after
10. 结合元素选择器： div.test  所有包含test类的div
11. 多类选择器： .testA.testB  选择同时包含这些类名的元素

选择器分组（分隔），用逗号 ` , `

## 选择器的优先级，权重计算
!important > 内联 > ID选择器 > 类选择器 > 元素选择器  

权重计算：  
有四个值，ABCD, 从左到右分别表示，内联样式，ID选择器，类选择器，元素选择器  
ul li .red { ... } 的权重表示 {A=0, B=0, C=1, D=2}, 即 {0, 0, 1, 2}  
#red { ... } 的权重表示 {0, 1, 0, 0}  
权重比较：将ABCD四个值，从左到右依次比较，大的则优先级更高。

## 浏览器如何解析css选择器
按照CSS类从上到下的定义解析，后定义的会覆盖前面的。
```html
<div class="a">text1</div>
<div class="b">text2</div>
<div class="a b">text3</div>
<div class="b a">text4</div>

<style>
.a {
  color: red;
}
.b {
  color: green;
}
.a.b {
  color: blue;
}
.b.a{
  color: orange;
}
</style>

text1 ===> red
text2 ===> green
text3 ===> orange
text4 ===> orange
```

## 伪类，伪元素
### 伪类： 用一个冒号
* :active  被激活的元素 
* :focus   
* :hover    
* :link   未被访问的链接 
* :visited   
* :first-child  
* :last-child 
* :nth-child(n) 第n个子元素
* :nth-of-type(n) 同类子元素中的第n个
* :not(selector) 除 selector 元素以外的元素
* :enabled
* :disabled
* :checked
### 伪元素： 用两个冒号
* ::before
* ::after
* ::first-line  文本第一行
* ::first-letter  第一个字符

## px, rem, em, vw/vh
* px 绝对单位
* rem 相对单位，相对根节点html元素的font-size
* em 相对单位，相对父级元素的font-size
* vw/vh 视口单位，整个视窗为100vw/vh

## 移动端适配
### 像素
1. css像素：js或者css代码中使用的px单位就是指的是css像素。  
2. 物理像素：物理像素也称设备像素，他是显示设备中一个最微小的物理部件。
3. 设备独立像素(逻辑像素)：设备独立像素也称为密度无关像素，可以认为是计算机坐标系统中的一个点，这个点代表一个可以由程序使用的虚拟像素(比如说CSS像素)，然后由相关系统转换为物理像素。
4. 设备像素比：设备像素比简称为dpr，其定义了物理像素和设备独立像素的对应关系。`设备像素比 ＝ 物理像素 / 设备独立像素（逻辑像素）`，可以由`window.devicePixelRatio`获得
5. 屏幕分辨率： 设备的物理像素个数。
6. 屏幕密度：屏幕密度是指一个设备表面上存在的物理像素数量。它通常以每英寸有多少像素来计算(PPI)。

综上所述：如iPhone6的分辨率：750 * 1334，则它的物理像素为：750 * 1334。他的像素比DPR为2，逻辑像素分辨率为：375 * 667。

### 视口
1. 布局视口（layout viewport）：移动端的默认布局行为，默认为980px。也就是说在不设置网页的viewport的情况下，pc端的网页默认会以布局视口为基准，在移动端进行展示。  
2. 视觉视口（visual viewport）：用户看到的网站的区域，用户可以通过移动缩放来查看网页的显示内容，从而改变视觉视口大小，比如本来看到的是375px，如果用户将网页缩小一倍，用户将看到750px的范围。  
3. 理想视口（ideal viewport）： 网页的内容，全部显示在设备的视口。
4. width=device-width设置网页的宽度跟设备的逻辑像素相等。就相当于让布局视口等于理想视口。
5. initial-scale = 理想视口 / 视觉视口。

综上所述：iPhone6的默认视口宽度980px，加上width=device-width，视口宽度为375px; 也就是说此时总宽度为CSS像素375px。如果initial-scale设为0.5则视口宽度为750px。

### 适配
假设设计师给了一个原型图，宽750px，里面有一个按钮，200px。怎么样使其在不同的页面上显示。
假设我们要在iPhone5, iPhone6，iPhone6 plus上显示，已知他们的逻辑宽度分别是320px，375px，414px
1. rem原理  
    * 假设将设计图等分成100rem，则1rem = 7.5px。即在宽度为750px的屏幕中，根元素font-size为7.5px。200px = 26.67rem。
    * 在不同的屏幕中，只需要保持上面的比例即可。例如宽度320px的屏幕中，定义根元素的font-size，为320 / 750 * 7.5 = 3.2px
    * 同理宽度375的屏幕为375 / 750 * 7.5 = 3.75px
    * 即是font-size = document.documentElement.clientWidth / 750 * 7.5
    * 设计图上宽度为200px的按钮则始终是26.67rem。
2. vw原理
    * 假设将设计图等分成100vw，则1vw = 7.5px。200px = 26.67vw。
    * 因为vw默认就是将设备视窗分成100份，所以我们无须单独为每个设备去计算，只需按照UI图上的vw比例写即可。
    * 如在宽320px的iPhone5中，也是26.67vw。保持这个比例即可。
3. 文本还用px
4. 上述rem和vw，都可以保持UI的比例跟设计图一样。但是如果涉及到必须使用px的地方。此时1px就可能显示多个像素点。所以还需要设置viewport的initial-scale

## 移动端 Retina 1px 像素问题的解决方案
不同的设备，像素比不一样。对于像素比为1的设备，渲染1px长需要一个物理像素，对于像素比为3的，需要3个物理像素。
1. 设置viewport的initial-scale = 1 / dpr。
2. 媒体查询结合transform缩放为相应尺寸。
```css
@media screen and (-webkit-min-device-pixel-ratio: 2) {
    div {
        height:1px;
        background:#000;
        transform: scaleY(0.5);
    }
}
@media screen and (-webkit-min-device-pixel-ratio: 3) {
    div {
        height:1px;
        background:#000;
        transform: scaleY(0.333);
    }
}
```

## 外边距折叠（合并）
块级元素的上下外边距(margin)在某些情况下会合并（折叠）起来，合并之后的大小为较大的margin。  
准确来说，外边距折叠应该叫垂直外边距折叠，因为只会发生在垂直方向上，而水平方向上不会发生。

会发生外边距折叠的情况：  
1. 相邻兄弟元素
2. 父元素的上外边距和第一个子元素的上外边距
3. 父元素的下外边距和最后一个子元素的下外边距
4. 空元素自己的上外边距会和自己的下外边距合并

解决方案：  
1. 变为BFC。但overflow:hidden不能解决相邻元素外边距重叠问题。
2. 使用padding或者border取代margin间距效果

## 什么是BFC? 如何产生BFC? BFC作用
BFC:  
块级格式化上下文(Block Formatting Context)，它是一个独立的渲染区域，BFC内部的元素与外部的元素互相隔离。  
主要是管理他的子元素的。

如何产生BFC：  
1. position: absolute、fixed
1. 弹性盒子元素，不是父元素（flex或inline-flex）
1. 浮动float
1. 根HTML元素
1. 具有overflow 且值不是 visible 的块元素
1. display: inline-block, table-cell, table-caption

BFC的作用：  
1. 解决父元素与子元素外边距重叠问题。
2. 去除浮动，解决浮动引起的高度塌陷

## 层叠上下文，层叠等级，层叠顺序，z-index
### 层叠上下文   
其实就是在Z轴上的使用权利，元素有了层叠上下文之后，会根据元素属性和优先顺序占用Z轴空间。
#### 怎么生成层叠上下文元素
1. position: relative , absolute 且z-index值不为auto
1. z-index不为auto的flex item元素(不是父元素)
1. opacity值小于1的元素
1. 根HTML元素
1. 其他一些 CSS3 属性，属性值不为none  
    * transform
    * filter
    * ...

### 层叠等级(层叠水平)
它决定了同一个层叠上下文中元素在 Z 轴上的显示顺序，越靠上等级越大

### 层叠顺序
元素在同一个层叠上下文中的顺序规则，从层叠的底部开始，共有七种层叠顺序：  
1. 背景和边框：形成层叠上下文的元素的背景和边框。
2. 负z-index值：层叠上下文内有着负z-index值的定位子元素，负的越大层叠等级越低；
3. 块级盒：文档流中块级、非定位子元素；
4. 浮动盒：非定位浮动元素；
5. 行内盒：文档流中行内、非定位子元素；
6. z-index: 0：z-index为0或auto的定位元素， 这些元素形成了新的层叠上下文；
7. 正z-index值：z-index 为正的定位元素，正的越大层叠等级越高；

### 总结：
1. 有层叠上下文的元素的子元素跟其他外部元素没有关联，也没有可比性
2. 层叠上下文可以嵌套
3. 不同的层叠上下文之间互相独立
4. 同一层叠上下文之间的元素按上述层叠顺序排列

### 怎么比较
两个元素比较先后顺序的时候，先看是否在同一个层叠上下文，如果是，按层叠顺序比较。如果不是，看他们的父元素层叠上下文的顺序。如果比较到后面，层叠顺序相等， 按照HTML出现的先后来判断，后出现的在上面。

## 为什么有时候人们用translate来改变位置而不是定位
改变transform或opacity不会触发浏览器重排或重绘，只会触发复合（compositions）。而改变绝对定位会触发重排，进而触发重绘和复合。

## 渲染层，复合层，层叠上下文
* 渲染层，是页面普通的文档流。绝对定位，相对定位，浮动定位虽然脱离文档流，但它仍然属于默认渲染层，共用同一个绘图上下文对象。普通文档流是由多个渲染层合成的。
* 复合层，某些特殊条件的渲染层，会被浏览器自动提升为复合层。拥有单独的绘图上下文，被GPU渲染，硬件加速。
* 层叠上下文，就是多个渲染层的一个堆叠关系。

### 渲染层 提升为 复合层
1. 一个渲染层就是上面说的一个层叠上下文元素
2. 满足下列条件会变为复合层
    * 3D transforms：translate3d、translateZ 等
    * video、canvas、iframe 等元素
    * 对 opacity、transform、fliter、backdropfilter 应用了 animation 或者 transition

## 文本显示行数控制
单行：  
```css
overflow:hidden;
text-overflow:ellipsis;
white-space:nowrap;
```
多行：  
```css
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;           // 将元素作为弹性伸缩盒子模型显示 。
-webkit-line-clamp: 2;          // 用来限制在一个块元素显示的文本的行数
-webkit-box-orient: vertical;   // 设置或检索伸缩盒对象的子元素的排列方式
```

## 长宽比解决方案
1. 垂直方向的padding。  
原理： 在CSS中padding-top或padding-bottom的百分比值是根据容器的width来计算的。
```css
div {
    width: 100%;
    padding-top: 56.25%; /* 长宽比是16：9*/
    /* 
    padding-top: calc(100% * 9 / 16)
    */
}
```
2. 视窗单位vw  
```css
div {
    width: 100vw;
    height: 56.25vw;
}
```
## css reset 和 normalize.css 有什么区别
Normalize.css是一种CSS reset的替代方案。它是一个很小的CSS文件，但它在默认的HTML元素样式上提供了跨浏览器的高度一致性。
* 保护有用的浏览器默认样式而不是完全去掉它们
* 修复浏览器自身的bug
* 一般化的样式

## 如何实现左侧宽度固定，右侧宽度自适应的布局
1. 固定宽度区浮动或者使用绝对定位，自适应区设置margin和剩余的宽度
```css
.left{
    float: left; 
    /* position: absolute; 或者绝对定位 */
    width: 100px;
    height: 100px;
    background: red;
}
.right{
    margin-left: 100px;
    width: calc(100% - 100px);
    height: 100px;
    background: green;
}
```
2. flex布局，固定宽度区用width或者flex-basis, 自适应区flex-grow=1, flex-shrink=1, flex-basis:auto
```css
.parent{
    display: flex;
    flex-direction: row;
}
.left{
    flex-basis: 100px;
    height: 100px;
    background: red;
}
.right{
    flex: 1 1 auto;
    height: 100px;
    background: green;
}
```

## Sass语法
1. 声明变量 `$`
2. css嵌套
3. 父选择器的标识符`&`，一般用于将父级直接拼在目标选择器之前，中间不加空格。
4. 导入文件 `@import`，可以省略.sass或.scss文件后缀，支持嵌套导入，即直接导入在某个css规则内。
5. 注释，`/*会出现在生成的css文件中*/   // 不会出现在生成的css文件中`
6. 混合器`@mixin`，实现大段样式的重用。用`@minxin`去定义，用`@include`调用。混合器可以传参`@mixin link-colors($normal, $hover, $visited) {...}`
7. 继承`@extend`来精简CSS。可以继承另一个类的所有样式。继承一般是基于类的。
8. sass可进行简单的加减乘除运算等。`width: 600px / 960px * 100%;`，可以使用小括号。
9. 函数`@function, @return`，前者创建函数，后者表明了函数将返回的值。`@function function-name($args) { @return value-to-be-returned; }`
10. 插值语句`#{}`可以在选择器或属性名中使用变量。`p.#{$name} { #{$attr}-color:blue; }`
11. 控制指令`@if @for @each @while`

## CSS-Sprites
CSS-sprites，又叫精灵图或者雪碧图。选择合适的图像素材去拼合到一张图片上，再利用定位将图像在网页中渲染出来。  
优点：  
将众多小图像拼合，一次性加载，减少http请求，提高网站性能  
缺点：  
不利于维护，当CSS-sprites上的一个图像需要改动时，往往需要改动其他的图像的CSS代码  
更好的方案：icon-font

## PNG,GIF,JPG的区别及如何选
* GIF:  
8 位像素，256 色  
无损压缩  
支持简单动画  
支持 boolean 透明  
适合简单动画  

* JPEG：  
颜色限于 256  
有损压缩  
不支持透明  
适合照片  

* PNG：  
无损压缩  
适合图标、背景、按钮  

    与GIF相比：  
    * 它通常会产生较小的文件大小。  
    * 它支持阿尔法（变量）透明度。  
    * 无动画支持  

    与JPEG相比：  
    * 文件更大  
    * 无损  

