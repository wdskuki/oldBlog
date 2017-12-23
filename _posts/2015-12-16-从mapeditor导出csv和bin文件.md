---
layout: post
title: '3.从mapeditor导出csv和bin文件'
description: ''
tags: DQMSL
---




> 一旦策划那边完成了地图编辑，剩下的事情就是软件工程师将相关的地图数据（csv和bin文件）导出到后台，来供后续做差分资源使用。

这里需要说明一下，策划那边所有的地图数据都会存在mapeditor服务器的DB表中，所谓`从mapeditor导出csv和bin文件`即将这些地图数据（不含图片资源，图片资源客户端会有，做差分的时候在将两者对应起来）从DB表里导出到csv文件，csv文件在进行压缩操作生成bin文件。其中csv文件保存在后台供后台使用；bin文件主要用来给客户端做差分用的。


## 登陆

导出mapeditor服务器DB表中的数据需要登录到对应服务器上进行，由于我们的很多操作咋图形界面操作起来比较方便，所以这里所谓的登录不是`ssh`方式,而是以`远程桌面`的形式登陆。


两种登陆mapeditor服务器操作的方式：

1. 通过teamviewer工具远程桌面到mapeditor服务器上
2. 在实际的mapeditor物理机上进行操作。

这里需要特别说明一下的是：程序这边`不能`通过本地的浏览器键入`http://10.21.23.68/mapeditor`。因为虽然可以实现连接到mapeditor服务器，但后续的一些操作只能在mapeditor服务器本地环境进行，而不是远端环境。而上述两种方式可以实现对mapeditor进行本地化操作。

后续我以teamviewer远程桌面的方式来说明具体导出流程。

## 将地图数据导出到csv文件中

我们还是以`area_394`为例，说明如何将`area_394`的地图数据倒出来。

点击如下：

![mapeditor_quest_list](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_quest_list.png)

或者在浏览器中键入：

```
http://10.21.23.68/mapeditor/data_manage/
```

会跳转到如下一个界面：

![mapeditor_data_manage](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_data_manage.png)

这里账户和密码都是：`monsters`

这样就登陆到所有地图信息的一个列表界面，如下：
![mapeditor_data_manage_list](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_data_manage_list.png)


选中策划那边绘制的一些地图资源。如果发现无法选中，例如如下：
![mapeditor_quest_unwrite](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_quest_unwrite.png)

一般情况是因为在quest_data.csv中没有配置对应的条目。这时候需要在quest_data.csv目录中插入对应的条目才行。quest_data.csv的全路径为：

```
D:\DQ\Server\mon_server\monsters-server\server\codeigniter\application\resource\csv\EN\csv1.0.8\quest_data.csv
```

这里我们可以不用考虑全路径中`EN`和`csv1.0.8`所带来的影响。因为硬编码的原因，我们只能修改这个全路径下的quest_data.csv，否则将无法生效。

插入area_394对应的条目：

![mapeditor_quest_write](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_quest_write.png)

这样我们发现可以选中了！

![mapeditor_quest_select](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_quest_select.png)

选中之后，我们可以导出对应的csv文件。这个操作可以在win8系统中（而非CentOS虚拟机系统下）进行。选中之后，点击如下：

![mapeditor_generate_dungeon_csv](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_generate_dungeon_csv.png)

然后在win8的`下载`文件夹下解压出刚才下载的zip文件。

将解压出来的dungeon目录中的内容拷贝如下目录：

```
D:\DQ\Server\mon_server\monsters-server\server\codeigniter\application\resource\csv\EN\csv1.0.8\dungeon\
```
如果该目录中有旧的文件，就删除旧的目录和文件。

拷贝完了之后，同时将一个脚本拷贝到上面那个目录中, 脚本名称为`cmp_mapStage_quest_type.sh`,脚本代码如下：

```
#! /bin/sh

# 

#将本脚本拷贝到resource/csv1.0.8/dungeon目录下运行

stageID_in_mapStage='stageID_in_mapStage.txt'
stageID_in_quest_type='stageID_in_quest_type.txt'
prefix='mapEventMonster_'
stageID_in_quest_type=${prefix}${stageID_in_quest_type}
tmp='in_mapStage_not_in_quest_type.txt'
target='mapStage.csv.new'

:> $stageID_in_mapStage
:> $stageID_in_quest_type
:> $tmp
:> $target
#get stage ID in mapStage.csv
cat mapStage.csv | awk -F ',' '{print $1}' | less | grep -v stageID | uniq | sort >> $stageID_in_mapStage

#get stage ID in quest_type_x
for dir in $(ls | grep quest_type_)
do
	for subdir in $(ls $dir)
	do
		#echo $subdir
		ls $dir/$subdir | grep $prefix | awk -F '_' '{print $2}' | awk -F '.' '{print $1}' | uniq | sort >> $stageID_in_quest_type
	done
done

#cat $stageID_in_mapStage | sort $stageID_in_quest_type

grep -vFf $stageID_in_quest_type $stageID_in_mapStage >> $tmp
# mv mapStage.csv mapStage.csv.old
grep -vFf ${tmp} mapStage.csv >> mapStage.csv.new

rm $tmp
rm $stageID_in_quest_type
rm $stageID_in_mapStage

cat mapStage.csv.new
```

我们通过命令行来运行这个脚本,届时将生成的mapStage.csv.new。我们用mapStage.csv.new来覆盖之前的mapStage.csv,覆盖后文件名仍然保留为 `mapStage.csv`。

```
[root@centos6 dungeon]# pwd
/product/monsters-server/server/codeigniter/application/resource/csv/EN/csv1.0.8/dungeon
[root@centos6 dungeon]# bash cmp_mapStage_quest_type.sh
stageID,longRoomCountMAX,wideRoomCountMAX,worldID,areaID,dungeonID,floorName,floorDescription
239401010,6,3,2,394,1,Blue Behemoth:easy,1f
239401020,5,6,2,394,1,Blue Behemoth:Easy,2f
239402010,7,4,2,394,2,Blue Behemoth:Meduim,1f
239402020,5,7,2,394,2,Blue Behemoth:Meduim,2f
239402030,5,2,2,394,2,Blue Behemoth:Meduim,3f
239403010,5,4,2,394,3,Blue Behemoth:Hard,1f
239403020,6,8,2,394,3,Blue Behemoth:hard,2f
239403030,6,5,2,394,3,Blue Behemoth:Hard,3f
239404010,7,7,2,394,4,Blue Behemoth:bonkers,1f
239404020,6,5,2,394,4,Blue Behemoth:bonkers,2f
[root@centos6 dungeon]# mv mapStage.csv.new mapStage.csv
mv：是否覆盖"mapStage.csv"？ y
[root@centos6 dungeon]#
```
这样， `D:\DQ\Server\mon_server\monsters-server\server\codeigniter\application\resource\csv\EN\csv1.0.8\dungeon\`
目录下的所有的csv文件（除了脚本cmp_mapStage_quest_type.sh）都是我们需要的csv文件。


## 将csv文件数据数据压缩为bin文件

完成了csv文件的导出操作之后，我们需要生成bin文件。从mapeditor的代码中可以看出bin文件本质为csv文件的压缩文件。

进行bin文件的生成操作，我们必须要在CentOS系统（GUI界面）下进行，否则的话无法正常完成bin生成流程。

我们进入到CenOS系统，打开firefox浏览器，登陆到`10.21.23.68/mapeditor/data_manage`,然后点击如下：

![mapeditor_generate_dungeon_bin](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_generate_dungeon_bin.png)

这样会在如下目录下生成对应的`bin`文件：

```
D:\DQ\Server\mon_server\monsters-server\server\dl\dungeon\
```


这里需要说明一下的是,目录`D:\DQ\Server\mon_server\monsters-server\server\dl\dungeon`必须要存在，
如果不存在，必须在`D:\DQ\Server\mon_server\monsters-server\server\dl`中新建一个空的dungeon目录。
方便的话可以清空`D:\DQ\Server\mon_server\monsters-server\server\dl\dungeon`目录，这样可以生成干净的bin文件。


##小结


生成好了bin文件之后,基本完成csv和bin文件的生成操作。其中：

* `dungeon csv`:  D:\DQ\Server\mon_server\monsters-server\server\codeigniter\application\resource\csv\EN\csv1.0.8\dungeon\
* `dungeon bin`:  D:\DQ\Server\mon_server\monsters-server\server\dl\dungeon\

我们可以压缩打包，利用飞鸽传书或者QQ，将这两个目录下的文件传到你本地的工作机器上（例如我的iMac）。

其中`dungeon bin`下的内容拷贝到服务器端的`dl/dungeon/`目录下；`dungeon csv`下的内容拷贝到`codeigniter\application\resource\csv\各个语言\对应版本\dungeon\`目录下，同时将`dungeon csv`中的mapStage.csv文件中内容添加到`codeigniter\application\resource\csv\各个语言\对应版本\dungeon\mapStage.csv`文件的最后。


最后需要强调一下的是，服务器端的dl/dungeon目录是所有地图数据的bin文件的所在目录，这个目录非常非常重要，客户端做差分的时候需要用这个目录中的资源进行差分操作（可以参考客户端做差分时候的三个脚本）。
