---
title: PTA甲级刷题记录——动态规划专题
date: 2020-03-29 09:33:56
categories: 
- PTA甲级刷题
tags:
- 动态规划
- 算法
- c++
---
PTA甲级刷题记录——动态规划题解，方便自己复盘
<!-- more -->

## A1007 Maximum Subsequence Sum (25分)(最大连续子序列和)

> ![这里是引用](https://img-blog.csdnimg.cn/20200502154018553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 给定一个数字序列a1,a2.....an,求i,j，使得ai+...+aj最大，输出最大和以及
>ai,aj.

**最大连续子序列和：**
（1）令状态dp[i]表示以A[i]作为末尾的连续序列的最大和（A[i]必须作为连续序列的末尾）
通过设置dp数组，要求的最大和其实是dp[0],dp[1].....dp[n-1]中的最大值
（2）dp[i]必须是以A[i]作为末尾连续序列，只有两种情况

 - 这个最大的连续序列只有一个元素，即以A[i]开始A[i]结束
 - 这个最大和的连续序列有多个元素，即从前面某处A[p]开始，一直到A[i]结尾。
 对于第一种情况，最大和就是A[i]本身
 对于第二种情况，最大和是dp[i-1] +A[i]
 则转台转移方程是dp[i] = max{A[i],dp[i-1] +A[i]};

以s[i]表示以A[i]作为结尾的最大连续子序列是从那个元素开始的（记录下标），根据上面的策略：

 - 第一种情况，只有一个元素，这个最大的连续子序列就是从A[i]，开始，于是s[i] = i；
 - 第二种情况，注意到dp[i]和dp[i-1]使用的是同一个其实元素p，因此s[i]=s[i-1]
 
 最后只需要得到dp[0]...dp[n-1]最大值的过程中记录最大值的下标k，然后输出dp[k]],A[s[k]],A[k]即可

```cpp
//
//  main.cpp
//  A1007
//
//  Created by 李超 on 2020/5/2.
//  Copyright © 2020 李超. All rights reserved.
//

#include <iostream>
using namespace std;
int a[10002] = {0},dp[10002]= {0};//a[i]存放序列  dp[i]存放以a[i]结尾的连续序列的最大和
int s[10002] = {0};//s[i]表示产生dp[i]的连续序列从a的那一个元素开始的
int main(int argc, const char * argv[]) {
    int n;//数字数量
    cin>>n;
    bool flag = false;//flag表示数组a中是否全小于0
    for(int i  = 0 ; i < n ; i++)
    {
        scanf("%d",&a[i]);
        if(a[i]>=0) flag =true;//只要有一个数>=0 flag=true
    }
    if(flag == false)//如果全为0 则输出
    {
        printf("0 %d %d\n",a[0],a[n-1]);
        return 0;
    }
    
    dp[0]  = a[0];//边界
    for(int i = 1 ; i< n ; i++)
    {
        if(dp[i-1] +a[i] > a[i])
        {
            dp[i] = dp[i-1] +a[i];
            s[i] = s[i-1];
        }
        else
        {
            dp[i] = a[i];
            s[i] = i;
        }
    }
    //因为dp[i]存放的是以a[i]结尾的连续最大和 因此需要遍历i得到最大的才是结果
    int k = 0;
    for(int i = 0 ; i< n ; i++)
    {
        if(dp[i] >dp[k])
        {
            k = i;
        }
    }
     printf("%d %d %d\n",dp[k],a[s[k]],a[k]);
    return 0;
}

```
