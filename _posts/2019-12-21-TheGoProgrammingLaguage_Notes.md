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

### uintptr

there is an unsigned integer type uintptr, whose width is not specified but is sufficient to hold all the bits of a pointer value. The uintptr type is used only for lowlevel programming , such as at the boundary of a Go program with a C library or an operating system.

## `&^` bit clear(AND NOT)

The &^ operator is bit clear (AND NOT): in the expression z = x &^ y, each bit of z is 0 if the corresponding bit of y is 1; other wise it equals the corresponding bit of x.

## Left/Right shifts

Left shifts fill the vacated bits with zeros, as do right shifts of unsigned numbers, but right shifts of signed numbers fill the vacated bits with copies of the sign bit. For this reason, it is important to use unsigned arithmetic when you’re treating an integer as a bit pattern.

## implicit uint type panic

```Go
medals := []string{"gold", "silver", "bronze"}
for i := len(medals) - 1; i >= 0; i-- {
    fmt.Println(medals[i]) // "bronze", "silver", "gold"
}
```

The alternative would be calamitous. If len returned an unsigned number, then i too would be a uint, and the condition i >= 0 would always be true by definition. After the third iteration, in which i == 0, the i-- statement would cause i to become not −1, but the maximum uint value (for example, 2^64−1), and the evalu ation of medals[i] would fail at runtime, or panic , by attempting to access an element outside the bounds of the slice.

For this reason, unsigned numbers tend to be used only when their bitwise operators or peculiar arithmetic operators are required, as when implementing bit sets, parsing binary file formats, or for hashing and cryptography. They are typically not used for merely non-negative quantities.

### string

A string is an immutable sequence of bytes.

Immutability means that it is safe for two copies of a string to share the same underlying memory, making it cheap to copy strings of any length. Similarly, a string s and a substring like s[7:] may safely share the same data, so the substring operation is also cheap. No new memory is allocated in either case.

The built-in len function returns the number of bytes (not runes) in a string , and the index operation s[i] retrieves the i-th byte of string s, where 0 ≤ i < len(s).

A []rune conversion applied to a UTF-8-encoded string returns the sequence of Unicode code points that the string encodes.

```Go
// "program" in Japanese katakana
s := "プログラム"
fmt.Printf("%x\n", s) // "e3 83 97 e3 83 ad e3 82 b0 e3 83 a9 e3 83 a0"
r := []rune(s)
fmt.Printf("%x\n", r) // "[30d7 30ed 30b0 30e9 30e0]"
```

### constant generator iota

A const declaration may use the constant generator iota, which is used to create a sequence of related values without spelling out each one explicitly. In a const declaration, the value of iota begins at zero and increments by one for each item in the sequence.

```Go
const (
    _ = 1 << (10 * iota) // omit 0
    KiB // 1024
    MiB // 1048576
    GiB // 1073741824
    TiB // 1099511627776 (exceeds 1 << 32)
    PiB // 1125899906842624
    EiB // 1152921504606846976
    ZiB // 1180591620717411303424 (exceeds 1 << 64)
    YiB // 1208925819614629174706176
)
```

### Untyped Constants

PS:
Go's constants can be compared with C++'s macro.

Constants in Go are a bit unusual. Although a constant can have any of the basic data types like int or float64, including named basic types like time.Duration, many constants are not committed to a particular type. The compiler represents these uncommitted constants with much greater numeric precision than values of basic types, and arithmetic on them is more precise than machine arithmetic; you may assume at least 256 bits of precision. There are six flavors of these uncommitted constants, called untyped boolean, untyped integer, untyped rune, untyped floating-point, untyped complex, and untyped string.

By deferring this commitment, untyped constants not only retain their higher precision until later, but they can participate in many more expressions than committed constants without requiring conversions. For example, the values ZiB and YiB in the example above are too big to store in any integer variable, but they are legitimate constants that may be used in expressions like this one:`fmt.Println(YiB/ZiB) // "1024"`

For literals(also untyped constants), syntax determines flavor.

```
var f float64 = 212
fmt.Println((f - 32) * 5 / 9) // "100"; (f - 32) * 5 is a float64
fmt.Println(5 / 9 * (f - 32)) // "0"; 5/9 is an untyped integer, 0
fmt.Println(5.0 / 9.0 * (f - 32)) // "100"; 5.0/9.0 is an untyped float
```

# 4. Composite Types

### array

PS: array like array in C++，and slice like vector in C++.

In an array literal, if an ellipsis ‘‘...’’ appears in place of the length, the array length is determined by the number of initializers.`q := [...]int{1, 2, 3}`

Indices can appear in any order and some may be omitted; as before, unspecified values take on the zero value for the element type. For instance,`r := [...]int{99: -1}`defines an array r with 100 elements, all zero except for the last, which has value −1.

### slice

Unlike arrays, slices are not comparable, so we cannot use == to test whether two slices contain the same elements. The standard library provides the highly optimized bytes.Equal function for comparing two slices of bytes ([]byte), but for other types of slice, we must do the comparison ourselves.

PS: the buildin function copy is like memcpy in C++, it can copy source slice elements to target slice and check whether the two slice has intersection in the underlying array.

### map

In practice, the order is random, varying from one execution to the next. This is intentional; making the sequence vary helps force programs to be robust across implementations.

### Struct Embedding and Anonymous Fields

In this section, we’ll see how Go’s unusual struct embedding mechanism lets us use one named struct type as ananonymous field of another struct type, providing a convenient syntactic shortcut so that a simple dot expression like x.f can stand for a chain of fields like x.d.e.f.

# 5. Functions

You may occasionally encounter a function declaration without a body, indicating that the function is implemented in a language other than Go. Such a declaration defines the function signature.

```Go
package math
func Sin(x float64) float64 // implemented in assembly language
```

### callstack size

Many programming language implementations use a fixed-size function callstack; sizes from 64KB to 2MB are typical. Fixed-size stacks impose a limit on the depth of recursion, so one must be careful to avoid a stack overflow when traversing large data structures recursively; fixed-size stacks may even pose a security risk. In contrast, typical Go implementations use variable-size stacks that start small and grow as needed up to a limit on the order of a gigabyte. This lets us use recursion safely and without worrying about overflow.

### Error-Handling Strategies

Because error messages are frequently chained together, message strings should not be capitalized and new lines should be avoided. The resulting errors may be long , but they will be self-contained when found by tools like grep. like: "genesis: crashed: no parachute: G-switch failed: bad relay orientation"

# 6. Methods

# 7. Interfaces

# 8. Goroutines and Channels

# 9. Concurrency with Shared Variables

# 10. Packages and the Go Tool

# 11. Testing

# 12. Reflection

# 13. Low-Level Programming
