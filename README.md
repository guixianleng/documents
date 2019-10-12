# Vue 开发必备技能
## 前言
自使用Vue以来，总结下自己开发过程中掌握的技能，务实下自己的vue基础的同事，方便后续翻阅和技术总结

## 1. Vue生命周期
### 1. 生命周期是什么
Vue 实例有一个完整的生命周期，也就是从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、卸载等一系列过程，这就是 `Vue生命周期`

### 2. 生命周期图示
![](./images/period.png)

### 3.各生命周期及其作用

|  生命周期钩子   | 作用  |
|  :----:  | :-----  |
| beforeCreate  | 实例未创建，组件属性未生效，data数据不能访问 |
| created  | 实例创建完成，未挂载DOM，不能访问$el，$ref，可以对data数据进行操作，若需DOM操作，可以在vm.$nextTick(callback)回调处理 |
| beforeMount  | 在挂载开始之前被调用，render函数被调用 |
| mounted  | 完成创建$el和双向绑定，完成挂载DOM和渲染；可在mounted钩子对挂载的dom进行操作 |
| beforeUpdate  | 数据更新之前，可在更新前访问现有的DOM |
| updated  | 组件DOM已完成更新 |
| activated  | keep-alive专属，组件被激活时调用 |
| deactivated  | keep-alive专属，组件被移除时使用 |
| beforeDestroy  | 组件销毁前调用 |
| destroyed   | 组件销毁后调用 |

### 4. 父子组件执行顺序
- 加载渲染过程

父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted

- 子组件更新过程

父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated

- 父组件更新过程

父 beforeUpdate -> 父 updated

- 销毁过程

父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

## 2. 组件之间传值
### 1. props（单项数据流）
1. 字符串数组形式
```js
Vue.component('my-component', {
  props: ['propA', 'propB', ...]
}
```

2. 规范式
```js
/**
  * @type 传入值类型检查
    可选值：String | Number | Boolean | Array | Object | Date | Function | Symbol
    @type 还可以是一个自定义的构造函数，并且通过 instanceof 来进行检查确认
  * @require 是否为必传
  * @default 默认值
  * @validator 自定义验证函数
*/
Vue.component('my-component', {
  props: {
    propA: {
      type: String
    },
    propB: {
      type: [Number, String] // 多个可能的类型
    },
    propC: {
      type: Object,
      default: () => {
        return {
          message: 'hello'
        }
      }
    }
    // 自定义验证函数
    propF: {
      type: String,
      default: 'success',
      required: true,
      validator: (value) => {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
}
```

### 2. $emit
自定义事件
```js
// 子组件
Vue.component('my-component', {
  // ...
  methods: {
    click () {
      this.$emit('myEvent', $event)
    }
  }
})

// 父组件
<my-component v-on:my-event="doSomething" />
```
### 3. vuex
vuex是一个状态管理器，提供数据共享

### 4. EventBus
背景：在vue1.x中，组件之间的通信主要通过`vm.$dispatch`沿着父链向上传播和`vm.$broadcast`向下广播来实现。然而在vue2.x中，已经废除了这种用法。vuex加入后，对组件之间的通信有了更加清晰的操作。
对于`中大型项目`来说，一开始就把vuex的使用是明智的选择。而对于`小型项目`，`eventBus`就是便捷的解决方法了。

实现原理：`$on` 和 `$emit` 实例化一个全局 vue 实现数据的共享
```js
// 在 main.js
Vue.prototype.$eventBus = new Vue()

// 传值
methods: {
  addEvent (ev) {
    this.$eventBus.$emit('eventTarget', ev.target)
  }
}

// 接收传值
mounted () {
  this.$eventBus.$on('eventTarget', target => {
    console.log('eventTarget', target)
  })
}
```
或者新建一个 `eventBus.js` 文件
```js
// eventBus.js
import Vue from 'vue'

const eventBus = new Vue()

export { eventBus }

// 传值组件内
import { eventBus } from '../eventBus'

Vue.component('sender', {
  template: `
    <div v-on:click="addEvent"></div>
  `,
  methods: {
    addEvent () {
      this.$eventBus.$emit('eventTarget', 'send值')
    }
  }
})
// 接收组件内
Vue.component('receiver', {
  mounted () {
    this.$eventBus.$on('eventTarget', target => {
      console.log('eventTarget', target) // send值
    })
  }
})
```
### 5. attrs、inheritAttrs和listeners
### 1. $attrs
场景：需要获取父组件多个值时，定义props比较繁琐，这时就可以使用attrs替代

```js
// 父组件
<parent title="标题" level="level" />

// 子组件
mounted () {
  console.log(this.$attrs) // {title: "标题", level: "level"}
}
```
**注意**：如果`props`定义了，`$attrs`中是获取不到的

### 2. 禁用特性继承`inheritAttrs`
如果不希望组件的根元素继承特性，你可以在组件的选项中设置inheritAttrs: false

```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```
**注意**：inheritAttrs: false 选项不会影响 style 和 class 的绑定。

### 3. $listeners
场景：子组件需要调用父组件中方法
```js
// 父组件
<parent v-on:click-left="handleClickLeft" />

// 子组件
mounted () {
  console.log(this.$listeners) // 可以获取handleClickLeft事件
}
```


### 6. provide和inject（多层传递值）
[`provide / inject`](https://cn.vuejs.org/v2/api/#provide-inject)

> 2.2.0 新增

> `provide` 和 `inject` 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码

简单的说就是在父组件中通过provider来提供变量，然后在子组件中通过inject来注入变量中。

- provide：Object | () => Object
- inject：Array<string> | { [key: string]: string | Symbol | Object }
```js
// 父组件
provide: {
  for: "demo"
}

// 任意个或多层级子组件
inject: ['for'],
data () {
  return: {
    demo: this.for
  }
}
console.log(this.demo) // demo
```

### 7. 插槽
### 1. 匿名插槽
```js
// 父组件
<parent>
  <template>
    <p>one slot.</p>
  </template>
</parent>

// 子组件
<slot></slot>

// 默认是 v-slot:default，可以不写
/* 上面为
  // 父组件
  <template v-slot:default>
    <p>one slot.</p>
  </template>
  // 子组件
  <slot name="default"></slot>
*/
```
**注意**：`v-slot`只能添加在一个 <template> 上， 若为多个则需要具名插槽
### 2. 具名插槽
顾名思义：插槽组件slot标签带name命名
```js
// 父组件
<parent>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>
</parent>

// 子组件
<slot name="header"></slot>
```
> 若想判断存在显示对应的插槽可以通过 v-if="$slots.name"；例如： v-if="$slots.default" v-if="$slots.header"

### 3. 作用域插槽
父组件可以访问子组件数据

```js
// 父组件
<parent>
  <template v-slot:user="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</parent>

// 子组件
Vue.component('my-component', {
  data: {
    user: {
      firstName: 'Smith',
      lastName: 'Harden'
    }
  },
  template: `
    <slot v-bind:user="user" name="user">
      {{ user.firstName }}
    </slot>
  `
})
// slotProps 可以随便命名
// slotProps 绑定的是子组件上的 user 对象
```
### 8. parent和children
指定已创建的实例之父实例，在两者之间建立父子关系。
子实例可以用 this.$parent 访问父实例，子实例被推入父实例的 $children 数组中。

### $children
- 类型：Array
- 详细：`$children` **并不能保证顺序, 也不是响应式的**

```js
// 父组件
console.log(this.$children) // 只能获取一级子组件的属性和方法

// 子组件
console.log(this.$parent) // 只能获取 parent 的属性和方法
```

### 8. `.sync` 修饰符
> 2.3.0+ 新增
```js
// 父组件
<my-component v-bind:title.sync="title" />
// 等同于
<my-component
  v-bind:title="title"
  v-on:update:title="val => title = val"
/>

// 子组件做了什么？
// 通过 $emit 触发 update方法更新 title
this.$emit('update:title', newTitle)
```
## 3. 不常用但实用的开发小技巧
### 1. watch
[vm.$watch( expOrFn, callback, [options] )](https://cn.vuejs.org/v2/api/#vm-watch)

### options：deep（深度监听） 和 immediate（立即执行）

```js
var data = { a: 1 }

var vm = new Vue({
  el: '#root',
  data: data
})

vm.$watch(
  'a',
  function (newVal, oldVal) {
    console.log(val, oldVal)
  },
  {
    // 深度 watcher
    deep: true,
    // 该回调将会在侦听开始之后被立即调用
    immediate: true
  }
)
```
### 2. v-cloak
场景：网络加载慢，是出现`{{}}`插值来不及渲染的情况，

解决方式：使用`v-cloak`指令
```html
<div v-cloak>
  {{ message }}
</div>
<!-- 不会显示，直到编译结束。 -->
```
css设置
```css
[v-cloak] {
  display: none;
}
```
### 3. $forceUpdate
场景：render函数没有自动更新，

解决方式：强制刷新，解决页面不会重新渲染的问题，`this.$forceUpdate()`

### 4. Object.freeze
[Object.freeze()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

该方法是ES5新增的特性，可以冻结一个对象，防止对象被修改。
```js
new Vue({
  data: {
      // vue不会对list里的object做getter、setter绑定
    list: Object.freeze([
      { value: 1 },
      { value: 2 }
    ])
  },
  created () {
    // 界面不会有响应
    this.list[0].value = 100;

    // 下面两种做法，界面都会响应
    this.list = [
      { value: 100 },
      { value: 200 }
    ]
    this.list = Object.freeze([
      { value: 100 },
      { value: 200 }
    ])
  }
})
```

### 5. template 调试
场景: 在Vue开发过程中, 经常会遇到template模板渲染时JavaScript变量出错的问题
```js
// main.js
Vue.prototype.$log = window.console.log

// 页面
<div>{{$log(info)}}</div>
```

### 6. transformToRequire 再也不用把图片写成变量了
场景：vue-cli3脚手架引入图片，需提前 require 传给一个变量再传给组件

解决方式：配置 `transformToRequire` 获取require地址
```js
// 配置之前
<template>
  <div>
    <avatar :default-src="imgUrl"></avatar>
  </div>
</template>
<script>
  export default {
    created () {
      this.imgUrl = require('./assets/default.png')
    }
  }
</script>
```
`vue.config.js`中配置
```js
{
  vue: {
    transformToRequire: {
      avatar: ['default-src']
    }
  }
}
```
```js
<template>
  <div>
    <avatar default-src="./assets/default.png"></avatar>
  </div>
</template>
```

### 7. img 图片加载失败
场景：移动端会出现很多图片的加载，某些图片加载不出来

解决方式：使用默认的图片替换加载失败的图片
1. vue-lazyload 图片懒加载插件

```shell
$ yarn add vue-lazyload
```

```js
// main.js
import VueLazyload from 'vue-lazyload'

Vue.use(VueLazyload, {
  loading: require('assets/images/default.png')
})

// 页面中 
<img width="60" height="60" v-lazy="item.imgUrl">
```

2. 使用原生error事件处理
```js
new Vue({
  template: `
    <img :src="imgUrl" @error="handleError" />
  `,
  data: {
    imgUrl: ''
  },
  methods: {
    handleError (ev) {
      ev.currentTarget.src = require('assets/images/default.png')
    }
  }
})
```

### 8. 路径别名配置
```js
// vue.config.js
resolve: {
  // 配置项目别名
  alias: {
    '@': resolve('./src'),
    'api': resolve('./src/api'),
    'assets': resolve('./src/assets'),
    'baseUI': resolve('./src/baseUI'),
    'components': resolve('./src/components'),
    'utils': resolve('./src/utils'),
    'views': resolve('./src/views')
  }
}
// 也可以
const path = require('path')

function resolve (dir) {
  return path.join(__dirname, dir)
}

module.exports = {
  chainWebpack: config => {
    config.resolve.alias
      .set(key, value) // 如set('@components', resolve('src/components'))
  }
}
```

### 9. 不同路由的组件复用
场景：相同的组件在不同路由下不刷新
```html
<router-view :key="$route.fullpath"></router-view>
```

### 10.  require.context() 自动化全局注册
[require.context()](https://webpack.docschina.org/guides/dependency-management/#require-context) 和 [Vue组件自动化全局注册](https://cn.vuejs.org/v2/guide/components-registration.html#%E5%9F%BA%E7%A1%80%E7%BB%84%E4%BB%B6%E7%9A%84%E8%87%AA%E5%8A%A8%E5%8C%96%E5%85%A8%E5%B1%80%E6%B3%A8%E5%86%8C)
```js
const requireComponent = require.context(
  // 其组件目录的相对路径
  './components',
  // 是否查询其子目录
  false,
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)

  // 获取组件的 PascalCase 命名
  const componentName = upperFirst(
    camelCase(
      // 获取和目录深度无关的文件名
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )

  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，
    // 否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```
如下我的路由自动化全局注册示例：
```js
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

let routes = [
  {
    path: '/',
    redirect: '/recommend'
  }
]

const routerContext = require.context('./modules', true, /\.js$/)
routerContext.keys().forEach(route => {
  const routerModule = routerContext(route)
  routes = [...routes, ...(routerModule.default || routerModule)]
})

export default new Router({
  mode: 'hash',
  base: process.env.BASE_URL,
  routes: routes
})

```
### 11. 深度作用选择器
[Scoped Css](https://vue-loader.vuejs.org/guide/scoped-css.html#deep-selectors)

当 `<style>` 标签有 `scoped` 属性时，它的 `CSS` 只作用于当前组件中的元素

场景：有些情况调用第三方组件并需要修改其样式，而又不想去除scoped属性造成组件之间的样式污染又可以影响其样式

#### 解决方式
1. `>>>`
```js
<div class="demo">
  <el-input v-model="text"></el-input>
</div>
<style scoped>
  .demo >>> .el-input { /* ... */ }
</style>
```
会编译成
```css
.demo[data-v-f3f3eg9] .b { /* ... */ }
```

2. `/deep/`
```js
<div class="demo">
  <el-input v-model="text"></el-input>
</div>
<style scoped>
  .demo /deep/ input { /* ... */ }
</style>
```
会编译成
```css
.demo[data-v-f3f3eg9] input { /* ... */ }
```

### 12. Vue.compile 渲染函数
在 render 函数中编译模板字符串。**只在独立构建时有效**
```js
var res = Vue.compile('<div><span>{{ msg }}</span></div>')

new Vue({
  data: {
    msg: 'hello'
  },
  render: res.render,
  staticRenderFns: res.staticRenderFns
})
```
### 11. Vue.observable 简单状态管理
[Vue.observable](https://cn.vuejs.org/v2/api/#Vue-observable)
> 2.6.0 新增

让一个对象可响应，返回的对象可以直接用于`渲染函数`和`计算属性`内，并且会在发生改变时`触发相应的更新`

```js
// store.js
import Vue from 'vue'

export const store = Vue.observable({ count: 0 })
export const mutations = {
  setCount (count) {
    store.count = count
  }
}

//使用
import { store, mutations } from '@/store'

Vue.component('', {
  template: `
    <div>
      <button @click="setCount(count + 1)">+</button>
      <span>{{count}}</span>
      <button @click="setCount(count - 1)">-</button>
    </div>
  `,
  computed: {
    count () {
      return store.count
    }
  },
  methods: {
    setCount: mutations.setCount
  }
})
```

### 13. 异步组件加载
#### 常用两种
```js
// 1. 
Vue.component(
  'async-webpack-example',
  // 这个 `import` 函数会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
// 2. 也是最常用的
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component')
  }
})
```
#### 处理加载状态
> 2.3.0+ 新增

```js
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})
```

### 14. 函数式组件（functional）
[functional](https://cn.vuejs.org/v2/api/#functional) 和 [函数式组件](https://cn.vuejs.org/v2/guide/render-function.html#%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BB%84%E4%BB%B6) 
>定义：它`无状态` (没有响应式数据)，也`没有实例` (没有 this 上下文)

一个函数式组件就像这样：
```js
Vue.component('my-component', {
  functional: true,
  // Props 是可选的
  props: {
    // ...
  },
  // 为了弥补缺少的实例
  // 提供第二个参数作为上下文
  render: function (createElement, context) {
    // ...
  }
})
```
单文件组件
```html
<template functional>
  <!-- ... -->
</template>
```

下面以smart-list组件举例：
```js
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true,
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  },
  render: function (createElement, context) {
    function appropriateListComponent () {
      var items = context.props.items
      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,
      context.children
    )
  }
})
```

## 4. Api 之间的区别
### 1. watch 和 computed
**watch**：自定义的侦听器，每当监听的数据变化时都会执行回调进行后续操作；

**computed**：计算属性，其值`有缓存`，除非依赖的响应式属性变化才会重新计算；
#### 运用场景：
- 当需要进行数值计算，并且依赖于其它数据时，应该使用 `computed`，利用其缓存特性，避免每次获取值时，都要重新计算
- 当需要在数据变化时执行异步或开销较大的操作时，应该使用 `watch`。

### 2. components 和 Vue.component
1. components: 局部注册组件
```js
export default{
  components: {
    compA,
    compB,
    ...
  }
}
```
2. Vue.component: 注册或获取全局组件
```js
// 注册组件，传入一个扩展过的构造器
Vue.component('my-component', Vue.extend({ /* ... */ }))

// 注册组件，传入一个选项对象 (自动调用 Vue.extend)
Vue.component('my-component', { /* ... */ })

// 获取注册的组件 (始终返回构造器)
var MyComponent = Vue.component('my-component')
```

### 3. v-show 和 v-if
**v-if**：是动态的向DOM树内添加或删除DOM元素；在切换时元素及它的数据绑定 / 组件被销毁并重建，`更消耗性能`

**v-show**：是基于css属性'display'简单切换

#### 运用场景：
- `v-if` 适合于不需要频繁切换条件的场景；
- `v-show` 适合于频繁切换。