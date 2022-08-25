---
title: "Error"
date: 2022-08-25T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: You will find the basics on errors and how to build your own.
thumbnail: "assets/images/thumbnail/2022-08-25-golang-error.svg"
comments: true
---

It can be interesting to **create its own errors**, for example to **contain more information** like an error code, for example a **StatusCode** to **be returned** in case of error so that the **HTTP server** returns this Code.

## New Error Type

In this section we will see how to **retype** our **error** to retrieve **its information**:

```go
package main

import (
	"errors"
	"fmt"
)

type RequestError struct {
	StatusCode int

	Err error
}

func (r *RequestError) Error() string {
	return fmt.Sprintf("%d - %s", r.StatusCode, r.Err)
}

func example(a int) error {
	switch a {
	case 1:
		return errors.New("example")
	default:
		return &RequestError{
			StatusCode: 503,
			Err:        errors.New("unavailable"),
		}
	}
}

func main() {
	err := example(2)
	if err != nil {
		var requestErr *RequestError
		switch {
		// usually used when you have one type of error
		case errors.As(err, &requestErr):
			fmt.Printf("%d: %s\n", requestErr.StatusCode, requestErr)
		default:
			fmt.Printf("unexpected error: %s\n", err)
		}
	}
	// or in case if you have multiple error types
	err = example(2)
	if err != nil {
		switch tErr := err.(type) {
		case *RequestError:
			fmt.Printf("%d: %s\n", tErr.StatusCode, tErr)
		default:
			fmt.Printf("unexpected error: %s\n", err)
		}
	}
}
```

Go playground: <https://go.dev/play/p/u-MkOyJ9L2v>.

Output:

```bash
503: 503 - unavailable
503: 503 - unavailable
```

To create your **custom error**, you will just have to create a **structure** that has the `Error()` method.

Prefer the **switch case** with **type** when you have **several types of errors** to manage.

As you can see we can **get the informations** from **our error**. In an REST API case we could use `StatusCode` and **return this HTTP status** to return the **correct status** related to the error.

{% include alerts/info.html content='When necessary, you can also customize the behavior of the <strong>errors.Is</strong> and <strong>errors.As</strong>. See <a href="https://go.dev/blog/go1.13-errors">this Go.dev blog</a> for an example.' %}

## Compare Error

`Is` allows us to **compare** **errors** to see if they are **equal**.
This function can be **redefined** in our custom error, in order to create our **own equality** rule:

```go
package main

import (
	"errors"
	"fmt"
)

type DBError struct {
	Code    int
	Message string
}

func (r *DBError) Error() string {
	return fmt.Sprintf("%d - %s", r.Code, r.Message)
}

func (r *DBError) Is(target error) bool {
	t, ok := target.(*DBError)
	if !ok {
		return false
	}

	return (r.Code == t.Code || t.Code == 0) &&
		(r.Message == t.Message || t.Message == "")
}

// predfined errors
var UnavailableDBError = &DBError{
	Code:    503,
	Message: "unavailable",
}

func doRequest() error {
	return UnavailableDBError
}

func main() {
	if err := doRequest(); err != nil {
		if errors.Is(err, UnavailableDBError) {
			fmt.Println("db result is unavailable")
		}
	}
}
```

Go playground: <https://go.dev/play/p/kubN4AwNems>.

Output:

```bash
db result is unavailable
```

You can often **find this usage** with **DB libraries** that will return `no rows` errors and that **you can compare** to **handle the error** your way:

```go
// with go-pg
if err := db.Model(&unit).Where("id = ?", 200).Select(); err != nil {
	if errors.Is(err, pg.ErrNoRows) {
		// handle record not found
	}
}
// with gorm
if err := userHandler.db.Where("email = ?", email).First(&user).Error; err != nil {
	if errors.Is(err, gorm.ErrRecordNotFound) {
		// handle record not found
	}
}
```

{% include alerts/info.html content='<strong>Preferred this</strong> solution to <strong>==</strong> (You will see <a href="#wrapping-error">in the next chapter</a>, with error wrapping you will not be able to use <strong>==</strong>).' %}

## Wrapping Error

Wrapping **errors** in Go means **adding extra context information** to the **returned error** like the name of the function where the error occurred, the cause, the type, etc. This technique is most commonly used to create **clear error messages**, which are especially useful for **debugging** when you want quickly and precisely **locate the source of problems**.

```go
package main

import (
	"errors"
	"fmt"
)

type DBError struct {
	Err error
}

func (r *DBError) Error() string {
	return fmt.Sprintf("this is an error of DBError - %s", r.Err)
}

func (r *DBError) Unwrap() error {
	return r.Err
}

func (r *DBError) Is(target error) bool {
	t, ok := target.(*DBError)
	if !ok {
		return false
	}
	return (r.Err == t.Err || t.Err == nil)
}

var exampleErr = errors.New("example")

func doRequest() error {
	return &DBError{
		Err: exampleErr,
	}
}

func main() {
	err := doRequest()
	err = fmt.Errorf("This is my wrapped error: %w", err)
	fmt.Printf("%s", err)
	fmt.Println()
	fmt.Printf("%s", errors.Unwrap(err))
	fmt.Println()
	fmt.Printf("%s", errors.Unwrap(errors.Unwrap(err)))
	fmt.Println()
	fmt.Printf("%s", errors.Unwrap(errors.Unwrap(errors.Unwrap(err))))
	fmt.Println()
	// Wrapping an error makes it available to errors.Is and errors.As:
	fmt.Printf("%v", errors.Is(err, exampleErr))
}
```

Go playground: <https://go.dev/play/p/-sVhNPRAxD9>.

Output:

```bash
This is my wrapped error: this is an error of DBError - example
this is an error of DBError - example
example
%!s(<nil>)
true
```

As you can see you can **create your own error** and define `Unwrap` method or use `fmt.Errorf("...%w...",..., err)` to wrap your error and thus **improve the error message** while **keeping** the **parent error**.

You'll also notice that `errors.Is` and `errors.As` **work** with **wrapped errors** so you can **compare** or **set** them more **easily**. This is why you should **avoid** using `==` to compare 2 errors.

{% include alerts/info.html content='Note that if you <strong>Unwrap</strong> an <strong>non-wrapped</strong> error you will get a <strong>nil</strong> error.' %}

## References

<https://go.dev/blog/go1.13-errors>

<https://earthly.dev/blog/golang-errors/>
