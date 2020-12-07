---
title: PTA甲级刷题记录——专题扩展
date: 2020-03-28 19:15:15
tags:
---
PTA甲级刷题记录——专题扩展题解（一些复杂的模拟题），方便自己复盘
<!-- more -->
## 1014 Waiting in Line (30分)（大模拟）

> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530135542599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：某个银行有N个窗口，每个窗口前最多可以排M个人。现在有K位客户需要服务，每位客户的服务时长已知。假设所有客户均在08:00按客户编号次序等在黄线外，且如果有窗口的排队人数没有排满（没有达到M人），当前在黄线外的第一个客户就会选择这样的窗口中排队人数最少的窗口排队（排队人数相同时选择窗口序号最小的窗口去排队）。给出Q个查询，每个查询给出一位客户的编号，输出这位客户的服务结束时间。注意，如果一个客户在17:00之后还没有被服务到，则不再服务，输出Sorry；而如果一个客户在17:00之前被服务，那么无论他的服务时长有多长，都会接受完整的服务。
>输入解释：第一行的四个数字表示，N（窗口数），M（每个窗口前最多的排队人数），K（客户数），Q（查询数）
>第二行位K位客户的处理时间
>第三行为需要查询的客户编号

思路：
（1）当一位客户进入某一窗口的队列时，他的服务结束时间就已经确定了，即当钱在窗口排队的所有人的服务时间之和。而在所有窗口排满后，剩余客户能够去排队的时间点是所有窗口最早结束的队首客户，也就是说，在所有窗口排满的情况下，每当有一个窗口的队首客户服务结束（结束时间相同时选id小的），剩余客户的第一个就会排到那个窗口最后面去。
（2）在8:00的时候，只要窗口队列没满，就把客户按照窗口编号为0，1，2.。。。n-1的循环书序进入队列，且安排过程中不堵啊更新窗口的endTime和popTime，其中endTime将直接作为刚入队客户的服务结束时间保存下来，而pop仅在安排每个窗口的第一客户时更新。
（3）如果（2）中已经把所有窗口排满(显然如果没有排满,就不存在剩余在黄线外的客户),那么在该步中将剩下的客户想办法入队。由步骤1可以知道,在所有窗口排满的情况下,每当有一个窗口的队首客户服务结束(结束时间相同的,窗口ID小的视为先结束)剩余客户的第一个就会排到那个窗口最后面去。这样对每一个剩余的客户,可以选出当前所有窗口中 poptime最小的窗口( pop'time相同的选择窗口D较小的),将客户排到该窗口的队列后面,并更新该窗口的 endtime和 poptime,其中 endtime将作为刚入队的客户的服务结束时间保存下来。
（4）对每一个输入的查询客户编号，如果他的服务开始时间在17:00以后，则输出“sorry”，否则，输出服务结束时间。
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <string>
#include <queue>
#include <vector>
int n,m,k,q;//第一行的输入
int query;//当前查询
using namespace std;
struct Window
{
    int end;//d当前窗口队列的最后服务时间
    int pop;//队首客户服务结束时间
    queue<int> q;
};

int convertToMin(int h,int m)
{
    return h*60 +m;//时间处理为分数
}
Window window[20];//所有的窗口数
int needTime[1010];
int ans[1010];
int main(int argc, const char * argv[]) {
    // insert code here...
    scanf("%d%d%d%d",&n,&m,&k,&q);
    for(int i = 0 ; i< k ; i++)
    {
        scanf("%d",&needTime[i]);//读入服务需要时间
        
    }
    for(int i = 0 ; i < n ; i++)//初始化每个窗口的时间为8:00
    {
        window[i].pop = window[i].end = convertToMin(8, 0);
    }
    int index = 0;//d当前第一个为入队的人的编号
    for(int i = 0 ; i< min(n*m , k) ; i++)
    {
        window[index % n].q.push(index);//循环入队
        
        window[index % n].end += needTime[index];//更新次窗口的服务结束时间
        //对窗口的第一个客户，更新popTime
        if(index <n)
            window[index].pop = needTime[index];
        
        //将当前k入队客户的服务结束时间世界保存作为答案
        ans[index] = window[index % n].end;
        index++;
    }
    //处理剩余客户的入队
    for(;index<k ; index++)
    {
        int idx = -1;
        int min = 1<<30;//初始化为2^30
        //寻找所有窗口最小的pop（出队时间）
        for(int i = 0 ; i< n ; i++)
        {
            if(window[i].pop < min)
            {
                idx = i;
                min  = window[i].pop;
            }
        }
        //找到最小的pop的窗口编号为idx，更新该队列的情况
        window[idx].q.pop();
        window[idx].q.push(index);
        window[idx].end += needTime[index];
        window[idx].pop += needTime[window[idx].q.front()];
        //客户index的服务结束时间为该窗口的endtime
        ans[index] = window[idx].end;
        
    
    }
    for(int i = 0 ;i <q ; i++)
    {
        scanf("%d",&query);
        if(ans[query-1] - needTime[query-1] >= convertToMin(17, 0))
        {
            printf("Sorry\n");//r服务开始时间到达17.00
            
        }
        else
        {
              printf("%02d:%02d\n",ans[query-1]/60 ,ans[query-1]%60);
        }
    }
    
    return 0;
}

```
