---
layout: post
title: 'Minimum Window Substring'
date: '2015-11-27'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---

[Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)

题目大意：
给定一个字符串S 和 另一个字符串T，在S中找到一个最短长度的子串包含T中所有的字母。 时间期望是O(N)
如：
```S = "ADOBECODEBANC"```
```T = "ABC"```
最短子串是 ```"BANC"```

提示：
 
* 如果没有找到子串，则返回```""```空串
* 如果有多个满足条件的字串，题目保证最短子串只有唯一的一个

思路：
维护一个计数数组CountTable统计字符串T所出现的所有字母次数，再开始遍历字符串S，同时开始统计T所出现的字母在S所出现的次数。 如果遍历过程中统计的字母次数和T的长度相同，说明我们遍历的得到的这个子串就是一个 Window Substring，满足条件！记录这个subString，开始缩小这个Window Substring，直到条件不满足 继续遍历寻找下一个 Window Substring。


```
class Solution {
public:
    string minWindow(string s, string t) {
        string ans = "";
        if(t.length() > s.length()) return ans;
        
        vector<int> countTable(256,0);
        
        for_each(t.begin(), t.end(), [&](const char ch){
            countTable[ch]++;
        });
        
        vector<int> foundTable(256,0);
        queue<unsigned long> Q;
        unsigned long maxLen(s.length()),count(0);
        for(size_t i = 0;i<s.length();i++){
            if(countTable[s[i]] > 0){
                foundTable[s[i]] ++;
                Q.push(i);
                if(foundTable[s[i]] <= countTable[s[i]]) count++;
                if(count == t.length()){
                    unsigned long idx;
                    do {
                        idx = Q.front();
                        Q.pop();
                        foundTable[s[idx]]--;
                    } while (foundTable[s[idx]] >= countTable[s[idx]]);
                    if(i-idx < maxLen) ans = s.substr(idx,i-idx+1),maxLen = ans.length();
                    count --;
                }
            }
        }
        return ans;
    }
};
```

![Accepted](https://raw.githubusercontent.com/778477/778477.github.io/73252cd0a2f49ca59bc5caee15fb1261bfae7abb/img/Minimum%20Window%20Substring.png)