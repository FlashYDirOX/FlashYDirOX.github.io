---
title: '网络流的初步应用[USACO4.2]草地排水'
date: 2018-01-21 11:49:17
tags:
---
# 	[USACO4.2]草地排水DrainageDitches

- 网络流的应用日渐广泛，[USACO4.2]草地排水作为一道最大流的基本问题，可以说是裸的模板最大流问题，在此将其作为网络流的第一道例题，正方便了对网络流的学习以及理解，也为以后的诸多变形打下基础。

  <!-- more -->

## 题目简介：

- #### 中文翻译（转自nocow）：

- 农夫约翰知道每一条排水沟每分钟可以流过的水量，和排水系统的准确布局（起点为水潭而终点为小溪的一张网）。需要注意的是，有些时候从一处到另一处不只有一条排水沟。

  根据这些信息，计算从水潭排水到小溪的最大流量。对于给出的每条排水沟，雨水只能沿着一个方向流动，注意可能会出现雨水环形流动的情形。

  ## 题目背景

  在农夫约翰的农场上，每逢下雨，贝茜最喜欢的三叶草地就积聚了一潭水。这意味着草地被水淹没了，并且小草要继续生长还要花相当长一段时间。因此，农夫约翰修建了一套排水系统来使贝茜的草地免除被大水淹没的烦恼（不用担心，雨水会流向附近的一条小溪）。作为一名一流的技师，农夫约翰已经在每条排水沟的一端安上了控制器，这样他可以控制流入排水沟的水流量。

  ## 题目描述

  农夫约翰知道每一条排水沟每分钟可以流过的水量，和排水系统的准确布局（起点为水潭而终点为小溪的一张网）。需要注意的是，有些时候从一处到另一处不只有一条排水沟。

  根据这些信息，计算从水潭排水到小溪的最大流量。对于给出的每条排水沟，雨水只能沿着一个方向流动，注意可能会出现雨水环形流动的情形。

  ## 输入输出格式

  输入格式：


  第1行: 两个用空格分开的整数N (0 <= N <= 200) 和 M (2 <= M <= 200)。N是农夫John已经挖好的排水沟的数量，M是排水沟交叉点的数量。交点1是水潭，交点M是小溪。

  第二行到第N+1行: 每行有三个整数，Si, Ei, 和 Ci。Si 和 Ei (1 <= Si, Ei <= M) 指明排水沟两端的交点，雨水从Si 流向Ei。Ci (0 <= Ci <= 10,000,000)是这条排水沟的最大容量。



  输出格式：



  输出一个整数，即排水的最大流量。

  ## 输入输出样例

  ###### 输入：

  ```
  输入：
  5 4
  1 2 40
  1 4 20
  2 4 20
  2 3 30
  3 4 10
  ```

  ###### 输出：

  ```
  50
  ```

  ## 分析：

对于一道网络流的初级问题来说，使用Dinic算法未免有些小题大			   做，现在我们使用广搜的办法求解该题，在此之前我们首先需要了解一下解决此网络流问题需要的基本定义。

---

  **定义**：

  > **增广路径 **：可以理解为从源点s到汇点t的一条流量不为0的路径，在这里我们不考虑负流量的情况

  > **残余流量**：即当前路径下容量减去当前流量的值

---

​													(以上仅为一些定义，如接受扔有困难请自行移步百度)

首先，对于一个有向图，若不需直接求出两点之间的最大流量，我们可以考虑使用邻接表储存。否则使用邻接矩阵则较为方便。想要求出最大流量，我们可以不停的使用广搜寻找出**可行**的增广路径，对于当前搜索的增广路径上的每一个节点，每次对它所可以抵达的每一个点进行扩展，同时记录下该节点的前一个节点，这个扩展需要被扩展的点与当前节点上联通的路径的残余流量为正。当当前节点已经抵达目标节点时，即已经找到了一条可行的增广路径，则可以更新一下最优值。注意：**当更新的时候**，一定要给当前的流量的反向弧加上与它**相等**的容量。

例如：

```c++
f[u][v] = 10
```

则需要加上一个反向容量

```c++
c[v][u] += 10;
```

那么这样做的意义何在？我们有这样的一条定理：当当前残余网络上没有可行的增广路时，则找到了最大流。那么我们给它添加一个反向容量，即为下一次对增广路的搜索做出准备，以找到最大流。

具体的看下面代码实现：

### 代码实现:

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <iostream>
#define REP(i, N, M) for(register int i = N; i <= M; i++)

const int MAXN = 200 + 10;
const int inf = 0x3f3f3f3f;//定义inf最大值，以便比较求出最小值

using namespace std;

queue<int>Q;//队列以进行广搜

int Capacity[MAXN][MAXN]/*存取两点间的容量*/,
Flow[MAXN][MAXN]/*存取两点间的流量*/, Previous[MAXN];
//记录当前节点链接的上一个节点
bool SignQ[MAXN];//标记进行判重
int N, M, Bottlenneck, cnt;

inline void IN();//输入函数

inline bool Search_Augmenting_Path();//求取增广路是否存在

inline void Search_Augmenting_Path_Bottlenneck();
//求取当前增广路上最大的流量，即残存网络下的最大流量，也即剩余的容量最小值

inline void Calc_Flow();//对当前增广路上的流量进行统计

inline void OUT();//输出

int main()
{
    IN();

    while(Search_Augmenting_Path())
      //搜索增广路是否存在， 当增广路存在时，
      //即有一条路从源点到汇点的流量可以大于0，此时还可以增加流量;
    {
        Search_Augmenting_Path_Bottlenneck();
        Calc_Flow();
    }

    OUT();

    return 0;
}

inline void IN()
{
    cin >> N >> M;
    REP(i, 1, N)
    {
        int Node_u, Node_v, Node_Capacity;
        cin >> Node_u >> Node_v >> Node_Capacity;
        Capacity[Node_u][Node_v] += Node_Capacity;
      //读取从Node_u到Node_v路径上的容量
      //注意，从一个点到另一个点的路径可能有多条，
     // 我们可以把它们视为同一条路径，
       //其容量为从该点到另一个店的所有容量之和

    }
}

inline bool Search_Augmenting_Path()
{
    while(!Q.empty())//清空队列
        Q.pop();
    memset(SignQ, 0, sizeof(SignQ));//清空标记

    Q.push(1);//入队，开始广搜
    SignQ[1] = true;
    while(!Q.empty())
    {
        int Node = Q.front();//取出队首
        REP(i, 1, M)
        {
            if(!SignQ[i] && Capacity[Node][i] > Flow[Node][i])
            {//枚举所有结点，当容量大于当前流量时，可以汇入流量
                Q.push(i);
                SignQ[i] = true;
              //将当前节点入队，标记设为true
                Previous[i] = Node;
              //记录当前节点的上一个结点，在后面统计当前增广路的流量时需要用到
                if(i == M)
                  //当i等于M时，即找到了汇点，为一条增广路
                    return true;
            }

        }
        Q.pop();
    }
    return false;
  //否则返回false，此时没有增广路了，需要对最终答案进行统计
}

inline void Search_Augmenting_Path_Bottlenneck()
{
    Bottlenneck = inf;
  //将当前增广路的最大可行流量设为inf，即可以从这条增广路汇入源点的流量
    int Last_Node = M;
    while(Previous[Last_Node]){
      //当当前的结点不为源点时，继续执行
        Bottlenneck =
          min(Capacity[Previous[Last_Node]][Last_Node] - Flow[Previous[Last_Node]][Last_Node], Bottlenneck);
                //更新当前的最大可行流量，即最小的剩余容量
        Last_Node = Previous[Last_Node];
      //将当前节点跳至上一节点，继续更新Bottlenneck
    }
}

inline void Calc_Flow()
{
    int Last_Node = M;
    while(Previous[Last_Node])//当当前节点不为源点时
    {
        Flow[Previous[Last_Node]][Last_Node] += Bottlenneck;
        Capacity[Last_Node][Previous[Last_Node]] += Bottlenneck;
        Last_Node = Previous[Last_Node];
//这里很重要！！当前增广路上的每一条路径需要加上此时的最大流量，
      //同时为了返回上一节点，
      //需要将此路径的反向路径容量加上当前最大流量，
      //这使得下一次可以返回此节点以寻找下一条路
    }
}

inline void OUT()
{
    REP(i, 2, M)
    {
        cnt += Flow[1][i];
    }
    cout << cnt;
}
```
