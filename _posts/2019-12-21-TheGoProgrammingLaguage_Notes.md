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

### Package main

Package main is special. It defines a standalone executable program, not a library. Within package main the function main is also special—it’s where execution of the program begins.

- PS: but the folder name may not named 'main'.

### semicolons in line ends

In effect, new lines following certain tokens are converted into semicolons, so where newlines are placed matters to proper parsing of Go code.

### map

A map is a reference to the datastructure created by make. When a map is passed to a function, the function receives a copy of the reference, so any changes the called function makes to the underlying datastructure will be visible through the caller’s map reference too.

### switch

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

### Short Variable Declarations

One subtle but importantpoint: a short variable declaration does not necessarily declare all the variables on its left-hand side. If some of them were already declared in the same lexical block, then the short variable declaration acts like anassignment to those variables.

### new

Since new is a predeclared function, not a keyword, it’s possible to redefine the name for something else within a function.

### package

Packages in Go serve the same purposes as libraries or modules in other languages, supporting modularity, encapsulation, separate compilation, and reuse.

Only one file in each package should have a package doc comment. Extensive doc comments are of ten placed in a file of their own, conventionally called doc.go.

### implicit lexical blocks

```Go
if x := f(); x == 0 {
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```

The second if statement is nested within the first, so variables declared within the first statement’s initializer are visible within the second. Similar rules apply to each case of a switch statement: there is a block for the condition and a block for each case body.

At the package level, the order in which declarations appear has no effect on their scope, so a declaration may refer to itself or to another that follows it, letting us declare recursive or mutually recursive types and functions. The compiler will report an error if a constant or variable declaration refers to itself, however.

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
