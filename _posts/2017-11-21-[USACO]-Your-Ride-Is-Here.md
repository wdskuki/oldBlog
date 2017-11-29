---
layout: post
title: '[USACO] Your Ride is Here'
description: ''
category: USACO
tags: USACO
---

# [USACO] Your Ride is Here

## ## 题目描述

每一颗彗星背后都隐藏着都一架UFO！！外星人来地球寻找支持UFO的团体，来把他们接走（类似于三体人来找叶文洁）。不过由于UFO飞船里的空间有限，因此每次只能有一个团体能够有幸登上UFO，和外星人Happy。地球上支持UFO的团体那么多，那到底那个团体会被接走呢？别急，外星人已经提前放风给地球上的支持团体了：只有团体的名字需和彗星的名字，在某种运算规则下一致，该团体才会被接走。团体的名字和彗星的名字都是有大写英文字母组成，字母A-Z对应数字1-26，名字会转化为数字，转化规则为“USACO” => 21 * 19 * 1 * 3 * 15 = 17955，另外数字通过对47进行取模运算，看是否相等。如果想等表示一致，可以被接走，反之不然。

## 题目分析

题目表示简单。这里就不做特别分析了。将名字转化为数字，然后mod47，最后比较一下是否相等即可。

## 代码

```c++
/*
ID: dswei191
LANG: C++
TASK: ride
*/

#include <iostream>
#include <fstream>
#include <string>

using namespace std;

int main(){
    string filename = "ride";
    ofstream fout("ride.out");
    ifstream fin("ride.in");
    string str1, str2;
    int pro1, pro2;
    int i, j;
    while(fin>>str1>>str2){
        pro1 = 1;
        pro2 = 1;
        for(i = 0; i < str1.length(); i++){
            pro1 *= (str1[i] - 'A' + 1);
        }
        for(j = 0; j < str2. length(); j++){
            pro2 *= (str2[j] - 'A' + 1);
        }

        if((pro1 % 47) == (pro2 % 47)){
            fout << "GO" << endl;
        }else{
            fout << "STAY" << endl;
        }
    }
    return 0;
}
```



