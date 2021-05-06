---
layout: post
title: "Algorithm Line Sweep"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Line Sweep 的算法实现

### "850. Rectangle Area II"

```C++
int rectangleArea(vector<vector<int>>& rectangles) {
    if (rectangles.empty() || rectangles[0].empty()) return 0;
    const int N = rectangles.size(), OPEN = 0, CLOSE = 1;
    vector<vector<int>> events(N * 2, vector<int>(4));
    for (int i = 0; i < N; ++i) {
        events[2 * i] = {rectangles[i][1], OPEN, rectangles[i][0], rectangles[i][2]};
        events[2 * i + 1] = {rectangles[i][3], CLOSE, rectangles[i][0], rectangles[i][2]};
    }
    sort(events.begin(), events.end());
    vector<vector<int>> active;
    int cur_y = events[0][0];
    long ans = 0;
    for (auto &e : events) {
        int y = e[0], type = e[1], x1 = e[2], x2 = e[3];
        long query = 0;
        int cur = -1;
        for (auto &a : active) {
            cur = max(cur, a[0]);
            query += max(a[1] - cur, 0);
            cur = max(cur, a[1]);
        }
        ans += query * (y - cur_y);
        if (type == OPEN) {
            active.push_back({x1, x2});
            sort(active.begin(), active.end());
        } else {
            for (int i = 0; i < active.size(); ++i) {
                if (active[i][0] == x1 && active[i][1] == x2) {
                    active.erase(active.begin() + i);
                    break;
                }
            }
        }
        cur_y = y;
    }
    ans %= 1000000007;
    return ans;
}
```
