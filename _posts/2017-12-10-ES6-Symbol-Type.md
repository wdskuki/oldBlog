---
layout: post
title: 'ES6中Symbol类型'
description: ''
tags: ES6
---



>  JS中对象的属性名是字符串型，对象属性的动态增减有时出现属性名冲突的情况。

举个例子：

```javascript
//confg.js
const config = {
  protocol : 'https:',
}
module.exports = config
```

假设很多模块都需要`import config from 'config'`来读取里面配置信息。如果有工程师在无意中设置了例如`config.protocol='http'` ，那么这个操作可能会影响到其他引用这个对象的模块。而具体config.protocol的当前值是什么取决于不同模块的运行顺序和程序当前的运行状态，这样人为增加了项目复杂度和维护成本。

ES6引入了`Symbol`类型，所有`Symbol`类型的值都是唯一的，从语法层面保证避免出现上述情况。



## Symbol类型

Symbol类型和其他一些基本类型地位相同(数值型、字符型、布尔类型、...)，它表示一个独一无二的值。

```javascript
let s = Symbol()
typeof s; //“symbol”
```

typeof可以看到，这里`s`的类型为`symbol`类型。另外特别注意的是，定义`Symbol`类型的时候无需加`new`，否则会报错。



两个独立定义的`Symbol`的类型，值不相等。

```javascript
let s1 = Symbol()
let s2 = Symbol()
s1 == s2 // false
```

正如之前描述的那样，每次`Symbol（）`都会生成一个唯一的值。

还有一种定义方法，`Symbol`允许输入一个字符串，方便区分。

```javascript
let s1 = Symbol('foo')
let s2 = Symbol('foo')
s1 == s2 //false
```



## 对象属性为Symbol

```javascript
//xxx.js
let HELLO = Symbol('hello')
let foo = {
	[HELLO]: 'world',
}
foo[HELLO] //'world'

module.exports = foo
```

我们显示定义一个`Symbol` 类型的变量`HELLO`, 然后对象`foo`中的属性设置为`HELLO`，这里注意一点的是`Symbol`的属性需要用中括号`[]`包住，否则会被作为字符串属性的`HELLO`，而非`Symbol`类型；同时访问的属性的时候也需用`[]`操作符访问，否则读取的是字符串类型的属性。

其他模块在调用`foo`对象的时候无法修改属性`HELLO`的值：

```javascript
foo.HELLO //undefined， 读取字符串属性‘HELLO’
let Hi = Symbol('hello')
foo[Hi] // undefined， 虽然读取属性都是Symbol('hello')，但Symbol对象具有唯一性
foo[Symbol('hello')] // undefined, 理由同上
```



## 修改开头的例子

```javascript
//config.js
const PROTOCOL = Symbol('protocol')
const config = {
  [PROTOCOL] : 'https:',
}
module.exports = config
```

唯一需要注意的是，因为不支持点操作符读取`Symbol`类型的属性，所以其他模块文件在读取`config`的属性时，必须通过`[]`操作符访问；同时对对象属性的动态添加，属性类型也需`Symbol`，这样可以防止属性名冲突的问题。



## 结论

使用`Symbol`是可以有效避免对象属性冲突，但同时也增加了开发成本。

之前我们在增加属性的时候，一行代码即可搞定:`foo.hello = 'world'`；现在则需要先定义一个`Symbol`的属性名，然后再设置属性。

```javascript
const HELLO = Symbol('hello')
obj[HELLO] = 'world'
```

可以看到，后者需要额外多维护一个`Symbol`属性名。

两者之间如何选择需要视具体项目的而定。
