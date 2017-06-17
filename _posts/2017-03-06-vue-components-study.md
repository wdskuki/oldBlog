---
layout: post
title: 'Vue.js组件化开发实践'
description: 'Vue学习之父子组件通信'
category: JS
tags: Vue
---



## 什么是Vue.js

> Vue.js 是一套构建用户界面的 **渐进式框架**。它非常容易与其它库或已有项目整合，而无须从头开始重构整个项目；另一方面，Vue 完全有能力驱动采用**单文件组件**来开发的更为复杂的单页应用。
>



目前在我参与开发维护的项目中已经使用上了Vue.js的一些基本功能，下面两幅图来自项目截图。

![vue-in-cur-poj.png](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/vue-in-cur-poj.png)

![vue-in-cur-poj-2.png](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/vue-in-cur-poj-2.png)


## Vue.js基本功能

> * 视图元素响应式
> * 数据双向绑定，解放DOM操作
> * 只关注视图层，渐进式插件



### 视图元素响应式

```html
//视图
<div id="app">{{msg}}</div>
```

```javascript
//JS逻辑
var vm = new Vue({
  el: '#app',
  data: {
    msg: 'hello world'
  }
})
```



上图中，我们在视图里声明一个变量`msg`，它被包在一个双花括号"{{}}"中，以此表明它是一个Vue所管理的视图变量元素。同时，我们在JS中新建一个Vue对象，其中的`el`对应"#app"，表示改Vue对象管辖的视图范围为id是`app`所对应的区域；`data`中有一个`msg`属性，对应视图中的双花括号变量`msg`。一旦我们新建好这个Vue对象，所有对于该对象data属性中的msg进行操作，会同步反应在视图中的{{msg}}上，这个视图变量元素即具有响应式。



### 数据双向绑定，解放DOM操作

> * 双向绑定：可以理解为JS逻辑中的数据的更改会实时的反映在视图上；同时任何从视图中过来的数据或者事件，能够实时的反映在逻辑中。
> * 一旦视图和逻辑之间规定好需要绑定的数据和事件，那么业务逻辑就能专注数据处理，而无需手动管理DOM，这样就实现了视图和逻辑各司其职。

**一个例子：**

输入框中的输入的字符串实时显示在视图上；同时当用户点击视图中的一个按钮时，视图中的字符串反向之后在输出到视图上。

**DOM操作**

```html
//视图
<div id="app">
    <p id="msg"></p><input id="inputMsg" type="text">
    <p id="gsm"></p><button id="reverseMsg">Reverse</button>
</div>
```

```javascript
//逻辑
document.getElementById('inputMsg').addEventListener('input', function({
  document.getElementById('msg').innerText = this.value
}), false)

document.getElementById('reverseMsg').addEventListener('click', function({
  var msg = document.getElementById('msg').innerText
  document.getElementById('gsm').innerText = msg.split('').reverse().join('')
}), false)

```



**Vue处理**

```html
//视图
<div id="app">
    <p>{{msg}}</p><input v-model="msg" type="text">
    <p>{{gsm}}</p><button v-on:click="reverseMsg">Reverse</button>
</div>
```

```javascript
//逻辑
var vm = new Vue({
    el: '#app',
    data: {
        msg: "",
        gsm: ''
    },
    methods: {
        reverseMsg: function(){
            this.gsm = this.msg.split('').reverse().join('')
        }
    }
})
```





## Vue.js组件

> * 易维护
> * 易复用



组件系统是 Vue.js 另一个重要概念，因为它提供了一种抽象，让我们可以用独立可复用的小组件来构建大型应用。如果我们考虑到这点，几乎任意类型的应用的界面都可以抽象为一个组件树。

![components-concept.png](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/components-concept.png)


#### 一个自定义的按钮

```html
<div id='app'>
  <wds-button></wds-button>
</div>

<template id='template-wds-button'>
  <button class="wds-button">默认按钮</button>
</template>
```

```javascript
var wdsButton = {
  template: '#template-wds-button'
}

var vm = new Vue({
  el: '#app',
  components: {
    'wds-button': wdsButton
  }
})
```



##### 父组件传递数据到子组件: `props`

```html
<div id='app'>
  <wds-button></wds-button>
  <wds-button type="success" btn-name="成功按钮"></wds-button>
</div>

<template id='template-wds-button'>
  <button v-bind:class="['wds-button', type ? 'wds-button--' + type: '']">{{btnName}}</button>
</template>
```

```javascript
var wdsButton = {
    props: {
        btnName: {
            type: String,
            default: '默认按钮'
        },
        type: {
            type: String,
            default: ''
        }
    },
    template: '#template-wds-button'
}

var vm = new Vue({
  el: '#app',
  components: {
    'wds-button': wdsButton
  }
})
```



##### was-button 组件描述

__Props__

| 参数      | 说明   | 类型     | 可选值                    | 默认值  |
| ------- | ---- | ------ | ---------------------- | ---- |
| btnName | 按钮名字 | String | —                      | 默认按钮 |
| type    | 按钮类型 | String | success/warning/danger | —    |







##### 子组件将数据传回父组件：`自定义Event`

```html
<div id='app'>
  <wds-button v-on:wds-button-click-event="alertButtonName" type="warning" btn-name="警告按钮"></wds-button>
</div>

<template id='template-wds-button'>
  <button class="wds-button" v-bind:class="['wds-button', type ? 'wds-button--' + type: '']" v-on:click="wdsButtonClick">{{btnName}}</button>
</template>
```

```javascript
var wdsButton = {
    props: {
        btnName: {
            type: String,
            default: '默认按钮'
        },
        type: {
            type: String,
            default: ''
        }
    },
    methods: {
      wdsButtonClick: function(){
        this.$emit('wds-button-click-event', this.btnName, this.type)
      }
    },
    template: '#template-wds-button'
}

var vm = new Vue({
  el: '#app',
  components: {
    'wds-button': wdsButton
  },
  methods: {
    alertButtonName: function(btnName, type){
      alert(btnName)
    }
  }
})
```



##### was-button 组件描述

__Props__

| 参数      | 说明   | 类型     | 可选值                    | 默认值  |
| ------- | ---- | ------ | ---------------------- | ---- |
| btnName | 按钮名字 | String | —                      | 默认按钮 |
| type    | 按钮类型 | String | success/warning/danger | —    |

__events__

| 事件名称                   | 说明       | 回调参数          |
| ---------------------- | -------- | ------------- |
| wds-button-click-event | 点击按钮回调事件 | btnName, type |






## Vue.js单文件组件

> * 组件应该内聚自己的样式(HTML/CSS)和逻辑(JS)
> * 一个组件对应一个文件



文件和组件一一对应：` .vue`文件


![vue-file-sample.png](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/vue-file-sample.png)

`.vue`文件无法直接运行在浏览器上，通过**webpack + vue-loader**的方式来将Vue组件转化为JS模块。

> * **webpack**： 前端资源模块化管理打包工具。
> * **vue-loader**:  webpack下处理`.vue`文件的的插件。



一个例子：

一个页面中有多个按钮，每点击一个按钮弹出toast信息，`按钮`和`toast`都自定义。代码可以下载[这里](http://www.jianshu.com/writer#/notebooks/8411855/notes/9820034/preview)。


![vue-webpack-demo-directory.png](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/vue-webpack-demo-directory.png)




## 小结

> **前端技术日新月异，如果你今年还没开始学某门技能，明年就可以用不学它了。**

![end-emoji.jpeg](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/end-emoji.jpeg)
