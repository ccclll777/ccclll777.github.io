---
title: PTA甲级刷题记录——数据结构专题
date: 2020-03-29 09:33:19
categories: 
- PTA甲级刷题
tags:
- 数据结构
- 算法
- c++
---
PTA甲级刷题记录——数据结构题解，方便自己复盘
<!-- more -->
## A1004 Counting Leaves (30分) (树的遍历)

> ![这里是引用](https://img-blog.csdnimg.cn/20200502113816772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 给出一颗树，问每一层各有多少叶节点。
> 样例解释：
> 这棵树有两个节点，其中一个节点是非叶节点。
> 根结点01号有一个孩子——02号。这棵树有两层，第一层只有根节点01号，且有子节点02号，第二层只有一个节点02号，没有孩子节点。


**1.DFS**
```cpp
#include <iostream>
#include <cmath>
#include <vector>
using namespace std;
vector<int> graph[101];//存放树
int leaf[101] = {0};//每层叶节点数量
int depth = 1;//树的深度
void dfs(int s,int h)
{
    depth = max(h,depth);
    if(graph[s].size() == 0)
    {
        leaf[h]++;
        return;
    }
    for(int i = 0 ; i<graph[s].size() ; i ++)
    {
        dfs(graph[s][i], h+1);
    }
    
    
}
int main(int argc, const char * argv[]) {
    // insert code here...
    int n,m,parent,child,k;
    scanf("%d%d",&n,&m);
    for(int i = 0 ; i < m ; i ++)
    {
          scanf("%d%d",&parent,&k);//父节点编号以及子节点个数
        for(int j = 0 ; j< k ; j++)
        {
            scanf("%d",&child);
            graph[parent].push_back(child);
        }
    }
    dfs(1, 1);
    cout<<leaf[1];
    for(int i =2 ; i <= depth ; i++)
    {
        cout<<" "<<leaf[i];
    }
    return 0;
}
```

2.BFS

```cpp
#include <iostream>
#include <cmath>
#include <vector>
#include <queue>
using namespace std;
vector<int> graph[101];//存放树
int leaf[101] = {0};//每层叶节点数量
int depth = 1;//树的深度
int h[101] = {0};//存储最大深度
void dfs(int s,int h)
{
    depth = max(h,depth);
    if(graph[s].size() == 0)
    {
        leaf[h]++;
        return;
    }
    for(int i = 0 ; i<graph[s].size() ; i ++)
    {
        dfs(graph[s][i], h+1);
    }
    
    
}
void bfs(int s)
{
    h[1] = 1;
    queue<int> q;
    q.push(s);//从节点s开始bfs
    while (!q.empty()) {
        int id = q.front();//弹出队首
        q.pop();
        depth = max(depth, h[id]);//更新最大深度
        if(graph[id].size() == 0)//如果该节点是叶子
        {
            leaf[h[id]] ++;
        }
        for(int i = 0 ; i<graph[id].size() ; i++)//枚举所有子节点
        {
            h[graph[id][i]] = h[id] +1;
            q.push(graph[id][i]);
        }
    }
}
int main(int argc, const char * argv[]) {
    // insert code here...
    int n,m,parent,child,k;
    scanf("%d%d",&n,&m);
    for(int i = 0 ; i < m ; i ++)
    {
          scanf("%d%d",&parent,&k);//父节点编号以及子节点个数
        for(int j = 0 ; j< k ; j++)
        {
            scanf("%d",&child);
            graph[parent].push_back(child);
        }
    }
    
    bfs(1);
    cout<<leaf[1];
    for(int i =2 ; i <= depth ; i++)
    {
        cout<<" "<<leaf[i];
    }
    return 0;
}

```
