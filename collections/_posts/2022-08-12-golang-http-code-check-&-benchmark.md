---
title: "HTTP Code Check & Benchmark"
date: 2022-08-12T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: A simple trick to make your code look swag with an introduction to benchmark testing.
thumbnail: "assets/images/thumbnail/2022-08-12-golang-http-code-check-&-benchmark.jpg"
comments: true
---

I'll show you a code **trick** to check if an **HTTP status** is OK (between 200 and 299).

It will also allow me to **introduce the benchmark** in Go.

## Example

**code.go**:

```go
package main

func Code(n int) bool {
	return n/100 == 2
}

func Code2(n int) bool {
	if http.StatusOK > n || n > 299 {
		return false
	}
	return true
}
```

Go playground: <https://go.dev/play/p/ARWCYyeR4rf>.

Simply **divide the HTTP code** by **100** and **compare** it to an **integer**.
This can help you to **avoid errors** on the condition, missing an `=` or the `<` being on the wrong side etc.

Now that we have an excuse to **compare performance** let's see how to write a **Go benchmark**.

Go already **comes with a complete tooling** to benchmark your functions.

**code_test.go**:

```go
package main

import "testing"

func BenchmarkCode(b *testing.B) {
	// run the Code function b.N times
	for n := 0; n < b.N; n++ {
		Code(n)
	}
}

func BenchmarkCode2(b *testing.B) {
	// run the Code2 function b.N times
	for n := 0; n < b.N; n++ {
		Code2(n)
	}
}
```

Let's look in detail at the signature of the functions:

```go
BenchmarkCode(b *testing.B)
```

You can see that the **input** of the **benchmark** test is **testing.B** as **opposed** to a **unit test** which has **testing.T**.

{% include alerts/info.html content='If you want to run both types of tests you can use <strong>testing.TB</strong>. Checkout <a href="https://pkg.go.dev/testing">https://pkg.go.dev/testing</a> to get more details.' %}

To **run** your benchmark, use this **command**:

```bash
go test -bench=.
```

The loop on **b.N** allows us to set the **number of iterations** with parameters to the **go test** command using **-benchtime=1000x**.

You can also use **-count 10** to **run your test several times** and get an **average** at the end.

{% include alerts/info.html content='If you want to go further into the parameters, see <a href="https://pkg.go.dev/cmd/go#hdr-Testing_flags">https://pkg.go.dev/cmd/go#hdr-Testing_flags</a> for more details.' %}

## Result of Code2 function

```console
$ go test -benchmem -run=^$ -bench ^BenchmarkCode2$ test -count 10
goos: linux
goarch: amd64
pkg: test
cpu: Intel(R) Xeon(R) CPU E5-2689 v4 @ 3.10GHz
BenchmarkCode2-4   	1000000000	         0.3477 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3361 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3312 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3594 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3381 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3759 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3297 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3923 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3461 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode2-4   	1000000000	         0.3335 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	test	3.956s
```

## Result of Code (the trick) function

```console
$ go test -benchmem -run=^$ -bench ^BenchmarkCode$ test -count 10
goos: linux
goarch: amd64
pkg: test
cpu: Intel(R) Xeon(R) CPU E5-2689 v4 @ 3.10GHz
BenchmarkCode-4   	1000000000	         0.3229 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3264 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3440 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3449 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3391 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3276 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3180 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3219 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3540 ns/op	       0 B/op	       0 allocs/op
BenchmarkCode-4   	1000000000	         0.3352 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	test	3.749s
```

## Conclusion

After several runs of the benchmark we can see that the **Code** solution performs **slightly better**.

{% include alerts/green.html content='Benchmark can be interesting, especially if you have algorithmic parts with a large volume of data to process.' %}
