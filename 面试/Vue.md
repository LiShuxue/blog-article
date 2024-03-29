## vue 生命周期

- 创建前/后, 载入前/后, 更新前/后, 销毁前/后  
  beforeCreate/created, beforeMount/mounted, beforeUpdate/updated, beforeDestroy/destroyed
- keep-alive 组件激活时 activated，keep-alive 组件停用时 deactivated

- errorCaptured， 组件级错误监控，监控后代组件的错误。监控不到事件中的和异步中的错误

- errorHandler，全局错误处理函数，可以监控生命周期，vue 自定义事件，v-on Dom 事件，如果生命周期或者事件是 Promise，可以监控到，如果是其他的异步，监控不到。

## 第一次页面加载会触发哪几个钩子？

第一次页面加载时会触发 beforeCreate, created, beforeMount, mounted 这四个钩子

## DOM 渲染在哪个周期中就已经完成？

在 mounted 中就已经完成了

## 每个生命周期适合哪些场景？

- beforecreate : 此时 data 和 method 都还没初始化，可以在这加个 loading 事件，在加载实例时触发，在 created 时进行移除。
- created : 初始化完成时的事件写在这里，需要异步请求数据的方法可以在此时执行，完成数据的初始化。
- mounted : 当需要操作 dom 的时候执行
- updated : 当数据更新要做统一业务处理的时候

## Vue 的父组件和子组件生命周期钩子函数执行顺序

组件的调用顺序都是先父后子

加载渲染：

- 父 beforeCreate --> 父 created --> 父 beforeMounted --> 子 beforeCreate --> 子 created --> 子 beforeMounted --> 子 mounted -->父 mounted

父组件更新：

- 影响到子组件： 父 beforeUpdate -> 子 beforeUpdate->子 updated -> 父 updted
- 不影响子组件： 父 beforeUpdate -> 父 updated

子组件更新：

- 父 beforeUpdate --> 子 beforeUpdate --> 子 updated --> 父 updated

父组件销毁：

- 父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

子组件销毁：

- 子 beforeDestroy -> 子 destroyed

## beforeCreate 的时候能拿到 Vue 实例么

可以拿到组件实例，也就是可以用 this。但是不能获取到 data 和 methods。如果想强行获取 data，可以用 this.$options.data()

## 什么情况下会触发组件销毁，销毁的时候会卸载自定义事件和原生事件么

- 页面关闭
- 路由切换（没有 keep-alive 时）
- v-if

Vue 本身的一些自定义事件监听，比如@click, @blur 等会自动销毁，但是原生的 document.addEventListener 事件比如 scroll，keydown，keyup，vue 监测不到，无法移除监听，你可以自己销毁。

其实组件销毁后 DOM 元素也被移除，根据 JS 的垃圾回收机制，节点的销毁，会顺带把该节点所有的监听事件置空，所以绑定在 DOM 元素上的事件自然也就被移除了。但是绑定的是 document 的话，因为 document 对象没有被清楚，所以事件没有被卸载。

## 哪个生命周期调用异步请求

created、beforeMount、mounted 中都可以，因为此时 data 已经创建。一般在 created 中。

## 父组件怎么监听子组件的生命周期

1. 子组件生命周期中 emit，在子组件上监听，并触发父组件的方法。

   ```js
   // Parent.vue
   <Child @mounted="doSomething"/>

   // Child.vue
   mounted() {
       this.$emit("mounted");
   }
   ```

2. 使用@hook 监听

   ```js
   //  Parent.vue
   <Child @hook:mounted="doSomething" ></Child>

   //  Child.vue
   mounted(){
       console.log('子组件触发 mounted 钩子函数 ...');
   }
   ```

## v-show 与 v-if 区别

1. v-show 是 css 切换,display:none。
2. v-if 是完整的销毁和重新创建

频繁切换时用 v-show，运行时较少改变时用 v-if

## computed watch

computed 和 watch 都起到监听/依赖一个数据，并执行相应操作

1. computed：
   - 计算属性，值会缓存，下次使用此属性，直接使用缓存的值。它依赖的属性值变化后，此值会重新计算。
   - 可以依赖多个属性
   - 需要有 return
   - 不支持异步
2. watch：
   - 数据的监听，监听的数据变化后，回调函数会执行。页面重新渲染时值不变化也会执行。
   - 支持异步
   - immediate：watch 默认在数据从无到有的过程是不进行监听的，如果需要监听这个过程，可将 immediate 设置为 true；
   - deep：若对象没有改变，但是对象内部的属性改变了，需要监听此变化就将 deep 设置为 true。

## 常用指令有哪些

`v-if v-else v-for v-show v-modal`

## 绑定 class 的方法

```html
<!-- 对象语法，直接传给class一个对象 -->
<p v-bind:class="{ active: isActive, test1: isActive}">通过对象语法绑定class</p>
<!-- 对象语法，传给class一个计算属性，这个计算属性会返回一个对象 -->
<p v-bind:class="classObject">通过对象语法绑定class</p>

<!-- 数组语法，直接传给class一个数组，数组里的每个值都是data值，代表一个class名 -->
<p v-bind:class="[className1, className2]">通过数组语法绑定class</p>
<!-- 数组语法，三元表达式 -->
<p v-bind:class="[isActive ? className1 : className2]">通过数组语法绑定class</p>
```

## 动态组件

is: 后面指定一个组件的名字

```js
<component v-bind:is="currentTabComponent"></component>
```

## 组件通信

1. props/$emit 父子通信  
   如果子组件需要修改父组件传进来的 props，可以用.sync 来减少父组件监听和修改的代码

   ```js
   // before
   // 父组件中
   <child :val="name" @update="modify">
   modify(newVal){
     this.name=newVal
   }
   // 子组件中
   <input :value=val @input="$emit('update', $event.target.value)"/>

   // after
   // 父组件中，省略了写监听函数
   <child :val.sync="name">
   // 子组件中
   <input :value=val @input="$emit('update:val', $event.target.value)"/>
   ```

2. 使用 vuex 父子，兄弟，隔代都可以用
3. EventBus 父子，兄弟，隔代都可以用

   ```js
   const EventBus = new Vue();
   EventBus.$emit('aMsg', '来自A页面的消息');
   EventBus.$on('aMsg', (msg) => {
     this.msg = msg;
   });
   ```

4. provide/inject 隔代通信  
   祖先组件中通过 provide 来提供变量，然后在子孙组件中通过 inject 来注入变量

   ```js
   // 祖先
   provide: {
     test: 'demo';
   }
   // 子孙
   inject: ['test'];
   ```

5. `$attrs/$listeners` 隔代通信

   - $attrs：当子组件的props中没有声明父组件传下来的prop属性时，那么父组件传下来的prop属性会被保存在子组件的$attrs 属性上( class 和 style 除外 )。  
     子组件加了 inheritAttrs:false，DOM 上就不会继承未声明的 props。

     ````js
     // 父
     <child :foo="foo" :coo="coo"></child>

         // 子，继承了所有属性并传给下一代
         <p>attrs:{{$attrs}}</p>
         <grandChild v-bind="$attrs"></grandChild>

         // 孙子，只声明了coo，并设置inheritAttrs: false表示不继承
         props:["coo"],
         inheritAttrs:false
         <p>coo:{{coo}}</p>
         ```

     ````

   - $listeners：包含了父作用域中的 (不含 .native 修饰器的)  v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件

     ```js
     // 父
     <child @parentTest="parentTestMethod"></child>
     parentTestMethod(value){
         console.log(value)
     }

     // 子，相当于也监听了事件
     <grandChild v-on="$listeners"></grandChild>

     // 孙子
     <button @click="test">我要发射火箭</button>
     test(){
         this.$emit("parentTest",'test');
     }
     ```

6. ref 与 $parent / $children 父子通信
   - 使用 this.$parent 查找当前组件的父组件实例。
   - 使用 this.$children 查找当前组件的直接子组件，可以遍历全部子组件，需要注意 $children 并不保证顺序，也不是响应式的。
   - 使用 this.$refs 查找命名子组件。
   - 使用 this.$root 查找根组件，并可以配合$children 遍历全部组件。

## provide/inject 实现原理

通过原型链实现的传参。

- provides

  实例上挂载的 provides 要么是直接拿的父实例上的 provides，要么是创建的一个原型指向父实例 provides 的对象。

- inject

  若当前实例没有父实例则取根实例上的 provides 否则取父实例上的 provides。
  拿到 provides 后，会遍历原型上的属性去取内容。

## 自定义组件实现双向绑定 v-model

子组件中需要定义 value props, 同时定义 data innerValue, 作为子组件内部的表单组件的绑定值。

初始化设置 innerValue = value，同时 watch value，给 innerValue 赋值

innerValue 变化后，要触发 input 事件。可以通过 watch innerValue 或者子组件内部组件的 change 事件。

```js
// 子组件
<template>
  <div>
      <el-input v-model="innerValue"/>
      {{value}}
  </div>
</template>

<script>
export default {
    props: {
        value: {
            type: String
        }
    },
    data() {
        return {
            innerValue: this.value
        }
    },
    watch: {
        value(val) {
            this.innerValue = val;
        },
        innerValue(value) {
            this.$emit('input', value);
        }
    }
}
</script>
```

## template 和 jsx 的优缺点

### template

优点：

- 基于 dom 结构，方便，易读，易上手，学习成本低

缺点：

- 不够灵活，只能基于提供的指令去写逻辑

### jsx

优点：

- 基于 js 语法表达各种逻辑，十分灵活

缺点：

- 可读性差，容易写的很乱
- 没有编译优化

## 单向数据流

1. 数据只能从父组件流向子组件，props 向下传，emit 触发事件，父组件中去更新 data。子组件修改 props, vue 将会报错。
2. vuex 的单向数据流，状态只能通过提交 mutation 去更改。

## v-for 循环为什么要加 key

使用 v-for 更新已渲染的元素列表时,默认用”就地复用“策略;列表数据修改的时候,他会根据 key 值去判断某个值是否修改,如果修改,则重新渲染这一项,否则复用之前的元素。

从原理上来说，使用 key 来给每个节点做一个唯一标识，Diff 算法就可以正确的识别此节点，key 的作用主要是为了高效的更新虚拟 DOM。

## key 可以是 index 吗，可以是随机数吗？会有什么问题？

判断两个节点是否为同一节点（也就是是否可复用），标准是 key 相同且 tag 相同。

如果循环渲染某个数组，数组项不变的话，用 index 没问题，数组项改变的话，用 index 可能会导致下列不必要的更新。如果只是最后 push 不会有问题。

1. 用 index 作为 key，如果数组元素都变了，比如 arr.reverse()，所有列表项会重新创建渲染，不会复用。
2. 用 index 作为 key，如果删除了数组第一个元素，虚拟 diff 的结果会是删除了最后一个元素，其他元素内容改变，导致所有列表项会重新创建渲染，不会复用。
3. 用 index 作为 key 时，在对数据进行，逆序添加，逆序删除等破坏顺序的操作时，会产生没必要的真实 DOM 更新，从而导致效率低。
4. 用随机数作为 key，所有列表项会重新创建渲染，不会复用。

## v-for 为什么不能跟 v-if 一起使用

v-for 优先级高于 v-if，这意味着 v-if 将分别重复运行于每个 v-for 循环中，效率比较低。  
推荐将 v-if 移到父元素

## 组件中的 data 为什么是函数而不是对象

一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝。

如果 data 是一个纯碎的对象,则所有的实例将共享引用同一份 data 数据对象,无论在哪个组件实例中修改 data,都会影响到所有的组件实例

如果 data 是函数,每次创建一个新实例后,调用 data 函数,从而返回初始数据的一个全新副本数据对象

## Vue 的数据为什么频繁变化但只会更新一次 DOM

1：监听到数据变化

2：开启一个变化后数据的队列

3：在同一事件循环中缓冲所有数据改变

4：队列去重重复的 Watcher id，使其只更新一次

Vue 的 dom 更新是异步的，当数据发生变化，vue 并不是直接去更新 dom，而是开启一个队列。跟 JavaScript 原生的同步任务和异步任务相同。

现在有这样的一种情况，mounted 的时候 test 的值会被循环执行++1000 次。 每次++时，都会根据响应式触发 setter->Dep->Watcher->update->run。 如果这时候没有异步更新视图，那么每次++都会直接操作 DOM 更新视图，这是非常消耗性能的。 所以 Vue 实现了一个 queue 队列，在下一个 tick（或者是当前 tick 的微任务阶段）统一执行 queue 中 Watcher 的 run。同时，拥有相同 id 的 Watcher 不会被重复加入到该 queue 中去，所以不会执行 1000 次 Watcher 的 run。最终更新视图只会直接将 test 对的 DOM 的 0 变成 1000。 保证更新视图操作 DOM 的动作是在当前栈执行完以后下一个 tick（或者是当前 tick 的微任务阶段）的时候调用，大大优化了性能。

## Vue 框架怎么实现对象和数组的监听？

- 通过递归遍历对象，利用 Object.defineProperty() 也能对对象进行监听
- 通过重写数组的方法， push, pop....实现对数组的监听。

## 直接给一个数组项赋值，Vue 能检测到变化吗？怎么解决？

不能检测到以下变化：  
当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`  
当你修改数组的长度时，例如：`vm.items.length = newLength`

解决方法：

- 解决第一种问题：  
  `vm.$set(vm.items, indexOfItem, newValue)`  
  `vm.items.splice(indexOfItem, 1, newValue)`
- 解决第二种问题：  
  `vm.items.splice(newLength)`

## Vue 如何实现的数组的监听，为什么 Vue 没有对数组下标修改做劫持

重写数组的方法。不对数组下标做劫持就是因为性能问题。

初始化 data 时数组的元素，如果是对象，也会监听。如果是普通数组，需要用提供的数组方法或者 this.$set 才会变。

## v-model 的原理

v-modal 只是一个语法糖，相当于执行了两步：

1. 在组件上面定义一个名为 value 的 props
2. 组件内部数据的变化，触发 input 事件更新绑定的值。

```js
// 自定义组件
<child v-model="msg">
// 相当于
<child :value="msg" @input="msg = $event"/>
// 规定组件上的 v-model 默认会利用名为 value 的 prop 和名为 input 的事件，所以自定义组件为：
<template>
    <div class="text">
        <input type="text" :value="value" @input="changeVal">
    </div>
</template>
<script>
export default {
  props: {
    value,
  },
  methods:{
      changeVal(e){
          this.$emit('input', e.target.value)
      }
  }
}
</script>
```

## nextTick()

获取更新后的 DOM。

因为数据更新时，并不会立即更新 DOM。如果在更新数据之后的代码执行 DOM 操作，有可能达不到预想效果。

```js
this.msg = 'hello';
// 此时dom还没有更新
this.$nextTick(() => {
  // 此时dom已经更新
  console.log(this.$refs.msg.innerHTML);
});
```

## nextTick 底层是怎么实现的？跟事件循环是什么关系

nextTick 会接收回调函数，先放入一个数组中，然后定义一个批次清空数组的函数。这个批次清空的函数会被放入浏览器的微任务队列中。

将这个函数放入微任务或则宏任务队列，主要借助 Promise.resolve().then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0)。

当微任务执行到这个方法时，就会执行所有的 nextTick 接收的回调函数。

```js
// 模拟 nextTick
let callbacks = [];
let pending = false;
let timerFunc;

// 当微任务或者宏任务队列，执行到这个方法时，会清空所有的回调
const flushCallbacks = () => {
  pending = false;
  const copy = callbacks.slice(0);
  callbacks.length = 0;
  for (let i = 0; i < copy.length; i++) {
    copy[i]();
  }
};

if (typeof Promise === 'function' && /native code/.test(Promise.toString())) {
  // 原生有Promise，用then的微任务执行
  timerFunc = () => {
    Promise.resolve().then(flushCallbacks); // Promise.resolve()会立即执行，将then的回调放入微任务队列
  };
} else if (
  typeof MutationObserver === 'function' &&
  /native code/.test(MutationObserver.toString())
) {
  // 没有Promise就用MutationObserver，也是微任务
  const observer = new MutationObserver(flushCallbacks); // 监控DOM节点的变化，监听到变化后的将回调放入微任务队列
  const textNode = document.createTextNode(String(counter)); // 创建一个需要监听的节点
  observer.observe(textNode, {
    // 监控这个节点
    characterData: true,
  });
  let counter = 0;
  timerFunc = () => {
    counter = (counter + 1) % 2; // 主动触发变化，然后会将回调的flushCallbacks放入微任务队列
    textNode.data = String(counter);
  };
} else {
  // 实在不行就用setTimeout，宏任务
  timerFunc = setTimeout(flushCallbacks, 0);
}

const nextTick = (cb, ctx) => {
  if (cb) {
    // push一个函数，函数中再去执行这个真正的回调cb
    callbacks.push(() => {
      cb.call(ctx);
    });
  }
  if (!pending) {
    pending = true;
    timerFunc();
  }
};

window.$nextTick = function (cb) {
  nextTick(cb, this);
};
```

## 清空 nextTick 收集的回调时，为啥会拿到更新后的 Dom 呢

首先明确首先是数据变化，引起的 DOM 更新。说数据改变之后，dom 不是同步改变的，所以我们不能直接拿到最新的 dom。如果想拿，需要使用 nextTick。

1. 修改响应式数据
2. 触发 Object.defineProperty 中的 set
3. 发布通知
4. 触发 Watcher 中的 update 方法，
5. update 方法中把 Watcher 缓冲到一个队列
6. 刷新队列的方法(其实就是更新 dom 的方法)传到 nextTick 方法中
7. nextTick 方法中把传进来的 callback 都放在一个数组 callbacks 中，然后放在异步队列中去执行

然后这时你又调用了 $nextTick 方法，传进来一个获取最新 dom 的回调，这个回调也会推到那个数组 callbacks 中，此时遍历 callbacks 并执行所有回调的动作已经放到了异步队列中，到这（假设你后面没有其他的代码了）所有的同步代码就执行完了，然后开始执行异步队列中的任务，更新 dom 的方法是最先被推进去的，所以就先执行，你传进来的获取最新 dom 的回调是最后传进来的所以最后执行，显而易见，当执行到你的回调的时候，前面更新 dom 的动作都已经完成了，所以现在你的回调就能获取到最新的 dom 了。

## keep-alive

- `<keep-alive>` 是 Vue 内置的一个组件，可以使被包含的组件保留状态，避免重新渲染。

- `<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。

- 它是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

- 当组件在 `<keep-alive>` 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。

## 自定义指令

一般自定义指令解决的问题或者说使用场景是对普通 DOM 元素进行底层操作。

### 全局自定义指令

`Vue.directive('test', hookOptions)`

### 局部自定义指令

在组件的 directives 选项中进行声明。

```js
directives: {
    test: {
        bind() {

        },
        inserted() {

        },
        update() {

        }
    }
}
```

上面的 hookOptions 用来定义指令的行为。有 bind, inserted, update, componentUpdated, unbind 共 5 个 hook 函数。

- bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
- componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
- unbind：只调用一次，指令与元素解绑时调用。

## 自定义过滤器

过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式中。被添加在 JavaScript 表达式的尾部，由“管道”符号指示：

```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```

### 全局过滤器

```
Vue.filter('test', value => { ... })
```

### 局部过滤器

定义在组件的 filters 选项中。

```js
filters: {
    test() { ... }
}
```

## Vue 的运行流程，new Vue 都做了什么

1. initLifecycle(vm) 初始化生命周期
2. initEvents(vm) 初始化事件中心
3. initRender(vm) 初始化渲染
4. callHook(vm, 'beforeCreate') 触发 beforeCreate 生命周期
5. initInjections(vm) 初始化别处注入过来的 inject 对象
6. initState(vm) 初始化 state， props, computed, watcher
7. initProvide(vm) 初始化给别处提供的 Provide 对象
8. callHook(vm, 'created') 触发 created 生命周期
9. vm.$mount()
   1. compile 将 el 或者 template 编译成 render 方法
   2. render 通过执行 createElement 方法生成虚拟 DOM 节点
   3. vnode // create diff patch
   4. patch 将虚拟 DOM 树插入到真实的 DOM 中

## Vue 响应式原理(数据绑定原理)

Vue 采用数据劫持结合发布—订阅模式的方法，通过 Object.defineProperty() 来劫持各个属性的 setter，getter，在 set 时通知订阅者更新，get 的时候收集订阅者。模板编译的时候，生成对应的订阅者，调用这个属性的 get，主动去完成这个订阅者收集。所以一共分为 4 部分：

1. 对数据对象进行遍历，包括子属性对象的属性，利用 Object.defineProperty() 对属性都加上 setter 和 getter。调用 getter 的时候，进行依赖收集，将 Observer 存起来。调用 setter 的时候通知更新。

2. 模板编译的时候，对模板上的每一个属性，生成对应的订阅者，这样数据变化时，可以通知模板视图的更新。

3. 实现一个任务调度中心：采用 发布-订阅 设计模式，用来收集订阅者 Observer，以及通知 Observer 去更新。

4. 实现一个 Observer 类，用来作为订阅者类。订阅者初始化的时候主动调用属性的 getter，将自己作为订阅这个属性变化的，完成依赖收集。

```js
class Vue {
  constructor(options) {
    this.data = options.data;
    this.walkData(this.data);
    this.compile();
  }

  // 第一步
  walkData(data) {
    Object.keys(data).forEach((key) => {
      let val = data[key];
      let eventCenter = new EventCenter();
      Object.defineProperty(data, key, {
        set(value) {
          val = value;
          eventCenter.notify(); // 通知所有的observer更新
        },
        get(value) {
          eventCenter.collect(); // 最开始collect的时候， 因为EventCenter.target是null， 所以并没有依赖，只有编译页面的时候， 才将依赖收集上来
          return value;
        },
      });
    });
  }

  // 第二步
  compile() {
    new Observer(); // 指定target 和 调用get，完成依赖收集
  }
}

// 第三步
class EventCenter {
  constructor() {
    this.observers = []; // 存储所有的observer
  }
  addObserver(ob) {
    this.observers.push(ob);
  }
  notify() {
    this.observers.forEach((ob) => {
      ob.update();
    });
  }
  collect() {
    // 依赖收集
    if (EventCenter.target) {
      // 这里是重点，借用静态属性target存储当前的observer实例。
      EventCenter.target.addSelf(this);
    }
  }
}
EventCenter.target = null;

// 第四步
class Observer {
  constructor() {
    this.init();
  }
  init() {
    EventCenter.target = this;
    // 调用get， 完成依赖收集
  }
  update() {}
  addSelf(eventCenter) {
    eventCenter.addObserver(this); // 将自己添加到事件调度中心的订阅者队列
  }
}
```

## vue 响应式原理的两大缺陷，解决办法

1. 无法监听到对象属性的动态添加和删除。用 this.$set 或者this.$delete 解决
2. 无法监听到数组下标和 length 长度的变化。用 arr.splice 解决

## Object.defineProperty(obj, prop, descriptor)

会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的。

1. obj 要在其上定义属性的对象。
2. prop 要定义或修改的属性的名称。
3. descriptor 将被定义或修改的属性描述符。属性描述符有两种主要形式：数据描述符和存取描述符。描述符必须是这两种形式之一，不能同时是两者。

   数据描述符

   - writable  
     当且仅当该属性的 writable 为 true 时，value 才能被赋值运算符改变。默认为 false。
   - value  
     该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。

   存取描述符

   - get  
     当访问该属性时，该方法会被执行，默认为 undefined。
   - set  
     当属性值修改时，触发执行该方法，该方法将接受唯一参数，即该属性新的参数值。默认为 undefined

   数据描述符和存取描述符均具有以下可选键值

   - configurable  
     当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变(也就是可以被重新 defineProperty)，同时该属性也能从对应的对象上被删除。默认为 false。
   - enumerable  
     当且仅当该属性的 enumerable 为 true 时，该属性才能够出现在对象的枚举属性中。默认为 false。

## Proxy 与 Object.defineProperty 的优劣势对比

Proxy 的优势如下:

- Proxy 可以直接监听对象而非属性；
- Proxy 可以直接监听数组的变化；
- Proxy 有多达 13 种拦截方法, 很多是 Object.defineProperty 不具备的；
- Proxy 返回的是一个新对象,我们可以只操作新的对象达到目的,而 Object.defineProperty 只能遍历对象属性直接修改；

Object.defineProperty 的优势如下:

兼容性好，支持 IE9，而 Proxy 的存在浏览器兼容性问题,而且无法用 polyfill 磨平，因此 Vue 的作者才声明需要等到下个大版本( 3.0 )才能用 Proxy 重写。

## watch 实现原理

initWatch 的过程中其实就是实例化 new Watcher 完成观察者的依赖收集的过程。  
参考 Vue 响应式原理。

## computed 实现原理

除了依赖收集，他还有个动态计算的过程，只有依赖的数据发生变化，他才重新计算。  
参考 Vue 响应式原理。

## EventBus 的实现原理

其实就是发布订阅模式的事件调度中心的实现。

1. 有一个对象或者 Map，来存储事件，以及对应的处理函数。
2. 一个 on 方法，来监听某事件，然后将对应的处理函数 push 进去。
3. 一个 off 方法，来删除某事件的监听。
4. 一个 trigger 方法， 来触发某事件的处理函数。

## 虚拟 DOM（Virtual Dom）原理

虚拟 dom 只是一层对真实 DOM 树的抽象，对这颗抽象树进行创建节点,删除节点以及修改节点的操作， 经过 diff 算法得出一些需要修改的最小单位,再更新视图，减少了 dom 操作，提高了性能。

虚拟 DOM 的实现原理主要包括以下 3 部分：

1. 用 JavaScript 对象模拟真实 DOM 树，对真实 DOM 进行抽象；一般虚拟 DOM 都有以下几个属性。

   - 节点名称 tag
   - 节点属性 props 对象
   - 子节点 children 数组

   ```html
   <div id="app">
     <p class="text">hello world!!!</p>
   </div>
   ```

   转化为虚拟 dom

   ```js
   {
       tag: 'div',
       props: {
           id: 'app'
       },
       children: [
           {
               tag: 'p',
               props: {
                   className: 'text'
               },
               chidren: [
                   'hello world!!!'
               ]
           }
       ]
   }
   ```

2. diff 算法 — 比较两棵虚拟 DOM 树的差异；
   - 比较只会在同层级进行, 不会跨层级比较
   - 在 diff 比较的过程中，循环从两边向中间比较
3. pach 算法 — 将两个虚拟 DOM 对象的差异应用到真正的 DOM 树。

## 真实 DOM 更快还是虚拟 DOM 更快？

框架的意义在于为你掩盖底层的 DOM 操作，让你用更声明式的方式来描述你的目的，从而让你的代码更容易维护。

没有任何框架可以比纯手动的优化 DOM 操作更快，因为框架的 DOM 操作层需要应对任何上层 API 可能产生的操作，它的实现必须是普适的。

针对任何一处基准，我们都可以写出比任何框架更快的手动优化，但是那有什么意义呢？在构建一个实际应用的时候，你难道为每一个地方都去做手动优化吗？出于可维护性的考虑，这显然不可能。框架给你的保证是，你在不需要手动优化的情况下，我依然可以给你提供过得去的性能。

## vue scoped 是怎么实现的

1. 给 HTML 的 DOM 节点加一个不重复 data 属性(形如：data-v-2311c06a)来表示他的唯一性。
2. 在每句 css 选择器的末尾（编译后的生成的 css 语句）加一个当前组件的 data 属性选择器的哈希特征值（如[data-v-2311c06a]）来私有化样式。

## 路由跳转

1. 声明式，`<router-link to='home'>` router-link 标签会渲染为`<a>`标签

2. 编程式导航，比如`this.$router.push('/home')`

## 路由导航守卫

1. 全局前置守卫 `router.beforeEach((to, from, next) => {...})`
2. 全局解析守卫 `router.beforeResolve((to, from, next) => {...})`，和 router.beforeEach 类似，区别是在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用。
3. 全局后置钩子 `router.afterEach((to, from) => {...})`，没有 next 函数也不会改变导航本身
4. 路由独享守卫 `beforeEnter((to, from, next) => {...})`，在路由配置上直接定义
5. 组件内的守卫
   - beforeRouteEnter (to, from, next) {...}在渲染该组件的对应路由被 confirm 前调用
   - beforeRouteUpdate (to, from, next) {...}在当前路由改变，但是该组件被复用时调用
   - beforeRouteLeave (to, from, next) {...}导航离开该组件的对应路由时调用

## 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用离开守卫 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。

## 路由鉴权

```js
meta: {
    requireAuth: true
}

router.beforeEach((to, from, next) => {
    if (to.matched.some(record => record.meta.requireAuth)) { ... }
}
```

### 为什么不直接用 to.mate 判断，而要用 to.matched 来判断

to.matched 能够拿到父级的组件的路由对象，用 to.matched 则只需要给较高一级的路由添加 requiresAuth 即可，其下的所有子路由不必添加。

## 路由动态加载与动态删除

动态添加: router.addRoutes([...])

动态删除：

1. 刷新页面
2. 替换 router.matcher 为一个新的 matcher

```js
// 创建静态的路由表
const createRouter = () => new Router({
  mode: 'hash',
  scrollBehavior: () => ({ y: 0 }),
  isAddDynamicRoutes: false, // 是否已经动态添加路由
  routes: [...]
})
const router = createRouter()

// 动态添加路由
const addDynamicRoute = () => {
  let routes = dynamicRoutes() // 获得动态添加的路由表
  router.addRoutes(routes)
  router.options.isAddDynamicRoutes = true
}

// 重置路由，即动态删除添加的路由
const resetRoute = () => {
  const newRouter = createRouter()
  router.matcher = newRouter.matcher // reset router
  router.options.isAddDynamicRoutes = false
}
```

## vue-router 实现原理

### 前端路由

路由模块的本质 就是建立起 url 和页面之间的映射关系，在单页面应用程序中，动态替换 DOM 内容并同步修改 url 地址。

### hash 模式

使用 URL 的 hash 来模拟一个完整的 URL，只改变 hash#后的部分。通过监听 hashchange 事件，监测 hash 值变化，实现更新页面部分内容。

### history 模式

利用 HTML5 History API, pushState 和 replaceState，通过这两个 API 可以改变 url 地址且不会发送请求，同时还有 popstate 事件。

### hash 模式和 history 模式的区别

- 一般使用场景没啥区别，他们俩也都可以使用浏览器的前进后退按钮。
- hisotry 模式需要配置服务器， 否则刷新页面可能会导致 404
- hisotry 不支持 IE9 以下
- hash 模式带#号，一般不能用来做分享的 url，因为有的 app 里面 url 是不允许带有#号的

## 手写 mini router

## vuex

### state

state 对象可以包含本应用全部的状态。在组件中通过 this.$store.state 来访问

```js
const state = {
  testState: 'this is a state',
};
```

### getter

getter 可以认为是 store 的计算属性。在组件中通过 this.$store.getters 来访问

```js
const getters = {
  testGetter: (state) => {
    return state.testState + ' testGetters';
  },
};
```

### mutation

更改 store 中的状态的唯一方法是提交 mutation

mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数

```js
const mutations = {
  // ES6中属性命名也可以用 []，里面是表达式
  [types.TEST_MUTATION_CHANGE_STATE](state, payload) {
    state.testState = payload.newState;
  },
};

this.$store.commit('TEST_MUTATION_CHANGE_STATE', {
  newState: 'this is new state',
});
```

### action

Action 类似于 mutation，不同在于：Action 提交的是 mutation，而不是直接变更状态； Action 可以包含任意异步操作。  
Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters

```js
testActionChangeState ({ commit }, payload) {
    setTimeout(() => {
        commit(types.TEST_MUTATION_CHANGE_STATE, {
            newState: payload.newState
        })
    }, 1000)
}

this.$store.dispatch('testActionChangeState', {
    newState: 'this is new state'
})
```

### module

module 可以将 store 分割成小模块

```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

### mapState mapGetters mapMutations mapActions

mapState 和 mapGetters 可以将 state 和 getter 映射成组件中的计算属性  
mapMutations 和 mapActions 可以映射成组件中的方法

```js
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'

computed: {
    ...mapState([
        'testState'
    ]),

    ...mapGetters([
        'testGetter'
    ])
},

methods: {
    ...mapMutations([
        'TEST_MUTATION_CHANGE_STATE'
    ]),
    ...mapActions([
        'testActionChangeState'
    ])
}
```

### 手写 mini vuex

## vue 项目优化

### 代码层面的优化

- v-if 和 v-show 区分使用场景
- computed 和 watch 区分使用场景
- v-for 遍历必须为 item 添加 key，且避免同时使用 v-if
- 事件的销毁
- 图片资源懒加载
- 路由懒加载
- 第三方插件的按需引入
- 服务端渲染 SSR or 预渲染

### Webpack 层面的优化

- Webpack 对图片进行压缩
- 减少 ES6 转为 ES5 的冗余代码
- 提取公共代码
- 模板预编译
- 提取组件的 CSS
- 优化 SourceMap
- 构建结果输出分析
- Vue 项目的编译优化

### 基础的 Web 技术的优化

- 开启 gzip 压缩
- 浏览器缓存
- CDN 的使用
- 使用 Chrome Performance 查找性能瓶颈
