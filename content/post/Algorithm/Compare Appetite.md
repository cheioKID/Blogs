---
date: 2018-03-25
title: "Compare Appetite"
tags:
- Algorithm
- Brute force method
categories:
- Algorithm
comment: true
---

# Compare Appetite

## 比饭量

- 描述

3个人比饭量，每人说了两句话： 
A说：B比我吃的多，C和我吃的一样多 
B说：A比我吃的多，A也比C吃的多 
C说：我比B吃得多，B比A吃的多。 
事实上，饭量和正确断言的个数是反序的关系。 
请编程按饭量的大小输出3个人的顺序。

- 输入

无输入

- 输出

按照饭量大小输出3人顺序，比如：
ABC

- 样例输入

```
无
```

样例输出

```
无
```
```c++
#include <iostream>
using namespace std;
int main() {
    int a,b,c,A,B,C;
    char arr[] = "ABC";
    for (a=0; a<=2; a++) {
        for (b=0; b<=2; b++) {
            for (c=0; c<=2; c++) {
                A=((b>a)+(c==a));
                B=((a>b)+(a>c));
                C=((c>b)+(b>a));
                if(a==0 && A!=2) break;
                if(a==1 && A!=1) break;
                if(a==2 && A!=0) break;
                if(b==0 && B!=2) break;
                if(b==1 && B!=1) break;
                if(b==2 && B!=0) break;
                if(c==0 && C!=2) break;
                if(c==0 && C==1) break;
                if(c==1 && C!=1) break;
                if(c==2 && C!=0) break;
                if(A+B+C == 3) {
                    int n[3],no1,no2,no3,min,i,tmp;
                    n[0]=a;
                    n[1]=b;
                    n[2]=c;
                    for (i=0,min=3; i<3; i++) {
                        if(n[i]<min) {
                            min=n[i];
                            no1=i;
                        }
                    }
                    tmp=min;
                    for (i=0,min=3; i<3; i++) {
                        if(n[i]<min && n[i]>tmp) {
                            min=n[i];
                            no2=i;
                        }
                    }
                    
                    for (i=0,min=tmp; i<3; i++) {
                        if(n[i]>min) {
                            min=n[i];
                            no3=i;
                        }
                    }
                    cout<<arr[no1]<<arr[no2]<<arr[no3];
                }
            }
        }
    }
    return 0;
}
```

However, there is something wrong with 

```c++
if(c==0 && C!=2) break;
```

it will suddenly quit the `for (c=0; c<=2; c++)` circle. I wonder why, but it is still not solved.

```c++
#include <iostream>
using namespace std;
int main() {
    int a,b,c,A,B,C;
    char arr[] = "ABC";
    for (a=0; a<=2; a++) {
        for (b=0; b<=2; b++) {
            for (c=0; c<=2; c++) {
                A=((b>a)+(c==a));
                B=((a>b)+(a>c));
                C=((c>b)+(b>a));
                if(a==0 && A!=2) break;
                if(a==1 && A!=1) break;
                if(a==2 && A!=0) break;
                if(b==0 && B!=2) break;
                if(b==1 && B!=1) break;
                if(b==2 && B!=0) break;
                //if(c==0 && C!=2) break; what's wrong?
                if(c==0 && C==1) break;
                if(c==1 && C!=1) break;
                if(c==2 && C!=0) break;
                if(A+B+C == 3) {
                    int n[3],no1,no2,no3,min,i,tmp;
                    n[0]=a;
                    n[1]=b;
                    n[2]=c;
                    for (i=0,min=3; i<3; i++) {
                        if(n[i]<min) {
                            min=n[i];
                            no1=i;
                        }
                    }
                    tmp=min;
                    for (i=0,min=3; i<3; i++) {
                        if(n[i]<min && n[i]>tmp) {
                            min=n[i];
                            no2=i;
                        }
                    }
                    
                    for (i=0,min=tmp; i<3; i++) {
                        if(n[i]>min) {
                            min=n[i];
                            no3=i;
                        }
                    }
                    cout<<arr[no1]<<arr[no2]<<arr[no3];
                }
            }
        }
    }
    return 0;
}
```

That is `Assertion` problem, we can use

```c++
A=((b>a)+(c==a));
B=((a>b)+(a>c));
C=((c>b)+(b>a));
```

to get true assertions' numbers. Thanks to `C` 's feature, true means `1` , false means `0`.
