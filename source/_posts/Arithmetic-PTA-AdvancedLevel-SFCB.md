---
title: PTA甲级刷题记录——算法初步
date: 2020-03-29 09:32:43
categories: 
- PTA甲级刷题
tags:
- 算法
- c++
---
PTA甲级刷题记录——算法初步题解，方便自己复盘
<!-- more -->

## 1010 Radix (25分)（二分）

> ![这里是引用](https://img-blog.csdnimg.cn/20200502181308777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502181322594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 输入四个整数N1，N2，tag，radix。其中tag==1表示N1为radix进制数，tag==2表示N2为radix进制数。范围：N1和N2均不超过10个数位，且每个数位均为0～9或a～z，其中0～9表示数字，a～z表示数字10～35.
> 求N1和N2中未知进制的那个数是否存在，并满足某个进制时和另一个数在十进制下相等的条件。若存在，则输出满足条件的最小进制，否则，输出Impossible

 

 - 将已经确定进制的数放在N1，未确定进制的数放在N2，以便在后面进行统一的计算。
 - 将N1转化为十进制，使用long long类型来存储。考虑对一个字符串来说，它的进制越大，将该数字串转化为十进制的结果也就越大，因此就可以使用二分法。二分N2的进制，将N2从该进制转化为十进制，令其与N1的十进制比较：如果大于N1的十进制，说明N2的当前进制太大，应该往左子区间继续二分；如果小于N2的十进制，说明N2的当前进制太小，应往右子区间继续二分，当二分结束时即可判断解是否存在。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <string>
#include <map>
using namespace std;
typedef  long long LL;
map<char, int>  Map;//0~9 a~z与0～35的对应
LL inf = 9223372036854775807 ;//llong long 的最大值是2^63-1

LL convertNum10(string a,LL radix,LL t)//将a转化为十进制 t为上界
{
    LL ans = 0;
    for(int i = 0 ; i<a.length() ; i++)
    {
        ans = ans * radix +Map[a[i]];//进制转化
        if(ans<0 || ans >t) return -1;//溢出或超过N1的十进制
    }
    return ans;
}
int cmp(string n2,LL radix,LL t )//N2的十进制与t表示
{
    LL num = convertNum10(n2, radix, t);
    if(num < 0) return 1;//溢出 n2>t
    if(t>num)  //t较大返回-1
    {
        return -1;
    }
    else if(t== num)
    {
        return 0;//相等 返回0
    }
    else
    {
        return 1;//num较大 返回1
        
    }
}
LL binarySearcj(string n2 ,LL left,LL right ,LL t)//二分求解N2的进制
{
    LL mid;
    while(left <= right)
    {
        mid = (right+left)/2;
        int flag = cmp(n2, mid, t);
        if(flag == 0)
        {
            return mid;//找到解
        }
        else if(flag == -1)
        {
            left = mid+1;//向右区间查找
        }
        else
        {
            right = mid-1;//向左区间查找
        }
     
    }
       return -1;//解不存在
}
LL findLargestDigit(string n2)//求最大数位
{
    LL ans = -1;
    for(int i = 0 ; i<n2.length() ; i++)
    {
        if(Map[n2[i]] >ans)
        {
            ans = Map[n2[i]];
        }
    }
    return ans +1;
}
string N1,N2, temp;
int tag,radix;
int main(int argc, const char * argv[]) {
    //初始化映射关系
    for(char c = '0' ; c <= '9' ; c++)
    {
        Map[c] = c-'0';
    }
    for(char c = 'a' ; c <= 'z' ; c++)
    {
        Map[c] = c-'a' +10;
    }
    cin>>N1;
    cin>>N2;
    cin>>tag;
    cin>>radix;
//    scanf("%d %d",&tag,&radix);
    if(tag == 2)
    {
        temp = N1;
        N1 = N2;
        N2 = temp;
    }
    LL t = convertNum10(N1, radix, inf);//将N1从radixz进制转化为十进制
    LL low = findLargestDigit(N2);//找到N2中 位数最大的加一，j当成二分下界
    LL high  = max(low,t) +1;//上界
    LL ans = binarySearcj(N2, low, high, t);
    if(ans == -1) printf("Impossible\n");
    else
    {
        printf("%lld\n" ,ans);
    }
    return 0;
}

```
## 1012 The Best Rank (25分)（排序）

> ![这里是引用](https://img-blog.csdnimg.cn/20200529095651512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200529095701239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
题目大意：
已知n个考生的3门课分数C,M,E，而平均分数A可以由这3个分数得到。现在分别按这4个分数对n个考生从高到低排序，对每个考生来说，会有四个排名，且每个分数都会有一个排名。接下来会有m个查询，每个查询输入一个考生的ID，输出该考生4个排名中最高的那个排名以及对应是A，C，M，E中的哪一个。如果不同课程排名相同，则按优先级A>C>M>E输出；如果查询的考生ID不存在，则输出N/A。

思路：
（1）用map<int,vector<int>>存储Ranks的值，key为id，value为每一个成绩的排名，顺序为A>C>M>E
（2）结构体Students存放id和四个分数，顺序为A>C>M>E
（3）读入考生id和三个分数，计算平均分（这里我们计算平均分，直击算了总分，因为平均分和总分排序是相同的）
（4）然后用自带的sort函数对每一个成绩进行排序，now为当前正在计算的成绩，然后将序列存入Ranks
（5）查询时根据query从Ranks中取出对应的value，输出，如果Ranks.count(query) ==0 说明这个query不存在，输出N/A.
```cpp
//
//  main.cpp
//  A1012
//
//  Created by 李超 on 2020/5/29.
//  Copyright © 2020 李超. All rights reserved.
//

#include <iostream>
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <string>
#include <map>
#include <vector>
using namespace std;
struct Student{
    int id;
    int grade[4];
    Student(int s,int a,int c,int m,int e )
    {
        id =s;
        grade[0] = a;
        grade[1] = c;
        grade[2] = m;
        grade[3] = e;
    }
};
vector<Student> students;
int now = 0;//当前正在排序什么元素
bool cmp(Student a,Student b)
{
    return a.grade[now] >b.grade[now];
}
int main(int argc, const char * argv[]) {
    // insert code here...
    int n,m;
    scanf("%d %d",&n,&m);
    int s;
    int c,math,e;
    //l课程与序号的映射
    map<int,char> course;
    course[0] = 'A';
    course[1] = 'C';
    course[2] = 'M';
    course[3] = 'E';
    map<int,vector<int>> Ranks;
    for(int i = 0 ; i< n ; i++)
    {
        scanf("%d%d%d%d",&s,&c,&math,&e);
        int avg = c+math+e;//直接求三门课的总分 不取平均
        students.push_back(Student(s,avg,c,math,e));
        //初始化ranks的map 刚开始所有id的课程排名都设为0 顺序为A C M E
        vector<int> temp;
        temp.push_back(0);
        temp.push_back(0);
        temp.push_back(0);
        temp.push_back(0);
        Ranks[s] = temp;
    }
    for( now = 0 ; now <4 ; now++)//枚举A C M E中的一个
    {
        sort(students.begin(), students.end(), cmp);//对所有考生按该分数从大到小排序
        Ranks[students[0].id][now] = 1;//排序完，将分数最高的设为rank1
        for(int i = 1; i< n ; i++)
        {
            //若与前一位考生分数相同
            if(students[i].grade[now] == students[i-1].grade[now])
            {
                Ranks[students[i].id][now] = Ranks[students[i-1].id][now];//则他们排名相同
            }else
            {
                Ranks[students[i].id][now] =i+1;//否则 设为正确的 排名
                
            }
        }
    }
    int query ;
    for(int i = 0 ; i< m ; i++)
    {
        scanf("%d",&query);
        if(Ranks.count(query) <=0)
        {
            printf("N/A\n");//如果这个id不存在 则输出N/A
        }
        else
        {
            int k = 0 ;
            for(int j = 0 ; j< 4 ; j++)
            {
                if(Ranks[query][j] <Ranks[query][k])
                {
                    k = j;
                }
            }
            printf("%d %c\n",Ranks[query][k],course[k]);
            
        }
        
    }
    return 0;
}


```
