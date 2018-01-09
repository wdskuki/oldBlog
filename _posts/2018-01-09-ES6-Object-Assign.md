---
layout: post
title: 'Object.assign在Babel中无法转换的问题'
description: ''
tags: ES6
---

项目压缩打包后，主逻辑的JS文件大小也有1.6M，已经有些臃肿了。我们希望从减少三方库角度入手，减小包体积。排查了一圈，移除了几个占体积的三方库。其中有一个是[lodash](https://lodash.com/)，它在项目里主要用来做些数组处理。本来引用它的地方也就两个文件，另外它本身压缩后的体积就达100多K，有点"杀鸡用牛刀"的感觉。因此，我们直接移除lodash，原来对应的地方用自己写的逻辑替换了。

交给了测试同学，测试下来没问题，就部署上线；运行了几天，客服助理说有用户无法打开页面了。开始以为是个别情况的网络问题，后来得知反馈用户不止一个。感觉问题有些严重，马上组织同事一起排查。手头的机型无法无限用户爆出的问题，于是联系用户，同时针对性的hotfix了一些感觉有问题的地方，但还是没有解决。后来引导用户进入我们的测试环境，于是我们再测试环境回滚到上一个版本，解决页面可以正常显示了。可见，出问题的地方就是之前改动进入的。于是逐行代码比对`git diff`前后两个版本的差异代码，终于找到的问题。

## Object.assign

![](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2018-01/object-assign.jpg)

我们将之前lodash使用的地方(这里的`_`表示lodash)替换为`Object.assign`, 而恰恰是这个`Object.assign`导致了问题。

Object.assign是[ES6新增的方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign#Specifications),  我们通过查看打包之后的文件，发现Object.assign没有被转化为ES5方法！这就奇了怪了，我们的项目是用webpack构建的，同时也引入了babel-loader插件将相关ES6语法转换为ES5语法，为何这个没有被转换呢？

查阅网[上大牛的博客](http://www.ruanyifeng.com/blog/2016/01/babel.html)发现了如下惊人的大坑！

>  Babel默认之转换新的JS语法，而不转换新的API，以及一些再全局对象上的法官法（比如Object.assign）都不会转码！

原来如此，为此要解决Object.assign的无法转换为ES5的问题，可由以下两种方法解决：

**1. 引用babel-polyfill库**

```javascript
//命令部分
npm install --save babel-polyfill
...

//代码部分
require('babel-polyfill')
Object.assign({}, .....)
```

babel-polyfill库文件比较大，它会将所有ES6的语法和API进行ES5转换。如果只是单纯兼容Object.assign，有点小题大做了，解决问题的同时徒增了文件体积。因此这里可以使用方法2，它仅仅针对Object.assign进行变换：

**2. 安装webpack插件：babel-plugin-transform-object-assign**

```javascript
//命令
npm install --save-dev babel-plugin-transform-object-assign

//.babelrc文件
{
    "presets": [
        ["es2015"]
    ],
    "plugins": [
        "transform-object-assign"
    ]
}
```

如此一来，再次查看打包后的文件，发现Object.assign的地方进行了修改。DONE。

![](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2018-01/object-assign-transform-es5.jpg)