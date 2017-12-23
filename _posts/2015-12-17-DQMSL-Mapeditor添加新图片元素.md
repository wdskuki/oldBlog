---
layout: post
title: 'Mapeditor添加新图片元素'
description: ''
category: DQMSL
---


# 4. Mapeditor添加新图片元素


> 当前，讨伐活动很多地图资源都是从2.0.9版本里引入的，而目前mapeditor的版本主要还是和1.0.8对应。也就是说，目前mapeditor中没有关于2.0.9的讨伐活动的相关图片资源。那么我们该如何编辑一些2.0.9的讨伐活动的地图呢？这里，我们首先要做的就需要将2.0.9的相关图片资源导入到当前的mapeditor系统中才行。

## 背景

这里我们简要讲解一下地图编辑器中的两个基本概念。

### Room

![mapeditor_map_rooms](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_map_rooms.png)

一张地图由多个Room构成，图中这些一个一个的虚线格子就是所谓的`Room`,即房间的意思。


### Tile

![mapeditor_room_tiles](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_room_tiles.png)

每个Room有多个Tile组成，图中标出来得那些每个小方格就是所谓的`Tile`,可以理解为`地砖`。

每个Room中，从上往下，从左到右，每个Tile都有一个编号来唯一标示该Tile所在的位置(1,2,3,4,....)。该编号可以认为对应这个Tile在该Room的坐标。
这个坐标可以定位Room中一些图片元素摆放位置。


## mapEditor代码简介


远程`ssh`到mapeditor服务器上，进入到mapeditor的逻辑代码所在的目录一探究竟。

```
[root@centos6 mapeditor2]# pwd
/product/mapeditor2
[root@centos6 mapeditor2]# ls /product/mapeditor2
app  README.md
[root@centos6 mapeditor2]# git branch
* chinese_version
  master
[root@centos6 mapeditor2]# git remote -v
origin	git@54.169.78.60:/data/dq_map.git (fetch)
origin	git@54.169.78.60:/data/dq_map.git (push)
[root@centos6 mapeditor2]#
```

之前说过，mapeditor相关的逻辑代码保存在`/product/mapeditor2`目录下，这里我将该目录下的所有文件用git进行版本控制管理，同时上传到远端服务git服务器上，位置为：

```
git@54.169.78.60:/data/dq_map.git
```
这里我们当前的工作分支为`chinese_version`,该分支上我进行了相关的bugfix和部分汉化工作，所以如果要正常使用mapeditor,就必须要在该分支上进行，而不需要却换到master分支上。

mapeditor逻辑代码也是基于CI框架，这一点和后台代码框架一致。

当前，讨伐活动很多地图资源都是从2.0.9版本里引入的，而目前mapeditor的版本主要还是和1.0.8对应。也就是说，目前mapeditor中很多2.0.9的讨伐活动的相关图片资源是没有的。那么我们该如何编辑一些2.0.9的讨伐活动的地图呢？这里就需要将2.0.9的相关图片资源导入到当前的mapeditor系统中才行。

这里我们以`导入2.0.9的新关卡所涉及到的地图元素`为例，说明其工作流程。



## 添加新图片元素


所有的地图图片元素放到如下目录中:

```
/product/mapeditor2/app/img/event/import
```
导入完所有地图图片元素资源到上述目录后，需要在`/product/mapeditor2/app/js/mockjax/events_tile.js`文件中添加对应条目。这样策划才能在浏览器中看到新加入的地图图片元素，并且可以通过拖拽使用这些图片元素。

为了方便自动化生成相关条目，我这里写了一个小的python脚本: `209mapeditor_map.py`,只要稍微修改一下就能运行：

```
#!/usr/bin/python
# Filename : 209mapeditor_map.py

import os

sample_json_str =  '''{ category: '90822038', title: 'Devil City', tiles: [ {name: '0', label: "event-90822038", src: "img/event/import/mc90822038.png"}, ]  },'''
mapdirpath = '/Users/wds/Downloads/map-devil-city/'


idx1 = '90822038'

pngfileslist = os.listdir(mapdirpath)

print sample_json_str.find(idx1)


for filename in pngfileslist:
	pngId = filename[2:].split('.')[0]
	#print pngId+''
	print sample_json_str.replace(idx1, pngId).replace('mt', '')
	#print sample_json_str

```

这里需要修改的地方有如下两点：

* `sample_json_str`： 可能需要在`mc90822038`之后添加`mt`，即修改为`mc90822038mt`,不过这个**不是必须的**，可以根据实际情况修改（主要看导入的图片的文件名是否有这个后缀）。
* `mapdirpath`： 目前为我本地iMac机器上保存新图片元素目录的全路径，可能需要根据你的情况修改。这个**修改是必须的**。

将生成的条目保存到`events_tile.js`中对应位置就可以了。





## 一键配置


这里一键配置，既可以对这个地图（所有Room）进行整体配置，也可以针对单个Room逐一进行配置。

需要特别说明一下的是，由于浏览器缓存JS代码,因此一旦配置修改完之后，需要**清空你的本地浏览器缓存**看到这些添加修改的内容。


### Room背景设置

地图背景的配置在如下文件中设置：

```
/product/mapeditor2/app/js/mockjax/assets.js
```

其中在`roomBackgrounds`需要建对应的背景图片的信息添加到该数组中，添加的格式类似于:

```
...
{name: '12211000', label: 'South Island:1', src: 'img/event/import/mc12211000mt.png'},
{name: '12212000', label: 'South Island:2', src: 'img/event/import/mc12212000mt.png'},
{name: '56811000', label: 'Devil City:1', src: 'img/event/import/mc56811000.png'},
{name: '56812000', label: 'Devil City:2', src: 'img/event/import/mc56812000.png'},
...
```
这里我们假设配置完`South Island:1`,我们可以在如下web页面中看到一个下拉菜单，其中就有`South Island:1`相关的条目。选中该条目，就能设置对应的图片为Room的背景。

![mapeditor_room_background](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_room_background.png)

### Room中道路配置

Room道路配置在如下文件中设置：

```
/product/mapeditor2/app/js/mockjax/preset_ways.js
```

这里我们还是以`South Island:2`为例说明：

```
        {
            "themeID": 122,
            "label": "South Island:2",
            "tiles": {
                "always": [
                    {
                        "tileID": 148,
                        "eventID": 12221013,
                        "type": 0,
                        "z_index": 24
                    }
                ],
                "topWay": [
                    {
                        "tileID": 8,
                        "eventID": 12221001,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "rightWay": [
                    {
                        "tileID": 151,
                        "eventID": 12221002,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "bottomWay": [
                    {
                        "tileID": 208,
                        "eventID": 12221003,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "leftWay": [
                    {
                        "tileID": 141,
                        "eventID": 12221004,
                        "type": 0,
                        "z_index": 30
                    }
                ]
            },
            "alterTiles": {
                "always": [
                    {
                        "tileID": 148,
                        "eventID": 12221013,
                        "type": 0,
                        "z_index": 24
                    }
                ],
                "topWay": [
                    {
                        "tileID": 8,
                        "eventID": 12221001,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "rightWay": [
                    {
                        "tileID": 151,
                        "eventID": 12221002,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "bottomWay": [
                    {
                        "tileID": 208,
                        "eventID": 12221003,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "leftWay": [
                    {
                        "tileID": 141,
                        "eventID": 12221004,
                        "type": 0,
                        "z_index": 30
                    }
                ]
            }
        },
```

* "themeID": 具体数值无所谓，一般取添加这组图片元素的ID的前三位（例如这里为568）;同时和preset.js中对应的主题值一样就行。
* "label": 名称, 取一个容易记住的名字就行了
* "tiles"和 "alterTiles"：一般不需要特别处理"alterTiles"里的内容，直接拷贝一份"tiles"的内容到"alterTiles"就行了
* "tiles"和“alterTiles”分别有`always`/`topWay`/`rightWay`/`bottomWay`/`leftWay`。
  * `always`: 为道路中间的位置
  * `topWay`/`rightWay`/`bottomWay`/`leftWay`： 分别对应道路的上、右、下、左
* `tileID`: 对应了该元素图片（图片为长方形，对应的起点在左上角）在room的位置
* `eventID`: 对应了道路图片编号的ID
* `type`:  一般使用默认值
* `z_index`: 可以使用默认值

这里需要特别说明一下，每个图片对应的位置(`tileID`)可能存在或多或少的偏移，因此需要不断的微调才能确定每个图片的`tileID`值。

一般的做法是：先拷贝之前的一份作为模板，然后修改`themeID`/`label`/`eventID`等，从浏览器中打开查看效果。如果有些偏差，然后调整各个`eventID`的`tileID`值，进行微调修改。

不过根据我的经验，一般直接拷贝其他的一份作为模板，一般无需修改`tileID`就可以直接使用。
### Room中围墙配置

```
/product/mapeditor2/app/js/mockjax/preset.js
```

```
       {
            "themeID": 568,
            "label": "Devil City:1",
            "tiles": {
                "corner": [
                    {
                        "tileID": 1,
                        "eventID": 56831001,
                        "type": 0,
                        "z_index": 24
                    },
                    {
                        "tileID": 12,
                        "eventID": 56831002,
                        "type": 0,
                        "z_index": 24
                    },
                    {
                        "tileID": 221,
                        "eventID": 56831004,
                        "type": 0,
                        "z_index": 24
                    },
                    {
                        "tileID": 232,
                        "eventID": 56831003,
                        "type": 0,
                        "z_index": 24
                    }
                ],
                "top": [
                    {
                        "tileID": 4,
                        "eventID": 56831005,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "right": [
                    {
                        "tileID": 72,
                        "eventID": 56831006,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "bottom": [
                    {
                        "tileID": 224,
                        "eventID": 56831007,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "left": [
                    {
                        "tileID": 61,
                        "eventID": 56831008,
                        "type": 0,
                        "z_index": 30
                    }
                ]
            },
            "alterTiles": {
                "corner": [
                    {
                        "tileID": 1,
                        "eventID": 56832001,
                        "type": 0,
                        "z_index": 24
                    },
                    {
                        "tileID": 12,
                        "eventID": 56832002,
                        "type": 0,
                        "z_index": 24
                    },
                    {
                        "tileID": 221,
                        "eventID": 56832004,
                        "type": 0,
                        "z_index": 24
                    },
                    {
                        "tileID": 232,
                        "eventID": 56832003,
                        "type": 0,
                        "z_index": 24
                    }
                ],
                "top": [
                    {
                        "tileID": 4,
                        "eventID": 56832005,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "right": [
                    {
                        "tileID": 72,
                        "eventID": 56832006,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "bottom": [
                    {
                        "tileID": 224,
                        "eventID": 56832007,
                        "type": 0,
                        "z_index": 30
                    }
                ],
                "left": [
                    {
                        "tileID": 61,
                        "eventID": 56832008,
                        "type": 0,
                        "z_index": 30
                    }
                ]
            }
        },
```

这里逐个说明一下：

* "themeID": 具体数值无所谓，一般取添加这组图片元素的ID的前三位（例如这里为568）;同时和preset_ways.js中对应的主题值一样就行。
* "label": 名称, 取一个容易记住的名字就行了
* "tiles"和 "alterTiles"：
  *	"tiles"主要处理如下几类Room的图片元素摆放位置：
     * ![mapeditor_tiles_in](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_tiles_in.png)
       *而"alterTiles"主要处理如下几类Room的图片元素摆放位置：
       *![mapeditor_alterTiles_in](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_alterTiles_in.png)
* "tiles"和“alterTiles”分别有`corner`/`top`/`right`/`bottom`/`left`。
  * `corner`: 包含四个角落
  * `top`/`right`/`bottom`/`left`: 分别对应上、右、下、左
* `tileID`: 对应了该元素图片（图片为长方形，对应的起点在左上角）在room的位置
* `eventID`: 对应了图片编号的ID
* `type`:  一般使用默认值
* `z_index`: 可以使用默认值

这里需要特别说明一下，每个图片对应的位置(`tileID`)可能存在或多或少的偏移，因此需要不断的微调才能确定每个图片的`tileID`值。

一般的做法是：先拷贝之前的一份作为模板，然后修改`themeID`/`label`/`eventID`等，从浏览器中打开查看效果。如果有些偏差，然后调整各个`eventID`的`tileID`值，进行微调修改。




