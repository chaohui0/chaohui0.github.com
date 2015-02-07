---
layout: post
title: 数据结构与算法 
category: secret
description: 要拿出CLC顽强撸到1250的斗志！
---

###约瑟夫环

找了好久终于找到一个看得懂的推导过程：

新的序列：0  1  2  3 ...  m-2        m   m+1 ... n-1     从 编号 m 开始新的一轮游戏，
                                                              ||   因为从编号m开始，把后半段放到前面
                                                              \/         
                                 m   m+1   ...  n-1       0      1      2   ...   m-2
                                                              ||   编号变换 <1>, 右半部分加上 n
                                                              \/
                                 m    m+1  ...   n-1      n     n+1   n+2 ...  n-m-2
                                                              ||  编号变换 <2>, 全部减去 m
                                                              \/
                                 0    1   2   3  .......X .............    n - 2
假设红色数字 X 是 [ 0 1 2 ... n-2] 中最终存留的序号，我们的目标是得到这个编号经过了两次编号变换前的编号 x'。
两次编号变换的逆变换其实就是                   x'  = ( X + m) % n;
因此，我们得到了递推关系式   f [n] = ( f[n-1] + m ) % n;
代码如下：
// copyright @ L.J.SHOU Mar.11, 2014
// jossef circle
#include <iostream>
#include <vector>
using namespace std;
int JosefCircle(int m, int n)
{
  vector<int> f(n+1, 0);
  for(int i=2; i<=n; ++i)
    f[i] = (f[i-1] + m) % i;
  return f[n];
}


[LinChaohui]:    http://www.linchaohui.com  "LinChaohui"
