---
date: 2018-06-15
title: "LL(1) Grammer"
tags:
    - Compiling principle
    - Grammar
categories:
    - CS
comment: true
---

#### LL(1)分析表的构造

1.       LL(1)文法没有公共左因子，也不含左递归，并且不是二义的。根据文法可以得到FIRST和FOLLOW集合，如果文法的任意两个产生式A→𝛼|𝛽，FIRST(𝛼) 和FIRST(𝛽)交集为空，即没有公共左因子，若𝛽经过若干步推导为𝜀，那么FIRST(𝛼) 和FOLLOW(A)交集为空，那么𝛽→𝜀的情况就是具体的，也不存在任何冲突。这样的文法就属于LL(1)文法。

2.   根据FIRST和FOLLOW集合可以的到LL(1)分析表，分析表的行对应非终结符，列对应终结符，将每个产生式根据产生式左部的非终结符和产生式右部的FIRST集合填入分析表，如果是推出𝜀的情况，需要根据产生式右部的FOLLOW集合来确定，如果某个非终结符属于产生式右部的FOLLOW集合，那么将该产生式填入分析表。

3.   采用LL(1)文法预测分析过程中没有左递归导致的无限循环，也没有公共左因子导致的回溯现象，根据预测分析表，每一步的推到都是确定的。

#### 预测分析过程

1.   预测分析器中，符号栈初始状态下只有一个开始符号，开始符号根据输入栈栈顶的字符进行推导。
2.   如果符号栈栈顶为非终结符，根据该非终结符和面临的输入栈栈顶的终结符可以得到产生式动作，将符号栈栈顶的产生式左部推导为产生式右部。推导的过程中，产生式左部的字符即符号栈栈顶的字符出栈后，产生式右部压栈。因为预测分析采用最左推导，所以采用反序压栈，使最左边的非终结符在出现在栈顶，优先被推导。如果该非终结符和面临的输入栈栈顶的终结符在LL(1)分析表中没有对应的产生式动作，则出错。
3.   如果符号栈栈顶为终结符，判断是否和面临的输出栈栈顶的终结符匹配，如果匹配，两个栈的栈顶元素都出栈，继续分析下一个栈顶符号。如果不匹配则出错情况。
4.   循环到符号栈和输入栈都为空，则预测分析完毕。

#### 紧急错误恢复

1.       紧急错误恢复是一种简单的错误恢复方法，发现错误时弹出输入记号，直到输入记号属于同步记号集为止。

2.       如果终结符在栈顶但是不匹配，最简单的方法就是弹出这个终结符，然后继续分析栈顶记号。

3.       非终结符的FOLLOW集合中的终结符可以作为该非终结符的同步记号，这样可以保证错误恢复后的文法属于LL(1)文法。在实验中采用了这种做法，当得到sync时，弹出栈顶的非终结符，恢复分析

```c
#include<string.h>
#include<stdio.h>
int ll1[5][6]={{1,0,0,1,99,99},
				{0,2,0,0,3,3},
				{4,99,0,4,99,99},
				{0,6,5,0,6,6},
				{8,99,0,7,99,99}};

int main() {
	char stack[10] = {'$','E'};
	char input[10];
	char str[10];
	char ch;//当前看到的字符
	int i,j,m,n;
	int l = 1;//栈的大小
	int k = 0;//当前看到的字符的指针
	int action;
	int step = 1;
	int length = 0;
	
	printf("请输入，输入$结束:");
	do {
		scanf("%c",&ch);
		if(ch == '\n')
			continue;
		input[length]=ch;
		str[length]=ch;
		length++;
	} while(ch != '$');

	printf("--------------------------------------------------\n");
	printf("No\t栈\t\t\t输入\t动作\t\t\n");
	printf("--------------------------------------------------\n");
	
	do {
		ch=str[k];
		
		printf("%d\t",step);
		
		for(i=0;i<=l;i++)
			printf("%c",stack[i]);
		printf("\t\t\t");

		for(i=0;i<k;i++) {
			input[i]=' ';
			printf("%c",input[i]);
		}
		for(i=k;i<length;i++)
			printf("%c",input[i]);
		printf("\t");

		switch(ch) {
			case 'i':
				j=0;break;
			case '+':
				j=1;break;
			case '*':
				j=2;break;
			case '(':
				j=3;break;
			case ')':
				j=4;break;
			case '$':
				j=5;break;
			defult:
				j=-1;break;
		}/* switch(ch)*/
		
		if(j != -1) { //看到非终结符
			if(stack[l] != ch) { //栈顶元素和当前看到的字符不匹配
				if(stack[l] != 39) { //栈顶元素为E,T,F
					switch(stack[l]) {
						case 'E':
							m=0; break;
						case 'T':
							m=2;break;
						case 'F':
							m=4;break;
						default:
							m=-1;break;
					}
				} else { //栈顶元素为E',T'
					switch(stack[l-1]) {
						case'E':
							m=1; break;
						case 'T':
							m=3;break;
						default:
							m=-1;break;
					}
				}/* if(stack[l]) == '''*/
			}/*if stack[l] != ch*/
			
			if(m != -1) { //栈顶元素为任一非终结符
				if(stack[l] != ch) { //栈顶元素和当前看到的字符不匹配
					if(stack[l] == 'i' || 
							stack[l] == '+' || 
							stack[l] == '*' || 
							stack[l] == '(' || 
							stack[l] == ')') {
						l=l+1;
						printf("%c被弹出",ch);
						step+=1;
						//break;
					}


					action=ll1[m][j]; //根据非终结符和终结符得到产生式动作
					if(action == 1) {
						printf("输出E→TE'\n");
						n=3;
						l=l+n-1;
						stack[l]='T';
						stack[l-1]=39;
						stack[l-2]='E';
						step=step+1;
						
					} else if(action == 2) {
						printf("输出E'→+TE'\n");
						n=4;
						l=l+n-2;
						stack[l]='+';
						stack[l-1]='T';
						stack[l-2]=39;
						stack[l-3]='E';
						step = step+1;

					} else if(action == 3) {
						printf("输出E'→𝜀\n");
						l=l-2;//无任何元素入栈
						step=step+1;

					} else if(action == 4) {
						printf("输出T→FT'\n");
						n=3;
						l=l+n-1;
						stack[l]='F';
						stack[l-1]=39;
						stack[l-2]='T';
						step = step+1;
					} else if(action == 5) {
						printf("输出T'→*FT'\n");
						n=4;
						l=l+n-2;
						stack[l]='*';
						stack[l-1]='F';
						stack[l-2]=39;
						stack[l-3]='T';
						step = step+1;
					} else if(action == 6) {
						printf("输出T'→𝜀\n");
						l=l-2;
						step = step+1;
					} else if(action == 7) {
						printf("输出F→(E)\n");
						n=3;
						l=l+n-1;
						stack[l]='(';
						stack[l-1]='E';
						stack[l-2]=')';
						step = step+1;
					} else if(action == 8) {
						printf("输出F→i\n");
						n=1;
						l=l+n-1;
						stack[l]='i';
						step=step+1;
					} else if(action == 0){
						printf("出错：跳过%c\n",ch);
						k+=1;
						step+=1;
					} else if(action == 99) {
						if(m==0 || m==2 || m==4) {
							printf("出错：%c正好在%c的同步记号集合中，
									无需跳过任何记号;%c被弹出\n",
									ch,stack[l],stack[l]);
							l=l-1;
						}
						else if(m==1 || m==3) {
							printf("出错：%c正好在%c%c的同步记号集合中
									，无需跳过任何记号;%c%c被弹出\n",
									ch,stack[l-1],stack[l],stack[l-1],stack[l]);
							l=l-2;
						}
						else
							printf("不是正确的非终结符\n");
					} else {
						printf("未定义的错误");	
					}//判断查表所得的动作

				} else { // 栈顶元素和当前看到的字符匹配
					if(ch == '$' && stack[l] == '$') {
						printf("分析成功，结束\n");
						exit(0);
					} else {
						printf("匹配%c\n",ch);
						l=l-1; //栈的大小减一
						k=k+1; //输入指针调整
						step+=1;
					}
				}
			} else {
				printf("出错，跳过\n");
				exit(0);
			} 
		} else { //没有看到正确的终结符
			printf("错误的终结符\n");
			exit(0);
		}
	} while(l>=0); //栈大小

	return 0;
}

```