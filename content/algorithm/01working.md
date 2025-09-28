---
title: "😪 懒惰学习算法"
---

# 🌚 懒懒

### 快速幂

#### 题目描述

给你三个整数 $a,b,p$，求 $a^b \bmod p$。

#### 输入格式

输入只有一行三个整数，分别代表 $a,b,p$。

#### 输出格式

输出一行一个字符串 `a^b mod p=s`，其中 $a,b,p$ 分别为题目给定的值， $s$ 为运算结果。

#### 输入输出样例 #1

##### 输入 #1

```
2 10 9
```

##### 输出 #1

```
2^10 mod 9=7
```

#### 说明/提示

**样例解释**

$2^{10} = 1024$，$1024 \bmod 9 = 7$。

**数据规模与约定**

对于 $100\%$ 的数据，保证 $0\le a,b < 2^{31}$，$a+b>0$，$2 \leq p \lt 2^{31}$。

#### Solution

```c++
// 快速幂模板
#include <iostream>
using namespace std;
long long int fastpow(long long int a,long long int b,long long int p){
  long long int res=1;
  while(b){
    if(b&1)res=res*a%p;
    a=a*a%p;
    b>>=1;
  }
  return res;
}
int main()
{
  long long int t;cin>>t;
  while(t--){
    long long int n,m,p;
    cin>>n>>m>>p;
    cout<<fastpow(n,m,p)<<'\n';
  }
  return 0;
}
```



### 最大公约数

```c++
// 最大公约数模板
#include <bits/stdc++.h>
using namespace std;

int gcd(long long int a,long long int b){
	if(b==0)return a;
	else return gcd(b,a%b);
}

int main(){
	long long int n;cin>>n;
	while(n--){
		long long int a,b;cin>>a>>b;
		cout<<gcd(a,b)<<'\n';
	}
	return 0;
}
```



## 动态规划

### 最长公共子序列

#### 输入

> 输入包括多组测试数据。每组数据包括一行，给出两个长度不超过200的字符串，表示两个序列。两个字符串之间由若干个空格隔开。

#### 输出

> 对每组输入数据，输出一行，给出两个序列的最大公共子序列的长度。

#### 样例输入

```plain
abcfbc abfcab
programming contest
abcd mnp
```

#### 样例输出

```plain
4
2
0
```

#### Solution

```c++
#include <bits/stdc++.h>
using namespace std;
int dp[1005][1005];
int main(){
    string a,b;
    while(cin>>a>>b){
        memset(dp,0,sizeof(dp));
        int n=a.size(),m=b.size();
        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){
                if(a[i]==b[j]){
                    dp[i+1][j+1]=dp[i][j]+1;
                }
                else{
                    dp[i+1][j+1]=max(dp[i+1][j],dp[i][j+1]);
                }
            }
        }
        cout<<dp[n][m]<<'\n';
    }
    return 0;
}
```

