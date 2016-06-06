---
layout: post
title: 'Regular Expression Matching'
date: '2015-10-01'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---


https://leetcode.com/problems/regular-expression-matching/

这个题目比较有意思，大意是实现正则表达式中的'*'和'.'这种两种操作符。

我们来看看 这两种操作符是做什么的

> '.' Matches any single character.

> '*' Matches zero or more of the preceding element.

意思是说，'.'这个操作符能匹配任意单个字符，'\*' 这个操作符能匹配 **任意次** 在它前面的符号。恩，看起来'\*'是压缩连续字符串的。

我们再看看题目中的几个case，分析一下最后三个吧：

>case 1 isMatch("aa", ".*") → true 

>case 2 isMatch("ab", ".*") → true 

>case 3 isMatch("aab", "c*a*b") → true 

case 1比较好理解，'.'匹配第一个'a'，'\*'匹配了第二个'a'. 因为 '\*'前面一个字符是'a'

case 2 这个case，我一开始一直理解不了，一度以为是题目搞错了。重新读题的时候，才发现题意说的很明白，是我误解了题意。

**'\*' Matches zero or more of the preceding element.**

反复理解这句话，意思是指'\*'能表示任意个在它前面的字符。也就是说，它前面如果是'.'这种操作符的话，".\*"可以是".","..","..."。

那么，之前case 1的理解也是错的，'\*'操作符的优先级比较高，匹配时要优先考虑'\*'的存在。

case 1应该是这么理解 先是'\*'表示了一个在它前面的'.'，使得模式串变成了".."，最后再去匹配。

case 2也能这么解释。 

case 3，第一个'\*'的存在把前面的'c'表示了0次，第二个'\*'的存在把前面的'a'表示了两次，把模式串变成了"aab"

```C
bool isMatch(char* s, char* p) {
    if(*p == 0) return *s == 0;
    /**
     *  因为 '*'优先级较高,且影响前一个字符。所以优先考虑'*'存在的情况
     *  如果 当前字符的下一个字符不是 '*'的话, 进行正常的匹配。 注意'.'的情况。
     */
    if(*(p+1) != '*'){
        // 当前字符匹配成功的条件是 *s 存在 且 *s和*p相同
        if(*s && ((*s == *p) || (*p == '.'))){
            return isMatch(s+1,p+1);
        }
        // 当前字符匹配不通过，无须匹配后面的字符了。
        return false;
    }
    else{
        // 枚举 '*'操作符 表示n次当前字符,直到当前字符失效(失效的意思就是说 '*'影响当前字符的所有情况都我们枚举过了,不能考虑'*'的意义了,做下一步工作吧)
        while(*s && ((*s==*p) || (*p == '.'))){
            if(isMatch(s, p+2)) return true;
            s++;
        }
        // 放弃这个 '*'操作符的机会，也就是说'*'表示了前一次字符0次。
        return isMatch(s, p+2);
    }
}
```

![Regular Expression Matching Accepted](https://raw.githubusercontent.com/778477/778477.github.io/89a4ae382fa08862778215d211fb5a0e766df6ba/img/Regular%20Expression%20Matching.png)
