---
layout: post
title: "Algorithm TrieTree"
date: 2020-12-25 22:00:00 +0800
tags: Algorithm Leetcode
---

记录 TrieTree(Prefix Tree) 的算法实现

### "208. Implement Trie (Prefix Tree)" Golang

```Go
type Trie struct {
    root *node
}
type node struct {
    isEnd bool
    child [26]*node
}

func Constructor() Trie {
    return Trie{
        root: &node{
            isEnd: true,
        },
    }
}

func (this *Trie) Insert(word string)  {
    cur := this.root
    for _, c := range word {
        if cur.child[c-'a'] == nil {
            cur.child[c-'a'] = new(node)
        }
        cur = cur.child[c-'a']
    }
    cur.isEnd = true
}

func (this *Trie) Search(word string) bool {
    cur := this.root
    for _, c := range word {
        if cur.child[c-'a'] == nil {
            return false
        }
        cur = cur.child[c-'a']
    }
    return cur.isEnd
}

func (this *Trie) StartsWith(prefix string) bool {
    cur := this.root
    for _, c := range prefix {
        if cur.child[c-'a'] == nil {
            return false
        }
        cur = cur.child[c-'a']
    }
    return true
}
```

### "212. Word Search II"

```Go
type Node struct {
    child [26]*Node
    isWord bool
}

var root *Node
func add(s string) {
    cur := root
    for _, c := range s {
        idx := c - 'a'
        if cur.child[idx] == nil {
            cur.child[idx] = new(Node)
        }
        cur = cur.child[idx]
    }
    cur.isWord = true
}

func findWords(board [][]byte, words []string) (ret []string) {
    root = new(Node)
    for _, s := range words {
        add(s)
    }

    m, n := len(board), len(board[0])
    visited := make([][]bool, m)
    for i := range visited {
        visited[i] = make([]bool, n)
    }
    dirs := [4][2]int{ {-1,0},{1,0},{0,-1},{0,1} }
    var pre []byte
    var dfs func(int, int, *Node)
    dfs = func(x int, y int, node *Node) {
        visited[x][y] = true
        pre = append(pre, board[x][y])
        defer func(){
            visited[x][y] = false
            pre = pre[:len(pre)-1]
        }()

        if node.isWord {
            ret = append(ret, string(pre))
            node.isWord = false
        }

        for _, d := range dirs {
            x1, y1 := x+d[0], y+d[1]
            if 0 <= x1 && x1 < m && 0 <= y1 && y1 < n && !visited[x1][y1] &&
                node.child[board[x1][y1]-'a'] != nil {
                dfs(x1, y1, node.child[board[x1][y1]-'a'])
            }
        }
    }

    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if root.child[board[i][j] - 'a'] != nil {
                dfs(i, j, root.child[board[i][j] - 'a'])
            }
        }
    }

    return ret
}
```

### "616. Add Bold Tag in String"

```Go
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
            if len(bolds) > 0 && i <= bolds[len(bolds)-1][1]+1 {
                bolds[len(bolds)-1][1] = max(bolds[len(bolds)-1][1], end)
            } else {
                bolds = append(bolds, &[2]int{i, end})
            }
        }
    }

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

### "1707. Maximum XOR With an Element From Array"

```Go
const L = 30

type trie struct {
    children [2]*trie
}

func (t *trie) insert(val int) {
    node := t
    for i := L - 1; i >= 0; i-- {
        bit := val >> i & 1
        if node.children[bit] == nil {
            node.children[bit] = &trie{}
        }
        node = node.children[bit]
    }
}

func (t *trie) getMaxXor(val int) (ans int) {
    node := t
    for i := L - 1; i >= 0; i-- {
        bit := val >> i & 1
        if node.children[bit^1] != nil {
            ans |= 1 << i
            bit ^= 1
        }
        node = node.children[bit]
    }
    return
}

func maximizeXor(nums []int, queries [][]int) []int {
    sort.Ints(nums)
    for i := range queries {
        queries[i] = append(queries[i], i)
    }
    sort.Slice(queries, func(i, j int) bool { return queries[i][1] < queries[j][1] })

    ans := make([]int, len(queries))
    t := &trie{}
    idx, n := 0, len(nums)
    for _, q := range queries {
        x, m, qid := q[0], q[1], q[2]
        for idx < n && nums[idx] <= m {
            t.insert(nums[idx])
            idx++
        }
        if idx == 0 { // 字典树为空
            ans[qid] = -1
        } else {
            ans[qid] = t.getMaxXor(x)
        }
    }
    return ans
}
```
