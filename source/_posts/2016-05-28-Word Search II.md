---
layout: post
title: 'Word Search II'
date: '2016-05-28'
header-img: "img/home-bg.jpg"
tags:
     - LeetCode
     
author: '778477'
---

# Word Search II
---


[Word Search](https://leetcode.com/problems/word-search-ii/)

> Given a 2D board and a list of words from the dictionary, find all words in the board.

> Each word must be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once in a word.

```
For example,
Given words = ["oath","pea","eat","rain"] and board =

[
  ['o','a','a','n'],
  ['e','t','a','e'],
  ['i','h','k','r'],
  ['i','f','l','v']
]
Return ["eat","oath"].
```

题目大意：给定一个二维面板和一组字符串，找出二维面板上存在的字符串。
**必须使用面板上的字母且水平连续或垂直连续拼接单词。**


## 思路1： 深度优先搜索(DFS)  <span style="color:red">Time Limit Exceeded</span>

先遍历每个单词，如果单词首字母面板上有存在则开始深度优先搜索单词剩余的字母。

来分析一下超时的Test Case：

```
["aaaa","aaaa","aaaa","aaaa","bcde","fghi","jklm","nopq","rstu","vwxy","zzzz"]


["aaaaaaaaaaaaaaaa","aaaaaaaaaaaaaaab","aaaaaaaaaaaaaaac","aaaaaaaaaaaaaaad","aaaaaaaaaaaaaaae","aaaaaaaaaaaaaaaf","aaaaaaaaaaaaaaag","aaaaaaaaaaaaaaah","aaaaaaaaaaaaaaai","aaaaaaaaaaaaaaaj","aaaaaaaaaaaaaaak","aaaaaaaaaaaaaaal","aaaaaaaaaaaaaaam","aaaaaaaaaaaaaaan","aaaaaaaaaaaaaaao","aaaaaaaaaaaaaaap","aaaaaaaaaaaaaaaq","aaaaaaaaaaaaaaar","aaaaaaaaaaaaaaas","aaaaaaaaaaaaaaat","aaaaaaaaaaaaaaau","aaaaaaaaaaaaaaav","aaaaaaaaaaaaaaaw","aaaaaaaaaaaaaaax","aaaaaaaaaaaaaaay","aaaaaaaaaaaaaaaz","aaaaaaaaaaaaaaaa","aaaaaaaaaaaaaaab","aaaaaaaaaaaaaaac","aaaaaaaaaaaaaaad","aaaaaaaaaaaaaaae","aaaaaaaaaaaaaaaf","aaaaaaaaaaaaaaag","aaaaaaaaaaaaaaah","aaaaaaaaaaaaaaai","aaaaaaaaaaaaaaaj","aaaaaaaaaaaaaaak","aaaaaaaaaaaaaaal","aaaaaaaaaaaaaaam","aaaaaaaaaaaaaaan","aaaaaaaaaaaaaaao","aaaaaaaaaaaaaaap","aaaaaaaaaaaaaaaq","aaaaaaaaaaaaaaar","aaaaaaaaaaaaaaas","aaaaaaaaaaaaaaat","aaaaaaaaaaaaaaau","aaaaaaaaaaaaaaav","aaaaaaaaaaaaaaaw","aaaaaaaaaaaaaaax","aaaaaaaaaaaaaaay","aaaaaaaaaaaaaaaz","aaaaaaaaaaaaaaba","aaaaaaaaaaaaaabb","aaaaaaaaaaaaaabc"]
```

发现这个Test Case很有意思，有大量重复相同的前缀`aaaaaaaaaaaaaaaa`。使用DFS遍历图的话，如果没有高效的剪枝策略。光是这个前缀的深度优先搜索效率就很低下了。


## 思路2 ： 字典树(Trie) + 深度优先搜索(DFS) <span style="color:green">Accepted</span>

建立Trie字典树，能有效的避免大量重复前缀的搜索。 

LeetCodeOJ 上有[关于Trie建树的题目](https://leetcode.com/problems/implement-trie-prefix-tree/)

```

const int TrieChildNodeMax(26);
class Trie{
public:
    class TrieNode{
    public:
        TrieNode(){
            isWord = false;
            word = "";
            memset(childNode, NULL, sizeof(TrieNode *) * TrieChildNodeMax);
        }
        TrieNode *childNode[TrieChildNodeMax];
        string word;
        bool isWord;
    };
    TrieNode *root;
    
    Trie(){
        root = new TrieNode();
    }
    
    
    void insert(const string word,TrieNode* node,int idx = 0){
        if(idx == word.length()) return ;
        int k = word[idx] - 'a';
        if(node->childNode[k] == NULL){
            node->childNode[k] = new TrieNode();
        }
        if(idx == word.length() - 1){
            node->childNode[k]->isWord = true;
            node->childNode[k]->word = word;
        }
        else{
            insert(word, node->childNode[k],idx+1);
        }
    }
    
};

class Solution {
public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        vector<string> ans;
        
        
        for_each(words.begin(), words.end(), [&](const string word){
            trie.insert(word, trie.root);
        });
        
        
        n = (int)board.size();
        m = 0;
        if(n > 0) m = (int)board[0].size();
        
        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){
                memset(vis, false, sizeof(bool)*1024*1024);
                if(boardHasWord(board,i,j,trie.root,ans)){
                    cout<<"Yes,find word in borad"<<endl;
                }
            }
        }
        
        sort(ans.begin(), ans.end());
        
        return ans;
    }
private:
    bool judge(const int x,const int y){
        return (x>-1&&x<n&&y>-1&&y<m&&!vis[x][y]);
    }
    bool boardHasWord(const vector<vector<char>> board,int x,int y,Trie::TrieNode* root,vector<string>& ans){
        int k = board[x][y] - 'a';
        vis[x][y] = true;
        if(root->childNode[k]){
            if(root->childNode[k]->isWord){
                root->childNode[k]->isWord = false;
                ans.push_back(root->childNode[k]->word);
            }
            
            
            for(int i=0;i<4;i++){
                int xx = x + dir[i][0];
                int yy = y + dir[i][1];
                
                if(judge(xx, yy)){
                    vis[xx][yy] = true;
                    if(boardHasWord(board, xx, yy, root->childNode[k], ans)) return true;
                    vis[xx][yy] = false;
                }
            }
        }
        
        return false;
    }
    int n,m;
    const int dir[4][2] = { {1,0},{0,1},{-1,0},{0,-1} };
    bool vis[1024][1024];
    Trie trie;
};



int main(){
    
    freopen(INPUT,"r",stdin);
    
    string buf;
    vector<vector<char>> board;
    while(cin>>buf){
        vector<char> chars;
        chars.assign(buf.begin(), buf.end());
        board.push_back(chars);
    }
    
    Solution solve;
    vector<string> words({"aaaaaaaaaaaaaaaa","aaaaaaaaaaaaaaab","aaaaaaaaaaaaaaac","aaaaaaaaaaaaaaad"});
    vector<string> ans = solve.findWords(board,words);
    
    
    for_each(ans.begin(), ans.end(), [](const string str){
        cout<<str<<endl;
    });
    
    
    return 0;
}



/*	Wrong Answer */
/*
["ab","aa"]
["aba","baa","bab","aaab","aaa","aaaa","aaba"]
*/



/* Time Limit Exceeded */
/*
 
 ["aaaa","aaaa","aaaa","aaaa","bcde","fghi","jklm","nopq","rstu","vwxy","zzzz"]
 ["aaaaaaaaaaaaaaaa","aaaaaaaaaaaaaaab","aaaaaaaaaaaaaaac","aaaaaaaaaaaaaaad"]
*/

```


![Accepted](https://raw.githubusercontent.com/778477/778477.github.io/63215efbb2bc92122d932f3e33a6423b79a4d49f/img/Word_Search_II.png)