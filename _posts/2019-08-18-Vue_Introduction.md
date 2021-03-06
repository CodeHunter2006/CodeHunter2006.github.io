---
layout: post
title: "Vue 介绍"
date: 2019-08-18 23:00:00 +0800
tags: Vue Javascript
---

![Vue](/assets/images/2019-08-18-Vue_Introduction_1.png)
PS:最近入坑 Vue，发现了个宝藏，要好好总结。

# Feature(功能/特性)

Vue 读作/vju:/(与 view 同音)，是一个基于 js 的前端渐进式框架。它按照 MVVM 模式设计，专注于 View 层。

## 界面及数据互动更新

Vue 对象可以与一个 Dom 对象绑定(两者可以是复杂嵌套对象)，底层数据的任何变化（经过过滤）都可以表达在界面上，而 input 类型的 Dom 对象的任何输入变化也都可以(经过过滤，将有效值
)自动写入绑定的底层数据对象。这就是 MVVM 模式的基本逻辑。

## 渐进式

Vue 的渐进式搭建方法可以让你从小到大不同规模下使用 Vue，从而进行平滑过度。

- 只用于 form 表单 -> 整个页面用 Vue 对象管理 -> 组件 -> 模板(大规模组件复用) -> VueX 复杂状态管理 -> vue-router 控制路由 -> 与服务端通信 axios

## 语法易用

Vue 语法设计非常人性化，稍做适应就能很容易的实现原本大量代码才能实现的功能，适合极限编程。

## 社区活跃、组件丰富

由于 Vue 优秀的性能、人性化的语法、完整的文档，吸引了大量开源社区力量开发了丰富的组件。例如知名的 element(饿了么组件)提供丰付的 UI 组件、storybook 响应式 UI 开发及测试组件等。同时文档、教学视频资源丰富，很容易上手，形成了良性社区。

前置技术：

- HTML+CSS+JavaScript，Vue 是一个 Web 前端框架，所以基础是需要的。
- MVC(Model View Controller)设计模式(24 种常见设计模式的综合模式)，Vue 是 MVVM 设计模式(MVC 设计模式的增强版)，所以理解基本设计模式有助于理解 Vue。
- Ajax(Asynchronous Javascript And XML)，Vue 可以看做 Ajax 的延伸，只对局部 HTML 元素用 JavaScrip 进行操作，通过 HTTP 请求与服务端通信。

# Structure(关键机制)

## 观察者模式

![Vue](/assets/images/2019-08-18-Vue_Introduction_2.png)
如图，A B C 是最底层被观察的对象，与 D E F G 一起形成一个观察模式网络结构。当 B 发生变化时，会通知到 D E 更新，进而通知到 F（如果 F 绑定了某个页面上的 Dom 元素，这时就会发生用户可见的变化）。

Vue 的基本设计思路就是通过观察者模式对 Vue 对象中的数据等进行监听，然后做出合理的动作，围绕变化的对象进行工作，避免轮训造成性能低下。而这种监听是自动建立的，只要我们不要违反 Vue 的设计框架，就能轻松使用。观察者与被观察者之间可以是多对多的关系，形成双向的树形结构，当被监听的对象发生变化时会通知相关对象，形成通知链。

- 对于一个 Vue 对象，其中的数据(data、computed 字段)都是被监听的，而这种监听实际上是在上面封装一层对象实现的，所以我们在使用时注意不要对 Vue 对象随意增加属性，要么在创建时就存在、要么通过`$set`操作增加，否则会导致数据变化不能被监听到。
- 对于一个 Vue 对象，我们通常通过 this 指针操作它的属性和函数，所以在函数绑定时要注意，箭头函数可能会造成 this 指针问题，除非你知道什么时候该用箭头函数。
- 在进行监听链设计时，要单向依赖，避免形成环装依赖。

## 缓存机制

Vue 对象的 computed 属性可以根据 data 数据动态计算结果（比如根据各项属性值综合决定称号），多个 computed 和被使用的数据(data、computed)之间可以形成网络结构。当 computed 被读取时，会形成一系列连锁计算。
为避免每次读取都要重新计算，Vue 会对上一次的运算结果进行缓存，只有在网络末端数据发生变化后才会重新计算，在方便的同时性能没有任何多余损耗。

## 虚拟 Dom 机制

通畅我们会直接给 Dom 对象的属性赋值，这样来最终改变显示。Vue 给每个 Dom 对象绑定了一个虚拟 Dom 对象，当有外部消息进入后(键盘、通信)、监听的 data 发生变化，会先把数据更新到虚拟 Dom 对象。如果一次消息可能导致同一个 Dom 属性的多次变更，那这些变更都会只作用在虚拟 Dom 对象，这样避免频繁改变 Dom 属性导致性能下降。然后，等这次通信结束、所有变更完成后，Vue 通过异步机制(promise、timeout 0 等方式)统一将变更赋值于 Dom 对象。

## 组件(对象)间通信

### 父组件调用子组件(向子对象传递消息)

- (不推荐)1.可以直接通过`this.$refs.子对象.方法()`调用，但是这种调用方法没有利用好 Vue 的数据驱动模式，并且耦合性高。
- (推荐)2.父对象通过 prop 参数在子对象构造时传入并绑定子对象监听函数，当参数对象发生改变时双方都能做出响应。

### 子组件调用父组件函数

- (不推荐)1.子组件通过`this.$parent.方法()`直接调用，同样的耦合姓高。
- (推荐)2.子组件通过`$emit`向上抛出某种消息，由父组件决定捕获这种消息后怎样处理。

### 非子父关系的组件间通信

通过对 VueX 对象的监听、mutation 赋值、getter 查询进行间接通信。

## 生命周期 Hook

<img src="/assets/images/2019-08-18-Vue_Introduction_3.png" alt="drawing" width="800" height="517"/>
如图，在Vue对象生命周期的各个阶段的时机点，你都有机会执行特定代码。只要在对象构造时注册各阶段的Hook函数即可。

# Key-Words(关键概念）

## Directive(指令)

刚开始接触 Vue 看到这个词会一头雾水，其实就是指标签上添加的动态属性。(大概是因为 Vue 开发者是后端程序员，导致的这种命名...)

Vue 的指令名称都是以`v-xxx`开头的，v 表示 Vue。例如：

```
<!-- 当Vue对象的isShow成员变量为true时显示标签 -->
<div v-show='isShow'> 你好 </div>
```

- 指令后边可以跟指令参数，如`<button @click.stop="doThis">`，具体每种指令可以加哪些参数要查看 API 文档。
- Vue 为最常用的`v-bind`、`v-on`分别指定了缩略形式`:`、`@`。(从这点也可以看出 Vue 的人性化设计)
- 可以添加自定义指令，这样可以把常用处理过程写成通用指令函数

## Scope(作用域)

一个 Vue 对象被创建后形成一个作用域，在其中以`this`指针来调用 Vue 对象的变量(data)和函数(method)，一个 Vue 对象通过 id 与 Dom 对象绑定。

```
<!-- html -->
<div id="app">
  {{ message }}
</div>

// javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

- 有些情况使用箭头函数会破坏变量的作用域，所以在 Vue 开发时使用箭头函数要注意。

## expression（表达式）

在后端开发中，我们经常会用到 View 模板，其本质就是把页面上一些静态内容用动态代码计算得出结果后填入。Vue 也有相同的功能，而且遍及各处。

### 在指令(v-bind/v-if 等)后面的字符串中使用表达式

```
<img :src="'/path/to/images/' + fileName">
```

- 指令字符串中的表达式有多种用法:普通语句、三目运算符等，需查看 API 文档。

### 将模板内容替换为表达式

{% raw %}
在非指令的 html 文本中通过后端模板常用的`{{ }}`符号使用表达式
{% endraw %}

```
{% raw %}
<span>{{ msg }}</span>
{% endraw %}
```

## 数据成员字段：data/props/computed/filter

- data 字段存放最基本的成员变量，可被动态监听或赋值
- props 字段是 Vue 对象作为组件时，由调用者(父组件)传入的参数成员变量，同样可以提供动态监听或赋值
- computed 字段定义的变量可以对内部依赖的 data 进行监控，如果发生变化则改变自己的值，同时提供缓存功能避免重复计算
- filter 与 computed 类似，不同点是每次被读取都会重新计算一遍，不论它内部依赖的 data 对象是否发生变化。如果计算时使用的所有对象都是可监控的，一般用 computed；反之，如果有无法监控的对象，则使用 filter 来获得最新结果。

## 函数成员字段：methods/Lifecycle Hooks/watch

- methods 字段可以定义属于本对象的函数，可以在本作用域的任何地方通过`this.xxx()`的形式调用
- Lifecycle Hooks 前面已经说过，定义的函数可以根据生命周期的各个阶段被回调。
- watch 与 computed 类似，可以监控某成员变量，重点在于这个变量发生变化时执行的函数动作

## 其它字段：component/mixin/slot

- component(组件) 用于指定当前 Vue 对象会用到哪些组件。
- mixin(混合) 可以实现类似面向对象的继承机制
  - 具有相同成员时，以当前组件成员为准
  - 具有相同 hook 时，都会被调用，被 mixin 的会先被调用
  - 可以在全局范围 mixin，将影响所有 Vue 对象，所以要小心
  - 可以设定全局的自定义混合策略
- slot(插槽) 为定制 component 提供了一种机制，可以在 component 特定位置插入调用 component 时才决定的 html 结构

# Convention(惯例约定)

# 常用语法

## 数组元素赋值

Vue 数组对象的元素直接通过`arr[0] = {...}`赋值，会由于没有调用重写的函数而没有通知监听者更新。
需要使用下面函数才能正常更新数组：

```Javascript
数组对象赋值'='(是整体数组，不是数组元素)
push
pop
shift
unshift
splice
sort
reverse
```

也可以使用 Vue 自带的赋值函数来增加一个元素，同时进行绑定：

```Javascript
// this 是Vue对象
this.$set(this.arr, 0, {...})

// 对于非数组元素可以用
this.$set(this.obj, "newAttribuit", 123)
```

## watch 的两个可选参数

通常的 watch 定义是一个函数:

```Javascript
watch: {
    watchedValue(newValue, oldValue) {
      // xxx
    }
}
```

也可以增加两个可选参数，形式如下：

```Javascript
  obj: {
    handler(newValue, oldValue) {
      // xxx
    },
    immediate: true,  // 第一次绑定时就调用(通常刚绑定是不调用的)
    deep: true  // 子孙对象属性变化时也监听(通常只监听要监听的对象本身的属性或元素)
```

# 性能优化(包括一般 Javascript 优化)

由于 JS 运行于 UI 线程，所以出现性能问题后会直接影响用户体验。

- 过多的 watch、computed
- 描述
  由于 watch 特别方便，所以在代码中可能会滥用，当处理数据量增长到 500 ～ 1000 以上，性能问题开始显现。
- 方案
  - 减小 watch 的范围，能对一个元素 watch 就不要对一个数组 watch
  - 减小 watch 处理时的嵌套循环，由于 watch 触发较多，如果其中存在 O(n^2)时间复杂度以上的循环，就很可能造成性能问题
  - computed 与 watch 相比有缓存，可以好很多，但同样会产生类似问题，在性能遇到问题时也要和 watch 同等对待

* 列表元素频繁析构、重建
* 描述
  由于列表更新机制，每次全部删除列表元素后再重新添加，在量大时，会导致性能较差
* 方案
  - Vue 列表元素可以绑定到一个数组对象，利用数组自带的 filter、map 操作元素时，性能会较好。
  - 可以固定一些列表元素，绑定数组对象，更新时只通过修改元素属性内容变更，避免析构元素。

- 大量对象创建、释放
- 描述
  通常在设计时，应该避免客户端承受过多数据量，因为客户端的用户难以消化所有信息，同时也会造成通信过程中有大量的对象创建和释放
- 方案
  - 与界面元素绑定，超出界面显示数量的数据不处理，减少对象创建和释放

* 大数据量 json 解析 parse 较慢
* 描述
  大数据量情况下做什么都慢，不论是输入操作还是 json 转对象
* 方案
  将大数据量分割成小数据量进行处理

- 过多的消息通信
- 描述
  过多的消息通信(无论是发送还是接收，达到几百条)会影响性能
- 方案
  - JS 无解
  - 通过于服务端协商协议，聚合消息，减少通知次数

* 一些不必要的重排影响性能
* 描述
  有些 flex 自适应的容器控件，其中的元素少量变化时可能引起大面积的重排。现象是 UI 线程卡顿，通过 Chrome 的 Performence 查看 Rendering 耗时较多。
* 方案
  - 这种情况往往伴随着 flex 自适应容器，需要设定`overflow=hidden`、`min-height=xxx`等属性，将不应被重排的容器尺寸锁定在一定范围。
