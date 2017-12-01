---
layout: post
title: '[USACO] Castle'
description: ''
category: USACO
tags: USACO
---

## 题目描述

农场主John在爱尔兰赢得了一座城堡。城堡里有好多个房间，John想知道城堡的确切房间数，以及当拆掉一面墙之后，最大的房间能有多大。我们的任务就是帮助John解决这个问题。

城堡为矩形（N*M个单元组成），每个单元由一个数字构成，这个数字表示这个单元的墙面情况。

* 1表示东面（W）；
* 2表示北面（N）；
* 4表示西面（E）；
* 8表示南面（S）；

每个单元的数字为该单元各面墙的值之和，例如13表示有W、E、S三个方向有墙（1 + 4 + 8 = 13）。同时相邻两个单元之间的墙面可能会有共享。

求：

* 城堡的房间数；
* 城堡最大的房间数；
* 城堡拆掉一面墙之后的最大房间数；
* 拆掉这面墙的位置；

## 题目分析

该题是**IOI'94**年的一道题目。

题中城堡每个房间由一个或者多个单元构成，这些单元之间相互连通。

我们将每个单元看作节点，相邻单元之间如果连通，则可以看作节点之间无向连接。

则整个题目既可以转化为：

1. 求解图中有多少个连通子图（房间数）；
2. 图中的连通分量（最大房间数）；
3. 通过添加一条连接边，能够获得最大的连通分量（拆除一面墙之后的最大房间数）；

这里求解连通子图的个数，我们可以使用传统的FloodFill算法，通过遍历每个节点，对每个节点进行DFS算法遍历，凡是遍历过的节点进行标记，防止重复遍历。时间复杂度为：
$O(N*M+E)$, 其中E为所有的连接数。

在遍历的过程中，我们记录每个连通分量的大小，取最大的一个。

拆除每个单元的每一面墙，每次拆除一面墙时，跑一边FloodFill算法，记录最大的连通分量大小，同时保留着该墙的位置信息。时间复杂度为：
$$
O(E*((N*M) + E))
$$

## 代码

```c++

/*
ID: dswei191
LANG: C++
TASK: castle
*/

#include <iostream>
#include <fstream>
#include <cmath>
#include <string>
#include <string.h>

#define MAX_VALUE 50
using namespace std;
ofstream fout("castle.out");
ifstream fin("castle.in");

int N, M;
string wall[MAX_VALUE+1][MAX_VALUE+1];
int visit[MAX_VALUE+1][MAX_VALUE+1];

int numRoom = 0;
int maxRoomSize = 0;
int maxCreatedRoomSize = 0;
int idxA = -1;
int idxB = MAX_VALUE+1;
int idxK;

// (W, N, E, S) <=> (0, 1, 2, 3) <=> (左、上、右、下)
string transform(int a){
    string ret = "";
    for(int i = 0; i < 4; i++){
        if(a == 0){
            ret += "0";
        }else{
            if(a % 2 == 1){
                ret += "1";
            }else{
                ret += "0";
            }
            a = a / 2;
        }
    }
    return ret;
}

//DFS遍历连通子图
int dfs(int a, int b){
    if(a <1 || a > N || b < 1 || b > M || visit[a][b] == 1){
        return 0;
    }
    int ret = 1;
    visit[a][b] = 1;
    for(int i = 0; i < 4; i++){ //0, 1, 2, 3 => 左、上、右、下
        if(wall[a][b][i] == '0'){
            //没墙
            switch(i){
                case 0:
                    ret += dfs(a, b-1);
                    break;
                case 1:
                    ret += dfs(a-1, b);
                    break;
                case 2:
                    ret += dfs(a, b+1);
                    break;
                case 3:
                    ret += dfs(a+1, b);
                    break;
            }
        }
    }
    return ret;
}

//洪水算法遍历连通图
void floodFill(){
    for(int i = 1; i <= N; i++){
        for(int j = 1; j <= M; j++){
            if(visit[i][j] == 0){
                ++numRoom;
                int t = dfs(i, j);
                maxRoomSize = (t > maxRoomSize) ? t : maxRoomSize;
            }
        }
    }
}

// 将数字转化为方向字母
char direction(int i){
    char ret;
    switch(i){
        case 0:
            ret = 'W';
            break;
        case 1:
            ret = 'N';
            break;
        case 2:
            ret = 'E';
            break;
        case 3:
            ret = 'S';
            break;
    }
    return ret;
}

//遍历每个房间的每一堵墙移去之后的连通图
void removeWall(){
    for(int i = 1; i <= N; i++){
        for(int j = 1; j <= M; j++){
            for(int k = 0; k < 4; k++){
                memset(visit, 0, sizeof(visit));
                if (wall[i][j][k] == '1') {
                    wall[i][j][k] = '0';
                    if (visit[i][j] == 0) {
                        int t = dfs(i, j);
                        if (t > maxCreatedRoomSize) {
                            maxCreatedRoomSize = t;
                            idxA = i;
                            idxB = j;
                            idxK = k;
                        } else if (t == maxCreatedRoomSize) {
                            if (idxB > j) {
                                maxCreatedRoomSize = t;
                                idxA = i;
                                idxB = j;
                                idxK = k;
                            } else if (idxB == j) {
                                if (idxA < i) {
                                    maxCreatedRoomSize = t;
                                    idxA = i;
                                    idxB = j;
                                    idxK = k;
                                }
                            }
                        }
                    }
                    wall[i][j][k] = '1';
                }
            }
        }
    }
}

int main(){
    fin >> M >> N;
    for(int i = 1; i <= N; i++){
        for(int j = 1; j <= M; j++){
            int t;
            fin >> t;
            wall[i][j] = transform(t);
        }
    }

    floodFill();
    removeWall();

    fout<< numRoom<<endl;
    fout<< maxRoomSize<<endl;
    fout<< maxCreatedRoomSize<<endl;
    fout << idxA << " "<<idxB << " "<<direction(idxK)<<endl;
    return 0;
}
```

