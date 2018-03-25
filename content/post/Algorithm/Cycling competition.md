---
date: 2018-03-25
title: "循环比赛"
tags:
- Algorithm
- Divide and Conquer
categories:
- Algorithm
comment: true
---

# 循环比赛

## Cycling competition

- 描述

  设有N个选手的循环比赛。其中N=2^M(2的M次方)，要求每名选手要与其他N-1名选手都赛一次，每名选手每天比赛一次，循环赛共进行N-1天，要求每天没有选手轮空。

- 输入

  M

- 输出

  表格形式的比赛安排表

- 样例输入

  ```
  3
  ```

- 样例输出

  ```
  1 2 3 4 5 6 7 8 
  2 1 4 3 6 5 8 7
  3 4 1 2 7 8 5 6
  4 3 2 1 8 7 6 5
  5 6 7 8 1 2 3 4
  6 5 8 7 2 1 4 3
  7 8 5 6 3 4 1 2
  8 7 6 5 4 3 2 1
  ```

  ​

- 提示

  M的大小不会超过8

```c++
#include <iostream>
using namespace std;
int table[256][256]={0};
void match(int n) {
    int size=1<<n;
    int half=size/2;
    if(n==1) {
        table[0][0]=1;
        table[0][1]=2;
        table[1][0]=2;
        table[1][1]=1;
    } else {
        match(n-1);
        for (int i=0; i<half; i++) {
            for (int j=0; j<half; j++) {
                table[half+i][half+j]=table[i][j];
                table[i][half+j]=table[i][j]+half;
                table[i+half][j]=table[i][j]+half;
            }
        }
    }
}
int main() {
    int n;
    cin>>n;
    int size=1<<n;
    match(n);
    for (int i=0; i<size; i++) {
        for (int j=0; j<size; j++) {
            cout<<table[i][j]<<" ";
        }
        cout<<endl;
    }
    return 0;
}
```

