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
