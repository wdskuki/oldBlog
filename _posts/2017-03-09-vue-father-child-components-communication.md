---
layout: post
title: 'Vue父子组件通信（一）'
description: 'Vue父子组件通信（一）'
category: JS
tags: Vue
---



父子组件通信的场景很多，例如：

>父 -> 子：页面上(父)有两个自定义按钮(子)，每个按钮的颜色不一样；
>
>子 -> 父：点击页面(父)上按钮(子)，页面(父)上一个数字发生了递增



## 父 -> 子

> 我们以上面讲到的自定义按钮为例子来说明父组件如何调用子组件。



### 自定义按钮

定义一个自定义按钮组件:`wds-button`。如果想使用它，可以在html中这么写：

```html
<wds-button><wds-button>
```

得到的效果如下：

![自定义按钮](http://omcg5d6c2.bkt.clouddn.com/image/vue-parent-child-commucation/single-button.png)

具体实现逻辑：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title></title>
    <style scoped>
        .wds-button{
            line-height: 1;
            white-space: nowrap;
            cursor: pointer;
            background: #fff;
            border: 1px solid #bfcbd9;
            color: #1f2d3d;
            outline: none;
            margin: 0;
            padding: 10px 15px;
            font-size: 14px;
            border-radius: 4px;
        }
    </style>
</head>

<body>
<div id="app">
    <wds-button></wds-button>
</div>
</body>

<template id="template-wds-button">
    <button class="wds-button">默认按钮</button>
</template>

<script src="http://cdn.bootcss.com/vue/2.1.10/vue.min.js"></script>
<script>
    var wdsButton = {
        template: '#template-wds-button'
    }

    var vm = new Vue({
        el: '#app',
        components: {
            'wds-button': wdsButton
        }
    })

</script>
</html>
```



### 多个不同颜色的自定义按钮

如果我们想实现在页面上显示两个不同颜色自定义按钮呢？我们可能希望通过如下的方式来使用这个自定义按钮，来实现效果。

```html
<wds-button color="red"></wds-button>
<wds-button color="yellow"></wds-button>
```

如果说`wds-button`是子组件，那么使用该组件的当前页面则为父组件。这里的color是子组件的自定义属性名（因为用法和html的标签属性的用法一致），而"red"、"yellow"是父组件传递给子组件的值。类似于函数的形参和实参。

因此如果按照上面的那种调用组件的方法，总结一下：

> 子组件得拥有一个自定义的属性：color;
>
> 父组件通过子组件(wds-button)中的自定义属性名(color)，可以将不同的值(red/yellow)**传递**给子组件，从而达到复用子组件的效果。



那具体在代码层面如何实现呢？

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title></title>
  <style scoped>
  .wds-button{
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    background: #fff;
    border: 1px solid #bfcbd9;
    color: #1f2d3d;
    outline: none;
    margin: 0;
    padding: 10px 15px;
    font-size: 14px;
    border-radius: 4px;
  }
  .wds-button--red{
    background: red;
  }
  .wds-button--yellow{
    background: yellow;
  }
  </style>
</head>
<body>
  <div id='app'>
    <wds-button></wds-button>
    <wds-button color="red"></wds-button>
    <wds-button color="yellow"></wds-button>
  </div>
</body>

<template id='template-wds-button'>
  <button v-bind:class="[color ? 'wds-button wds-button--' + color: 'wds-button']">默认按钮</button>
</template>

<script src="http://cdn.bootcss.com/vue/2.1.10/vue.min.js"></script>
<script>
var wdsButton = {
  props: {
    color: {
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
</script>
</html>
```

![不同颜色的自定义按钮](http://omcg5d6c2.bkt.clouddn.com/image/vue-parent-child-commucation/multi-buttons.png)



这里子组件的JS对象中需要申明一个属性(`props`)，告知调用者暴露的自定属性名字是什么（这里props里又一个属性color），同时给出对应的类型和默认值（type & default）。

```javascript
var wdsButton = {
  props: {
    color: {
      type: String,
      default: ''
    }
  },
  template: '#template-wds-button'
}
```



同时在视图模版中，给自定义按钮绑定一个css class，其值由属性color的值来定。

```html
<template id='template-wds-button'>
  <button v-bind:class="[color ? 'wds-button wds-button--' + color: 'wds-button']">默认按钮</button>
</template>
```



按照上面的方式来，就实现了一个能够传入颜色属性的自定义的组件按钮。同时这个`wds-button`组件可以拥有了了一份属于它自己的类似于接口的文档:

**Props**

| 参数    | 说明   | 类型     | 可选值       | 默认   |
| ----- | ---- | ------ | --------- | ---- |
| color | 按钮颜色 | String | red/color | —    |




