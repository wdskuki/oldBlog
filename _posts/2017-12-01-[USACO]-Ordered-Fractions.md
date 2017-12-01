---
layout: post
title: '[USACO] Ordered Fractions'
description: ''
category: USACO
tags: USACO
---



## 题目描述

给定一个数字N（1 <= N <= 160），求所分数满足以下条件的分数：

* 分子和分布都在[0, N]之间；
* 分子分母不可约分；
* 分数值在[0, 1]之间；

将得到的分数按照升序排序。



## 题目分析

用一个二维数组表示这个分数的分子和分母，通过扫描二维数组的一半（因为只考虑分数小于等于1的情况），标记不可约分的点。我们这里利用辗转相除法来获得是否存在大约1的最大公约数来判断是否可约分。

获得所有满足条件的分数之后，就剩下排序了。当时考虑到的是因为当分子固定，分数值随着分母的减小而降低。所以一开始我打算利用归并排序来降低排序的时间复杂度。后来看了下规模一共也就最多160 * 160 ／ 2个数据，直接调用库函数sort()一把嗖^_^。



## 代码

```c++

/*
ID: dswei191
LANG: C++
TASK: frac1
*/

#include <iostream>
#include <fstream>
#include <cmath>
#include <string>
#include <vector>
#include <algorithm>

using namespace std;
ofstream fout("frac1.out");
ifstream fin("frac1.in");

#define MAX_N 160
int N;
int f[MAX_N+1][MAX_N+1];

vector<pair<int, int> > vec;

// 最大公约数（辗转相除法）
int maxCommDivider(int a, int b){
    if(a == 1 || b == 1){
        return 1;
    }else if(a == 0 || b == 0){
        return 0;
    }
    if(a < b){
        int t = a;
        a = b;
        b = t;
    }
    while(a % b != 0){
        int t = a - b;
        if(t > b){
            a = t;
        }else{
            a = b;
            b = t;
        }
    }
    return b;
}

//最小公倍数
int minCommMultiple(int a, int b){
    if(a== 0 || b == 0){
        return 0;
    }
    int t = maxCommDivider(a, b);
    return (b / t) * a;
}



//比较两个分数a1/a2和b1/b2的大小
bool comp(pair<int, int > & a, pair<int, int > & b){
    int t = minCommMultiple(a.second, b.second);
    return a.first * (t / a.second) < b.first * (t / b.second);
}


int main(){
    fin >> N;

    f[0][1] = 1;
    vec.push_back(make_pair(0, 1));
    for(int i = 1; i <= N; i++){
        for(int j = i; j <= N; j++){
            int t = maxCommDivider(i, j);
            int i1 = i/t;
            int j1 = j/t;
            if(f[i1][j1] == 0){
                f[i1][j1] = 1;
                vec.push_back(make_pair(i1, j1));
            }
        }
    }

    sort(vec.begin(), vec.end(), comp);

    for(int i = 0; i < vec.size(); i++){
        fout<< vec[i].first << "/" <<vec[i].second <<endl;
    }
    return 0;
}
```





