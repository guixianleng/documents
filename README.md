# Vue 开发必备技能
## 前言
自17年使用Vue以来，总结下自己开发过程中掌握的技能，务实下自己的vue基础的，方便后续翻阅和掌握的技术拾取。

## 1. Vue生命周期
### a. 生命周期是什么
Vue 实例有一个完整的生命周期，也就是从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、卸载等一系列过程，这就是 `Vue生命周期`

### b. 生命周期图示
![](./images/period.png)

### c.各生命周期及其作用

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

### d. 父子组件执行顺序
- 加载渲染过程

父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted

- 子组件更新过程

父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated

- 父组件更新过程

父 beforeUpdate -> 父 updated

- 销毁过程

父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

## 2. watch
[vm.$watch( expOrFn, callback, [options] )](https://cn.vuejs.org/v2/api/#vm-watch) 详细说明

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
  // 深度 watcher
  {
    deep: true,
    // 该回调将会在侦听开始之后被立即调用
    immediate: true
  }
)
```

## 3. 组件之间传值
### 1. props（单项数据流）
1. 字符串数组形式
```js
Vue.component('my-component', {
  props: ['propA', 'propB', ...]
}
```

2. 标准形式
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

### 4. attrs、inheritAttrs和listeners
1. $attrs

场景：需要获取父组件多个值时，定义props比较繁琐，这时就可以使用attrs替代

```js
// 父组件
<parent title="标题" level="level" />

// 子组件
mounted () {
  console.log(this.$attrs) // {title: "标题", level: "level"}
}
```
注意：如果props定义了，`$attrs`中是获取不到的

2. 禁用特性继承`inheritAttrs`

如果不希望组件的根元素继承特性，你可以在组件的选项中设置inheritAttrs: false

```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```
::: tips
  注意 inheritAttrs: false 选项不会影响 style 和 class 的绑定。
:::

3. $listeners

场景：子组件需要调用父组件方法
```js
// 父组件
<parent v-on:click-left="handleClickLeft" />

// 子组件
mounted () {
  console.log(this.$listeners) // 可以获取handleClickLeft事件
}
```


### 5. provide和inject（多层传递值）
> 2.2.0 新增

provide 和 inject 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码

[provide / inject](https://cn.vuejs.org/v2/api/#provide-inject)：简单的说就是在父组件中通过provider来提供变量，然后在子组件中通过inject来注入变量中。

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

### 6. `.sync` 修饰符
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
### 7. 插槽
1. 匿名插槽
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
注意 v-slot 只能添加在一个 <template> 上， 若为多个则需要具名插槽

2. 具名插槽

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

3. 作用域插槽
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