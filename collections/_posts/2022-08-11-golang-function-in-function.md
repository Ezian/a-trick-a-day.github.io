---
title: "Function in Function"
date: 2022-08-11T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: Need to pass more parameters to a function type parameter, you will know everything at the end.
thumbnail: "assets/images/thumbnail/2022-08-11-golang-function-in-function.png"
comments: true
---

When we do API configuration we often find ourselves having to **pass functions in parameters**.
For example **http.HandleFunc** need a function in second parameter:

```go
func http.HandleFunc(pattern string, handler func(http.ResponseWriter, *http.Request))
```

But you would like to be able to **pass an additional parameter** or you would like to **extend this function**.

Here is an example:

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/hello", middlewareOne("hello", http.HandlerFunc(hello)))
	http.HandleFunc("/headers", middlewareOne("headers", http.HandlerFunc(headers)))

	http.ListenAndServe(":8080", nil)
}

func middlewareOne(value string, next http.Handler) func(http.ResponseWriter, *http.Request) {
	return func(w http.ResponseWriter, r *http.Request) {
		log.Printf("Executing middlewareOne: %s", value)
		next.ServeHTTP(w, r)
	}
}

func hello(w http.ResponseWriter, r *http.Request) {
	log.Print("Executing hello")
	w.Write([]byte("OK"))
}

func headers(w http.ResponseWriter, r *http.Request) {
	log.Print("Executing headers")
	w.Write([]byte("OK"))
}
```

Go playground: <https://go.dev/play/p/NsXH7pH085X>.

**middlewareOne** allows us to log a message at each HTTP request and **takes as input a new parameter** that was not accepted by **http.HandleFunc**.

```console
$ go run .
2022/08/10 11:40:45 Executing middlewareOne: hello
2022/08/10 11:40:45 Executing hello
2022/08/10 11:41:05 Executing middlewareOne: headers
2022/08/10 11:41:05 Executing headers
```

For **most of the HTTP libraries**, when you will have to **develop middleware** you will have to do it this way, thanks to this trick you will be **able to pass additional configuration parameters** like the configuration of your logger or your opentelemetry lib etc.

Check <https://github.com/justinas/alice>, that can help you to **chain your HTTP middlewares**.

We can take **another case** that can happen more often.
Let's say you want to **loop** over an **interfaced function slice**, but the interface **prevents** you from **passing parameters**.
You can do it like this:

```go
package main

import "fmt"

type Task func()

func main() {
	defaultTask := func(id string) Task {
		return func() {
			fmt.Println(id)
		}
	}
	tasks := []Task{
		func() {
			fmt.Println("1")
		},
		defaultTask("2"),
		defaultTask("3"),
	}
	for _, t := range tasks {
		t()
	}
}
```

Go playground: <https://go.dev/play/p/RHYBqB9qeZ3>.

```console
$ go run .
1
2
3
```
