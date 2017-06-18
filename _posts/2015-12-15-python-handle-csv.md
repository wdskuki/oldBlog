---
layout: post
title: 'Python对csv文件的处理'
description: 'Python对csv文件的处理'
category: Python
tags: Python
---


# Python对csv文件的处理
公司策划那边是不是会有从相关csv文件提取相关属性所对应的所有值的需求。 为了避免手动一个文件一个文件的查找&复制，写了一个脚本进行对应查找和提取。

 比如前两天有个需求是：
>给出一组questid列表，在每个questid(例如99070333)后面添加一个`0`（变成了99070333`0`）,然后根据这个`questid + "0"`(我们定义为stageid)找出应目录下的csv文件，这些csv文件的文件名格式形式为：**`prefix_stageid`.`surfix`**。在项目中`prefix`为**mapEventMonster**, `surfix`为**csv**。因此以questid = 99070333为例，对应的csv文件名为：

```
mapEventMonster_990703330.csv
```

这些csv文件格式如下（属性和值之间由**逗号**分隔,这里为了表示的方式来显示效果）:

| stageID | roomID | rate | enemyGroupID| actionFlg|messageNo_0|messageNo_1|messageNo_2|messageNo_3|messageNo_4|worldID|areaID|dungeonID|
|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|
|990701690|2|0|90080169|1|-1|-1|-1|-1|-1|9|907|1|

这里我们提取**enemyGroupID**属性所对应的所有值。


这里还有一个问题是：所有csv文件按照相关规则保存在对应目录中，例如：

```
...
dungeon/quest_type_3/area_907/mapEventMonster_990701690.csv
dungeon/quest_type_3/area_999/mapEventMonster_999999990.csv
...
...
```
所以我们需要找深入到目录（及其子目录中；这里一级目录为`dungeon`）找到对应csv文件。同时最好建立一个`key-value`的字典结构用来保存所有的查找结果。这里`key`为stageId(即questId + "0"); `value`为对应csv文件的全路径。

代码如下：

```
#!/usr/bin/python
# coding:utf8

import os
import csv

# 递归目录及其子目录下的文件
def dirlist(path, allfile):
	filelist = os.listdir(path)
	for filename in filelist:
		filepath = os.path.join(path, filename)
		if os.path.isdir(filepath):
			dirlist(filepath, allfile)
		else:
			allfile.append(filepath)
	return filepath

# 从文件的全路径中截取文件名(一般处于全路径的最后一个'/'的后面)
def cutFileName(fullpath):
	return fullpath.split('/')[-1]

# 打印list所有元素
def printList(myList):
	for item in myList:
		print item

# 判断文件名是否满足指定后缀
def isSurfix(path, surfix):
	filename = cutFileName(path)
	return path.split('.')[-1] == surfix

# 判断文件名是否满足指定前缀
def isPrefix(path, prefix):
	filename = cutFileName(path)
	return filename.find(prefix) == 0

# 建立key-value的字典结构保存搜索结果
# key: 		stageId
# value:  	对应csv的全路径
def setDict(allfile, prefix, surfix):
	myDict = dict()
	for item in allfile:
		if isSurfix(item, surfix) and isPrefix(item, prefix):
			filename = cutFileName(item)
			beg = filename.find(prefix)
			end = filename.find(surfix)
			stageId = filename[beg + len(prefix): end-1]
			myDict[stageId] = item
	return myDict

# 获得对应csv文件的某一列的所有值
def getCsvValue(filepath, colnum, ret):
	# print filepath
	reader = csv.reader(file(filepath, 'rb'))
	for line in reader:
		if reader.line_num == 1:
			# 跳过第一行属性名
			continue
		ret.append(line[colnum])

# 递归创建目录
def createDirs(path,outputpath):
	idx = path.find('dungeon')
	subpath = path[idx : ]
	#print subpath
	outputfullpath = os.path.join(outputpath, subpath)

	idx = outputfullpath.rfind("/")
	outputfullpath = outputfullpath[0:idx]
	#print outputfullpath
	if not os.path.exists(outputfullpath):
		os.makedirs(outputfullpath)

# 拷贝对应文件
def copyFile(srcfile, outputpath):
	idx = srcfile.find('dungeon')
	subpath = srcfile[idx : len(srcfile)]
	outputfullpath = os.path.join(outputpath, subpath)
	# 拷贝文件
	shutil.copyfile(srcfile, outputfullpath)

# main函数
if __name__ == '__main__':

	#以下各变量需要根据实际情况补全，
	# 否则无法运行
	path = 		# "需要扫描的目录路径"
	prefix = 	# "例如:mapEventMonster_"
	surfix = 	# "例如：csv"
	inputfile =	# 例如：open('输入文件名（含路径）', 'rb')

	allfile = []
	dirlist(path, allfile)

	myDict = setDict(allfile, prefix, surfix)

	ret = list()
	for line in inputfile:
		key = line.strip("\n") + "0"
		getCsvValue(myDict[key], 3, ret)
	ret = list(set(ret)) # 去重
	ret.sort() # 排序
	printList(ret) # 打印
	inputfile.close()

```
