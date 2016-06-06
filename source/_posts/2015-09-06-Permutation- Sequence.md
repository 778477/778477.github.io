---
layout: post
title: 'Permutation Sequence'
date: '2015-09-06'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---


[Permutation Sequence](https://leetcode.com/problems/permutation-sequence/)

题目大意：一个[1,2,3,...,n]n个数字的集合有 n！个唯一的排列组合。对这些排列组合排序我们能得到一个有序的数组，现在给你一个n和k求第k个顺序的排列。

ie，for n = 3，k = 4

1. "123"
2. "132"
3. "213"
4. "231"
5. "312"
6. "321"

应该返回第4个排列情况"231"。

**Note** 给定的n在[1,9]之间

解题思路：一开始我使用C++ STL自带的next_permutation API,以为能水过去。结果超时了(Time Limit Exceeded) 看来不能偷懒。

我们来重新看一下题目，n个数字有n！种排列情况，求第k个排列组合(排序之后)。大致能理解为 给n个数字，组成第k小的数。 这么想的话，我们可以思考一下 如果有n个数，求组成第k小的数，我们第一位数字应该如何选择呢？

还是这个(n = 3, k = 4)的case，我们手上有1，2，3这三个数字，我们要组成第4小的数，我们第一个数字应该选2，为什么选2需要解释一下：

我们要组成第4小的数，手上有3个数，有3！= 6种排列情况(组成情况)。要先选一个最高位数， 我们看一下 剩下有2个数，有2种组成情况。显然最高位选1的话，最多也是一个第2小的数。选3的话，怎么组成也是第5小的数起跳。选2的话，组成的数区间是第4-第5。所以我们选择2作为 第4小的数的最高位。

由此规律为：最高位数字是 (k-1)/(n-1)！
同理 选择次最高位思路是一样的

```
class Solution {
public:
    string getPermutation(int n, int k) {
        
        string str = "";
        if(k == factorial(n)){
            for(int i=n;i>0;i--) str+=('0'+i);
            return str;
        }
        
        vector<int> numbers;
        for(int i=1;i<=n;i++) numbers.push_back(i);
        for(int i=1;i<n;i++){
            int num = factorial(n-i);
            int index = (k-1)/num;
            str += ('0' + numbers[index]);
            numbers.erase(numbers.begin() + index);
            k = k-index*num;
        }
        str+= ('0' + numbers[0]);
        return str;
    }
private:
    int factorial(int n){
        int ans = n;
        while(--n) ans*=n;
        return ans;
    }
};
```

![accepted](https://raw.githubusercontent.com/778477/778477.github.io/master/img/Permutation%20Sequence.png)


> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。