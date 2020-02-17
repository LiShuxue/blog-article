React使用Jest作为测试框架，但是在React中是如何使用的，如何测试React组件呢？下面给出了一些常用的测试方法，并给出了我们推荐的做法。

## Smoke Test 冒烟测试
这一术语源自硬件行业，对一个硬件或硬件组件进行更改或修复后，直接给设备加电。如果没有冒烟，则该组件就通过了测试。

在React中，主要代表，如果组件渲染没有出错，即测试通过。这是一种比较简单的测试方法。
```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

it('renders without crashing', () => {
  const div = document.createElement('div');
  ReactDOM.render(<App />, div); // 如果这里没有报错就代表测试通过
});
```
---

## Shallow Rendering 浅渲染
浅渲染是指渲染组件的第一层，不包含任何子组件的渲染。

浅渲染对分离的单元测试非常有用。只测试自己组件本身的功能，子组件由他自己的测试文件测试。

浅渲染 功能由Enzyme框架提供。

## Enzyme
### 1. 安装
`yarn add enzyme enzyme-adapter-react-16 react-test-renderer`

### 2. 配置
Enzyme需要对你使用的react版本配置适配器，在你的src/setupTests.js中
```js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
configure({ adapter: new Adapter() });
```

### 3. 使用
* 通过shallow方法实现浅渲染
```js
import React from 'react';
import { shallow } from 'enzyme';
import App from './App';

it('renders without crashing', () => {
    const wrapper = shallow(<App />); // 只渲染App组件本身，子组件不会渲染，即使子组件渲染报错，这个测试也会通过
});
```

* 通过mount方法实现完整渲染  
有时候我们想测试完全渲染的组件，包含任何子组件
```js
import React from 'react';
import { shallow } from 'enzyme';
import App from './App';

it('renders without crashing', () => {
    const wrapper = mount(<App />);
});
```

* API  
Enzyme通过模仿JQuery的api来操作DOM元素，更多API参考：https://airbnb.io/enzyme/docs/api/
```js
// .contains 判断是否包含某个DOM元素
it('renders welcome message', () => {
  const wrapper = shallow(<App />);
  const welcome = <h2>Welcome to React</h2>;
  expect(wrapper.contains(welcome)).toEqual(true);
});

// .find 根据选择器返回DOM节点
it('renders welcome message', () => {
  const wrapper = shallow(<App />);
  let node = wrapper.find('.foo'); // find的参数是css选择器
  let components = wrapper.find(Foo); // find的参数是React组件
  let components = wrapper.find('MyComponent'); // find的参数是React组件的名字
});
```

* jest-enzyme 可以提供更多的匹配器来提升你的测试
---

## React-Testing-Library
> react-testing-library是可以取代Enzyme的，我们推荐使用这个库测试React组件，create-react-app创建的app默认集成了这个库

react-testing-library是一个库，用于以类似于最终用户使用组件的方式来测试React组件。   
它非常适合于React组件和应用程序的单元，集成和端到端测试。   
它的工作方式更接近DOM的操作，因此建议与jest-dom一起使用以改进断言。

它可以通过label文本查找DOM元素，查找link和button等所有DOM操作。

它基于DOM Testing Library来处理DOM相关的测试。参考 https://testing-library.com/docs/dom-testing-library/intro

### 安装
`yarn add @testing-library/react @testing-library/jest-dom`

### 配置  
在你的src/setupTests.js中，导入jest-dom对Jest的扩展匹配器，以实现更多功能  
`import '@testing-library/jest-dom/extend-expect';`

### 使用
```js
import React from 'react';
import { render } from '@testing-library/react';
import App from './App';

it('renders welcome message', () => {
  const { getByText } = render(<App />);
  expect(getByText('Learn React')).toBeInTheDocument();
});
```

### <b>render方法</b>
render方法将React元素呈现到DOM中。将创建一个div并将该div附加到document.body，然后渲染React组件到这个div。更多详情参考： https://testing-library.com/docs/react-testing-library/api#render

并且返回一系列查询函数，供你查询DOM节点。如
* `getBy*** (ByText, ByTitle, ByRole, ByDisplayValue...等等)` 返回第一个匹配的DOM节点，如果没有匹配的就抛错
* `getAllBy*** (ByText, ByTitle, ByRole, ByDisplayValue...等等)` 返回所有匹配的DOM节点，以数组形式返回，如果没有匹配的就抛错
* `queryBy*** (ByText, ByTitle, ByRole, ByDisplayValue...等等)` 返回第一个匹配的DOM节点，如果没有匹配到就返回null，如果匹配到多个就抛错，适用于断言一个元素没有呈现
* `queryAllBy*** (ByText, ByTitle, ByRole, ByDisplayValue...等等)` 返回所有匹配的DOM节点，如果没有匹配到就返回[]数组
* `findBy*** (ByText, ByTitle, ByRole, ByDisplayValue...等等)` 返回一个Promise，如果有一个匹配到就resolve，没有匹配或者匹配了多个就reject
* `findAllBy*** (ByText, ByTitle, ByRole, ByDisplayValue...等等)` 返回一个Promise，如果匹配到DOM节点就resolve，如果没有匹配到任何节点就reject
* 更多详情参考：https://testing-library.com/docs/dom-testing-library/api-queries
```js
const { getByText, getByRole } = render(<App />);
```

还可以返回：
* container React组件所渲染在哪个节点，默认是div
* baseElement React组件所渲染的节点所在的容器，默认是body
* debug() 该方法可以打印你的DOM信息，相当于console.log()。不传参打印所有的，传参只打印你所传的
* rerender，跟render方法相同，可以渲染组件。一般用来测试组件的props update
* unmount，卸载你渲染的组件
* asFragment相当于获取当前的快照，一般用来测试你的组件响应一些事件后的变化，可以通过库snapshot-diff去比较两个快照，https://github.com/jest-community/snapshot-diff
```js
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import App from './App';

it('test renders', () => {
    const { 
        getByText, 
        container, 
        baseElement, 
        debug, 
        rerender, 
        asFragment } = render(<App number={1} />);

    const node = getByText('test'); // 获取节点
    debug(); // 打印整个组件渲染的信息
    debug(node); // 打印这个节点的信息

    rerender(<App number={2} />); // 传递不同的props，重新渲染

    unmount(); // 卸载渲染的组件

    const currentUI = asFragment(); // 获取当前渲染的信息
    fireEvent.click(node); // 触发一个点击事件
    const afterClickUI = asFragment(); // 再次获取当前渲染的信息
    expect(currentUI).toMatchDiffSnapshot(afterClickUI); // https://github.com/jest-community/snapshot-diff
});
```

### <b>fireEvent方法</b>
fireEvent方法允许你触发一个事件，模拟用户操作，比如click, change, keyDown等，更多支持的事件参考：https://github.com/testing-library/dom-testing-library/blob/master/src/events.js
```js
fireEvent.click(getByText('Submit')) // 左键点击
fireEvent.click(getByText('Submit'), { button: 2 }) // 右键点击
```

### <b>获取异步渲染组件 wait, waitForElement, waitForDomChange</b>
有时候我们某些组件是异步渲染的，可以用await...等待DOM元素出现或者消失。
```js
test('test async', async () => {
    await wait(() => getByLabelText('username'));
    await waitForElement(() => getByLabelText('username'));

    const container = document.createElement('div');
    waitForDomChange({ container }).then(() => console.log('DOM changed!'));

    waitForElementToBeRemoved(() => document.querySelector('div.getOuttaHere'))
    .then(() => console.log('Element no longer in DOM'));
});
```

### <b>匹配器</b>
Jest本身提供了很多的匹配器，但是用来判断DOM的场景不是很适用，所以通过jest-dom扩展了很多关于DOM的匹配器，如
* toBeDisabled 判断元素是否是disabled
* toBeEmpty 判断元素是否有内容
* toBeInTheDocument 判断某元素是否在文档内
* toBeVisible 判断元素是否可见
* toContainHTML 判断是否包含某个元素
* toHaveAttribute 是否有某个属性
* toHaveClass 是否有某个class
* toHaveStyle 是否有某个具体的css样式
* 更多匹配器参考：https://github.com/testing-library/jest-dom#custom-matchers  

### <b>自定义配置</b>
自定义render和queries，参考：https://testing-library.com/docs/react-testing-library/setup  

---

## 快照测试
每当你想要确保你的UI不会有意外的改变，快照测试是非常有用的工具。快照代表某一时刻(UI)的状态。

比如当我们呈现了一个UI组件，拍了快照，然后将其与组件旁边存储的参考快照文件进行比对。如果两个快照不匹配，则测试将失败。可能是意外的更改导致的，或者是参考快照需要根据最新的渲染去更新。

### 创建快照，验证快照 
`.toMatchSnapshot()` 是一个匹配器，可以用来验证快照是否一致，但是也可以创建快照，当最开始没有快照的时候。

但是我们怎么样创建一个UI组件的快照呢，我们知道React会将JSX转化成一个js对象，即虚拟DOM节点，包含type, props，children等。我们可以用这个虚拟DOM对象创建快照。

这时我们就需要借助`react-test-renderer`这个库，他提供了一个create方法，我们可以传递一个React组件，返回一个渲染器。再通过toJSON方法把组件渲染成js对象（虚拟DOM），这个对象就包含render tree的信息。
```js
import renderer from 'react-test-renderer';

it('renders app correctly', () => {
  const testRenderer = renderer.create(<App />)
  const dom = testRenderer.toJSON();
  // const dom = renderer.tomachcreate(<App />).toJSON();
  expect(dom).toMatchSnapshot();
});
```
toMatchSnapshot会根据这个虚拟DOM的信息创建一个快照文件，包含DOM节点的层级关系。

第一次运行测试文件的时候，会创建快照。第二次运行的时候，会比较是否与之前保存的快照文件一致。如果不匹配，则测试将失败。可能是意外的更改导致的，或者是参考快照需要根据最新的渲染去更新。

### 更新快照
`jest --updateSnapshot` 更新所有失败的快照，可以传递`--testNamePattern` 来更新名字匹配的快照。

或者是在watch mode下交互式的更新。根据命令行提示执行操作。

### 属性匹配器
快照测试并不只是可以用于UI组件，还可以用于任何可以序列化的值。目的是测试输出是否正确。

当我们为某个想要测试的 js 对象创建快照时，有可能这个对象包含了例如 `new Date()`这样的属性值，这样的话创建的快照肯定是当时的日期，以后每次验证都不对。这时我们就需要用到属性匹配器`expect.any(...)`
```js
it('will check the matchers and pass', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  // 如果我们不指明这两个属性的话，他自动创建出来的快照，以后每次都会失败。其余没指明的属性会自动保存进快照。
  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    id: expect.any(Number)
  }); 
});
```

### 最佳实践
* 快照文件会自动生成，保存在`__snapshots__`文件夹下，这个文件夹会跟你的xxx.test.js文件同级。推荐将它们放在`__tests__`文件夹下。
* 快照文件也应该提交进repo，并且跟代码一样需要被review
* 测试应该是确定性的，即运行多次结果都应该不变。
* 快照测试的名字也应该是明确的，这会让reviewer清楚的知道你这个快照文件是存的什么，利于verify

> 用了快照测试还需要单元测试吗？  
快照测试只是Jest附带的20多个断言之一。   
快照测试的目的不是要替换现有的单元测试，而是要提供附加的价值并使测试变得轻松。   
在某些情况下，快照测试可能会消除对特定功能集（例如React组件）进行单元测试的需求，但它们也可以一起工作。

## React 测试相关的全局设置
如果你使用了浏览器的API，你想要在测试文件中mock它，或者你想添加一些全局的配置，你可以加在src/setupTests.js文件中，他会在开始执行测试用例之前自动执行。

```js
// src/setupTests.js
const localStorageMock = {
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
  clear: jest.fn(),
};
global.localStorage = localStorageMock;
```