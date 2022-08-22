---
title: "Build your Own Error"
date: 2022-08-24T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: TODO.
thumbnail: "assets/images/thumbnail/2022-08-24-golang-build-your-own-error.jpg"
comments: true
---

## As

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

func doRequest() error {
	return &RequestError{
		StatusCode: 503,
		Err:        errors.New("unavailable"),
	}
}

func main() {
	err := doRequest()
	if err != nil {
		var requestErr *RequestError
		switch {
		case errors.As(err, &requestErr):
			fmt.Printf("%d: %s\n", requestErr.StatusCode, requestErr)
		default:
			fmt.Printf("unexpected error: %s\n", err)
		}
	}
}
```

Go playground: <https://go.dev/play/p/M-TnEuRxAdJ>.

Output:

```bash
503: 503 - unavailable
```

{% include alerts/info.html content='When necessary, you can also customize the behavior of the <strong>errors.Is</strong> and <strong>errors.As</strong>. See <a href="https://go.dev/blog/go1.13-errors">this Go.dev blog</a> for an example.' %}

## Is

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
	if errors.Is(doRequest(), UnavailableDBError) {
		fmt.Println("db result is unavailable")
	}
}
```

Go playground: <https://go.dev/play/p/4AuD-BActHe>.

Output:

```bash
db result is unavailable
```

## Unwrap

## References

<https://earthly.dev/blog/golang-errors/>
