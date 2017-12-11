---
layout: post
title: '[USACO] Name That Number'
description: ''
tags: USACO
---

## 题目描述

老式手机键盘，一个数字按键上有多个字母。现在给定一组数字和一张字典表，求这组数字对应的所有字母序列中，在这张字典表的字母序列有哪些，结果按照字典序排序。

## 题目分析

假如给定的数字为4734，那么这四个数字所组成的所有字母序列个数有
$$
3^4
$$
个,同时每个序列都要进行判断是否在字典里，则时间复杂度为:
$$
O(3^NM)
$$
其中N为数字序列长度，M为字典大小。这是一个比较大的时间开销。

我们尝试反过来思考下:将字典里每个字符串转化为数字序列，那么我们判断给定的数字是否在字典里，只需查找一次即可。复杂度将为
$$
O(M) + O(M) = O(M)
$$
这种反向的思维方式瞬间降低了程序的时间复杂度！

## 代码

```c++
/*
ID: dswei191
LANG: C++
TASK: namenum
*/

#include <iostream>
#include <fstream>

using namespace std;

ofstream fout( "namenum.out");
ifstream fin("namenum.in");

string transform(string str){
    string ret = "";
    for(int i = 0; i < str.length(); i++){
        if(str[i] == 'A' || str[i] == 'B' || str[i] == 'C'){
            ret += "2";
        }else if(str[i] == 'D' || str[i] == 'E' || str[i] == 'F'){
            ret += "3";
        }else if(str[i] == 'G' || str[i] == 'H' || str[i] == 'I'){
            ret += "4";
        }else if(str[i] == 'J' || str[i] == 'K' || str[i] == 'L'){
            ret += "5";
        }else if(str[i] == 'M' || str[i] == 'N' || str[i] == 'O'){
            ret += "6";
        }else if(str[i] == 'P' || str[i] == 'R' || str[i] == 'S'){
            ret += "7";
        }else if(str[i] == 'T' || str[i] == 'U' || str[i] == 'V'){
            ret += "8";
        }else if(str[i] == 'W' || str[i] == 'X' || str[i] == 'Y'){
            ret += "9";
        }
    }
    return ret;
}

string dict[5000];
string tDict[5000];
int main(){
    ifstream fdict("dict.txt");
    int i = 0;
    while(fdict >> dict[i]){
        tDict[i] = transform(dict[i]);
        ++i;
    }

    string numStr;
    fin >> numStr;
    bool flag = false;
    for(int j = 0; j < i; j++){
        if(tDict[j] == numStr){
            flag = true;
            fout << dict[j] << endl;
        }
    }
    if(flag == false){
        fout << "NONE" <<endl;
    }
    return 0;

}
```
