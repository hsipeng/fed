# Vue

## Vue 2.x 
### 生命周期

![](https://cn.vuejs.org/images/lifecycle.png)

### 组件

* 基础组件
```javascript
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

* 组件注册
全局组件

```javascript
Vue.component(‘my-component-name’, {
  // ... 选项 ...
})
```

局部组件

```javascript
var ComponentA = { /* … */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* … */ }

new Vue({
  el: ‘#app’,
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})

```

多组件注册

```javascript
import Vue from ‘vue’
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

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

* prop
我们可以为组件的 prop 指定验证要求，例如你知道的这些类型。如果有一个需求没有被满足，则 Vue 会在浏览器控制台中警告你。这在开发一个会被别人用到的组件时尤其有帮助。
为了定制 prop 的验证方式，你可以为 props 中的值提供一个带有验证需求的对象，而不是一个字符串数组。例如：

```javascript
Vue.component(‘my-component’, {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning’, ‘danger'].indexOf(value) !== -1
      }
    }
  }
})
```
* 自定义事件
# 事件名
不同于组件和 prop，事件名不存在任何自动化的大小写转换。而是触发的事件名需要完全匹配监听这个事件所用的名称。举个例子，如果触发一个 camelCase 名字的事件：

```javascript
this.$emit(‘myEvent’)

```

则监听这个名字的 kebab-case 版本是不会有任何效果的：
```javascript
<!-- 没有效果 -->
<my-component v-on:my-event=“doSomething”></my-component>

```

不同于组件和 prop，事件名不会被用作一个 JavaScript 变量名或 property 名，所以就没有理由使用 camelCase 或 PascalCase 了。并且 v-on 事件监听器在 DOM 模板中会被自动转换为全小写 (因为 HTML 是大小写不敏感的)，所以 v-on:myEvent 将会变成 v-on:myevent——导致 myEvent 不可能被监听到。
因此，我们推荐你**始终使用 kebab-case 的事件名**。

* 插槽
```html
// 模版
<a
  v-bind:href=“url”
  class=“nav-link”
>
  <slot></slot>
</a>

// 使用

<navigation-link url=“/profile”>
  <!-- 添加一个 Font Awesome 图标 -->
  <span class="fa fa-user"></span>
  Your Profile
</navigation-link>

```

* 动态组件

`<keep-alive>`
当在这些组件之间切换的时候，你有时会想保持这些组件的状态，以避免反复重渲染导致的性能问题。
异步组件
```javascript
// Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染。例如：
Vue.component(‘async-example’, function (resolve, reject) {
  setTimeout(function () {
    // 向 `resolve` 回调传递组件定义
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})

//一个推荐的做法是将异步组件和 webpack 的 code-splitting 功能一起配合使用：
Vue.component(‘async-webpack-example’, function (resolve) {
  // 这个特殊的 `require` 语法将会告诉 webpack
  // 自动将你的构建代码切割成多个包，这些包
  // 会通过 Ajax 请求加载
  require(['./my-async-component'], resolve)
})


// 你也可以在工厂函数中返回一个 Promise，所以把 webpack 2 和 ES2015 语法加在一起，我们可以写成这样

Vue.component(
  ‘async-webpack-example’,
  // 这个 `import` 函数会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)


//当使用局部注册的时候，你也可以直接提供一个返回 Promise 的函数：

new Vue({
  // …
  components: {
    ‘my-component’: () => import(‘./my-async-component')
  }
})
```

## Vue 3.x

尤雨溪已经很详细的介绍了Vue3.0带来的优势及其一些改变。其实大致可以体现在以下几点：

(0) 使用 ES2015 Proxy 作为其观察者机制。 这消除了以前存在的警告，使速度加倍，并节省了一半的内存开销
(1) 重写了虚拟Dom、diff算法优化
(2) update性能提高1.3~2倍、SSR速度提高了2~3倍
(3) 生命周期的改变，API 通过命名导出、可以按需导入生命周期函数
(4) 更直观的代码复用实现方式(hooks)
(5) 更友好且更耦合的数据通信方式(provide & inject)
(6) API 通过命名导出,更好的支持Tree Shaking
(7) 新的更新策略： block tree（区分动态节点和静态节点）
(8) 更好地支持typescript


```html
<template>
  <div class="test">
    <h1>test count: {{count}}</h1>
    <div>count * 2 = {{doubleCount}}</div>
    <button @click="add">add</button>
  </div>
</template>

<script>
  import { ref, computed, watch } from 'vue'

  export default {
    setup () {
      const count = ref(0)
      const add = () => {
        count.value++
      }
      watch(() => count.value, val => {
        console.log(`count is ${val}`)
      })
      const doubleCount = computed(() => count.value * 2)
      return {
        count,
        doubleCount,
        add
      }
    }
  }
</script>

```

## 参考
* [介绍 — Vue.js](https://cn.vuejs.org/v2/guide/)
* [Vue 3.0 全家桶抢先体验 - 掘金](https://juejin.im/post/5e99c21b6fb9a03c590dfea8)