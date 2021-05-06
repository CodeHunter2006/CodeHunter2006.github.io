---
layout: post
title: "Algorithm Minimax"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Minimax 的算法实现

### "913. Cat and Mouse"

```C++
int calc(int curM, int curC, int isM, vector<vector<vector<int>>>& dp, vector<vector<int>>& graph) {
    if(curC==0 || curM==0)
        return 1;
    if(curC==curM)
        return 2;
    if(dp[curM][curC][isM]!=-1) {
        return dp[curM][curC][isM];
    }
    dp[curM][curC][isM]=0;
    bool canDraw=false;
    if(isM) {
        for(int i=0;i<graph[curM].size();i++) {     //If any neigbor is 0 return 1
            if(graph[curM][i]==0) {
                dp[curM][curC][isM]=1;
                return 1;
            }
        }
        for(int i=0;i<graph[curM].size();i++) {
            int temp=calc(graph[curM][i], curC, 0, dp, graph);
            if(temp==1) {
                dp[curM][curC][isM]=1;
                return 1;
            }
            else if(temp==0) {
                canDraw=true;
            }
        }
        if(!canDraw)
            dp[curM][curC][isM]=2;
    } else {
        for(int i=0;i<graph[curC].size();i++) {     //If any neighbor is current pos of mouse return 2
            if(graph[curC][i]==curM) {
                dp[curM][curC][isM]=2;
                return 2;
            }
        }
        for(int i=0;i<graph[curC].size();i++) {
            int temp=calc(curM, graph[curC][i], 1, dp, graph);
            if(temp==2) {
                dp[curM][curC][isM]=2;
                return 2;
            }
            else if(temp==0) {
                canDraw=true;
            }
        }
        if(!canDraw)
            dp[curM][curC][isM]=1;
    }
    return dp[curM][curC][isM];
}

int catMouseGame(vector<vector<int>>& graph) {
    int n=graph.size();
    vector<vector<vector<int>>> dp(n, vector<vector<int>>(n, vector<int>(2, -1)));
    return calc(1, 2, 1, dp, graph);
}
```
