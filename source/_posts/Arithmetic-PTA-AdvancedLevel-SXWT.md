---
title: PTA甲级刷题记录——数学问题
date: 2020-03-29 09:32:48
categories: 
- PTA甲级刷题
tags:
- 算法
- c++
---

PTA甲级刷题记录——数学问题题解，方便自己复盘
<!-- more -->
## A1008 Elevator (20分)(简单数学)

> ![这里是引用](https://img-blog.csdnimg.cn/20200502174542382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：
> 有一部电梯，最开始停在第0层，上一层需要6s，下一层需要4s，每次到达当前的目的楼层需要停留5s。现给出电梯要去的楼层的顺序，求总共花费的时间（最后不需要回到第0层）


```cpp

#include <iostream>
using namespace std;
int main(int argc, const char * argv[]) {
    // insert code here...
    int n;
    cin>>n;
    
    int now = 0;//当前位置
    int ans = 0;//消耗
    int to;//目的位置
    for(int i = 0 ; i< n ; i ++)
    {
        cin>>to;
        if(to>now)
        {
            ans += (to-now)*6;
            
        }
        else
        {
            ans += (now-to) *4;
        }
        ans+=5;
        now = to;
    }
    cout<<ans<<endl;
    return 0;
}

```


## 1015 Reversible Primes (20分)(素数）

> ![这里是引用](https://img-blog.csdnimg.cn/20200530192915899.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
> 题目大意：给出正整数N和进制radix，如果N是素数且N在radix进制下反转后的数在十进制下也是素数，则输出“YES”,否则输出“NO”。

思路：（1）判断是否为素数，从2开始到n的平方根，看有没有数能整除n，如果没有，n就是素数

```cpp
#include <iostream>
#include <iostream>
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <string>
#include <queue>
#include <vector>
#include <cmath>
using namespace std;
bool isPrime(int n)
{
    //判断n是否为素数
    if(n <= 1) return false;
    for(int i = 2 ; i<= int(sqrt(n*1.0)) ; i++)
    {
        if( n%i ==0 )
        {
            return false;
        }
    }
    return true;
}
int main(int argc, const char * argv[]) {
    // insert code here...
    int n,radix;
    int d[110];
    while (scanf("%d",&n) != EOF) {
        if(n < 0) break;
        scanf("%d",&radix);
        if(isPrime(n) == false) //如果不是素数，输出No，结束算法
        {
            printf("No\n");
        }
        else{
            int len = 0;
           do {
                d[len++] = n%radix;
                n = n/radix;
           } while (n!=0);
            
            for(int i = 0 ; i< len ; i++) // 逆序进制
            {
                n = n*radix +d[i];
            }
            if(isPrime(n) ) printf("Yes\n");
            else printf("No\n");
        }
    }
    return 0;
}

```

