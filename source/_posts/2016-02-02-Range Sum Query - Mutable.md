---
layout: post
title: 'Range Sum Query Mutable'
date: '2016-02-02'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---

[Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/)


给定一个整形数组。会有两个操作

操作1：查询区间(i,j)的总和

操作2：更新元素(i,val) 下标为i的元素 值更新为val


提示：

 * 数组元素的修改只有通过 update 方法
 * update 和 sumRange 方法调用是不确定次数和顺序的（distributed evenly 分布式）
 

这个是一个经典的问题。类似 RMQ(Range Minimum/Maximum Query)，区间最值查询。不过我们注意到有操作2，就表示说数据是可变的，离线算法就不考虑了。

这类题目有一个现成的数据结构可以解决：[SegmentTree 线段树](https://en.wikipedia.org/wiki/Segment_tree)

由于线段树是平衡二叉树，所以深度是logn。就是算查询的区间和的时间复杂度是 O(logn)的。

![segment-tree1](http://d1gjlxt8vb0knt.cloudfront.net//wp-content/uploads/segment-tree1.png)

由上图可知，叶子节点均为数组元素。我们在做ACM题的常常会提到一个线段树常数，其实是指线段树的空间复杂度。按照数组n来推算线段树的空间复杂度。

时间复杂度大体分为： 

* 初始化 建树O(n) 
* 区间查询和 O(logn) 
* 更新单值O(logn) 

时间复杂度理想是线性的，不因为n的变化产生剧烈抖动。空间复杂度是O(nlogn)

> A segment tree T on a set I of n intervals uses O(nlogn) storage.

> Proof:
Lemma: Any interval [x, x′] of I is stored in the canonical set for at most two nodes at the    same depth.

> Proof: Let v1, v2, v3 be the three nodes at the same depth, numbered from left to right; and let p(v) be the parent node of any given node v. Suppose [x, x′] is stored at v1 and v3. This means that [x, x′] spans the whole interval from the left endpoint of Int(v1) to the right endpoint of Int(v3). Note that all segments at a particular level are non-overlapping and ordered from left to right: this is true by construction for the level containing the leaves, and the property is not lost when moving from any level to the one above it by combining pairs of adjacent segments. Now either p(v2) = p(v1), or the former is to the right of the latter (edges in the tree do not cross). In the first case, Int(p(v2))'s leftmost point is the same as Int(v1)'s leftmost point; in the second case, Int(p(v2))'s leftmost point is to the right of Int(p(v1))'s rightmost point, and therefore also to the right of Int(v1)'s rightmost point. In both cases, Int(p(v2)) begins at or to the right of Int(v1)'s leftmost point. Similar reasoning shows that Int(p(v2)) ends at or to the left of Int(v3)'s rightmost point. Int(p(v2)) must therefore be contained in [x, x′]; hence, [x, x′] will not be stored at v2.
The set I has at most 4n + 1 elementary intervals. Because T is a binary balanced tree with at most 4n + 1 leaves, its height is O(logn). Since any interval is stored at most twice at a given depth of the tree, that the total amount of storage is O(nlogn).




```
typedef unsigned long lusigned;

struct SegmenTreeNode{
    int val;
    lusigned star,end;
    SegmenTreeNode *left,*right;
    SegmenTreeNode(lusigned _star,lusigned _end,int _val){
        val = _val;
        star = _star, end = _end;
        left = right = NULL;
    }
};

class NumArray {
public:
    NumArray(vector<int> &nums) {
        numbers = nums;
        if(nums.size() >= 1){
            rootNode = buildSegmentTree(0, 0, nums.size()-1, nums);
        }
    }
    
    void update(int i, int val){
        updateNodeVal(rootNode, i, val);
    }
    
    int sumRange(int i, int j) {
        if(j < i) return 0;
        return querySumValInSegmentTree(rootNode, 0, numbers.size()-1, i, j);
    }
    
private:
    vector<int> numbers;
    SegmenTreeNode *rootNode;
    SegmenTreeNode *buildSegmentTree(int root,lusigned star,lusigned end,const vector<int> nums){
        
        SegmenTreeNode *rootNode = new SegmenTreeNode(star, end, 0);
        
        if(star == end){
            rootNode->val = nums[star];
        }
        else{
            lusigned mid = (star + end)/2;
            rootNode->left = buildSegmentTree(root*2+1, star, mid, nums);
            rootNode->right =  buildSegmentTree(root*2+2, mid+1, end, nums);
            
            rootNode->val = rootNode->left->val + rootNode->right->val;
        }
        
        return rootNode;
    }
    
    int updateNodeVal(SegmenTreeNode *root, int idx,int value){
        if(root->star == root->end && root->star == idx)  return (root->val = value);
        else if(root->star == root->end) return root->val;
        
        lusigned mid = (root->star + root->end)>>1;
        if(idx <= mid){
            root->left->val = updateNodeVal(root->left, idx, value);
        }
        else{
            root->right->val = updateNodeVal(root->right, idx, value);
        }
        return (root->val = root->left->val + root->right->val);
    }
    
    int querySumValInSegmentTree(SegmenTreeNode *root, lusigned star,lusigned end,lusigned qSidx,lusigned qEidx){
        
        if(qSidx <= star && qEidx >= end) return root->val;
        
        if(qEidx < star || qSidx > end) return 0;
        
        lusigned mid = (star + end) / 2;
        
        int leftSumVal = querySumValInSegmentTree(root->left, star, mid, qSidx, qEidx);
        int rightSumVal = querySumValInSegmentTree(root->right, mid+1, end, qSidx, qEidx);
        
        return leftSumVal + rightSumVal;
    }
};

```

![accepted](https://raw.githubusercontent.com/778477/778477.github.io/master/img/Range%20Sum%20Query%20Accepted.png)

不太明白为什么 leetCode Judge 中 C/C++ 的效率最低，Java最高。这个明显是不合理的。。

最后 关于 线段树的更新是有优化空间的。看一看这组操作

 1. 操作1
 2. 操作2 
 3. 操作2
 4. 操作2
 5. 操作2
 6. 操作2
 7. 操作2

其实 后面的操作2 如果没有接着操作1的话，其实我们没有必要实时更新单点值。这里就有一个延迟更新的优化点，也是线段树的精髓。 我之后有空补充:)