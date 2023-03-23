## 组合式API
### 介绍
组合式 API (Composition API) 是一系列 API 的集合，使我们可以使用函数而不是声明选项的方式书写 Vue 组件。它是一个概括性的术语，涵盖了以下方面的 API：
* 响应式 API：例如 ref() 和 reactive()，使我们可以直接创建响应式状态、计算属性和侦听器。
* 生命周期钩子：例如 onMounted() 和 onUnmounted()，使我们可以在组件各个生命周期阶段添加逻辑。
* 依赖注入：例如 provide() 和 inject()，使我们可以在使用响应式 API 时，利用 Vue 的依赖注入系统。

### 为什么要有组合式 API？
* 解决了过去组件过长时，options api 带来的难以维护的问题。
* 解决了 mixins复用模式 的所有缺陷。
* 更好的类型推导能力。
* 更小的生产包体积。

### [VueUse](https://vueuse.org/)

vue工具型组合式函数集合。

## setup
### 使用时机及方式
setup 是在组件中使用组合式 API 的入口，setup()函数通常只在以下情况下使用：

* 基于选项式 API 的组件中集成基于组合式 API 的代码时。
* 在非单文件组件中使用组合式 API 时。

其他情况下，都应优先使用 `<script setup>` 语法，它是使用组合式 API 的编译时语法糖。

### 特性
* setup会在每次组件实例被创建的时候执行。先执行setup、然后是beforeCreate、create。

* setup 函数的第一个参数是组件的 props。props是响应式的，并且会在传入新的 props 时同步更新。如果你解构了 props 对象，解构出的变量将会丢失响应性。推荐通过 props.xxx 的形式来使用其中的 props。如果你确实需要解构 props 对象，或者需要将某个 prop 传到一个外部函数中并保持响应性，可以使用 toRefs() 和 toRef() 这两个工具函数。
* setup 函数的第二个参数是一个 Setup 上下文对象context。包含：attrs，slots，emit，expose等。但是在 `<script setup>`中我们无法访问组件实例或 context 上下文参数。

* 任何在 `<script setup>` 声明的顶层的绑定 (包括变量，函数声明，以及 import 导入的内容) 都能在模板中直接使用。
* setup函数中不能使用this。如果确实需要使用，可以用`const { proxy } = getCurrentInstance();`

* `<script setup>` 中可以使用顶层 await。代码会被编译成 async setup()。

### setup中的生命周期
* onBeforeMount()
* onMounted()
* onBeforeUpdate()
* onUpdated()
* onBeforeUnmount()
* onUnmounted()
* onErrorCaptured()
* onActivated()
* onDeactivated()
```js
onMounted(() => {
    // 组件挂载完毕，DOM 生成，可以使用ref了
})
```

## reactive & ref
* reactive()：可接受对象或数组等对象类型，他能够将复杂数据类型变为响应式数据。响应式是深层次的。如果只想保留对这个对象顶层次访问的响应性，请使用 shallowReactive() 作替代

* ref()：返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性.value。ref的参数可以是任何类型。如果将一个对象赋值给 ref，那么这个对象将通过 reactive() 转为具有深层次响应式的对象。若要避免这种深层次的转换，请使用 shallowRef() 来替代。

* ref 有一个 value 属性可以用来重新赋值，而 reactive 的对象不可重新赋值（会丢失响应性），也不可解构。

```js
// reactive
let obj = reactive({ count: 0 })
obj.count++ // 直接使用，正确
const { count } = obj // 解构，错误
obj = reactive({ count: 1 }) // 重新赋值，错误

// ref
const num = ref(0)
console.log(num.value) // 0，使用num.value正确。
num.value++
console.log(num.value) // 1

const person = ref({ name: 'zhangsan', age: '24' })
console.log(person)
const { name } = person; // 解构，错误，没有person，需使用person.value
let { age } = person.value; // 解构，错误，没有响应式了
person.value.age++; // 正确

// 结合reactive和toRefs，可用来解构，模板中不用加value，但是js还需要写value
const bigObj = toRefs(
  reactive({
    x: 1,
    y: 2,
  })
);
let { x, y } = bigObj; // 正确
x++; // 错误
x.value++; // 正确
```

### 使用ref，什么时候加value，什么时候不加value?
* 在模板中访问ref中的数据，系统会自动帮我们添加.value，
* 在setup的JS中访问ref中的数据，需要手动添加.value。

### ref 和 reactive 到底用哪个。
reactive对象存在解构丢失响应性的问题，而 ref 需要到处使用 .value 则感觉很繁琐，并且在没有类型系统的帮助时很容易漏掉 .value。

### $ref语法糖
为了解决繁琐的value问题。可以使用$ref宏来声明变量。目前是一个实验性功能，默认是禁用的，需要显式选择使用。参考：https://cn.vuejs.org/guide/extras/reactivity-transform.html
```js
<script setup>
let count = $ref(0)

console.log(count)

function increment() {
  count++
}
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

## toRefs & toRef & unref
* toRefs()：将一个响应式对象转换为一个普通对象，这个普通对象的每个属性都是指向源对象相应属性的 ref。转换后的对象可以解构。

* toRef()：基于响应式对象上的一个属性，创建一个对应的 ref。如`toRef(props, 'foo')`

* unref()：如果参数是 ref，则返回内部值，否则返回参数本身。这是 `val = isRef(val) ? val.value : val` 计算的一个语法糖。

比如我们需要解构 props 对象，或者需要将某个 prop 传到一个外部函数中并保持响应性，可以使用 toRefs() 和 toRef() 这两个工具函数。

```js
setup(props) {
    // 将 `props` 转为一个其中全是 ref 的对象，然后解构
    const { title } = toRefs(props)
    // `title` 是一个追踪着 `props.title` 的 ref
    console.log(title.value)

    // 或者，将 `props` 的单个属性转为一个 ref
    const title = toRef(props, 'title')
}
```

## computed $ watch & watchEffect
* computed()：接受一个函数，返回一个响应式ref对象。函数中的响应式变量为依赖（自动收集依赖）
    ```js
    const plusOne = computed(() => count.value + 1)
    console.log(plusOne.value) // 2
    ```
* watch()，跟vue2中差不多，第一个参数是要监听的值，可以是多个，ref或者reactive, getter等。第二个是回调，第三个参数是一些选项，控制执行时机等。
    ```js
    // watch ref
    watch(num, (newValue, oldValue) => {
        console.log('watch num 已触发', `${oldValue} --> ${newValue}`);
    });
    // watch reactive
    watch(
        () => obj.count, // 可以直接watch obj 但不能直接watch obj.count。 必须使用getter
        (newValue, oldValue) => {
            console.log('watch pobj.count 已触发', `${oldValue} --> ${newValue}`);
        }
    );
    ```
    
* watchEffect()，类似useEffect, 在组件渲染之前执行。第一个参数是函数，函数中的响应式变量为依赖（自动收集依赖），依赖变化时函数重新执行。第二个参数选项可以调整函数第一次执行时的机。返回值是一个用来停止该副作用的函数。

推荐大部分时候用 watch 显式的指定依赖以避免不必要的重复触发，也避免在后续代码修改或重构时不小心引入新的依赖。

## readonly()
接受一个对象 (不论是响应式还是普通的) 或是一个 ref，返回一个原值的只读代理。
```js
const original = reactive({ count: 0 })

const copy = readonly(original)

// 更改源属性可以成功
original.count++

// 更改该只读副本将会失败，并会得到一个警告
copy.count++ // warning!
```

## defineProps()，defineEmits() 和 defineExpose()
* 这三个都是只能在 `<script setup>` 中使用的编译器宏。他们不需要导入
* defineProps 接收与 props 选项相同的值
* defineEmits 接收与 emits 选项相同的值。
* 传入到 defineProps 和 defineEmits 的选项会从 setup 中提升到模块的作用域。因此，传入的选项不能引用在 setup 作用域中声明的局部变量。
* defineExpose 指定在 `<script setup>` 组件中要暴露出去的属性，同expose选项。使用 `<script setup>` 的组件是默认不会暴露任何在 `<script setup>` 中声明的绑定。

```js
<!-- 父组件 -->
<script setup>
import Child from './Child.vue'
const testEvent = (e) => {
    console.log('parent event-triggerd')
    console.log(e)
}
</script>

<template>
  <Child some-prop="parent prop" @my-event="testEvent"/>
</template>

<!-- 子组件 -->
<script setup>
const props = defineProps({
  someProp: {
    type: String,
    required: true
  }
})
console.log(someProp) // 不能用，必须用props调用，因为模块作用域
console.log(props.someProp) // parent prop，

const emit = defineEmits(['my-event'])
const handleChange = (event) => {
  emit('my-event', event)
}
</script>

<template>
  <!-- 使用 someProp 或 props.someProp -->
  <div>{{ someProp }}</div>
  <div>{{ props.someProp }}</div>
  <div @click="handleChange">trigger event</div>
</template>
```
## 自定义组合式函数，useXxx(自定义hook)
```js
// mouse.js
import { ref, onMounted, onUnmounted } from 'vue'

// 按照惯例，组合式函数名以 use 开头
export function useMouse() {
  // 组合式函数管理的数据
  const x = ref(0)
  const y = ref(0)

  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  // 组合式函数可以挂靠在所属组件的生命周期上，来启动和卸载副作用
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  // 通过返回值暴露所管理的数据
  return { x, y }
}


// other.js
<script setup>
import { useMouse } from './mouse.js'

const { x, y } = useMouse()
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>
```

## vue3 的 v-modal和自定义组件
### vue2中的自定义组件v-modal
v-modal实现双向绑定，子组件更新的同时，父组件也更新。
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

### vue2的.sync 修饰符
在Vue中，props属性是单向数据传输的，父级的prop的更新会向下流动到子组件中，但是反过来不行。可是有些情况，我们需要对prop进行“双向绑定”。上文中，我们提到了使用v-model实现双向绑定。但有时候我们希望一个组件可以实现多个数据的“双向绑定”，而v-model一个组件只能有一个，这时候就需要使用到.sync。
```js
<com1 :a.sync="num" :b.sync="num2"></com1> 
// 它等价于
<com1 
  :a="num" @update:a="val=>num=val"
  :b="num2" @update:b="val=>num2=val">
</com1> 

// 子组件内：
methods:{
    changeVal(e){
        this.$emit('update:a', a)
        this.$emit('update:b', b)
    }
}
```

### .sync与v-model的异同
相同点:

* 两者的本质都是语法糖，目的都是实现组件与外部数据的双向绑定
* 两个都是通过属性+事件来实现的

不同点:

* 一个组件只能定义一个v-model,但可以定义多个.sync
* v-model与.sync对于的事件名称不同，v-model默认事件为input,可以通过配置model来修改，.sync事件名称固定为update:属性名

### vue3的v-modal和自定义组件
vue3中v-modal可以在一个组件使用多次，同时绑定多个props为双向绑定。

同时绑定多个
```js
<UserName
  v-model:first-name="first"
  v-model:last-name="last"
/>
// 定义多个
props: {
    firstName: String,
    lastName: String
},
emits: ['update:firstName', 'update:lastName']

```
v-modal只是语法糖。类似vue2中的.sync操作符。
```js
<CustomInput v-model="searchText" />
// 相当于
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```
默认情况下，v-model 在组件上都是使用 modelValue 作为 prop，并以 update:modelValue 作为对应的事件。
```js
<script setup>
const props = defineProps({
  modelValue: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="emit('update:modelValue', $event.target.value)"
  />
</template>
```

## 其他差异
template支持多节点，类似react fragments。

emit：声明由组件触发的自定义事件。在setup()中，使用context.emit()方法。在`<script setup>`中，使用defineEmits。

expose：父组件引用子组件实例时，默认暴露当前组件实例的全部内容。当使用 expose 时，只有显式列出的属性将在组件实例上暴露。

在setup()中，使用context.expose()方法暴露。在`<script setup>`中，使用defineExpose。

vue3的生命周期中没有了销毁之前（beforeDestroy ）以及销毁完毕（destroyed ）这两个生命周期。 取而代之的是卸载，卸载之前 (beforeUnmount)以及卸载完毕（unmounted ）

extends：可以继承另一个组件的组件选项。

nextTick：可以传入回调函数或者用await。
```js
<script>
import { nextTick } from 'vue'

...
await nextTick()
...
</script>

```

v-pre 跳过该元素及其所有子元素的编译。最常见的用例就是显示原始双大括号标签及内容。（因为如果编译，双大括号会消失）

v-once 仅渲染元素和组件一次，并跳过之后的更新。

`<Teleport>`将其插槽内容渲染到 DOM 中的另一个位置。
```js
<teleport to="#some-id" />
```

`<style>` 标签支持使用 v-bind 函数将 CSS 的值链接到动态的组件状态
```js
<script setup>
const theme = {
  color: 'red'
}
</script>

<style scoped>
p {
  color: v-bind('theme.color');
}
</style>
```

处于 scoped 样式中的选择器如果想要做更“深度”的选择，可以使用 :deep() 这个伪类。这是vue3自带的，类似sass语法中的v::deep
```js
<style scoped>
.a :deep(.b) {
  /* ... */
}
</style>
```

全局方法或全局变量：
```js
app.config.globalProperties.fn = () => {...}
app.config.globalProperties.data = {...}
```

## 异步组件defineAsyncComponent 与 defineComponent & defineCustomElement
* defineAsyncComponent：创建异步组件，一般配合import使用，可以传入一个loading组件和加载出错时的组件，也可以直接import。
```js
const AsyncComp = defineAsyncComponent({
  loader: () => import('./Foo.vue'),
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,
  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})
```
* defineComponent：为了让 TypeScript 正确地推导出组件选项内的类型，我们需要通过 defineComponent() 这个全局 API 来定义组件
```js
import { defineComponent } from 'vue'

export default defineComponent({
    props: {
        message: String
    },
    setup(props) {
        props.message
    }
})
```

* defineCustomElement: 创建web-components，和 defineComponent 接受的参数相同。注意需要配置app.config.compilerOptions.isCustomElement选项
```js
import { defineCustomElement } from 'vue'

const MyVueElement = defineCustomElement({
  // 这里是同平常一样的 Vue 组件选项
  props: {},
  emits: {},
  template: `...`,

  // defineCustomElement 特有的：注入进 shadow root 的 CSS
  styles: [`/* inlined css */`]
})

// 注册自定义元素
// 注册之后，所有此页面中的 `<my-vue-element>` 标签
// 都会被升级
customElements.define('my-vue-element', MyVueElement)
```

## vue3核心写法总结：
* 优先使用 `<script setup>` 语法，早期的setup函数不推荐。
* 组件的props可以用const props = defineProps({})来定义和接收。通过props.xxx来使用，模板中也可以直接用xxx，省略props.
* 组件的data可以用ref()在 `<script setup>` 中声明，js使用时加value。
* 使用的其他组件可以直接在 `<script setup>` 中导入，导入后模板中可以直接使用
* 组件中使用的方法可以直接在 `<script setup>` 中定义函数.
* 之前在created中写的代码，都可以在最外层写。包括初始化数据，从后台获取数据等。
* 组件中的生命周期，参考生命周期部分，变为onMounted等。
* watch computed就分别用watch()和computed()
* 父子通讯：父给子传递参数，子组件需要用defineProps。子组件触发事件，可以用defineEmits来实现以前的emit。父组件获取子组件的属性，需要子组件显式的暴露defineExpose
* 默认setup中无法使用 this 对象，如果想用可以通过getCurrentInstance获取。`const { proxy }  = getCurrentInstance()`

## 路由
### 组合式 API
因为我们在 setup 里面没有访问 this，所以我们不能再直接访问 this.$router 或 this.$route。

所以增加了新的组合式api：访问路由器useRouter，访问当前路由useRoute。

但是在模板中我们仍然可以访问 $router 和 $route，所以不需要在 setup 中返回 router 或 route。
```js
import { useRouter,useRoute } from 'vue-router'
const router = useRouter()
const route = useRoute()

router.push({
    path: '/about',
    query: {
        msg: 'hello vue3!'
    }
})

console.log(route.query.msg) // hello vue3!
```

setup中使用的组件内守卫：`import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'`

### 动态路由
主要通过两个函数实现。router.addRoute() 和 router.removeRoute()

### 路由插槽
之前可以直接传递一个模板，通过嵌套在 `<router-view>` 组件下，由路由组件的 <slot> 来渲染
```js
<router-view>
  <p>In Vue Router 3, I render inside the route component</p>
</router-view>
```

但现在，transition 和 keep-alive 现在必须通过 v-slot API 在 RouterView 内部使用
```js
<router-view v-slot="{ Component }">
  <transition>
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </transition>
</router-view>
```

## 状态管理Pinia
vue3不再用vuex。而使用更简单的Pinia。

createPinia 创建一个 pinia（根存储）并将其传递给应用程序。
```js
import { createPinia } from 'pinia';

const pinia = createPinia();
app.use(pinia);
```

defineStore 创建一个 store，来存储一些状态。可以定义多个 store，按模块划分。
```js
import { defineStore } from 'pinia'

export const useAMoudleStore = defineStore('a-moudle', {
  state: () => {
    return {
      name: 'a',
      counter: 1
    }
  },
  getters: {
    doubleCount: (state) => state.counter * 2,
  },
  actions: {
    increment() {
      this.counter++;
    },
  },
})

// 使用
import { useAMoudleStore } from '@/xxx'
const store = useAMoudleStore()

store.counter++; // 修改

const testAction = () => { // 调用action
  store.increment();
}
```

从 Store 中解构也会丢失响应性，需要使用storeToRefs()转化。
```js
import { storeToRefs } from 'pinia'
const store = useStore()
const { name, doubleCount } = storeToRefs(store)
```

修改store中的state。除了直接用 store.counter++ 修改 store，你还可以调用 $patch 方法。 它允许同时应用多个更改：
```js
store.$patch({
  counter: store.counter + 1,
  name: 'Abalam',
})
```

或者将其 $state 属性设置为新对象来替换 Store 的整个状态。
```js
store.$state = { counter: 666, name: 'Paimon' }
```

getter和actions中可以使用 this 访问到 整个 store 的实例。

Actions 相当于组件中的 methods。 可以使用 defineStore() 中的 actions 属性定义，并且它们非常适合定义业务逻辑。支持异步。

