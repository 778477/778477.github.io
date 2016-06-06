---
layout: post
title: 'Minimum Size Subarray Sum'
date: '2016-04-08'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---


# Question

[Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/)

> Given an array of n positive integers and a positive integer s, find the minimal length of a subarray of which the sum ≥ s. If there isn't one, return 0 instead.

> For example, given the array [2,3,1,2,4,3] and s = 7,
the subarray [4,3] has the minimal length under the problem constraint.


题目大意： 给定一个正数数组 nums和一个正数 s，找出最短长度的连续子数组 使其和 大于等于 s。如果没有的话，返回 0.


# Solution

## 暴力枚举 <span style="color:green;">Accepted</span>  

``` 
class Solution {
public:
    int minSubArrayLen(int s, vector<int>& nums) {
        size_t n = nums.size();
        int sum = 0;
        int ans = INT_MAX;
        for(int i=0;i<n;i++){
            sum = nums[i];
            if(sum >= s) return 1;
            for(int j=i+1;j<n;j++){
                sum += nums[j];
                if(sum >= s){
                    ans = min(ans,j-i+1);
                    break;
                }
            }
        }
        
        return ans==INT_MAX ? 0 : ans;
    }
};
```


![](https://raw.githubusercontent.com/778477/778477.github.io/master/img/Minimum%20Size%20Subarray%20Sum.png)


## 暴力枚举 + flag优化 <span style="color:green;">Accepted</span>

```
class Solution {
public:
    int minSubArrayLen(int s, vector<int>& nums) {
        size_t n = nums.size();
        int sum = 0;
        int ans = INT_MAX;
        
        bool flag;
        for(int i=0;i<n;i++){
            sum = nums[i];
            if(sum >= s) return 1;
            flag = false;
            for(int j=i+1;j<n;j++){
                sum += nums[j];
                if(sum >= s){
                    ans = min(ans,j-i+1);
                    flag = true;
                    break;
                }
            }
            
            if(!flag) break;
        }
        
        return ans==INT_MAX ? 0 : ans;
    }
};
```

![](https://raw.githubusercontent.com/778477/778477.github.io/master/img/Minimum%20Size%20Subarray%20Sum%202.png)

## 枚举小窗口 <span style="color:green;">Accepted</span>
```
class Solution {
public:
    int minSubArrayLen(int s, vector<int>& nums) {
        size_t n = nums.size();
        int sum = 0;
        int ans = INT_MAX;
        int p = 0,q = 0;
        
        while(q < n){
            sum += nums[q++];
            
            while(sum >= s){
                ans = min(ans,q-p);
                sum -= nums[p++];
            }
            
        }
        
        return ans==INT_MAX ? 0 : ans;
    }
};
```

![](https://raw.githubusercontent.com/778477/778477.github.io/master/img/Minimum%20Size%20Subarray%20Sum%203.png)