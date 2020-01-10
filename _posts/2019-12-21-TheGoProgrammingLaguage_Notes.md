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

PS: in go1.13.1 this problem is solved, because `len()` will return int not uint.

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

Because error messages are frequently chained together, message strings should not be capitalized and new lines should be avoided. The resulting errors may be long , but they will be self-contained when found by tools like grep.
PS: the error message should separated by ':'.

### Caveat: Capturing Iteration Variables

```Go
var rmdirs []func()
    for _, d := range tempDirs() {
        dir := d // NOTE: necessary!
        os.MkdirAll(dir, 0755) // creates parent directories too
        rmdirs = append(rmdirs, func() {
        os.RemoveAll(dir)
    })
}
// ...do some work...
for _, rmdir := range rmdirs {
    rmdir() // clean up
}
```

The reason is a consequence of the scope rules for loop variables. In the program immediately above, the for loop introduces a new lexical block in which the variable dir is declared. All function values created by this loop "capture" and share the same variable—an addressable storage location, not its value at that particular moment.

### Variadic Function

Although the ...int parameter behaves like a slice within the function body, the type of a variadic function is distinct from the type of a function with an ordinary slice parameter.

```Go
func f(...int) {}
func g([]int) {}
fmt.Printf("%T\n", f) // "func(...int)"
fmt.Printf("%T\n", g) // "func([]int)"
```

### Deferred Function Calls

Deferred functions run after return statements have updated the function’s result variables. Because an anonymous function can access its enclosing function’s variables, including named results, a deferred anonymous function can observe the function’s results or even change it.

On many file systems, notably NFS, write errors are not reported immediately but may be postponed until the file is closed. So the close function should not be called by defer to make sure the error were handled properly.

### Recover

PS: panic and recover is like try catch exception mechanism in C++.

If the built-in recover function is called within a deferred function and the function containing the defer statement is panicking, recover ends the current state of panic and returns the panic value. The function that was panicking does not continue where it left off but returns normally. If recover is called at any other time, it has no effect and returns nil.

Recovering from a panic within the same package can help simplify the handling of complex or unexpected errors, but as a general rule, you should not attempt to recover from another package’s panic. Public APIs should report failures as errors. Similarly, you should not recover from a panic that may pass through a function you do not maintain, such as a caller-provided callback, since you cannot reason about its safety.

For example, the net/http package provides a web server that dispatches incoming requests to user-provided hand ler functions. Rather than let a panic in one of these handlers kill the process, the server calls recover, prints a stack trace, and continues serving. This is convenient in practice, but it does risk leaking resources or leaving the failed handler in an unspecified state that could lead to other problems.

For all the above reasons, it’s safest to recover selectively if at all. In other words, recover only from panics that were intended to be recovered from, which should be rare. This intention can be encoded by using a distinct, unexported type for the panic value and testing whether the value returned by recover has that type. (We’ll see one way to do this in the next example.) If so, we report the panic as an ordinary error; if not, we call panic with the same value to resume the state of panic.

From some conditions there is no recovery. Running out of memory, for example, causes the Go runtime to terminate the program with a fatal error.

PS: this is not like C++, which return a null when no memory can be use for new.

# 6. Methods

method, receiver, selector, field

Go is often convenient to define additional behaviors for simple types such as numbers, strings, slices, maps, and sometimes even functions. Methods may be declared on any named type defined in the same package, so long as its underlying type is neither a pointer nor an interface.

### Methods with a Pointer Receiver

PS: interface satisfaction
it is legal to call a *T method on an argument of type T so long as the argument is a variable; the compiler implicitly takes its address. But this is mere syntactic sugar: a value of type T does not possess all the methods that a *T pointer does, and as a result it might satisfy fewer interfaces.

### Nil Is a Valid Receiver Value

### Method Values and Expressions

Usually we select and call a method in the same expression, as in p.Distance(), but it’s possible to separate these two operations. The selector p.Distance yields a method value, a function that binds a method(Point.Distance) to a specific receiver value p. This function can then be invoked without a receiver value; it needs only the non-receiver arguments.

```Go
p := Point{1, 2}
q := Point{4, 6}
distanceFromP := p.Distance // method value
fmt.Println(distanceFromP(q))
```

```Go
type Rocket struct { /* ... */ }
func (r *Rocket) Launch() { /* ... */ }
r := new(Rocket)
time.AfterFunc(10 * time.Second, func() { r.Launch() })

// The method value syntax is shorter:
time.AfterFunc(10 * time.Second, r.Launch)
```

Related to the method value is the method expression. When calling a method, as opposed to an ordinary function, we must supply the receiver in a special way using the selector syntax. A method expression, written T.f or (\*T).f where T is a type, yields a function value with a regular first parameter taking the place of t he receiver, so it can be called in the usual way.

```Go
p := Point{1, 2}
q := Point{4, 6}
distance := Point.Distance
// the next two lines are the same
fmt.Println(distance(p, q))
fmt.Println(p.Distance(q))
```

### Example: Bit Vector Type

# 7. Interfaces

contract, substituability

### embedding an interface

```Go
package io
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Closer interface {
    Close() error
}
type ReadWriter interface {
    Reader
    // Read(p []byte) (n int, err error)
    // the up one is OK too
    Writer
}
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

The syntax used above, which resembles struct embedding , lets us name another interface as a shorthand for writing out all of its methods. This is called embedding an interface.

### Interface Satisfaction

As a shorthand, Go programmers often say that a concrete type "is a" particular interface type, meaning that it satisfies the interface. For example, a *bytes.Buffer is an io.Writer; an *os.File is an io.ReadWriter.

An interface with more methods, such as io.ReadWriter, tells us more about the values it contains, and places greater demands on the types that implement it, than do es an interface with fewer methods such as io.Reader. So what does the type interface{}, which has no methods at all, tell us about the concrete types that satisfy it?
That’s right: nothing. This may seem useless, but in fact the type interface{}, which is called the empty interface type, is indispensable. Because the empty interface type places no demands on the types that satisfy it, we can assign any value to the empty interface.

Example: Parsing Flags with flag.Value

### Interface Values

interface value = dynamic type + dynamic value

Interface values may be compared using == and !=. Two interface values are equal if both are nil, or if their dynamic types are identical and their dynamic values are equal according to the usual behavior of == for that type. Because interface values are comparable, they may be used as the keys of a map or as the operand of a switch statement.

However, if two interface values are compared and have the same dynamic type, but that type is not comparable (a slice, for instance), then the comparison fails with a panic:

```Go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```

- can not comparable type: `slice/map/function`

### Caveat: An Interface Containing a Nil Pointer Is Non-Nil

```Go
const debug = true
func main() {
    var buf *bytes.Buffer
    // the correct one should be "var buf bytes.Buffer"
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
    }
    f(buf) // NOTE: subtly incorrect!
}

func f(out io.Writer) {
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```

When main calls f, it assigns a nil pointer of type *bytes.Buffer to the out parameter, so the dynamic value of out is nil. However, its dynamic type is *bytes.Buffer, meaning that out is a non-nil interface containing a nil pointer value, so the defensive check out != nil is still true.

The problem is that although a nil \*bytes.Buffer pointer has the methods needed to satisfy the interface, it doesn’t satisfy the behavioral requirements of the interface. In particular, the call violates the implicit precondition of (\*bytes.Buffer). Write that its receiver is not nil, so assigning the nil pointer to the interface was a mistake.

### Type Assertions

There are two possibilities. First, if the asserted type T is a concrete type, then the type assertion checks whether x’s dynamic type is identical to T. If this check succeeds, the result of the type assertion is x’s dynamic value, whose type is of course T. In other words, a type assertion to a concrete type extracts the concrete value from its operand. If the check fails, then the operation panics.

Second, if instead the asserted type T is an interface type, then the type assertion checks whether x’s dynamic type satisfies T. If this check succeeds, the dynamic value is not extracted; the result is still an interface value with the same type and value components, but the result has the interface type T. In other words, a type assertion to an interface type changes the type of the expression, making a different (and usually larger) set of methods accessible, but it preserves the dynamic type and value components inside the interface value.

No matter what type was asserted, if the operand is a nil interface value, the type assertion fails. A type assertion to a less restrictive interface type (one w ith fewer methods) is rarely needed, as it behaves just like an assignment, except in the nil case.

### Querying Behaviors with Interface Type Assertions

```Go
// writeString writes s to w.
// If w has a WriteString method, it is invoked instead of w.Write.
func writeString(w io.Writer, s string) (n int, err error) {
    type stringWriter interface {
        WriteString(string) (n int, err error)
    }
    if sw, ok := w.(stringWriter); ok {
        return sw.WriteString(s) // avoid a copy
    }
    return w.Write([]byte(s)) // allocate temporary copy
}
```

What’s curious in this example is that there is no standard interface that defines the WriteString method and specifies its required behavior. Further more, whether or not a concrete type satisfies the stringWriter interface is determined only by its methods, not by any declared relationship between it and the interface type. What this means is that the technique above relies on the assumption that if a type satisfies the interface below, then WriteString(s) must have the same effect as Write([]byte(s)).

### Type Switches

- object-oriented programming, subtype polymorphism, ad hoc polymorphism

```Go
switch x.(type) {
    case nil:         // ...
    case int, uint:   // ...
    case bool:        // ...
    case string:      // ...
    default:          // ...
```

As with an ordinary switch statement, cases are considered in order and, when a match is found, the case’s body is executed. Case order becomes significant when one or more case types are interfaces, since then there is a possibility of two cases matching. The position of the default case relative to the others is immaterial. No fallthrough is allowed.

```Go
switch y := x.(type) { /* ... */ }
```

# 8. Goroutines and Channels

- CSP(communicating sequential processes)
  In discussions of concurrency, when we say x happens before y, we don’t mean merely that x occurs earlier in time than y; we mean that it is guaranteed to do so and that all its prior effects, such as updates to variables, are complete and that you may rely on them.
  When x neither happens before y nor after y, we say that x is concurrent with y. This doesn’t mean that x and y are necessarily simultaneous, merely that we cannot assume anything about their ordering .

### range loop to iterate over channels

```Go
go func() {
    for x := range naturals {
        squares <- x * x
    }
    close(squares)
}()

// this one is clumsy
go func() {
    for {
        x, ok := <-naturals if !ok {
            break // channel was closed and drained
        }
        squares <- x * x
    }
    close(squares)
}()
```

### Unidirectional Channel Types

- it is a compile-time error to attempt to close a receive-only channel.

### Multiplexing with select

```Go
select {
    case <-ch1:
    // ...
    case x := <-ch2:
    // ...use x...
    case ch3 <- y:
    // ...
    default:
    // ...
}
```

The general form of a select statement is shown above. Like a switch statement, it has a number of cases and an optional default. Each case specifies a communication(as end or receive operation on some channel) and an associated block of statements. A receive expression may appear on its own, as in the first case, or within a short variable declaration, as in the second case; the second form lets you refer to the received value.
A select waits until a communication for some case is ready to proceed. It then performs that communication and executes the case’s associated statements; the other communications do not happen. A select with no cases, select{}, waits forever.And a select with default is non-blocking, which specifies what to do when none of the other communications can proceed immediately.

# 9. Concurrency with Shared Variables

### The Race Detector

Just add the -race flag to your go build, go run, or go test command. This causes the compiler to build a modified version of your application or test with additional instrumentation that effectively records all accesses to shared variables that occurred during execution, along with the identity of the goroutine that read or wrote the variable. In addition, the modified program records all synchronization events, such as go statements, channel operations, and calls to (\*sync.Mutex).Lock, (\*sync.WaitGroup).Wait, and so on. (The complete set of synchronization events is specified by the The Go Memory Model document that accompanies the language specification.)
The race detector studies this stream of events, looking for cases in which one goroutine reads or writes a shared variable that was most recently written by a different goroutine without an intervening synchronization operation. This indicates a concurrent access to the shared variable, and thus a data race. The tool prints a report that includes the identity of the variable, and the stacks of active function calls in the reading goroutine and the writing goroutine. This is usually sufficient to pinpoint the problem.

### Growable Stacks

Each OS thread has a fixed-size block of memory (often as large as 2MB) for its stack, the work area where it saves the local variables of function calls that are in progress or temporarily suspended while another function is called. This fixed-size stack is simultaneously too much and too little. A 2MB stack would be a huge waste of memory for a little goroutine, such as one that merely waits for a WaitGroup then closes a channel. It’s not uncommon for a Go program to create hundreds of thousands of goroutines at one time, which would be impossible with stacks this large. Yet despite their size, fixed-size stacks are not always big enough for the most complex and deeply recursive of functions. Changing the fixed size can improve space efficiency and allow more threads to be created, or it can enable more deeply recursive functions, but it cannot do both.

In contrast, a goroutine starts life with a small stack, typically 2KB. A goroutine’s stack, like the stack of an OS thread, holds the local variables of active and suspended function calls, but unlike an OS thread, a goroutine’s stack is not fixed; it grows and shrinks as needed. The size limit for a goroutine stack may be as much as 1GB, orders of magnitude larger than a typical fixed-size thread stack, though of course few goroutines use that much.

### Goroutine Scheduling

Because OS threads are scheduled by the kernel, passing control from one thread to another requires a full context switch, that is, saving the state of one user thread to memory, restoring the state of another, and updating the scheduler’s datastructures. This operation is slow, due to its poor locality and the number of memory accesses required, and has historically only gotten worse as the number of CPU cycles required to access memory has increased.

The Go runtime contains its own scheduler that uses a technique known as m:n scheduling, because it multiplexes (or schedules) m goroutines on n OS threads. The job of the Go scheduler is analogous to that of the kernel scheduler, but it is concerned only with the goroutines of a single Go program.

Unlike the operating system’s thread scheduler, the Go scheduler is not invoked periodically by a hardware timer, but implicitly by certain Go language constructs. For example, when a goroutine calls time.Sleep or blocks in a channel or mutex operation, the scheduler puts it to sleep and runs another goroutine until it is time to wake the first one up. Because it doesn’t need a switch to kernel context, rescheduling a goroutine is much cheaper than rescheduling a thread.

### Goroutines Have No Identity

Go encourages a simpler style of programming in which parameters that affect the behavior of a function are explicit. Not only does this make programs easier to read, but it lets us freely assign subtasks of a given function to many different goroutines without worrying about their identity.

# 10. Packages and the Go Tool

### why Go compile faster ?

Go compilation is notably faster than most other compiled languages, even when building from scratch. There are three main reasons for the compiler’s speed.

- First, all imports must be explicitly listed at the beginning of each source file, so the compiler does not have to read and process an entire file to determine its dependencies.
- Second, the dependencies of a package form a directed acyclic graph, and because there are no cycles, packages can be compiled separately and perhaps in parallel.
- Finally, the object file for a compiled Go package records export information not just for the package itself, but for its dependencies too. When compiling apackage, the compiler must read one object file for each import but need not look beyond these files.

### The Package Declaration

There are three major exceptions to the "last segment" convention:

- The first is that a package defining a command (an executable Go program) always has the name main, regardless of the package’s import path. This is a signal to go build that it must invoke the linker to make an executable file.
- The second exception is that some files in the directory may have the suffix \_test on their package name if the file name ends with \_test.go. Such a directory may define two packages: the usual one, plus another one called an external test package. The \_test suffix signals to go test that it must build both packages, and it indicates which files belong to each package. External test packages are used to avoid cycles in the import graph arising from dependencies of the test;
- The third exception is that some tools for dependency management append version number suffixes to package import paths, such as "gopkg.in/yaml.v2". The package name excludes the suffix, so in this case it would be just yaml.

### Import Declarations

A renaming import may be useful even when there is no conflict. If the name of the imported package is unwieldy, as is sometimes the case for automatically generated code, an abbreviated name may be more convenient. The same short name should be used consistently to avoid confusion. Choosing an alternative name can help avoid conflicts with common local variable names. For example, in a file with many local variables named path, we might import the standard "path" package as pathpkg.

- blank import

### The Go Tool

\$ go ...
build compile packages and dependencies
clean remove object files
doc show documentation for package or symbol
env print Go environment information
fmt run gofmt on package sources
get download and install packages and dependencies
install compile and install packages and dependencies
list list packages
run compile and run Go program
test test packages
version print Go version
vet run go tool vet on packages
mod package dependency management
Use "go help [command]" for more information about a command.
...

### build tags

### Internal Packages

To address these needs, the go build tool treats a package specially if its import path contains a path segment named internal. Such packages are called internal packages. An internal package may be imported only by another package that is inside the tree rooted at the parent of the internal directory. For example, given the packages below, net/http/internal/chunked can be imported from net/http/httputil or net/http, but not from net/url. However,net/url may importnet/http/httputil.

```
net/http
net/http/internal/chunked
net/http/httputil
net/url
```

# 11. Testing

- Test Banchmark Example

- profile

# 12. Reflection

- reflect.Type, reflect.TypeOf
- reflect.Value, reflect.ValueOf

```Go
func display(path string, v reflect.Value) {
    switch v.Kind() {
        case reflect.Invalid:
            fmt.Printf("%s = invalid\n", path)
        case reflect.Slice, reflect.Array:
            for i := 0; i < v.Len(); i++ {
                display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
            }
        case reflect.Struct:
            for i := 0; i < v.NumField(); i++ {
                fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
                display(fieldPath, v.Field(i))
            }
        case reflect.Map:
            for _, key := range v.MapKeys() {
                display(fmt.Sprintf("%s[%s]", path, formatAtom(key)), v.MapIndex(key))
            }
        case reflect.Ptr:
            if v.IsNil() {
                fmt.Printf("%s = nil\n", path)
            } else {
                display(fmt.Sprintf("(*%s)", path), v.Elem())
            }
        case reflect.Interface:
            if v.IsNil() {
                fmt.Printf("%s = nil\n", path)
            } else {
                fmt.Printf("%s.type = %s\n", path, v.Elem().Type()) display(path+".value", v.Elem())
             }
         default: // basic types, channels, funcs
            fmt.Printf("%s = %s\n", path, formatAtom(v)
    }
}
```

### A Word of Caution

There is a lot more to the reflection API than we have space to show, but the preceding examples give an idea of what is possible. Reflection is a powerful and expressive tool, but it should be used with care, for three reasons.

The first reason is that reflection-based code can be fragile. For every mistake that would cause a compiler to report a type error, there is a corresponding way to misuse reflection, but where as the compiler reports the mistake at build time, a reflection error is reported during execution as apanic, possibly long after the program was written or even long after it has started running.

The best way to avoid this fragility is to ensure that the use of reflection is fully encapsulated within your package and, if possible, avoid reflect.Value in favor of specific types in your package’s API, to restrict inputs to legal values. If this is not possible, perform additional dynamic checks before each risky operation.

Reflection also reduces the safety and accuracy of automated refactoring and analysis tools, because they can’t determine or rely on type information.

The second reason to avoid reflection is that since types serve as a form of documentation and the operations of reflection cannot be subject to static type checking, heavily reflective code is often hard to understand. Always carefully document the expected types and other invariants of functions that accept an interface{} or a reflect.Value.

The third reason is that reflection-based functions may be one or two orders of magnitude slower than code specialized for a particular type. In a typical program, the majority of functions are not relevant to the overall performance, so it’s fine to use reflection when it makes the program clearer. Testing is a particularly good fit for reflection since most tests use small datasets. But for functions on the critical path, reflection is best avoided.

# 13. Low-Level Programming

### unsafe.Sizeof, Alignof, and Offsetof

Despite their names, these functions are not in fact unsafe, and they may be helpful for understanding the layout of raw memory in a program when optimizing for space.

### unsafe.Pointer

Most pointer types are written *T, meaning "a pointer to a variable of type T." The unsafe.Pointer type is a special kind of pointer that can hold the address of any variable. Of course, we can’t indirect through an unsafe.Pointer using *p because we don’t know what type that expression should have. Like ordinary pointers, unsafe.Pointers are comparable and may be compared with nil, which is the zero value of the type.
An ordinary *T pointer may be converted to an unsafe.Pointer, and an unsafe.Pointer may be converted back to an ordinary pointer, not necessarily of the same type *T.

Some garbage collectors move variables around in memory to reduce fragmentation or book keeping . Garbage collectors of this kind are known as moving GCs. When a variable is moved, all pointers that hold the address of the old location must be updated to point to the new one. From the perspective of the garbage collector, an unsafe.Pointer is a pointer and thus its value must change as the variable moves, but a uintptr is just a number so its value must not change.

Recall from Section 5.2 that goroutine stacks grow as needed. When this happens, all variables on the old stack may be relocated to a new, larger stack, so we cannot rely on the numeric value of a variable’s address remaining unchanged throughout its lifetime.

```Go
// NOTE: subtly incorrect!
tmp := uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b) pb := (*int16)(unsafe.Pointer(tmp))
*pb = 42
```

At the time of writing , there is little clear guidance on what Go programmers may rely up on after an unsafe.Pointer to uintptr conversion (see Go issue 7192), so we strongly recommend that you assume the bare minimum. Treat all uintptr values as if they contain the former address of a variable, and minimize the number of operations between converting an unsafe.Pointer to a uintptr and using that uintptr. In our first example above, the three operations—conversion to a uintptr, addition of the field offset, conversion back—all appeared within a single expression.

### Calling C Code with cgo
