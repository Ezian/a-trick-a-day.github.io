---
title: Go & Context
date: 2022-08-18T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: Why it is interesting to propagate the context to lower layers.
thumbnail: "assets/images/thumbnail/2022-08-18-golang-go-&-context.jpg"
comments: true
---

One way to think of **context** in go is that it allows you to pass a "context" to your program. A context that **signals a time limit**, **deadline** or channel to **indicate when** to **stop working** and come back. For example, if you are **requesting a web server** or running a system command, it is usually a good idea to have a **timeout for production systems**. This is because if an API you depend on is running slowly, you wouldn't want to log **too many requests on your application**, as this could end up **increasing the load** and degrading the performance of all the requests you are serving. Like a waterfall effect. This is where a delay or delay context can come in handy.

```go
package main

import (
	"context"
	"fmt"
)

func doSomething(ctx context.Context) {
	fmt.Println("Doing something!")
}

func main() {
	ctx := context.TODO()
	doSomething(ctx)
}
```

Go playground: <https://go.dev/play/p/Kq7YyLxqgB3>.

{% include alerts/info.html content='Try to always put this parameter first in your function.' %}

{% include alerts/info.html content='The <strong>context.Background()</strong> function creates an empty context like <strong>context.TODO()</strong> does, but it is designed to be used when you intend to start a known context.<br/>Both functions do the same thing: they return an empty context that can be used like <strong>context.Background()</strong>. The difference is in the way you signal your intention to other developers. If you are not sure which function to use, <strong>context.Background()</strong> is a good default option.' %}

Another example, by **passing this context.Context** value into a function that then makes a **call to your DB**, the database **query** will also be **terminated** if it is still running when the **client disconnects**.

## Ending a Context

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func example(w http.ResponseWriter, req *http.Request) {
	ctx := req.Context()
	fmt.Println("start")
	defer fmt.Println("end")

	select {
	case <-time.After(10 * time.Second):
		fmt.Fprintf(w, "example\n")
	case <-ctx.Done():
		err := ctx.Err()
		fmt.Println("error:", err)
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func main() {
	http.HandleFunc("/", example)
	http.ListenAndServe(":8080", nil)
}
```

Go playground: <https://go.dev/play/p/3vneiEUIMfr>.

Output:

```bash
$ go run .
start
error: context canceled
end
```

This way **some of the libraries** you use or your own **can stop the actions** and **avoid overloading** your service.

## Share variables

You can also use the context to pass variables:

```go
package main

import (
	"context"
	"fmt"
)

func doSomething(ctx context.Context) {
	fmt.Printf("doSomething: myKey's value is %s\n", ctx.Value("myKey"))
}

func main() {
	ctx := context.Background()
	ctx = context.WithValue(ctx, "myKey", "myValue")
	doSomething(ctx)
}
```

Go playground: <https://go.dev/play/p/R1MhUD2v96Q>.

This is what is used a lot to **propagate the logger**. And many logging libraries will offer you a **handler to fill the logger** with information on each request, e.g. path, request ID etc. And **a function to retrieve this logger** from the context.

```go
package main

import (
	"context"
	"net/http"
	"os"
	"time"

	"github.com/justinas/alice"
	"github.com/rs/zerolog"
	"github.com/rs/zerolog/hlog"
	"github.com/rs/zerolog/log"
)

func main() {
	log := zerolog.New(os.Stdout).With().
		Timestamp().
		Str("role", "my-example").
		Logger()

	c := alice.New()

	// Install the logger handler with default output on the console
	c = c.Append(hlog.NewHandler(log))

	// Install some provided extra handler to set some request's context fields.
	// Thanks to that handler, all our logs will come with some prepopulated fields.
	c = c.Append(hlog.AccessHandler(func(r *http.Request, status, size int, duration time.Duration) {
		hlog.FromRequest(r).Info().
			Str("method", r.Method).
			Stringer("url", r.URL).
			Int("status", status).
			Int("size", size).
			Dur("duration", duration).
			Msg("")
		getDB(r.Context())
	}))
	c = c.Append(hlog.RemoteAddrHandler("ip"))
	c = c.Append(hlog.UserAgentHandler("user_agent"))
	c = c.Append(hlog.RefererHandler("referer"))
	c = c.Append(hlog.RequestIDHandler("req_id", "Request-Id"))

	// Here is your final handler
	h := c.Then(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Get the logger from the request's context. You can safely assume it
		// will be always there: if the handler is removed, hlog.FromRequest
		// will return a no-op logger.
		hlog.FromRequest(r).Info().Msg("Hello this is my example")
	}))
	http.Handle("/", h)

	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal().Err(err).Msg("Startup failed")
	}
}

func getDB(ctx context.Context) {
	log.Ctx(ctx).Info().Msg("get from db")
}
```

Go playground: <https://go.dev/play/p/hUrDLZVGxd2>.

Ouput:

```json
{"level":"info","role":"my-example","ip":"127.0.0.1:54224","user_agent":"curl/7.68.0","req_id":"cbuk241ludm8ge0akv80","time":"2022-08-17T21:27:12+02:00","message":"Hello this is my example"}
{"level":"info","role":"my-example","ip":"127.0.0.1:54224","user_agent":"curl/7.68.0","req_id":"cbuk241ludm8ge0akv80","method":"HEAD","url":"/","status":0,"size":0,"duration":0.2451,"time":"2022-08-17T21:27:12+02:00"}
{"level":"info","role":"my-example","ip":"127.0.0.1:54224","user_agent":"curl/7.68.0","req_id":"cbuk241ludm8ge0akv80","time":"2022-08-17T21:27:12+02:00","message":"get from db"}
```

As you can see even up to **getDB** you **get the logger** and its **information**.

{% include alerts/info.html content='Very useful for tracing to follow the logs of a request and debug more easily.' %}

## References

<https://www.digitalocean.com/community/tutorials/how-to-use-contexts-in-go>

<https://gobyexample.com/context>
