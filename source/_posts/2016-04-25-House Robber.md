---
layout: post
title: 'House Robber'
date: '2016-04-25'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---

# House Robber
- - -

[House Robber](https://leetcode.com/problems/house-robber/)

> You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

> Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.

题目大意：你是一个职业强盗，计划去洗劫一条街。这条街上每个店铺都有一定量的现金。但是如果两家相邻的店铺被同时打劫的话会惊动警察。给定一个非负数数组表示各个店铺的现金数，请制定一个合理的打劫顺序使今晚的洗劫行动收获最大。

思考： 
 
 1. **贪心策略**：只洗劫奇数店铺 或 偶数店铺。 但细想一下，该策略只适合店铺数少于4的情况。比如有4家店铺的话： Plans = {[1,3],[2,4],[1,4]}；贪心策略的一个反例。
 2. **暴力搜索**：深度搜索，枚举每个店铺的两种状态： 抢 或 不抢。抢依赖上一家不抢。类似01背包，背包没有容量限制。**可能会超时**
 3. 在想 思路2的时候，其实已经想到这是一个 动态规划的题了：）动规的时间复杂度比搜索要来的线性，稳定。动态方程式是 `dp[i] = max{dp[i-2]+nums[i],dp[i-1]} 2<=i<numsSize`
 
 
 
## 动态规划 <span style="color:green;">Accepted</span> 
 
```
int max(const int a,const int b){
    return a > b ? a : b;
}

int easyRob(int* nums, int numsSize){
    int odd = 0, eve = 0;
    
    for(int i=0;i<numsSize;i++){
        if(i%2) eve += nums[i];
        else odd += nums[i];
    }
    
    return max(odd,eve);
}

int rob(int* nums, int numsSize) {
    if(numsSize <= 3) return easyRob(nums,numsSize);
    
    int *dp = (int *)malloc(sizeof(int)*(numsSize));
    memset(dp, 0, sizeof(int)*(numsSize));
    dp[0] = nums[0];
    dp[1] = max(nums[1],nums[0]);
    dp[2] = max(dp[0]+nums[2],dp[1]);
    
    for(int i=3;i<numsSize;i++){
        dp[i] = max(dp[i-2]+nums[i],dp[i-1]);
    }
    
    int ans = dp[numsSize-1];
    free(dp);
    return ans;
}

```

![](https://raw.githubusercontent.com/778477/778477.github.io/master/img/house_robber.png)