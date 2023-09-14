# React 中使用 CSS

## 基本使用

为组件添加 class 样式，传递一个字符串作为 className 属性：

```jsx
render() {
  return <span className="menu navigation-menu">Menu</span>
}
```

有时候 class 依赖 state 或者 props，如

```jsx
render() {
  let className = 'menu';
  if (this.props.isActive) {
    className += ' menu-active';
  }
  return <span className={className}>Menu</span>
}
```

有时候会有很多使用 js 来动态判断是否为组件添加 class，可以用**classnames**库去简化它。

```jsx
import classnames from 'classnames';
<Button className={classnames(
    {
      base: true,
      isActive: this.props.isActive,
      'is-error': this.props.isActive
    },
    'style1',
    'style2')}>
<Button/>
```

也可以使用 style 属性。style 接受一个采用小驼峰命名属性的 JavaScript 对象，而不是 CSS 字符串。

1. 用双大括号
2. 驼峰命名，如 backgroundColor

## 行内样式

```jsx
return <div style={{ width: '200px', height: '200px' }}></div>;
```

## 内联样式

```jsx
const style={
    width:'200px',
    height:'200px'
}
render() {
    return (
        <div style={style}>内联样式</div>
    )
}
```

通常不推荐将 style 属性作为设置元素样式的主要方式。在多数情况下，应使用 className 属性来引用外部 CSS 样式表中定义的 class。style 在 React 应用中多用于在渲染过程中添加动态计算的样式。

## 外联样式

在当前组件开头使用 import 引入 css 文件(或者 sass 文件)，然后用 className 使用。这种方式引入的 css 样式，会作用于当前组件及其所有后代组件。所以会造成 css 污染。

可以确定好命名空间，使用 BEM 来减小这种方式的影响。

```jsx
import './test.css';

<div className="test">123</div>;
```

## CSS Module

将 css 文件(或者 sass 文件)作为一个模块引入，这个模块中的所有 css，只作用于当前组件。不会影响当前组件的后代组件。

使用 [name].module.css 命名。（create-react-app 已经默认配置了）

```jsx
import './test.module.css';

<div className="test">123</div>;
```

## CSS in JS => styled-components

结合**styled-components**库，可以定义一个带有样式的元素。然后在 jsx 中使用。

CSS in JS 可以和 JS 共享变量。

```jsx
const Button = styled.button`
  background: ${(props) => (props.primary ? 'palevioletred' : 'white')};
  color: ${(props) => (props.primary ? 'white' : 'palevioletred')};
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;
render(
  <div>
    <Button>Normal</Button>
    <Button primary={true}>Primary</Button>
  </div>
);
```
