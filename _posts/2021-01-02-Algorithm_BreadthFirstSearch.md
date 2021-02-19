---
layout: post
title: "Algorithm BFS"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 BFS 的算法实现

### "1036. Escape a Large Maze" Golang

```Golang
func isEscapePossible(blocked [][]int, source []int, target []int) bool {
    blockMap := make(map[Point]struct{})
    for _, b := range blocked {
        blockMap[Point{b[0], b[1]}] = struct{}{}
    }
    return bfs(Point{source[0], source[1]}, Point{target[0], target[1]}, blockMap) && bfs(Point{target[0], target[1]}, Point{source[0], source[1]}, blockMap)
}

type Point [2]int
func bfs(source Point, target Point, block map[Point]struct{}) (ret bool) {
    // 因为 jekyll 格式问题暂时注掉
    // dir := [][]int{{-1, 0}, {1, 0}, {0, -1}, {0, 1}}
    visited := map[Point]struct{}{source:struct{}{}}
    l := list.New()
    l.PushBack(source)
    steps := 0

    for l.Len() > 0 && steps < 20000 {
        cur := l.Remove(l.Front()).(Point)
        steps++
        if cur == target {
            return true
        }

        for i := 0; i < 4; i++ {
            next := Point{cur[0]+dir[i][0], cur[1]+dir[i][1]}
            if next[0] < 0 || next[1] < 0 || next[0] >= 1e6 || next[1] >= 1e6 {
                continue
            } else if _, exists := block[next]; exists {
                continue
            } else if _, exists := visited[next]; exists {
                continue
            }
            visited[next] = struct{}{}
            l.PushBack(next)
        }
    }
    if steps >= 20000 {
        return true
    }
    return false
}
```
