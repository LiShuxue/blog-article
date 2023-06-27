## 现状与需求

1. 使用的是 React 全家桶技术栈，组件库使用的 ant-design
2. ant-design 的默认样式不满足需求，所以我们需要自定义主题来覆盖默认样式
3. 所以我们需要创建一个 override 文件，来覆盖 antd 的组件的一些 less 变量
4. 此时，我们又想实现换肤功能，并且还想动态换肤
5. antd 使用的 less 作为 css 预编译器

## 解决方案探索与分析

1. 首先想到的是，两套主题文件切换
2. 所以我们可以创建多个 override 文件，每个文件都用不同的颜色，但是这样有弊端，我们的换肤不仅是针对 antd 的组件，还有很多自己的组件，页面等。
3. 所以想到了，创建多个 theme 文件，里面定义很多 less 变量，不同的 theme 定义成不同的值，但是变量名字一样，然后 override-antd 文件中引用自己定义的这些变量。
4. 理论上方法是可以的，但是在打包使用的时候有问题。因为现在用的是 less 预编译的，所以我们在每个使用这些变量的地方都要同时 import 这两个主题文件，而且后面的主题会把前面的覆盖，而且这两个主题文件也不能分开打包在不同的 css 文件里。
5. 所以目前的问题，如何将主题打包在不同的文件中，如何可以不用在任何需要的地方都 import 这些主题文件，如何动态切换。

## 解决方案

1. 主题打包在不同的文件，我们可以借助 webpack 的动态加载。import()
2. 如果不需要在每一个文件中引用这些主题文件，那第一次导入之后的这些值，就应该是全局的。
3. 所以想到了使用 css 变量，他不需要参与 less 的预编译，而且加载到浏览器就可以使用，问题是它不支持 IE。

所以，综上所述，我们需要有一个 override-antd 文件，里面包含全部的 antd 组件的变量覆盖。还需要不同的主题文件，比如 dark.css， light.less，里面包含不同的 css 变量来作为主题的变量。还需要动态导入不同的主题文件，然后使 webpack 可以分开打包。

```js
// override-antd.less
@body-background: var(--theme-color-body-background);

// dark.css
:root {
  --theme-color-body-background: black;
}

// light.css
:root {
  --theme-color-body-background: light;
}

// switchTheme.js
const changeToDark = () => {
  import(/* webpackChunkName: 'theme-dark' */ 'src/styles/theme/dark.less');
  setTheme('dark');
  localStorage.setItem('ratan-theme', 'dark');
};
const changeToLight = () => {
  import(/* webpackChunkName: 'theme-light' */ 'src/styles/theme/light.less');
  setTheme('light');
  localStorage.setItem('ratan-theme', 'light');
};
// 然后根据不同的theme state, 在useEffect中动态修改link标签的值。
useEffect(() => {
  if (theme === 'dark' && hasLightThemeFile()) {
    hideLightThemeFile();
  }
  if (theme === 'light' && hasDarkThemeFile()) {
    hideDarkThemeFile();
  }
}, [theme]);
```
