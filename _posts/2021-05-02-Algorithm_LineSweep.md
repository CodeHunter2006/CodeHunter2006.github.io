---
layout: post
title: "Algorithm Line Sweep"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Line Sweep 的算法实现

### "218. The Skyline Problem"

```Go
func getSkyline(buildings [][]int) (ret [][]int) {
    arr := make([]*[2]int, 0, len(buildings)*2)  // (坐标, 高度)
    for _, b := range buildings {
        // 用负数表示开始高度，以便排序时开始比结束更靠前
        arr = append(arr, &[2]int{b[0],-b[2]}, &[2]int{b[1],b[2]})
    }
    sort.Slice(arr, func(a, b int) bool {
        if arr[a][0] == arr[b][0] {
            return arr[a][1] < arr[b][1]    // 再按高度
        }
        return arr[a][0] < arr[b][0]    // 先按坐标
    })
    h := new(myHeap)

    maxHeight := 0
    for _, cur := range arr {
        if cur[1] < 0 {
            if -cur[1] > maxHeight {
                maxHeight = -cur[1]
                ret = append(ret, []int{cur[0], maxHeight})
            }
            heap.Push(h, &[2]int{cur[0], -cur[1]})
        } else {
            for i := 0; i < h.Len(); i++ {
                if (*h)[i][1] == cur[1] {
                    heap.Remove(h, i)
                    break
                }
            }
            if h.Len() > 0 {
                if (*h)[0][1] == maxHeight {
                    continue
                } else if (*h)[0][1] < maxHeight {
                    maxHeight = (*h)[0][1]
                }
            } else {
                maxHeight = 0
            }
            ret = append(ret, []int{cur[0], maxHeight})
        }
    }

    return ret
}

type myHeap []*[2]int   // (坐标, 高度)
func (p *myHeap) Len() int { return len(*p) }
func (p *myHeap) Less(a, b int) bool {
    return (*p)[a][1] > (*p)[b][1]
}
func (p *myHeap) Swap(a, b int) { (*p)[a], (*p)[b] = (*p)[b], (*p)[a] }
func (p *myHeap) Push(x interface{}) { (*p) = append((*p), x.(*[2]int)) }
func (p *myHeap) Pop() interface{} {
    tmp := (*p)[len(*p)-1]
    (*p) = (*p)[:len(*p)-1]
    return tmp
}
```

### "616. Add Bold Tag in String"

```Go
// TrieTree + LineSweep
type TreeNode struct {
    Children map[byte]*TreeNode
    IsEnd bool
}

var gRoot *TreeNode

func addWord(s string) {
    cur := gRoot
    pos := 0
    for pos < len(s) {
        if _, ok := cur.Children[s[pos]]; !ok {
            cur.Children[s[pos]] = &TreeNode{ Children: make(map[byte]*TreeNode) }
        }
        cur = cur.Children[s[pos]]
        pos++
    }
    cur.IsEnd = true
}

// findEnd 找到尽量靠后的单词结束位置
func findEnd(s string, pos int) (ret int) {
    ret = -1
    cur := gRoot
    for pos < len(s) {
        if c, ok := cur.Children[s[pos]]; ok {
            cur = c
            if cur.IsEnd {
                ret = pos
            }
            pos++
        } else {
            return ret
        }
    }
    return ret
}

func addBoldTag(s string, words []string) string {
    gRoot = &TreeNode{ Children: make(map[byte]*TreeNode) }
    for _, w := range words {
        addWord(w)
    }

    // 查找区间
    var bolds []*[2]int
    for i := range s {
        if end := findEnd(s, i); end != -1 {
            bolds = append(bolds, &[2]int{i, end})
        }
    }
    if len(bolds) == 0 {
        return s
    }

    // 合并区间
    lenIdx := 0
    for i := 1; i < len(bolds); i++ {
        if bolds[i][0] <= bolds[lenIdx][1]+1 {
            bolds[lenIdx][1] = max(bolds[lenIdx][1], bolds[i][1])
        } else {
            lenIdx++
            bolds[lenIdx] = bolds[i]
        }
    }
    bolds = bolds[:lenIdx+1]

    // 组装结果
    var builder strings.Builder
    lastPos := 0
    for _, b := range bolds {
        if lastPos < b[0] {
            builder.WriteString(s[lastPos:b[0]])
        }
        lastPos = b[1]+1
        builder.WriteString("<b>")
        builder.WriteString(s[b[0]:b[1]+1])
        builder.WriteString("</b>")
    }
    builder.WriteString(s[lastPos:])
    return builder.String()
}
```

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

### "986. Interval List Intersections"

```Go
// version 1
func intervalIntersection(firstList [][]int, secondList [][]int) (ret [][]int) {
    type changePoint struct {
        pos int
        action bool // false end; true start
    }
    allActions := make([]*changePoint, 0, len(firstList)*2 + len(secondList)*2)
    for _, v := range firstList {
        allActions = append(allActions, &changePoint{v[0], true}, &changePoint{v[1], false})
    }
    for _, v := range secondList {
        allActions = append(allActions, &changePoint{v[0], true}, &changePoint{v[1], false})
    }

    sort.Slice(allActions, func(a, b int) bool {
        if allActions[a].pos == allActions[b].pos {
            return allActions[a].action
        }
        return allActions[a].pos < allActions[b].pos
    })

    for i, count, lastStart := 0, 0, 0; i < len(allActions); i++ {
        if allActions[i].action {
            count++
            lastStart = allActions[i].pos
        } else {
            if count > 1 {
                ret = append(ret, []int{lastStart, allActions[i].pos})
            }
            count--
        }
    }
    return ret
}
```

```Go
// version 2
func intervalIntersection(firstList [][]int, secondList [][]int) (ret [][]int) {
    for i, j := 0, 0; i < len(firstList) && j < len(secondList); {
        low, high := max(firstList[i][0], secondList[j][0]), min(firstList[i][1], secondList[j][1])
        if low <= high {
            ret = append(ret, []int{low, high})
        }
        if firstList[i][1] < secondList[j][1] {
            i++
        } else {
            j++
        }
    }
    return ret
}
```

### "1288. Remove Covered Intervals"

```Go
func removeCoveredIntervals(intervals [][]int) (count int) {
    sort.Slice(intervals, func(a, b int) bool {
        if intervals[a][0] == intervals[b][0] {
            return intervals[a][1] > intervals[b][1]
        }
        return intervals[a][0] < intervals[b][0]
    })
    for i, end := 0, math.MinInt32; i < len(intervals); i++ {
        if intervals[i][1] > end {
            count++
            end = intervals[i][1]
        }
    }
    return count
}
```
