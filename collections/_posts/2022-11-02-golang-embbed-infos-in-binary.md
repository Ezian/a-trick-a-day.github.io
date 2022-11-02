---
title: "CI Traceability & Golang"
date: 2022-11-02T20:30:03+01:00
layout: post
authors: ["Fouillet Guillaume"]
categories: ["Golang", "Development", CI]
description: Embedding CI informations in Go Binaries.
thumbnail: "assets/images/thumbnail/2022-11-02-traceability.jpeg"
comments: true
---

Traceability should be a main concern for all software we develop and distribute. In case of failure of a software, there is a real value to be able to directly read the code that is actually run.

To do that, we need to embbed contextual information into our software, like:

* version
* commit id
* author
* build date
* ...

However, a simple way to do that, in some case, is to provide some metadata file that will hosts those information. But it comes with several drawbacks:

* Mutability, a simple file can be edited and information be compromised
* Security, in a web server, some security issue can cause the file to be exposed and this information are very valuable for an attacker
* Possibility, sometimes, like when you implement a CLI, it's just not possible to have a file to host those informations.

So... How to do that ?



The **generics** have finally **arrived** since the go **1.18**. You will find here the basics on its use.

A very basic example that you will find a lot in your code.
It's the **famous "Contains"** in a slice that we **recode** each time:

```go
package main

import "fmt"

func ContainsInt64(s []int64, v int64) bool {
	for _, t := range s {
		if t == v {
			return true
		}
	}
	return false
}

func ContainsFloat64(s []float64, v float64) bool {
	for _, t := range s {
		if t == v {
			return true
		}
	}
	return false
}

type Number interface {
	int64 | float64
}

// generics houra !!!
func Contains[V Number](s []V, v V) bool {
	for _, t := range s {
		if t == v {
			return true
		}
	}
	return false
}

// or
func Contains[V comparable](s []V, v any) bool {
	for _, t := range s {
		if t == v {
			return true
		}
	}
	return false
}

func main() {
	fmt.Println(ContainsInt64([]int64{1, 2, 3}, 1))
	fmt.Println(ContainsFloat64([]float64{1, 2, 3}, 1))

	fmt.Println(Contains([]int64{1, 2, 3}, 1))
	fmt.Println(Contains([]float64{1, 2, 3}, 1))
}
```

Go playground: <https://go.dev/play/p/xeUh3GPI3Ok>.

Another example with a function to **online some pointers**, that works with multiple types:

```go
package main

import "fmt"

func ToPtr[V any](v V) *V {
	return &v
}

func main() {
	fmt.Println(ToPtr("a"))
	fmt.Println(ToPtr(1))
}
```

Go playground: <https://go.dev/play/p/qcQB1oyCLgg>.

{% include alerts/info.html content='Note that you can create interfaces that contain your aggregate types.' %}

Thanks to these generics you will find **new libraries** like **<https://pkg.go.dev/golang.org/x/exp/slices>** that will **help you** a lot.

In the `slices` library, you will find **many functions** that you **used to code yourself**, which are **now available** as generics and as a library:

```go
package main

import (
	"fmt"

	"golang.org/x/exp/slices"
)

func main() {
	fmt.Println(slices.Contains([]int64{1, 2, 3}, 1))
	fmt.Println(slices.Contains([]float64{1, 2, 3}, 1))
}
```

Go playground: <https://go.dev/play/p/CsnGdu_EC8Z>.

You will find functions like:

- `Contains`
- `Compare`
- `Index`
- `Sort`

And more !

{% include alerts/info.html content='Don’t forget the <a href="https://pkg.go.dev/golang.org/x/exp/slices">slices library</a>, it will help you a lot!' %}

## References

<https://go.dev/doc/tutorial/generics>
