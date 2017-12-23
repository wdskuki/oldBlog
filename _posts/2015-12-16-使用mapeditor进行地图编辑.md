---
layout: post
title: '使用mapeditor进行地图编辑'
description: ''
category: DQMSL
---



# 2. 使用mapeditor进行地图编辑


> DQMSL需要添加相关的活动地图(例如制作常规的讨伐活动),首先需要策划那边利用mapeditor进行地图编辑。我们以配置讨伐活动`area_394`为例来说明如何进行地图编辑。


## 连接地图编辑器


在浏览器里键入`mapeditor服务器的IP地址 + '/mapeditor'`，这里我们以`10.21.23.68`(当前我所搭建的mapeditor服务器的内网IP地址）为例：

```
10.21.23.68/mapeditor
```

得到的效果如下：

![mapeditor_welcome](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_welcome.png)

其中URL中的`#M`为JS里的`锚点定位`,可以不用管它。

这样我们就连接到了mapeditor服务器，可以进行后续操作。



## 创建地图

进入到`10.21.23.68/mapeditor`页面之后，可以看到如下:

![mapeditor_quest_create](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_quest_create.png)

 这里我们稍微对相关的日文进行简单的翻译一下：

| 日文      | 翻译   |
| ------- | ---- |
| ワールドID  | 世界ID |
| エリアID   | 区域ID |
| ダンジョンID | 迷宫ID |
| フロアID   | 楼层ID |

这里area_394的相关配置如下：

1. 世界ID: 2
2. 区域ID: 394
3. 迷宫ID: 1 ~ 4
4. 楼层ID: 239401010, 239401020, 239402010, 239402020, 239402030, 239403010, 239403020, 239403030, 2394014010, 239404020

这里我们`239401010`为例说明图示说明如何配置：

![mapeditor_quest_create_area394](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_quest_create_area394.png)

这里`縦部屋数`和`横部屋数`为所绘制的地图大小，可以根据实际情况进行调整(这是我是随便填写的)。

icon可以不用管，选默认的就好了。

最下面的两个button中：

* `作成`： 是指生成一份新的空白地图
* `コピーとして作成`： 是指拷贝一份之前的地图，拷贝源地图的楼层ID可以从下拉菜单中选择。


## 编辑地图


点击`作成` 之后，就会在该页面中生成（可能需要手动刷新）一个新的地图，点击进去之后就可以进行地图编辑。

关于具体编辑的，可以询问相关策划人员，这里不具体细讲了。
