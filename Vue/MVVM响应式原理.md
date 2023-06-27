# MVVM 响应式原理

## vue 的基本功能

只讨论响应式原理涉及到的部分

1. 模板可以写表达式，用{{}}包裹
2. 数据值变了，视图会更新

```html
<body>
    <div id="app">
        <h2>{{a}}</h2>
        <h2>{{b.c}}</b>
    </div>
</body>
<script>
    var vm = new MVVM({
        el: '#app',
        data: {
            a: 'test',
            b: {
                c:'test2'
            }
        }
    });
</script>
```

## 思维导图

从上面的代码分析，我们想实现这个功能，则

1. 我们需要一个类 MVVM，可以用来 new
2. 类可以传入一个对象，包含 el，data 属性
3. 我们要把 data 中的所有数据变为响应式的，才能在数据变化的时候更新视图
   - 通过 Object.defineProperty()给每个数据设置 set 和 get
   - 需要用到发布订阅模式，有发布者，订阅者和事件处理中心
     - 事件处理中心，有一个数组存储订阅者，有一个方法触发所有订阅者更新，有一个方法进行订阅者收集
     - 发布者，就是具体的数据
     - 订阅者，订阅某个数据的变化，有一个更新方法，并执行更新的回调。初始化的时候要调用数据的 get 去将本身添加到该数据的事件处理中心。
   - 绑定一个事件处理中心，set 的时候通知变化，get 的时候去收集依赖
4. 我们需要编译模板，看模板上使用了哪些 data 中的数据，这样数据变化的时候才知道更新哪里
   - 解析指令和模板，每个有数据的地方都是要变化的，所以都是一个订阅者。
   - 在订阅者初始化的时候去调用本数据的 get 方法，将订阅者的更新方法收集到本数据的事件处理中心，即为依赖收集。
   - 初始化渲染

![mvvm](https://cdn.lishuxue.site/blog/image/Vue/mvvm.png)

```js
class MVVM {
  constructor(options) {
    this.$options = options || {};
    const { el, data, watch, computed } = this.$options;
    this._data = data;
    this.observe(data); // 初始化MVVM的时候，将data变为可观察的。即通过defineProperty定义每个数据的set get
    this.compile(el); // 编译模板，解析指令与变量，初始化渲染，然后给数据添加订阅者(观察者)与更新之后的回调。
  }

  observe(data) {
    let ob = new Observer();
    ob.observe(data);
  }

  compile(el) {
    new Watcher(this, '_data.a', () => console.log('观察到数据a变化，开始更新视图'));
    new Watcher(this, '_data.b', () => console.log('观察到数据b变化，开始更新视图'));
    new Watcher(this, '_data.b.c', () => console.log('观察到数据c变化，开始更新视图'));
  }
}

class Observer {
  observe(data) {
    if (!data || typeof data !== 'object') return;
    this.walk(data);
  }

  walk(data) {
    Object.keys(data).forEach((key) => {
      this.observePropery(data, key, data[key]);
    });
  }

  observePropery(data, key, val) {
    let dep = new Dep();
    this.observe(val); // 嵌套对象递归设置属性
    Object.defineProperty(data, key, {
      configurable: false, // 不可以再次被defineProperty
      enumerable: true,
      get() {
        dep.collectWatcher(); // get方法中进行依赖收集
        return val;
      },
      set(newVal) {
        val = newVal;
        dep.notify(); // set方法中通知订阅者进行更新
      },
    });
  }
}

class Watcher {
  constructor(vm, expOrFn, cb) {
    this.vm = vm;
    this.cb = cb; // 更新回调函数
    this.exp = expOrFn; // 表达式或者方法

    this.deps = []; // 用来存储这个watcher所观察过的dep，如果这个watcher已经存在于某个dep中，就不需要重新把这个watcher加到dep上

    if (typeof expOrFn === 'function') {
      this.getter = expOrFn;
    } else {
      this.getter = this.parseGetter(expOrFn);
    }
    this.init();
  }
  init() {
    // 初始化的时候，先将Dep.target赋值为本身实例，然后调用get方法，触发依赖收集，将本身实例添加到事件中心的订阅者数组中。
    Dep.target = this;
    this.getter.call(this.vm, this.vm);
    Dep.target = null;
  }
  update() {
    this.cb.call(this.vm);
  }
  addWatcherToDep(dep) {
    // watcher已经添加给这个dep了
    let isAdded = this.deps.some((value) => {
      value.id === dep.id;
    });
    if (!isAdded) {
      this.deps.push(dep);
      dep.addSub(this);
    }
  }
  parseGetter(exp) {
    // 将表达式a.b.c循环触发get方法，来进行依赖收集
    if (/[^\w.$]/.test(exp)) return;

    let exps = exp.split('.');
    return function (obj) {
      for (let i = 0, len = exps.length; i < len; i++) {
        if (!obj) return;
        obj = obj[exps[i]];
      }
      return obj;
    };
  }
}

uid = 1;
class Dep {
  constructor() {
    this.id = uid++;
    this.subs = []; // 用于存储本数据的订阅者
  }
  addSub(sub) {
    this.subs.push(sub);
  }
  notify() {
    this.subs.forEach((sub) => {
      sub.update();
    });
  }
  collectWatcher() {
    // 依赖收集
    if (Dep.target) {
      Dep.target.addWatcherToDep(this);
    }
  }
}
Dep.target = null; // 当前数据的订阅者实例

var data = { a: 'test', b: { c: 'test2' } };
var vm = new MVVM({ data });
vm._data.a = 'ceshi'; // 观察到数据a变化，开始更新视图
```
