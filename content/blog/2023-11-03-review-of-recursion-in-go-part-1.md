---
title: Review of Algorithms In Go - Part 1
date: 2023-11-03T12:24:20.633Z
description: A basic review of recursion and basic memoization
draft: false
showHero: true
showComments: true
thumbnail: /img/russian-dolls.jpeg
tags:
  - programming
  - basics
  - recursion
  - algorithms
  - tutorials
---
### Algoritmd In Go - Part I

This is a basic review of recursion for Go programmers. It was originally written in a Go Jupyter Notebook.

**Recursion** - A function that calls itself.

Recursive functions must have an exit clause, if they aren't going to be an infinite loop. The exit clause is the condition in which case the function should return a value and *not* call itself. Without one, the function will just call itself forever.

Here is a simple recursive function called factorial, as you may expect it calculated the factorial of an input:

```go
import "fmt"

func factorial(num uint) uint {
    if num == 1 {
        return num
    }
    return num * factorial(num - 1)
}

fmt.Sprintf("10! = %d", factorial(10))
```

```
10! = 3628800
```

### Basic Fibonnaci Sequence

Fibonnaci Sequences are classic examples of recursive functions. Here is the basic implementation of a fibonnaci sequence.

```go
func fib(n uint) uint {
    if n == 0 || n == 1 {
        return n
    }
    return fib(n - 2) + fib(n - 1)
}


fib(12)
```

```
144
```

### Memoization

This function will work, but it is very inefficient. We will not be able to exit any call until `fib(0) || fib(1)` are called. Each time we run fib, we need to call fib 2 more times. This will end up calling our function 465 times for `fib(12)`!!

```go
times_run := 0

func fib(n uint) uint {
    times_run++
    if n == 0 || n == 1 {
        return n
    }
    return fib(n - 2) + fib(n - 1)
}


fmt.Printf("fib(12) = %d\n", fib(12))
fmt.Sprintf("The fib function was run %d times!!!", times_run)
```

```
fib(12) = 144
The fib function was run 465 times!!!
```

In order to better understand this, lets run a modified function - with print statements for `fib(6)` - that equals 8 and should run 25 times:

```go
i := 0

func fib(n uint) uint {
    i++
    fmt.Printf("%d: ENTERED fib(%d)\n", i, n)
    if n == 0 || n == 1 {
        fmt.Printf("\tCALLING fib(%d)\n", n)
        return n
    }
    fmt.Printf("\tCALLING fib(%d) and fib(%d)\n", n - 2, n - 1)
    ans := fib(n - 2) + fib(n - 1)
    return ans
}


result := fib(6)
```

```
1: ENTERED fib(6)
	CALLING fib(4) and fib(5)
2: ENTERED fib(4)
	CALLING fib(2) and fib(3)
3: ENTERED fib(2)
	CALLING fib(0) and fib(1)
4: ENTERED fib(0)
	CALLING fib(0)
5: ENTERED fib(1)
	CALLING fib(1)
6: ENTERED fib(3)
	CALLING fib(1) and fib(2)
7: ENTERED fib(1)
	CALLING fib(1)
8: ENTERED fib(2)
	CALLING fib(0) and fib(1)
9: ENTERED fib(0)
	CALLING fib(0)
10: ENTERED fib(1)
	CALLING fib(1)
11: ENTERED fib(5)
	CALLING fib(3) and fib(4)
12: ENTERED fib(3)
	CALLING fib(1) and fib(2)
13: ENTERED fib(1)
	CALLING fib(1)
14: ENTERED fib(2)
	CALLING fib(0) and fib(1)
15: ENTERED fib(0)
	CALLING fib(0)
16: ENTERED fib(1)
	CALLING fib(1)
17: ENTERED fib(4)
	CALLING fib(2) and fib(3)
18: ENTERED fib(2)
	CALLING fib(0) and fib(1)
19: ENTERED fib(0)
	CALLING fib(0)
20: ENTERED fib(1)
	CALLING fib(1)
21: ENTERED fib(3)
	CALLING fib(1) and fib(2)
22: ENTERED fib(1)
	CALLING fib(1)
23: ENTERED fib(2)
	CALLING fib(0) and fib(1)
24: ENTERED fib(0)
	CALLING fib(0)
25: ENTERED fib(1)
	CALLING fib(1)
```

This is very inefficient, for example, look how many times we calculate `fib(0)`!! We should only be doing that once! 

***SIDE NOTE**: Another thing to notice is that while we seem to call `fib(4)` and `fib(5)` we only enter `fib(4)` - this is because we can't enter to `fib(5)` until we finish `fib(4)`. This is due to something called a call stack - which I will discuss more when I post about dynamic programming. ;)*

```go
fmt.Printf("fib(6) = %d\n", result)
fmt.Sprintf("Ran %d times!\n", i)
```

```
fib(6) = 8
Ran 25 times!
```

One method of reducing the amount of times the function is called is a method called memoization. Since the fibonacci of any given input number will always be the same, we don't need to repeat it each time. We can just map the result we get to the input the first time. If we've already calculated the fibonacci of any number, we just pull it out of our hash map instead of calculating it again. Here we can reduce the number of `fib(6)` from 25 to just 11!

```go
mi := 0

func fibInner(n uint, m map[uint]uint) uint {
        mi++
        fmt.Printf("%d) ENTERED fibInner(%d)\n", mi, n)
        if n == 0 || n == 1 {
            fmt.Printf("\t%d) CALLING fibInner(%d)\n", mi, n)
            return n
        }
        if m[n] == 0 {
            fmt.Printf("\tCALLING fibInner(%d) and fibInner(%d)\n", n-2, n-1)
            m[n] = fibInner(n-2, m) + fibInner(n-1, m)
        }
        return m[n]
    }

func mfib(n uint) uint {
    m := make(map[uint]uint);

    return fibInner(n, m)
}

mresult := mfib(6)
```

```
1) ENTERED fibInner(6)
	CALLING fibInner(4) and fibInner(5)
2) ENTERED fibInner(4)
	CALLING fibInner(2) and fibInner(3)
3) ENTERED fibInner(2)
	CALLING fibInner(0) and fibInner(1)
4) ENTERED fibInner(0)
	4) CALLING fibInner(0)
5) ENTERED fibInner(1)
	5) CALLING fibInner(1)
6) ENTERED fibInner(3)
	CALLING fibInner(1) and fibInner(2)
7) ENTERED fibInner(1)
	7) CALLING fibInner(1)
8) ENTERED fibInner(2)
9) ENTERED fibInner(5)
	CALLING fibInner(3) and fibInner(4)
10) ENTERED fibInner(3)
11) ENTERED fibInner(4)
```

``

```go
fmt.Printf("mfib(6) = %d\n", mresult)
fmt.Sprintf("Ran %d times!\n", mi)
```

```
mfib(6) = 8
Ran 11 times!
```

For the case of `fib(12)` we can reduce it from 465 to just 23!

```go
mi = 0

func fibInner(n uint, m map[uint]uint) uint {
        mi++
        if n == 0 || n == 1 {
            return n
        }
        if m[n] == 0 {
            m[n] = fibInner(n-2, m) + fibInner(n-1, m)
        }
        return m[n]
    }

func mfib(n uint) uint {
    m := make(map[uint]uint);

    return fibInner(n, m)
}

func fib(n uint) uint {
    if n == 0 || n == 1 {
        return n
    }
    return fib(n - 2) + fib(n - 1)
}


mresult := mfib(12)
```

```go
fmt.Printf("mfib(12) = %d\n", mresult)
fmt.Sprintf("mfib ran %d times!\n", mi)
```

```
mfib(12) = 144
mfib ran 23 times!
```

```go
mfib(50)
```

```
12586269025
```

if we remove the incrementer statements and just time how long it takes to finish each function, we can see how much more time it takes to fun a fibonacci sequence on a slightly higher number like 35

```go
import "time"

func fibInner(n uint, m map[uint]uint) uint {
        if n == 0 || n == 1 {
            return n
        }
        if m[n] == 0 {
            m[n] = fibInner(n-2, m) + fibInner(n-1, m)
        }
        return m[n]
    }

func mfib(n uint) uint {
    m := make(map[uint]uint);

    return fibInner(n, m)
}

start := time.Now()
mfib(35)
end := time.Now()

executionTime := end.Sub(start)

fmt.Sprintf("Executed in %v", executionTime)
```

```
Executed in 366.694µs
```

Well that was rather fast with memoization! Let's see how long it takes without memoization:

```go
start := time.Now()
fib(35)
end := time.Now()

executionTime := end.Sub(start)

fmt.Sprintf("Executed in %v", executionTime)
```

```
Executed in 3.045922618s
```

Holy Moly!! That was over 3 seconds!!

My results indicate that with memoization, it is 8,306 times faster!!

Results may vary slightly.