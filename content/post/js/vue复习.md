---
title: "Vue复习"
date: 2019-11-29T15:44:01+08:00
draft: false
---
# 时隔九月再次上手Vue

##### 注意项

1. 不要在选项属性或回调上使用箭头函数，比如 created: () => console.log(this.a) 或 vm.$watch('a', newValue => this.myMethod())。因为箭头函数并没有 this，this 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 Uncaught TypeError: Cannot read property of undefined 或 Uncaught TypeError: this.myMethod is not a function 之类的错误。

2. 任何复杂逻辑，你都应当使用计算属性
````
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
````

3. 计算属性缓存 vs 方法
计算属性会使用缓存，只有依赖的值更新后才会重新执行，方法每次执行

4. 计算属性 vs 侦听属性

watch更加通用监听某个属性变化后对其它属性的关联处理
计算属性有时会更加优雅
例如计算输入行与名，得到全名

5. 侦听属性
当异步会开销较大需要设置中间状态时使用

6. truthy（真值）
在 JavaScript 中，truthy（真值）指的是在布尔值上下文中，转换后的值为真的值。所有值都是真值，除非它们被定义为 假值（即除 false、0、""、null、undefined 和 NaN 以外皆为真值）。

7. v-if vs v-show
同计算属性与方法类似，v-if 每次销毁重建，v-show值为真时渲染，之后只是渲染

8. js中 === 与 == 

=== 会比较类型, ==不会
同时注意NaN,等特殊情况，详见https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness

