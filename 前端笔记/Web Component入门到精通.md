## Web Component

Web 组件是一种现代前端技术，用于构建可重用的、封装的自定义 HTML 元素。

它由自定义元素、Shadow DOM 和 HTML 模板等技术组成，让开发者能够将复杂的 UI 部件封装为独立的组件，并在不同项目中重复使用。

## 自定义元素（Custom Elements）

自定义元素是 Web 组件的核心。通过自定义元素，你可以创建自己的 HTML 标签，并为其定义行为和属性。

首先需要创建一个 JavaScript 类，该类继承自 HTMLElement 或其子类，然后在类中添加你想要的行为和属性。

通过 customElements.define 来注册自定义元素，就可以在页面上使用了，注册的时候要遵循 html 标签的格式，简单来说就是必须带短横线。

然后，就可以在 html 中使用该自定义元素 `<my-element></my-element>`。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Custom Element Demo</title>
  </head>
  <body>
    <my-element></my-element>

    <script>
      class MyElement extends HTMLElement {
        constructor() {
          super();
          // 在构造函数中初始化元素
          this.textContent = 'Hello, Web Component!';
        }
      }

      // 注册自定义元素
      customElements.define('my-element', MyElement);
    </script>
  </body>
</html>
```

一旦你在 html 中声明了自定义元素，你可以像使用任何其他 HTML 元素一样在页面上使用它。比如可以通过 JavaScript 来操作自定义元素。

```js
// 获取自定义元素
const myElement = document.querySelector('my-element');

// 修改元素的属性和内容
myElement.textContent = 'Updated Text';

// 添加事件监听器
myElement.addEventListener('click', () => {
  alert('Element Clicked!');
});
```

除了继承 HTMLElement，也可以继承其子类。有两种实现方式，一种是类继承，一种是 define 的时候继承。第一种最常用。

他们注册的自定义元素，使用方式也不同，第二种一般用 is 语法。

```js
// 第一种
// 类继承自 HTMLParagraphElement
class WordCount extends HTMLParagraphElement {
  constructor() {
    super();
    // ...
  }
}
customElements.define('word-count', WordCount);
// 使用标签
<word-count></word-count>;
```

```js
// 第二种
class WordCount extends HTMLElement {
  constructor() {
    super();
    // ...
  }
}

// 使用 { extends: "p" } 使它继承自p标签
customElements.define('word-count', WordCount, { extends: 'p' });
// 使用标签
<p is="word-count"></p>;
```

### 生命周期

自定义元素类是有生命周期方法的，它们允许你在元素的不同生命周期阶段执行自定义操作。

- constructor() 是自定义元素的构造函数。当自定义元素被创建时，这个方法会被调用。你可以在这里进行初始化操作，如设置初始属性、创建影子 DOM 等。

- connectedCallback() 方法在自定义元素被添加到文档中时被调用。这是一个很好的地方来执行与 DOM 相关的初始化操作，如添加事件监听器。

- disconnectedCallback() 方法在自定义元素从文档中移除时被调用。你可以在这里进行清理工作，如移除事件监听器。

- attributeChangedCallback() 方法在自定义元素的属性被添加、删除或修改时被调用。你可以在这里响应属性的变化并执行相应的操作。

在使用 attributeChangedCallback 方法时，通常需要配合 observedAttributes 属性。observedAttributes 是一个静态属性，你需要在自定义元素类中定义它，它返回一个包含需要监视的属性名称的数组。当定义了 observedAttributes 后，只有这些属性的更改会触发 attributeChangedCallback 的调用。

```js
class MyElement extends HTMLElement {
  static get observedAttributes() {
    return ['my-attribute', 'another-attribute'];
  }

  constructor() {
    super();
    // 初始化操作
  }

  connectedCallback() {
    // 当元素被添加到文档中时执行的操作
  }

  disconnectedCallback() {
    // 当元素从文档中移除时执行的操作
  }

  attributeChangedCallback(attributeName, oldValue, newValue) {
    // 当监视的属性发生变化时执行的操作
    console.log(`Attribute ${attributeName} changed from ${oldValue} to ${newValue}`);
  }
}
```

### 自定义属性

在自定义元素的类中，可以在构造方法中定义一些属性。

observedAttributes 中声明的属性不一定非要在构造函数中已经定义。你可以在 observedAttributes 中列出你希望监听的属性，而这些属性可以在构造函数中定义，也可以在后续的代码中动态添加或修改（比如直接在自定义标签中使用了该属性）。

```html
<!DOCTYPE html>
<html>
  <head> </head>
  <body>
    <my-element my-custom-attribute="test"></my-element>

    <script>
      class MyElement extends HTMLElement {
        constructor() {
          super();
          // 创建一个自定义属性
          this.myCustomAttribute = 'default value';
        }

        // 自定义元素被连接到文档时调用
        connectedCallback() {
          // 读取自定义属性的值
          const customAttributeValue = this.getAttribute('my-custom-attribute');
          if (customAttributeValue !== null) {
            this.myCustomAttribute = customAttributeValue;
          }

          // 使用自定义属性的值
          this.innerHTML = `My Custom Attribute: ${this.myCustomAttribute}`;
        }
      }

      customElements.define('my-element', MyElement);
    </script>
  </body>
</html>
```

### 自定义事件

自定义元素可以触发自己的自定义事件，以便与外部世界进行通信。你可以使用 dispatchEvent 方法触发事件，并使用 addEventListener 监听事件。

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
  }

  // 触发自定义事件
  fireEvent() {
    const event = new Event('customEvent', {
      bubbles: true, // 是否冒泡
      composed: true, // 是否穿越 Shadow DOM 边界
    });
    this.dispatchEvent(event);
  }
}
customElements.define('my-element', MyElement);

const myElement = document.querySelector('my-element');

// 监听自定义事件
myElement.addEventListener('customEvent', (event) => {
  console.log('自定义事件被触发了');
});

// 触发自定义事件
myElement.fireEvent();
```

Web 组件可以通过自定义事件来与其他组件进行通信。例如，你可以在一个组件内部触发自定义事件，然后在另一个组件中监听并做出反应。

```js
// 第一个组件，用来触发事件
class SenderElement extends HTMLElement {
  constructor() {
    super();
  }

  // 触发自定义事件
  sendCustomEvent() {
    const event = new Event('customEvent', {
      bubbles: true,
      composed: true,
    });
    this.dispatchEvent(event);
  }
}
customElements.define('sender-element', SenderElement);

// 第二个组件，用来监听事件
class ReceiverElement extends HTMLElement {
  constructor() {
    super();
    this.addEventListener('customEvent', this.handleCustomEvent.bind(this));
  }

  // 监听自定义事件
  handleCustomEvent(event) {
    console.log('自定义事件被接收到');
  }
}
customElements.define('receiver-element', ReceiverElement);

// 外部获取到这两个组件
const sender = document.querySelector('sender-element');
const receiver = document.querySelector('receiver-element');
// 触发自定义事件
sender.sendCustomEvent();
```

## 影子 DOM（Shadow DOM）

DOM 编程模型令人诟病的一个方面就是缺乏封装和隔离，不同组件之间的逻辑和样式很容易互相污染。Shadow DOM 允许你将 HTML、CSS 封装在一个独立的组件中，创建了一个隔离的作用域，使组件内部的样式和 DOM 结构不会受到外部样式和脚本的影响。这有助于防止全局作用域的命名冲突，从而提高了代码的可维护性。

定义 Shadow DOM，你需要在自定义元素的构造函数中使用 attachShadow 方法。这个方法接受一个配置对象，其中 mode 可以设置为 open 或 closed，来定义 Shadow DOM 是开放还是封闭。然后就可以将其他 DOM 元素 appendChild 到这颗 shadow tree 上面了。最终它会挂载在你的自定义元素上，以 shadow root 节点为起始根节点，下面是你添加的 shadow tree。

- 如果你希望自定义元素的内部结构和样式对外部可见和可定制，使用 open 模式。

- 如果你希望确保自定义元素的内部结构和样式不受外部影响，以保持良好的封装性，使用 closed 模式。

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    // 创建Shadow DOM
    const shadow = this.attachShadow({ mode: 'open' });

    // 在Shadow DOM中添加内容
    const paragraph = document.createElement('p');
    paragraph.textContent = 'This is Shadow DOM content.';

    // 将额外的DOM元素封装在shadow dom中
    shadow.appendChild(paragraph);
  }
}
customElements.define('my-element', MyElement);
```

在自定义元素内部，可以使用 this.shadowRoot 来获取 Shadow DOM 的根节点，然后使用标准 DOM 方法来操作它。只有在 open 模式下，才允许访问和修改 Shadow DOM 内部元素。

以下是一些常用的方法和属性：

- attachShadow(config)：在自定义元素上创建并附加 Shadow DOM，config 是一个配置对象，用于指定 mode。也可以用在普通元素上。Element.attachShadow()。

- this.shadowRoot：获取 Shadow DOM 的根节点，可以用来访问和操作 Shadow DOM 内部元素。

- shadowRoot.querySelector(selector)：在 Shadow DOM 内部查找匹配选择器的第一个元素。

- shadowRoot.querySelectorAll(selector)：在 Shadow DOM 内部查找匹配选择器的所有元素。

- shadowRoot.host：获取 Shadow DOM 的宿主元素，即拥有 Shadow DOM 的自定义元素。

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'open' });

    const paragraph = document.createElement('p');
    paragraph.textContent = 'This is Shadow DOM content.';
    shadow.appendChild(paragraph);

    // 访问Shadow DOM内部元素
    const p = this.shadowRoot.querySelector('p');
    // 修改Shadow DOM内部元素
    p.style.color = 'blue';
  }
}
customElements.define('my-element', MyElement);
```

## HTML 模板（HTML Templates）

使用 `<template>` 元素来创建 HTML 模板。其中可以包含任意的 HTML 结构、文本和变量占位符。此元素及其内容不会在 DOM 中立即呈现，直到你用 JavaScript 来克隆并插入到 Shadow DOM 中。

建议使用 `template.cloneNode` 来克隆模板内容，不要直接 `shadow.appendChild(template.content)`。尤其是当你的组件需要复杂的模板结构或者需要在不同的上下文中使用相同的模板时。这样可以更安全地操作模板内容，而不会影响原始文档或其他组件。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>HTML Template Demo</title>
  </head>
  <body>
    <my-element></my-element>

    <template id="my-template">
      <p>Hello, Web Component!</p>
    </template>

    <script>
      class MyElement extends HTMLElement {
        constructor() {
          super();
          const shadow = this.attachShadow({ mode: 'open' });

          // 使用HTML模板
          const template = document.getElementById('my-template');
          const clone = template.cloneNode(true);

          shadow.appendChild(clone);
        }
      }
      customElements.define('my-element', MyElement);
    </script>
  </body>
</html>
```

### 插槽（slot）

插槽是一种用于将外部内容插入到组件内部的机制。插槽允许你在组件内定义可插入的区域，使外部内容能够填充到这些区域中。这样，你可以在组件内部自由组合和分发内容。

同 Vue 一样，有默认插槽和命名插槽。

```html
<!-- 自定义元素中定义插槽的位置 -->
<template id="my-template">
  <p>我是自定义元素内容</p>
  <p>后面展示的是插槽的内容：<slot name="content"></slot></p>
</template>

<!-- 使用插槽 -->
<my-element>
  <span slot="content">我会放在插槽里</span>
</my-element>
```

### 使用 CSS

在 Shadow DOM 内部，你可以使用普通的 CSS 样式表来定义组件的样式。这些样式将局部作用于 Shadow DOM 内部，不会影响全局样式。

需要创建一个 style 标签，将标签添加到 shadow dom 中。

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'open' });

    // 创建一个 style 元素并添加样式
    const style = document.createElement('style');
    style.textContent = `
      h1 {
        color: blue;
      }
    `;

    // 将 style 元素添加到 Shadow DOM 中
    shadow.appendChild(style);

    // 在 Shadow DOM 内部创建 DOM 结构
    const heading = document.createElement('h1');
    heading.textContent = 'Hello from Shadow DOM!';
    shadow.appendChild(heading);
  }
}
customElements.define('my-element', MyElement);
```

如果需要在 Shadow DOM 中使用外部样式，可以将样式表链接或内联样式添加到 Shadow DOM 中。

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'open' });

    // 添加外部样式表链接
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = 'external-styles.css'; // 外部样式表文件路径
    shadow.appendChild(link);

    // 在 Shadow DOM 内部创建 DOM 结构
    const heading = document.createElement('h1');
    heading.textContent = 'Hello with External Styles!';
    shadow.appendChild(heading);
  }
}
customElements.define('my-element', MyElement);
```

如果是在模版中使用，基本跟上面一样，可以通过 style 标签或者 link 标签，也可以用内联样式。

```html
<!-- 内联样式 -->
<template id="my-template">
  <div style="color: blue; font-size: 18px;">This is a blue text with 18px font size.</div>
</template>

<!-- style标签 -->
<template id="my-template">
  <style>
    div {
      color: blue;
      font-size: 18px;
    }
  </style>
  <div>This is a blue text with 18px font size.</div>
</template>

<!-- 外部样式表 -->
<template id="my-template">
  <link rel="stylesheet" type="text/css" href="my-styles.css" />
  <div class="my-element">This element uses an external CSS file.</div>
</template>
```
