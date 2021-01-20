layout: post
title: "Vue 高级用法与精粹"
date: 2020-07-23 19:32:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- Document
tags:
- JavaScript
- Vue
- Framework
---
## 复用与组合

### Mixin

多重继承大致有两种模式，一种就是按照顺序将各个父类进行继承，例如当你需要一个类继承 A、B、C和D的特性，那么可以采用 A ⇒ B ⇒ C ⇒ D 这样一个继承链去继承，这种继承方式的好处是知道每一步继承的父子关系。而大多数情况我们不需要知道这些，只要像 duck typing 一样拥有这些父类的特性即可，Mixin 混入机制就是来解决这个问题。Vue 中也提供给了这样的机制：

```jsx
var myMixin = {
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

var Component = Vue.extend({
  mixins: [myMixin]
});
```

比较好理解的，像 methods、computed 这种本身是对象类型的属性，可以直接进行 merge，所遵循的混合规则如下：

- 数据对象在内部会进行递归合并，并在发生冲突时**以组件数据优先**
- 同名钩子函数将合并为一个数组，数组中的所有函数都将在此钩子生效时执行

混入也支持全局注入，但是需要注意！这会影响到之后的每一个 Vue 组件实例（不建议使用）：

```jsx
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})
```

## 自定义指令

除了 v-model 和 v-show 等自带的指令之外，也可以自定义指令。自定义指令可以全局注册，也可以在组件内进行局部注册（指令在标签中必须要以 v- 开头）：

```jsx
Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})
```

```jsx
directives: {
  focus: {
    inserted: function (el) {
      el.focus()
    }
  }
}
```

上面代码中出现的 inserted 是指令所在的钩子，指令的钩子有以下五种：

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

上述的钩子函数的参数有四个，分别是 el、binding、vnode、oldVnode，其中，指令的值需要从 binding.value 中获取。                                                             

自定义指令还可以添加动态参数，v-someDirection:[arg]，使用 binding.arg 来获取动态的指令：

```html
<div id="dynamicexample">
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>
```

```jsx
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

// 回调函数可以简写，只支持 bind 和 update 两个钩子同时触发时
Vue.directive('pin', function (el, binding, vnode) {});
```

自定义指令也支持传入对象字面量 `<div v-demo="{ color: 'white', text: 'hello!' }"></div>`

## 渲染函数

如果想要使用纯 js 的方式来实现视图，可以使用 Vue 提供的 render 函数属性来进行渲染，下面是一个简单的例子：

```jsx
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // 标签名称
      this.$slots.default // 子节点数组
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

createElement 这个函数可以创建出一个虚拟 Dom，它所包含的信息会告诉 Vue 页面上需要渲染什么样的节点，包括及其子节点的描述信息，所以也被称为 Vnode。

```jsx
// @returns {VNode}
createElement(
  // 必填项一个 HTML 标签名、组件选项对象，或者 resolve 了上述任何一种的一个 async 函数
  // {String | Object | Function}
  'div',

  // 一个与模板中 attribute 对应的**数据对象**，可选项，{Object}。
  {},

  // 子级虚拟节点 (VNodes)，由 `createElement()` 构建而成，也可以使用字符串来生成“文本虚拟节点”，可选。
	// {String | Array}
  [
    '先写一些文字',
    createElement('h1', '一则头条'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```

上面讲到的数据对象，也就是一个节点中的一些属性描述：

```jsx
{
  // 与 `v-bind:class` 的 API 相同，
  // 接受一个字符串、对象或字符串和对象组成的数组
  'class': {
    foo: true,
    bar: false
  },
  // 与 `v-bind:style` 的 API 相同，
  // 接受一个字符串、对象，或对象组成的数组
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // 普通的 HTML attribute
  attrs: {
    id: 'foo'
  },
  // 组件 prop
  props: {
    myProp: 'bar'
  },
  // DOM property
  domProps: {
    innerHTML: 'baz'
  },
  // 事件监听器在 `on` 内，
  // 但不再支持如 `v-on:keyup.enter` 这样的修饰器。
  // 需要在处理函数中手动检查 keyCode。
  on: {
    click: this.clickHandler
  },
  // 仅用于组件，用于监听原生事件，而不是组件内部使用
  // `vm.$emit` 触发的事件。
  nativeOn: {
    click: this.nativeClickHandler
  },
  // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`
  // 赋值，因为 Vue 已经自动为你进行了同步。
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // 作用域插槽的格式为
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // 如果组件是其它组件的子组件，需为插槽指定名称
  slot: 'name-of-slot',
  // 其它特殊顶层 property
  key: 'myKey',
  ref: 'myRef',
  // 如果你在渲染函数中给多个元素都应用了相同的 ref 名，
  // 那么 `$refs.myRef` 会变成一个数组。
  refInFor: true
}
```

## 插件

通过 Vue.use() 方法使用插件，必须要在 new Vue() 实例化之前进行注册。Vue.use 会自动阻止多次注册相同插件，届时即使多次调用也只会注册一次该插件。

## 过滤器

Vue.js 允许你自定义过滤器（filters 可作为与 computed 同一级的属性进行配置），可被用于一些常见的文本格式化。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式：

```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中，过滤器可以串联，前一个的输出会作为下一个的输入 -->
<div v-bind:id="rawId | capitalize | formatId"></div>
```