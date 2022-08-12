---
title: "func VS var f = func & Mocking"
date: 2022-08-16T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: Declaration vs literals function and how to optimize your tests.
thumbnail: "assets/images/thumbnail/2022-08-16-golang-func-vs-var-func-&-mocking.jpg"
comments: true
---

I have been asked a lot about how to **test certain cases** for certain functions.

The first solution can be to **split in more functions** and the other can be to **change your functions in literals**.

In Golang, you can use **2 types of function definitions**:

- A [function declaration](https://golang.org/ref/spec#Function_declarations) **binds** an identifier, the _function name_, to a function. So the function name will be an [identifier](https://golang.org/ref/spec#Identifiers) which **you can refer to**.

  ```go
  func do() {}
  ```

- A [function literals](https://golang.org/ref/spec#Function_literals) represents an anonymous function. Function literals are closures, they **capture the surrounding environment**: they may refer to variables defined in a surrounding function. Those variables are then shared between the surrounding function and the function literal, and they survive as long as they are accessible.

  ```go
  var do = func() {}
  ```

## Function Declaration

Here is a basic example of how to define a function declaration:

```go
package main

func do() {}

func main() {
	go do()
}
```

Go playground: <https://go.dev/play/p/mwxVp3BODjj>.

## Function Literals

Let's take the previous example and switch it to literals mode:

```go
package main

import "fmt"

var do  = func() { fmt.Println("Doing...") }

func main() {
	do()
	do = func() { fmt.Println("Not doing!") } // be careful to this do it only in tests
	do()
}

```

Go playground: <https://go.dev/play/p/UDsQfjJ4B3F>.

Output of the above example:

```console
Doing...
Not doing!
```

As you can see you can **assign a declarative** function **to a literal** function and **use** it **in your tested function** and **change** it **in your tests**.

{% include alerts/warning.html content='Be careful as you can see you can redefine your functions which is to be avoided!' %}

This writing is very useful to **mock functions in tests**, that's what we'll see next.

## Mocking functions

### Test incomming parameters

If you want to go further in your tests and **test the input parameters** of the **functions called** by the tested function you can do like this.

**main.go**:

```go
package main

import (
	"fmt"
	"os"
)

var osExit = os.Exit

func MyFunc() {
	fmt.Println("https://a-trick-a-day.github.io")
	osExit(1) // I want to test it
}
```

**main_test.go**:

```go
package main

import "testing"

func TestMyFunc(t *testing.T) {
	// Save current function and restore at the end:
	oldOsExit := osExit
	resetMock := func() { osExit = oldOsExit }
	defer resetMock()

	t.Run("success", func(t *testing.T) {
		// restore at each test to be sure
		resetMock()
		var got int
		osExit = func(code int) {
			got = code
		}

		MyFunc()
		if exp := 1; got != exp {
			t.Errorf("Expected exit code: %d, got: %d", exp, got)
		}
	})
}
```

```console
https://a-trick-a-day.github.io
PASS
coverage: 100.0% of statements
ok      foo        0.002s
```

{% include alerts/info.html content='This can be interesting especially if you have calculations on these parameters.' %}

### Force output parameters

This trick can also be used to **mock the returns of a function** and see how your function handle it.

**main.go**:

```go
package main

// this function can be in the same package or in another one
var MyFuncFromExternalPackage = func() error {
	return nil
}

func MyFunc() {
	if err := MyFuncFromExternalPackage(); err != nil {
		panic(err)
	}
}
```

**main_test.go**:

```go
package main

import (
	"errors"
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestMyFunc(t *testing.T) {
	// Save current function and restore at the end:
	oldMyFuncFromExternalPackage := MyFuncFromExternalPackage
	resetMock := func() { MyFuncFromExternalPackage = oldMyFuncFromExternalPackage }
	defer resetMock()

	t.Run("error_panic", func(t *testing.T) {
		// restore at each test to be sure
		resetMock()
		MyFuncFromExternalPackage = func() error {
			return errors.New("test")
		}

		assert.Panics(t, MyFunc)
	})
}
```

{% include alerts/info.html content='It can be useful to make your tests test only what you have coded in your function. And to avoid having like a waterfall on your test errors.' %}

## References

When to use function expression rather than function declaration in Go?: <https://stackoverflow.com/questions/46323067/when-to-use-function-expression-rather-than-function-declaration-in-go>

Testing os.Exit scenarios in Go with coverage information (coveralls.io/Goveralls): <https://stackoverflow.com/questions/40615641/testing-os-exit-scenarios-in-go-with-coverage-information-coveralls-io-goverall/40801733#40801733>
