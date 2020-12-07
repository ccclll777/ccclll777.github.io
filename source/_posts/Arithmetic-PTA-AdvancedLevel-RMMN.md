---
title: PTA甲级刷题记录——入门模拟
date: 2020-03-29 09:32:35
categories: 
- PTA甲级刷题
tags:
- 模拟
- 算法
- c++
---
PTA甲级刷题记录——入门模拟题解，方便自己复盘
<!-- more -->
## A1001  A+B Format (20分)(字符串处理）

> ![这里是引用](https://img-blog.csdnimg.cn/20200429093840489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 给出两个正数a,b（不超过10^9）,求a+b的值，并按照标准格式输出（xxx,xxx,xxx,xxx）


**思路：**
（1）将a，b累加，判断是否为负，为负则输出负号，并将sum取绝对值，为正则不作处理
（2）将sum模式取余，结果存入数组
（3）输出时从高位输出，每输出3个数字则输出1个逗号，最后不输出逗号
**注意：**
必须判断sum为0的情况，否则有个测试点过不去
```cpp
#include <iostream>
#include <cmath>
using namespace std;
int main(int argc, const char * argv[]) {
    int a,b,sum;
    scanf("%d%d",&a,&b);
    sum = a+b;
    if(sum < 0)//sum为负数时，输出负号，将sum取绝对值
    {
        cout<<"-";
        sum = abs(sum);
    }
   
    int num[20];
    int i = 0;
    //当sum为0是e特殊处理 否则有一个测试点过不去
    if(sum == 0)
    {
        num[i] = 0;
        i++;
    }
    //将每一位放入数组
    while (sum != 0) {
     
        num[i] = sum%10;
        sum = sum/10;
        i++;
    }
    //从高位输出，每三位c输出一个逗号，最后一位除外
    for(int j = i - 1 ; j>= 0 ; j--)
    {
         cout<<num[j];
        if(j>0 && j%3 == 0)
        {
            cout<<",";
        }
    }
    return 0;
}

```
## A1002 A+B for Polynomials (25分)（简单模拟）

> ![这里是引用](https://img-blog.csdnimg.cn/20200429100152182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
>给出两行，每行表示一个多项式，第一个数表示该多项式中系数非零项的项数，后面每两个数表示一项，这两个数分别表示该项的幂次和系数。试求两个多项式的和，并以前面相同的格式输出结果

（1）数组p存储多项式的系数，p[i]为幂次为i的系数
（2）输入多项式时，就存入对应的位置，相加
（3）输出时计算系数的非0项，按照系数从大到小输出即可
注意：格式化输出问题，输出一位小数(%.1f).
```cpp
#include <iostream>
#include <cstdio>
using namespace std;
double p[1111] = {0};
int main(int argc, const char * argv[]) {
    // insert code here...
    int a ;//项数
    scanf("%d",&a);
    //系数和幂次
    float b;
    int n;
    
    for(int i = 0 ; i< a ; i++)
    {
        scanf("%d%f",&n , &b);
        p[n] += b;
    }
     scanf("%d",&a);
      for(int i = 0 ; i< a ; i++)
       {
           scanf("%d%f",&n , &b);
           p[n] += b;
       }
    int count =0;
    for(int i = 0 ; i< 1111 ; i++)
    {
        if(p[i] != 0.0)
        {
            count ++;
        }
    }
     printf("%d",count);
    for(int i = 1111 ; i >= 0 ; i--)
    {
        if(p[i] != 0.0)
        {
            printf(" %d %.1f",i,p[i]);
        }
    }
    return 0;
}

```

## A1005 Spell It Right (20分)(字符串处理)

> ![这里是引用](https://img-blog.csdnimg.cn/20200430160546808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 给定一个非负数N,求出每一位只和，并用英语表示这个总和的每一位


使用map存放数字到单词的映射
使用string库进行字符串的按位相加即可
```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;
int main(int argc, const char * argv[]) {
    // insert code here...
    map<char,string> a;
    a['0'] = "zero";
    a['1'] = "one";
    a['2'] = "two";
    a['3'] = "three";
    a['4'] = "four";
    a['5'] = "five";
    a['6'] = "six";
    a['7'] = "seven";
    a['8'] = "eight";
    a['9'] = "nine";
    string s;
    cin>>s;
    int num = 0;
    for(int i = 0 ; i< s.length() ; i++)
    {
        num += (s[i]-'0');
    }
    string result = to_string(num);
    cout << a[result[0]];
    for(int i = 1 ; i <result.length() ; i++)
    {
        cout<<" "<<a[result[i]];
      
    }
    return 0;
}

```


## A1006 Sign In and Sign Out (25分)(查找元素)

> ![这里是引用](https://img-blog.csdnimg.cn/2020043016394950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 每天第一个到机房的人要开门，最后一个离开的人要关门。现在有一堆再按的机房签到，签离记录，请根据记录找出当天开门和关门的人。

（1）结构体记录每个人的信息
（2）将输入格式化后，存入vector
（3）遍历vector进行比较，取最小开始时间和最大结束时间，并记录。
（4）输出对应的id。
```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <vector>
using namespace std;
struct Node
{
    string id;
    int start_h,start_m,start_s;
    int end_h,end_m,end_s;
};

int main(int argc, const char * argv[]) {
    // insert code here...
    int n;
    scanf("%d",&n);
    int h,m,s;//输入的时 分 秒
    string id;//id
    vector<Node> list;
    for(int i = 0 ; i < n ; i++ )
    {
        cin>>id;
        scanf(" %d:%d:%d",&h,&m,&s);
        Node node;
        node.id = id;
        node.start_h = h;
        node.start_m = m;
        node.start_s = s;
         scanf("%d:%d:%d",&h,&m,&s);
        node.end_h = h;
        node.end_m = m;
         node.end_s = s;
        list.push_back(node);
    }
    Node  early = list[0];
    Node  late = list[0];
    for(int i = 1 ; i< list.size() ; i++)
    {
        Node temp = list[i];
        if((early.start_h > temp.start_h) ||(early.start_h == temp.start_h && early.start_m > temp.start_m)||(early.start_h == temp.start_h && early.start_m == temp.start_m && early.start_s == temp.start_s))
        {
            early = temp;
        }
      if((early.end_h < temp.end_h) ||(early.end_h == temp.end_h && early.end_m > temp.end_m)||(early.end_h == temp.end_h && early.end_m == temp.end_m && early.end_s == temp.end_s))
             {
                 late = temp;
             }
        
    }
    cout<<early.id<<" "<<late.id<<endl;
   
    return 0;
}

```

## A1009 Product of Polynomials (25分)(简单模拟)

> ![这里是引用](https://img-blog.csdnimg.cn/2020050108501960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 给出两个多项式，求这两个多项式的乘积。


（1）现保存第一个输入到vector
（2）读取第二行输入时，遍历vertor计算结果，存入ans中，ans的index为指数，里面的值为系数
（3）将ans中不为0的项从大到小输出
```cpp
#include <iostream>
#include <vector>
using namespace std;
struct Poly
{
    int exp;//指数
    double cof;//系数
    
};
int main(int argc, const char * argv[]) {
    // insert code here...
    int n,m;
    scanf("%d",&n);
    int exp;
    double cof;
    vector<Poly> list ;//第一个多项式
    for(int i = 0 ; i< n ; i++)
    {
        cin>>exp;
        cin>>cof;
        Poly poly;
        poly.cof = cof;
        poly.exp  = exp;
        list.push_back(poly);
    }
     scanf("%d",&m);
    double ans[2002] ={0.0};//存放结果
    for(int i = 0 ; i<m ; i++)
    {
        cin>>exp;
        cin>>cof;
        for(int j = 0 ; j < list.size() ; j ++)
        {
            ans[exp +list[j].exp] += (cof *list[j].cof);
        }
    }
    int count = 0;
    for(int i = 0 ; i< 2002 ; i++)
    {
        if(ans[i] != 0.0) count ++;
    }
    printf("%d",count);
    for(int i = 2001 ; i>=0 ; i--)
    {
        if( ans[i] != 0.0)
        {
            printf(" %d %.1f",i,ans[i]);
        }
    }
    return 0;
}

```
## 1011 World Cup Betting (20分)（查找元素）

 

> ![这里是引用](https://img-blog.csdnimg.cn/2020052908535365.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 给出三场比赛的赔率，正确率为65%，每次投2元，问最大期望收入，并输出最大收入时比赛的结果


问题：读取数据时，用scanf("%f",&a)；读取失败，不知道什么问题，改成了cin
```cpp

#include <iostream>
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <string>
#include <map>
using namespace std;
int main(int argc, const char * argv[]) {
    // insert code here...
    double ans = 1.0 ,temp;
    float a;
    map<int,char> m;
    m[0] = 'W';
     m[1] = 'T';
     m[2] = 'L';
    
    int index ;//每行最大数字的下标
    for(int i = 0 ; i< 3 ; i++)
    {
        temp = 0.0;
        for(int j = 0 ; j < 3 ; j++)//寻找每行最大的数
        {
            cin>>a;
         
            if(temp<a)
            {
                temp = a;
                index = j;
            }
        }
        ans *=temp;
        printf("%c ",m[index]);//输出比赛结果
        
    }
    printf("%.2f",(ans *0.65 -1) *2);
    return 0;
}

```


