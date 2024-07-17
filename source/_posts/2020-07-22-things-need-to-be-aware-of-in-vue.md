---
layout: post
title: "Vue 中容易被忽视的小功能"
date: 2020-07-22 19:03:00
comments: true
categories: 
- Document
tags:
- JavaScript
- Vue
- Framework
---

在使用 Vue 2.0 开发项目的过程中，会发现有很多容易忽略或者令人迷惑的一些小方法和概念，本文针对这些”遗珠“进行了一个整理，方便分享和以后查阅。

<!-- more -->

## 组件通信

### Prop相关

prop 是父组件向子组件传递数据的一种方式，在父组件中使用 kebab-case 的方式传入数据，在子组件中可以添加 props 属性来获取传入的数据。这是一个单向的通信过程。

可以通过 v-bind 传入动态变量，也可以直接传入字符串值到子组件。

如果子组件中 props 属性中没有声明某些值，那么父组件传入的就是**非 prop 的 attribute**，对于绝大多数 attribute 来说，**从外部提供给组件的值会替换掉组件内部设置好的值**。所以如果传入 type="text" 就会替换掉 type="date" 并把它破坏！庆幸的是，**class 和 style attribute** 会稍微智能一些，即两边的值**会被合并（merge）起来，从而得到最终的值**。

可以使用 vm.$attr 属性来获取除 class 和 style 之外的非 prop 的 attribute，可以在父组件中使用 `v-bind="$attr"` 传入，还可以通过子组件中的 `inheritAttrs: true` 字段来控制是否接受父组件传入的非 prop 的 attribute。

## 事件

### 自定义事件

使用 v-on 绑定事件时，**最好使用 kebab-case 命名法来命名事件**，因为 v-on 事件监听器在 DOM 模板中会被自动转换为全小写 （因为 HTML 是大小写不敏感的）。

可以使用 `v-on="$listeners"` 来将所有的事件监听器指向这个组件的某个特定的子元素，它是一个对象，里面包含了作用在这个组件上所有的监听器（监听事件）。这样，在子组件上可以绑定 vm.$listeners 中所包含的监听事件。

### 事件监听

除了 $emit 和 v-on 这一对好基友之外，vue 还提供了类似 总线（bus）机制的观察者模式的事件监听方案：

- `$on(eventName, eventHandler)` 侦听一个事件
- `$once(eventName, eventHandler)` 一次性侦听一个事件
- `$off(eventName, eventHandler)` 停止侦听一个事件

### 修饰符

- sync
    - 作用：对 prop 进行简单的“双向绑定”，可以实现简单的父子数据同步
    - 使用：在父组件需要双向绑定的数据上添加 .sync 修饰符，例如 `<div><child :some-prop.sync="someProp"></child></div>` ；在子组件上使用 vm.$emit 触发数据更新，例如 `this.$emit('update:some-prop', val)`

## 组件

### 动态组件

动态组件是通过 <component :is="componentName"></component> 来实现的，但是在切换组件时，会将每个组件的原有状态丢失，这时需要添加一个 keep-alive 标签，例如：

```html
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

### 异步组件

Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染。

```jsx
Vue.component(
  'async-webpack-example',
  // 这个动态导入会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```

## Slot

### 编译作用域

父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

### default内容

可以使用 `<slot> Default content </slot>` 来设置分发内容的默认值，当没有内容传入时会显示该内容。

### 具名插槽

有多个插槽需要进行内容分发时，可以使用具名插槽，具体用法如下：

```html
<!-- 子元素设置具名插槽 -->
<header>
    <slot name="header"></slot>
</header>
```

```html
<!-- 父元素设置分发内容，由 v-slot 属性来指明具体的插槽名 -->
<template v-slot:header>
    <h1>Here might be a page title</h1>
</template>
```

### 作用域插槽

绑定在 <slot> 元素上的 attribute 被称为插槽 prop，为了让父元素能够访问到子组件中的数据，可以通过 v-bind 进行作用域的绑定，具体代码如下：

```html
<!-- 子元素中需要在 slot 中使用 v-bind 绑定具体变量 -->
<slot v-bind:user="user">
    {{ user.lastName }}
</slot>
```

```html
<!-- 父元素通过 v-slot 来指明子元素所 bind 的所有数据 -->
<child>
	  <template v-slot="slotProps">
		    {{ slotProps.user.firstName }}
	  </template>
</child>
```

***v-slot 的缩写为 #***，这个在具名插槽中有

### 结构插槽Prop

作用域插槽的内部工作原理是将你的插槽内容包裹在一个拥有单个参数的函数里，这意味着 v-slot 的值实际上可以是任何能够作为函数定义中的参数的 JavaScript 表达式。

```html
<!-- prop 重命名 -->
<current-user v-slot="{ user: person }">
	  {{ person.firstName }}
</current-user>
```

```html
<!-- 为 prop 指定默认值 -->
<current-user v-slot="{ user = { firstName: 'Guest' } }">
	  {{ user.firstName }}
</current-user>
```

## 边界情况处理

### 访问元素&组件

- 访问根实例

    一般不建议触达另一个组件实例内部或手动操作 DOM 元素

    如果必须这么做，可以通过 this.$root 来访问根实例中的 data、computed 和 methods 等属性

- 访问父级元素

    同访问根实例，父级元素的访问是通过 this.$parent

- 访问子元素

    你可以通过 ref 这个 attribute 为子组件赋予一个 ID 引用，父组件可以通过 他会 this.$refs.someId 来访问子元素的所有属性

- 依赖注入

    当出现了多层嵌套的组件时，使用 $parent 属性无法很好获取到特定父级元素，这时需要采用依赖注入的方法 provide 和 inject，相当于爷组件和孙组件之间的 props：

    ```jsx
    // 爷组件中添加 provide 属性
    provide: function () {
      return {
        getMap: this.getMap
      }
    }

    // 孙组件中添加 inject 属性
    inject: ['getMap']
    ```

- 循环引用
    - 组件递归引用要避免
    - 不同组件间相互引用，如果产生循环引用（例如 Tree 文件系统或级联系统，模块系统会不知道到底哪个是初始组件），可以采用如下两种方式来解决：
        - 在生命周期钩子 beforeCreate 中进行组件的引入：

            ```jsx
            beforeCreate: function () {
              this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
            }
            ```

        - 引用组件时采用异步方式：

            ```jsx
            components: {
              TreeFolderContents: () => import('./tree-folder-contents.vue')
            }
            ```

## 过渡&动画

### 单元素/组件的过渡

使用 <transition name="someTransition"></transition> 封装所要添加过渡的元素。这种方式适用于以下四种情况（都是单个组件显隐）：

- 条件渲染 (使用 `v-if`)
- 条件展示 (使用 `v-show`)
- 动态组件
- 组件根节点

封装好元素后，还需要提供相应的 css 样式，分别由六种状态的样式名，需要根据 name 属性进行相应的设置：someTransition-enter 表示进入过渡开始状态，someTransition-enter-active 表示进入过渡生效状态，someTransition-enter-to 表示进入过渡结束状态，someTransition-leave，someTransition-leave-active，someTransition-leave-to。

也可以自定义类名，相对应上面六种默认的样式，可以在 transition 标签中添加 enter-class，enter-active-class，enter-to-class，leave-class，leave-active-class，leave-to-class。

transition 标签中可以指定进入和离开过渡的时长，可以通过 `:duration="{ enter: 500, leave: 800 }"` 实现

transition 标签中可以指定过渡各阶段的钩子，以用来绑定相应的事件，包括 @before-enter、@enter、@after-enter、@enter-cancelled、@before-leave、@leave、@after-leave、@leave-cancelled 八种钩子：

```jsx
// 在 enter 和 leave 钩子事件，如果没有 css 控制元素过渡，那么一定需要有 done() 回调
// 否则 enter 和 leave 钩子事件会同时完成
beforeEnter: function (el) {},
enter: function (el, done) {
    done()
}
```

transition 标签中可以使用 appear 属性来设置元素初始化过渡的样式，用法与 enter 和 leave 相同，可以使用 someTrasition-appear 类或者自定义类名属性，当然也支持钩子的实现。

### 多元素的过渡

多元素使用过渡时，可以在最外层使用 transition 标签进行包装，用法与单元素相同，但是有一点需要注意——使用 v-if 或者 v-for 设置多个元素时，一定要在每个元素上绑定自己的 key。

多个元素切换显隐状态时，Vue 提供了一个 mode 属性来进行平滑过渡：

```jsx
<transition name="fade" mode="out-in"></transition>
```

### 多组件的过渡

多组件的过渡，使用之前所提到的动态组件即可实现，在过渡时可以在组件外包装一层 transition，指定过渡的样式名 name 属性，就可以使用上面提到的样式或钩子进行过渡样式的实现，当然也可以添加所需要的 mode：

```jsx
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```

对于列表的过渡，可以使用 <transition-group> 包装 v-for 渲染出的列表，指明过渡样式名 name 属性，即可用上面相同的方式进行过渡的实现。在列表打乱或重新排序时，Vue 提供了 someTransition-move 的类名来实现：

```html
<button v-on:click="shuffle">Shuffle</button>
<transition-group name="flip-list" tag="ul">
    <li v-for="item in items" v-bind:key="item">
	      {{ item }}
    </li>
</transition-group>
```

```jsx
new Vue({
  el: '#flip-list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9]
  },
  methods: {
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```

```css
.flip-list-move {
  transition: transform 1s;
}
```