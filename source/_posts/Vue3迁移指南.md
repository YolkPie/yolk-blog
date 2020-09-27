---
title: Vue3迁移指南
date: 2020-09-27 16:47:09
tags:
  - vue
categories: vue
keywords: vue
description: 虚拟DOM的核心之一就是它的Diff算法，其中最为核心的就是核心Diff算法，只有在新旧虚拟DOM的子节点都是多个的时候，核心Diff算法才会派上用场。
cover: https://img10.360buyimg.com/imagetools/jfs/t1/122074/8/13483/538833/5f70524dEdc179584/a381558342598384.png
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/133083/40/11052/11173/5f70524dE5f2bac4f/6ac0bf70c2099db1.jpg
---
## 前言

`Vue 3`已经正式发布，我们现有的项目基本都是基于`Vue 2`进行开发的，如何由`Vue 2.x`迁移到`Vue 3.x`，[官方文档](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E6%A6%82%E8%A7%88)已经提供了迁移指南，本文是对官方迁移指南的总结。

## 重大改变
`Vue 3`相较于`Vue 2`有许多`Break Change`的重大改变，是我们在迁移时需要重点关注的：

### 全局API
- 变更为使用应用程序实例
- 全局和内部API可以被`tree shaking`

### 模版指令
- `v-model`用法的变更
- `v-for`节点上`key`用法的变更
- `v-if`和`v-for`优先级的变更
- `v-bind="object"`合并策略的变更
- `v-for`中的`ref`不再注册为`ref`数组

### 组件
- 只能使用普通函数创建功能组件
- `functional`属性及选项被遗弃
- 使用`defineAsyncComponent`创建异步组件

### 渲染函数
- 渲染函数的API变更
- `$scopedSlots`property已删除，所有插槽都通过`$slots`作为函数暴露
- 自定义指令的API和组组件的生命周期保持一致
- 某些过渡`class`被重命名
- 组件的`watch`选项和实例的`$watch`方法不再支持点分隔字符串路径
- 应用程序容器的`innerHTML`将替换为根组件模版

### 其他改变
- `destroyed`生命周期被重命名为`unmounted`，`beforeDestroy`被重命名为`beforeUnmount`
- 组件的默认属性不能访问`this`
- `data`选项应使用声明为函数
- `mixin`的`data`与组件的`data`选项只进行简单的合并
- `attribute`强制行为的变更
- `<template>`在没有特殊的指令标记时，将被视为普通的元素，而不是渲染其内容

### 移除API
- 按键修饰符
- 移除`$on`，`$off`和`$once`实例方法
- 移除过滤器，建议使用计算属性或方法
- 移除`inline-template`属性
- 移除`$destroy`实例方法

## 全局API——`createApp`
`Vue 2.x`有许多全局API和配置，这些API和配置可以全局改变`Vue`的行为。例如，要创建全局组件，可以使用`Vue.component`。

从技术上讲，`Vue 2`没有“app”的概念，我们定义的应用程序只是通过`new Vue()`创建的根`Vue`实例。从同一个`Vue`构造函数创建的每个根实例共享相同的全局配置，因此：
- 全局配置使得在测试期间很容易意外地污染其他测试用例；
- 全局配置使得在同一页面上的多个“app”之间共享同一个 Vue 副本非常困难。

`Vue 3.x`中引入了一个新的全局API：`createApp`，调用`createApp`返回一个应用实例：
```js
import { createApp } from 'vue'

const app = createApp({})
```

任何全局改变 Vue 行为的 API 现在都会移动到应用实例上，以下是当前全局 API 及其相应实例 API 的表：

|2.x 全局 API|3.x 实例 API (app)|
|----|----|
|Vue.config|app.config|
|Vue.config.productionTip|移除|
|Vue.config.ignoredElements|app.config.isCustomElement|
|Vue.component|app.component|
|Vue.directive|app.directive|
|Vue.mixin|app.mixin|
|Vue.use|app.use|

使用`createApp(/* options */)`初始化后，应用实例`app`可用于挂载具有`app.mount(domTarget)`。

Vue 3 应用程序实例还可以提供可由应用程序内的任何组件注入的依赖项：
```js
// 在入口
app.provide('guide', 'Vue 3 Guide')

// 在子组件
export default {
  inject: {
    book: {
      from: 'guide'
    }
  },
  template: `<div>{{ book }}</div>`
}
```

在应用程序之间共享配置 (如组件或指令) 的一种方法是创建工厂功能，如下所示：
```js
import { createApp } from 'vue'
import Foo from './Foo.vue'
import Bar from './Bar.vue'

const createMyApp = options => {
  const app = createApp(options)
  app.directive('focus' /* ... */)

  return app
}

createMyApp(Foo).mount('#foo')
createMyApp(Bar).mount('#bar')
```

## 全局 API Treeshaking

在`Vue 3`中，全局和内部`API`都经过了重构，并考虑到了`tree-shaking`的支持。因此，全局`API`现在只能作为`ES`模块构建的命名导出进行访问。例如，我们想要手动操作`DOM`：
```js
import { nextTick } from 'vue'

nextTick(() => {
  // 一些和DOM有关的东西
})
```

受影响的`API`如下：
- Vue.nextTick
- Vue.observable (用`Vue.reactive`替换)
- Vue.version
- Vue.compile
- Vue.set
- Vue.delete

除了公共`API`，许多内部组件/帮助器现在也被导出为命名导出，只有当编译器的输出是这些特性时，才允许编译器导入这些特性，例如以下模板：
```js
<transition>
  <div v-show="ok">hello</div>
</transition>
```

被编译为类似于以下的内容：
```js
import { h, Transition, withDirectives, vShow } from 'vue'

export function render() {
  return h(Transition, [withDirectives(h('div', 'hello'), [[vShow, this.ok]])])
}
```

这实际上意味着只有在应用程序实际使用了`Transition`组件时才会导入它。换句话说，如果应用程序没有任何 `Transition`组件，那么支持此功能的代码将不会出现在最终的捆绑包中。

## v-model

- `v-model`prop和事件默认名称已更改
- `v-bind`的`.sync`修饰符和组件的`model`选项已移除，可用`v-model`作为代替
- 可以在同一个组件上使用多个`v-model`进行双向绑定
- 可以自定义`v-model `修饰符

在`Vue 2`中，开发者使用`v-model`指令必须使用名为`value`的`prop`。如果开发者出于不同的目的需要使用其他的`prop`，他们就不得不使用`v-bind.sync`。

在`Vue 3`中，双向数据绑定的`API`已经标准化，减少了开发者在使用`v-model`指令时的混淆并且在使用 `v-model`指令时可以更加灵活。

在`3.x`中，自定义组件上的`v-model`相当于传递了`modelValue prop`并接收抛出的 `update:modelValue`事件：
```js
<ChildComponent v-model="pageTitle" />

<!-- 简写: -->

<ChildComponent
  :modelValue="pageTitle"
  @update:modelValue="pageTitle = $event"
/>
```

若需要更改`model`名称，而可以将一个`argument`传递给`model`：
```js
<ChildComponent v-model:title="pageTitle" />

<!-- 简写: -->

<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

这也可以作为`.sync`修饰符的替代，而且允许我们在自定义组件上使用多个`v-model`：
```js
<ChildComponent v-model:title="pageTitle" v-model:content="pageContent" />

<!-- 简写： -->

<ChildComponent
  :title="pageTitle"
  @update:title="pageTitle = $event"
  :content="pageContent"
  @update:content="pageContent = $event"
/>
```

## key attribute

- 对于`v-if`/`v-else`/`v-else-if`的各分支项`key`将不再是必须的，因为现在`Vue`会自动生成唯一的`key`
- `<template v-for>`的`key`应该设置在`<template>`标签上 ，而不是设置在它的子节点上

## v-if 与 v-for 的优先级对比

两者作用于同一个元素上时`v-if`会拥有比`v-for`更高的优先级。由于语法上存在歧义，建议避免在同一元素上同时使用两者。比起在模板层面管理相关逻辑，更好的办法是通过创建计算属性筛选出列表，并以此创建可见元素。

## v-bind 合并行为

`v-bind`的绑定顺序会影响渲染结果，如果一个元素同时定义了`v-bind="object"`和一个相同的单独的`property`，那么声明绑定的顺序决定了它们如何合并。
```html
<!-- template -->
<div id="red" v-bind="{ id: 'blue' }"></div>
<!-- result -->
<div id="blue"></div>

<!-- template -->
<div v-bind="{ id: 'blue' }" id="red"></div>
<!-- result -->
<div id="red"></div>
```

## v-for的Ref数组
在`Vue 2`中，在`v-for`里使用的`ref attribute`会用`ref`数组填充相应的`$refs property`。在`Vue 3`中，这样的用法将不再在`$ref`中自动创建数组。要从单个绑定获取多个`ref`，需要将`ref`绑定到一个更灵活的函数上：
```html
<div v-for="item in list" :ref="setItemRef"></div>
```
```js
import { ref, onBeforeUpdate, onUpdated } from 'vue'

export default {
  setup() {
    let itemRefs = []
    const setItemRef = el => {
      itemRefs.push(el)
    }
    onBeforeUpdate(() => {
      itemRefs = []
    })
    onUpdated(() => {
      console.log(itemRefs)
    })
    return {
      itemRefs,
      setItemRef
    }
  }
}
```

## 函数式组件

- 在`3.x`中，函数式组件的性能提升可以忽略不计，因此建议只使用有状态的组件
- 函数式组件只能使用接收`props`和`context`的普通函数创建
- `functional attribute`在单文件组件 (SFC) `<template>`已被移除
- `{ functional: true }`选项在通过函数创建组件已被移除

在`Vue 3`中，所有的函数式组件都是用普通函数创建的，换句话说，不需要定义`{ functional: true }`组件选项。此外，现在不是在`render`函数中隐式提供`h`，而是全局导入`h`。

## 异步组件

在`Vue 2`中异步组件是通过将组件定义为返回`Promise`的函数来创建的：
```js
const asyncPage = () => import('./NextPage.vue')
```

或者，对于带有选项的更高阶的组件语法：
```js
const asyncPage = {
  component: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}
```

在`Vue 3`中，由于函数式组件被定义为纯函数，因此异步组件的定义需要通过将其包装在新的`defineAsyncComponent`助手方法中来显式地定义，同时`component`选项现在被重命名为`loader`，以便准确地传达不能直接提供组件定义的信息：
```js
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// 不带选项的异步组件
const asyncPage = defineAsyncComponent(() => import('./NextPage.vue'))

// 带选项的异步组件
const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

## 渲染函数 API

在`Vue 3.x`中，`h`是全局导入的，而不是作为参数自动传入渲染函数：
```js
import { h } from 'vue'

export default {
  render() {
    return h('div')
  }
}
```

## 统一 Slot

在`Vue 3.x`中，插槽被定义为当前节点的子对象，当你需要以编程方式引用作用域`slot`时，它们现在被统一到`$slots`选项中。

## 自定义指令

在`Vue 3`中，将自定义指令的API与组件的生命周期保持一致：
- bind → beforeMount，指令绑定到元素后发生，只发生一次。
- inserted → mounted，元素插入父 DOM 后发生。
- beforeUpdate，在元素本身更新之前调用。
- 移除`update`，改用`updated`。
- componentUpdated → updated，组件和子级被更新，调用这个钩子。
- beforeUnmount，在卸载元素之前调用。
- unbind -> unmounted，指令被移除，调用这个钩子。

## 过渡的 class 名更改

过渡类名`v-enter`修改为`v-enter-from`、过渡类名`v-leave`修改为`v-leave-from`，变得更加明确易读。

## 在 prop 的默认函数中访问 this

在属性的默认函数中无法访问`this`，替代方案为：
- 把组件接收到的原始 `prop`作为参数传递给默认函数；
- 注入`API`可以在默认函数中使用：
```js
import { inject } from 'vue'

export default {
  props: {
    theme: {
      default (props) {
        // `props` 是传递给组件的原始值。
        // 在任何类型/默认强制转换之前
        // 也可以使用 `inject` 来访问注入的 property
        return inject('theme', 'default-theme')
      }
    }
  }
}
```

## Data 选项

`data`组件选项声明不再接收纯`JavaScript object`，只接受返回`object`的`function`。当合并来自`mixin`或`extend`的多个`data`返回值时，是浅层次合并的而不是深层次合并的(只合并根级属性)。

## attribute 强制行为

删除枚举`attribute`的内部概念，并将这些`attribute`视为普通的非布尔`attribute`。

2.x 和 3.x 行为的比较

|Attributes|v-bind value 2.x|v-bind value 3.x|HTML 输出|
|----|----|----|----|
|2.x “枚举attribute” i.e. contenteditable, draggable and spellcheck.|undefined, false|undefined, null|移除|
|2.x “枚举attribute” i.e. contenteditable, draggable and spellcheck.|true, 'true', '', 1, 'foo'|true, 'true'|"true"|
|2.x “枚举attribute” i.e. contenteditable, draggable and spellcheck.|null, 'false'|false, 'false'|"false"|
|其他非布尔attribute eg. aria-checked, tabindex, alt, etc.|undefined, null, false|undefined, null|移除|
|其他非布尔attribute eg. aria-checked, tabindex, alt, etc.|'false'|false, 'false'|"false"|

## 按键修饰符

- 不再支持使用数字 (即键码) 作为`v-on`修饰符
- 不再支持`config.keyCodes`

建议对任何要用作修饰符的键使用 kebab-cased (短横线) 大小写名称：
```html
<!-- Vue 3 在 v-on 上使用 按键修饰符 -->
<input v-on:keyup.delete="confirmDelete" />
```

## 事件 API

`$on`，`$off`和`$once`实例方法已被移除，应用实例不再实现事件触发接口，`$emit`仍然是现有`API`的一部分，因为它用于触发由父组件以声明方式附加的事件处理程序。

## 过滤器

在`Vue 3.x`中，`filters`已删除，不再受支持。建议用方法调用或计算属性替换它们。

## 内联模板 Attribute

在`Vue 2.x`中，为子组件提供了`inline-template attribute`，以便将其内部内容用作模板，而不是将其作为分发内容：
```html
<my-component inline-template>
  <div>
    <p>它们被编译为组件自己的模板</p>
    <p>不是父级所包含的内容。</p>
  </div>
</my-component>
```

在`Vue 3.x`中不再支持此功能，所有模板都直接写在`HTML`页面中。

- 使用`<script>`标签：
```js
<script type="text/html" id="my-comp-template">
  <div>{{ hello }}</div>
</script>
```
```js
const MyComp = {
  template: '#my-comp-template'
  // ...
}
```

- 默认 Slot
```html
<my-comp v-slot="{ childState }">
  {{ parentMsg }} {{ childState }}
</my-comp>
```
```html
<!--
  在子模板中，在传递时渲染默认slot
  在必要的private状态下。
-->
<template>
  <slot :childState="childState" />
</template>
```

## 片段

在`Vue 3`中，组件正式支持多根节点组件，即片段。

## 自定义元素

如果我们想添加在`Vue`外部定义的自定义元素 (例如使用`Web`组件`API`)，我们需要“指示”`Vue`将其视为自定义元素。以下面的模板为例：
```html
<plastic-button></plastic-button>
```

在`Vue 3.0`中，此检查在模板编译期间执行指示编译器将`<plastic-button>`视为自定义元素：

- 如果使用生成步骤：将`isCustomElement`传递给`Vue`模板编译器，如果使用`vue-loader`，则应通过 `vue-loader`的`compilerOptions`选项传递：
```js
// webpack 中的配置
rules: [
  {
    test: /\.vue$/,
    use: 'vue-loader',
    options: {
      compilerOptions: {
        isCustomElement: tag => tag === 'plastic-button'
      }
    }
  }
  // ...
]
```

- 如果使用动态模板编译，请通过`app.config.isCustomElement`传递：
```js
const app = Vue.createApp({})
app.config.isCustomElement = tag => tag === 'plastic-button'
```

自定义元素规范提供了一种将自定义元素用作自定义内置模板的方法，方法是向内置元素添加`is`属性。在`Vue 3.0`中，我们仅将`Vue`对`is`属性的特殊处理限制到`<component>`tag。

- 在保留的`<component>`tag上使用时，它的行为将与`Vue 2.x`中完全相同；
- 在普通组件上使用时，它的行为将类似于普通 prop：
```html
<foo is="bar" />
```
  - 2.x 行为：渲染`bar`组件。
  - 3.x 行为：通过`is`prop渲染`foo`组件。

- 在普通元素上使用时，它将作为`is`选项传递给`createElement`调用，并作为原生`attribute`渲染，这支持使用自定义的内置元素。
```html
<button is="plastic-button">点击我！</button>
```
  - 2.x 行为：渲染 plastic-button 组件。
  - 3.x 行为：通过回调渲染原生的 button。
```js
document.createElement('button', { is: 'plastic-button' })
```

在`Vue 3.x`中引入了一个新的指令`v-is`，用于`DOM`内模板解析解决方案。`v-is`指令像一个动态的`2.x :is`绑定。

## 参考文档

https://v3.cn.vuejs.org/guide/migration/introduction.html#%E6%A6%82%E8%A7%88
