---
layout: post
title: '[USACO] Greedy Gift Givers'
description: ''
category: USACO
tags: USACO
---

## 题目描述

一组小朋友分钱，每次每个小朋友将自己所有的钱均分给对应的一部分其他小朋友。

例如小朋友jack有100块钱，他将这100块钱均分给了kitty、mike、lucy三个人，则每个人分到33块，剩下的一块留给jack。求经过一轮分钱之后，每个小朋友还剩下的钱多少。

## 题目分析

杂题。简单的演绎一遍即可。

## 代码

```c++
/*
ID: dswei191
LANG: C++
TASK: gift1
*/

#include <iostream>
#include <fstream>
#include <string>
#include <map>

using namespace std;

int main(){
    int np;
    string *members;
    map<string, int> mapMoney;
    map<string, int> mapOralMoney;

    ofstream fout("gift1.out");
    ifstream fin("gift1.in");

    fin >> np;
    if(np <= 0){
        return -1;
    }

    members = new string[np];
    for(int i = 0; i < np; i++){
        fin >> members[i];
        mapMoney.insert(pair<string, int>(members[i], 0));
        mapOralMoney.insert(pair<string, int>(members[i], 0));
    }

    for(int i = 0; i < np; i++){
        string name;
        int total, divideNum, divideMoney;

        fin >> name;
        fin >> total >> divideNum;

        mapMoney[name] += total;
        mapOralMoney[name] = total;
        if(divideNum == 0){
            divideMoney = 0;
        }else{
            divideMoney = total / divideNum;
        }
        for(int j = 0; j < divideNum; j++){
            string divideName;
            fin >> divideName;
            mapMoney[divideName] += divideMoney;
            mapMoney[name] -= divideMoney;
        }
    }

    for(int i = 0; i < np; i++){
        fout << members[i] << ' ' << mapMoney[members[i]] - mapOralMoney[members[i]] <<endl;
    }
    return 0;
}

```

