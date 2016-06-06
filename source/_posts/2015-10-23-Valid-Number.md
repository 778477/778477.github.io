---
layout: post
title: 'Vaild Number'
date: '2015-10-23'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---

[Vaild Number](https://leetcode.com/submissions/detail/43806015/)

判断给定的一个字符串是否是数字。

Note: 题目对于数字的合法定义没有给出具体限定，你应该根据判断返回结果来调整自己的程序。 (这个超坑的，我差不多错误提交了8次。有些后台TestCase的合理性是有待讨论的[吐槽传送门](https://leetcode.com/discuss/3303/the-worst-problem-i-have-ever-met-in-this-oj))


言归正传。这类题目，我们一般称为 模拟题。是因为该题目没有对应具体算法和数据结构，我们只能按照题目描述的分支情况，模拟实现。其实，这个题目我在杭电OJ上做过了。。。

[Is It a Number](http://acm.hdu.edu.cn/showproblem.php?pid=1409)

![accetped](https://raw.githubusercontent.com/778477/778477.github.io/master/img/hud-1409-accepted.png)

当时我就是当模拟题做的，现在看来这种解法不太优雅。 看了一下 Discuss，发现大多数人使用 DFA(有限状态自动机)。城！会！玩！

按照这个思路，我们来动手实践一下。 首先是状态的划分，因为 鉴别是否是数字，字符集{'0'-'9','+','-','.','E'}。根据数字的定义，状态是有限的。 所以我们大体划分出这么集中状态：

	1. UnkownState 起始状态，没有任何字符输入。能接受的下一步字符为 数字|正负号|小数点
	2. NumberState 输入数字状态，能接受的下一步字符为 数字|小数点|科学计数法(E,e)
	3. OnlyNumberState 数字接着输入小数点状态，能接受下一步字符为 数字|科学计数法(E,e)
	4. OperatorState 输入运算法状态，能接受下一步字符为 数字|小数点
	5. DecimalPointState 输入小数点状态，能接受下一步字符为 数字
	6. ScientificState 输入科学计数法状态，能接受下一步字符为 数字|正负号
	7. NoscientificState 科学计数法接着输入正负号状态，能接受下一步字符为 数字
	8. IllegalState 非法输入状态 
	9. EndState 终态 能接受下一步字符 为数字


这里的状态命名比较随意。 状态之间的转移是关键。最后，我们认为 2，3，9状态为合法状态。停留在其他状态的输入情况，不能认为是数字。

```
#include <stdio.h>
#include <string.h>

typedef enum State{
    UnkownState  = 0,       
    NumberState = 1,         
    OnlyNumberState = 2,	 
    OperatorState = 3,      
    DecimalPointState = 4,  
    ScientificState = 5,    
    NoscientificState = 6, 
    IllegalState = 7,
    EndState,
}State;

int isNumber(char* s) {
    State state = UnkownState;
    while (*s == ' ')  s++;
    char *p = (s+strlen(s)-1);
    while (*p == ' ')  --p;
    *(p+1) = 0;


    while (*s) {
        switch (state) {
            case UnkownState:{
                state = IllegalState;
                if(*s == '+' || *s == '-') state = OperatorState;
                if(*s >= '0' && *s <= '9') state = NumberState;
                if(*s == '.') state = DecimalPointState;
                break;
            }
            case NumberState:{
                state = IllegalState;
                if(*s >= '0' && *s <= '9') state = NumberState;
                if(*s == '.') state = OnlyNumberState;
                if(*s == 'e' || *s == 'E') state = ScientificState;
                break;
            }
            case OnlyNumberState:{
                state = IllegalState;
                if(*s >= '0' && *s <= '9') state = OnlyNumberState;
                if(*s == 'e' || *s == 'E') state = ScientificState;
                break;
            }
            case OperatorState:{
                state = IllegalState;
                if(*s >= '0' && *s <= '9') state = NumberState;
                if(*s == '.' ) state = DecimalPointState;
                break;
            }
            case DecimalPointState:{
                state = IllegalState;
                if(*s >= '0' && *s <= '9') state = OnlyNumberState;
                break;
            }
            case ScientificState:{
                state = IllegalState;
                if(*s == '+' || *s == '-') state = NoscientificState;
                if(*s >= '0' && *s <= '9') state = EndState;
                break;
            }
            case NoscientificState:{
                state = IllegalState;
                if(*s >= '0' && *s <='9') state = EndState;
                break;
            }
            case EndState:{
                state = IllegalState;
                if(*s >= '0' && *s <='9') state = EndState;
                break;
            }
            case IllegalState:{
                return 0;
            }
        }
        s++;
    }


    return (state == NumberState || state == OnlyNumberState || state == EndState);

}

int main(){
    char str[1024];
    while(gets(str)){
        printf("%s %s\n",str,isNumber(str) ? "is a Number":"isn't a Number");
    }
    return 0;
}

```

![accepeted](https://raw.githubusercontent.com/778477/778477.github.io/master/img/vaild-number-accepted.png)