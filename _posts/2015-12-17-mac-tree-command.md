---
layout: post
title: 'Mac OS X下的tree命令'
description: 'tree'
category: Mac
tags: Mac
---

Mac下**没有**Linux下的`tree`命令，但是今天找到[一个非常牛逼的一条指令](http://www.the5fire.com/mac-ternimal-tree-command.html)可以模拟`tree`命令的功能:

```
find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'
```


你可以将他添加到你的`.bash_profile`文件下:

```
echo "alias tree=\"find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'\"" >> ~/.bash_profile && source ~/.bash_profile
```

这样你可以在你的Mac终端下是用`tree`命令实现类似的效果。

效果图：

![](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12-17/1340230-112e1f86b43d633d.png)
目前不太清楚这里面涉及到的正则表达式机制。先占个坑，等有空好好分析一下。
