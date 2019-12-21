---
layout: post
title: "《The Go Programming Laguage》Notes"
date: 2019-12-21 12:00:00 +0800
tags: Go
---

"The palest ink is better than the best memory."

# Preface

As Rob Pike put it, "complexity is multiplicative": fixing a problem by making one part of the system more complex slowly but surely adds complexity to other parts.

# 1. Turorial

Package main is special. It defines a standalone executable program, not a library. Within package main the function main is also special—it’s where execution of the program begins.

- PS: but the folder name may not named 'main'.

In effect, new lines following certain tokens are converted into semicolons, so where newlines are placed matters to proper parsing of Go code.

A map is a reference to the datastructure created by make. When a map is passed to a function, the function receives a copy of the reference, so any changes the called function makes to the underlying datastructure will be visible through the caller’s map reference too.

- Switch case expression are evaluated left-to-right and top-to-bottom
- default can be anywhere and only appear onece. if no case can be matched default will execute.

```Go
func Signum(x int) int {
    switch {
        case x > 0:
            return +1
        default:
            return 0
        case x < 0:
            return -1
    }
}
```

This form is called a tagless switch; it’s equivalent to switch true.

# 2. Program Structure

# 3. Basic Data Types

# 4. Composite Types

# 5. Functions

# 6. Methods

# 7. Interfaces

# 8. Goroutines and Channels

# 9. Concurrency with Shared Variables

# 10. Packages and the Go Tool

# 11. Testing

# 12. Reflection

# 13. Low-Level Programming
