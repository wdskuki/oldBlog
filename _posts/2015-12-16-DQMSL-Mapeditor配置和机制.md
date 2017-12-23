---
layout: post
title: 'DQMSL Mapeditor配置和机制'
description: ''
category: DQMSL
---



# 1. DQMSL Mapeditor配置和机制


> Mapeditor是DQMSL的地图编辑器，游戏策划师可以通过浏览器连接到运行Mapeditor服务器进行地图编辑; 当策划完成了地图编辑之后，后台工程师将策划师编辑好的地图导出到来，放在后台代码中，供后续客户端工程师进行差分操作。



## 简介
mapeditor运行于CentOS系统上。之前从日本拿到的就是整个ova格式的文件（非常大，接近30G），即整个centos虚拟机（含mapeditor相关）的镜像。也就是说，如果要使用mapeditor，需要在虚拟机上运行这个centos系统，同时配置好对应网络, 才能使用这个mapeditor。

一般来说，其实可以把mapeditor从对应的CentOS系统中独立出来，这样就可以将mapeditor部署到任何一台linux机器上，而不一定运行在虚拟机上(目前我在win8系统上跑virtualbox虚拟机，virtualbox虚拟机上再跑这个ova镜像CentOS系统)。但由于mapeditor需要对应的DB表支持，之前的CentOS镜像文件内自带了一些DB和对应的表，所以我就没有做对应的独立操作。目前运行状况是：Win8-> Virtualbox -> CentOS -> mapeditor。

目前我已经将mapeditor（即对应的centos虚拟机镜像）部署在团队的一台台式机上; 同时做了一些适配性的修改和一些bugfix。因此,如果后续需要重新部署mapeditor环境，或者需要进一步修改、配置或者定制等等，建议导出这份虚拟机镜像来，而不是以日本那边刚拿到的镜像文件。**建议直接使用我已经部署好的环境来进行相应的地图编辑器操作,而不是重新搭建一个新的环境。**后续中描述的各种环境都是基于我搭建的mapeditor环境来进行的，因此如果你自己搭了一个mapeditor环境啥的，可能流程与我后面描述的有些出入，但内容应该基本相同。

目前mapeditor机器的对应的内网IP为：

*`10.21.23.68`*

这个IP可能会变动（比如宕机重启啥的），以为我是DHCP模式获得IP的。如果想知道确切的IP，建议直接进入到CentOS系统通过`ifconfig`命令查看。后续我们还是以`10.21.23.68`作为mapeditor的IP地址。


一旦部署好mapeditor环境（配置好对应网络），我们就可以通过ssh的形式登陆到centos系统。需要说明的是，如果ssh连接不上，很可能CentOS系统没有开启对应的网络适配器。可以通过GUI界面进入到centos系统，然后将两个网络适配器都打开,如下图所示：

![mapeditor_netconfig_gui](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_netconfig_gui.png)


## 目录结构

本地命令行连接到CentOS系统：

```
ssh monsters@10.21.23.68
```

账户和密码都是：`monsters`

进入之后可以切换到root账户,密码也为: `monsters`

mapeditor相关的代码资源都保存在 `/product`目录下

```
[root@centos6 /]# cd /product/
[root@centos6 product]# ls -l
总用量 8
drwxrwxrwx 4 monsters monsters 4096 9月  27 18:00 mapeditor2
drwxrwxrwx 1 monsters monsters    0 6月  26 18:58 monsters
drwxrwxr-x 2 monsters monsters 4096 7月  13 20:43 monsters-mapeditor
drwxrwxrwx 1 monsters monsters    0 8月  31 16:40 monsters-server
```

这里特别说明一下这几个目录：

1. mapeditor2: 存放所有mapeditor相关的代码。针对这个目录，后续我会专门讨论一下。
2. monsters: 为一个共享文件夹，共享本地的`客户端`代码（不过一般在制作地图的过程中用不到）
3. monsters-mapeditor: 为一个共享文件夹,共享本地的mapeditor event command相关的逻辑代码（由于我不会做event command，所以目前也基本用不着）
4. monsters-server: 为一个共享文件夹，共享本地`服务器`代码，这个**非常重要**，不能缺少!

说明一下，这里所有的`本地`是指虚拟机运行的物理机环境，而非发起ssh连接的主机环境。比如我的工作环境是iMac，运行mapeditor的物理机环境系统为Win8， Win8上跑一个virtualbox虚拟机，virtualbox上跑CentOS镜像系统。


由于需要共享文件夹，因此需要将对应的资源拷贝到物理机环境中，这样才能和虚拟机共享资源。

当前物理机环境运行的时Win8操作系统，DQ相关的资源我都保存在`D:\DQ`目录下。

![mapeditor_share_directories](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_share_directories.png)

这里图中标出的三个目录分别对应了和CentOS系统共享的三个目录：

* Server (Win8) <--> monsters-server (CentOS)
* mapeditor_doc\monsters-mapeditor (Win8) <--> monsters-mapeditor (CentOS)
* Client (Win8) <--> monsters (CentOS)

我们可以通过VirtualBox来设置共享文件夹，来将物理机（Win8）上的对应目录资源共享到虚拟机上。

![mapeditor_VM_share_directories](http://dsweiblog.oss-cn-shanghai.aliyuncs.com/2015-12/mapeditor_VM_share_directories.png)

完成以上Virtualbox配置之后，需要在系统中将对应目录挂载到centos文件系统之中。

```
[root@centos6 /]# mount -t vboxsf Client /product/monsters
[root@centos6 /]# mount -t vboxsf monsters-mapeditor /product/monsters-mapeditor/
[root@centos6 /]# mount -t vboxsf Server /product/monsters-server
```
这样完成了共享文件夹的挂载操作，CentOS可以访问里面的资源了。


