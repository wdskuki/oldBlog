---
layout: post
title: 'Jquery中.attr()和.data()的区别'
description: 'Jquery'
tags: Jquery
---





> $.attr()和$.data()本质上属于`DOM属性`和`Jquery对象属性`的区别。

## Jquery对象属性和DOM属性

一个简单的例子

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Jquery中.attr和.data的区别</title>
    </head>
    <body>
        <p id="app" data-foo="hello"></p>
    </body>
    <script type="text/javascript" src="http://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
    <script type="text/javascript">
        var getAttr1 = $('#app').attr('data-foo');
        var getData1 = $('#app').data('foo');
        console.log('getAttr1: ' + getAttr1); //hello
        console.log('getData1: ' + getData1); //hello

        $('#app').attr('data-foo', 'world'); //$.attr 设置DOM元素属性值
        var getAttr2 = $('#app').attr('data-foo');
        var getData2 = $('#app').data('foo');
        console.log('getAttr2: ' + getAttr2); //world
        console.log('getData2: ' + getData2); //*** hello ***

        $('#app').data('foo', 'WORLD'); //$.data 设置DOM node属性值
        var getAttr3 = $('#app').attr('data-foo');
        var getData3 = $('#app').data('foo');
        console.log('getAttr3: ' + getAttr3); //world
        console.log('getData3: ' + getData3); //*** WORLD ***

    </script>
</html>
```



* $.attr()每次都从DOM`元素`中取属性的值，即和视图中标签内的属性值保持一致。
  * $.attr('data-foo')会从标签内获得data-foo属性值；
  * $.attr('data-foo', 'world')会将字符串'world'塞到标签的'data-foo'属性中；
* $.data()是从`Jquery对象`中取值，由于对象属性值保存在内存中，因此可能和视图里的属性值不一致的情况。
  * $.data('foo')会从`Jquery对象`内获得foo的属性值，不是从标签内获得data-foo属性值；
  * $.data('foo', 'world')会将字符串'world'塞到`Jquery对象`的'foo'属性中，而不是塞到视图标签的data-foo属性中。

结合上面代码和解释，大家应该能够理解两者的区别。



## 小结

所以$.attr()和$.data()应避免混合用，也就是应该尽量避免如下两种情况的出现：

1. 通过$.attr()来进行set属性，然后通过$.data()进行get属性值；
2. 通过$.data()来进行set属性，然后通过$.attr()进行get属性值。

同时从性能的角度来说，建议使用$.data()来进行set和get操作，因为它仅仅修改的`Jquey对象`的属性值，不会引起额外的DOM操作。
