---
title: "Anonymous functions"
date: 2022-08-03T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: Go language provides a special feature known as an anonymous function.
thumbnail: "assets/images/thumbnail/2022-08-03-golang-anonymous-functions.jpg"
comments: true
---

An anonymous function is a function which doesn’t contain any name. It is useful when you want to create an inline function. In Go language, an anonymous function can form a closure. An anonymous function is also known as function literal.

Go playground example: <https://go.dev/play/p/oRvkptTza2q>.

## Examples

### Example with pointer

```go
type Example struct {
	Name *string
}

func NewExampleLong() Example {
	name := "example"
	return Example{
		Name: &name,
	}
}

func NewExample() Example {
	return Example{
		// Anonymous function
		Name: func() *string {
			n := "example"
			return &n
		}(),
	}
}
```

This can be annoying when you have a lot of parameters, of course you can use pointer function libraries like <https://github.com/AlekSi/pointer>. But here, the goal is simply to use fewer variables to avoid confusion.

Go playground: <https://go.dev/play/p/EKd55TFq0Eb>.

### Example with if

```go
type Example struct {
	Name     *string
	Example2 string
}

func NewExample2Long(example bool) Example {
	if example {
		return Example{
			Example2: "if example",
			// Anonymous function
			Name: func() *string {
				n := "example"
				return &n
			}(),
		}
	}
	return Example{
		Example2: "else example",
		// Anonymous function
		Name: func() *string {
			n := "example"
			return &n
		}(),
	}
}

func NewExample2(example bool) Example {
	return Example{
		// Anonymous function
		Example2: func() string {
			if example {
				return "if example"
			}
			return "else example"
		}(),
		// Anonymous function
		Name: func() *string {
			n := "example"
			return &n
		}(),
	}
}
```

In that case, it's very useful and avoid copy past of your struct definition.

Go playground: <https://go.dev/play/p/AnxvcLhOBdN>.

### Example with a variable

```go
func main() {
	// Anonymous function
	example3 := func() {
		fmt.Println("Hi A Trick A Day !")
	}
	example3()
	example3()
	example3()
}
```

In Go language, you are allowed to assign an anonymous function to a variable. When you assign a function to a variable, then the type of the variable is of function type and you can call that variable like a function call as shown in the above example.

Go playground: <https://go.dev/play/p/RYPHKySpCEc>.

### Example with parameters

```go
func main() {
	example := "Hi A Trick A Day !"
	// Anonymous function
	func() {
		fmt.Println(example)
	}()
	// Anonymous function
	func(s string) {
		fmt.Println(s)
	}("Hi A Trick A Day !")
}
```

You can also pass arguments into the anonymous function.
Preferably reuse variables from your main function and use parameters when you have references like loop variables.

Go playground: <https://go.dev/play/p/B9LueTouvmc>.
