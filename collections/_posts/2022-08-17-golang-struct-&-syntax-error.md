---
title: "Struct & Syntax Error"
date: 2022-08-17T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: Thorough use of struct and avoid syntax compilation errors.
thumbnail: "assets/images/thumbnail/2022-08-17-golang-struct-&-syntax-error.jpg"
comments: true
---

```go
package main

type E1 struct {
	[]int
}

type E2 struct {
	map[string]string
}

type E3 struct {
	interface{}
}
```

Go playground: <https://go.dev/play/p/g2Jt6Nu9xI1>.

```console
$ go build .
# test
./main.go:4:2: syntax error: unexpected [, expecting field name or embedded type
./main.go:8:2: syntax error: unexpected map, expecting field name or embedded type
./main.go:12:2: syntax error: unexpected interface, expecting field name or embedded type
./main.go:13:1: syntax error: non-declaration statement outside function body
```

To solve this problem, you need to **embed it in a type**, and then **inherit** that type in **your struct**.

```go
type E1Sub []int
type E1 struct {
	E1Sub
}

type E2Sub []int
type E2 struct {
	E2Sub
}

type E3Sub []int
type E3 struct {
	E3Sub
}
```

Go playground: <https://go.dev/play/p/xF3ownQt3N7>.
