---
title: "Overview of the Go tooling"
date: 2022-08-08T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Tool", "CI"]
description: I'll show you some interesting Go tools that you can integrate into your project.
thumbnail: "assets/images/thumbnail/2022-08-08-golang-overview-of-the-go-tooling.png"
comments: true
---

## Go CLI

I found a very complete post about the tooling of the **go** cli here <https://www.alexedwards.net/blog/an-overview-of-go-tooling>.
If you want to have more details I advise you to check it out.
I am going to make a reminder of the comands that I use and recommend the most.

I wrote this post at the version: **1.18.5**

### Installing Go Packages

Update from the blog I quoted above, **go get** is now deprecated to **install packages locally**, <https://go.dev/doc/go-get-install-deprecation>.

Prefer to use **go install** :

```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint
golangci-lint -v
```

### Optimize your binary

In this part, I will show you how to optimize the size of your binary easily.

In this case, I will use a simple example:

```go
package main

import "fmt"

func main() {
	fmt.Println("optimize binary")
}
```

I get these 2 results when I compile my application:

```bash
-rwxr-xr-x 1 ... ... 1762620 août   5 15:08 app1
-rwxr-xr-x 1 ... ... 1183744 août   5 15:09 app2
```

As you can see app2 is **~30% lighter**!

- **app1** is built using:

  ```bash
  go build -o app1 .
  ```

- **app2** is built using optimizations:

  ```bash
  CGO_ENABLED=0 go build -a -ldflags "-s -w" -o app2 .
  ```

  - **CGO_ENABLED=0**, you got a [staticaly-linked binary](https://en.wikipedia.org/wiki/Static_buildd) that will work without any external dependencies (you can use **lighter docker images** like [scratch](https://hub.docker.com/_/scratch)).

  - **-ldflags "-s -w"**, you can use the [-s and -w linker](https://golang.org/cmd/link/) flags to strip the debugging information:

  ```bash
  $ go tool link
  ...
  -s  disable symbol table
  ...
  -w  disable DWARF generation
  ...
  ```

{% include alerts/green.html content='This is a must-have to put inside your <strong>CI</strong>!<br/>If you want to go deeper to optimize your binary check <a href="https://upx.github.io/">upx</a>.' %}

### Testing deeper

When handling goroutines quite a bit, feel free to run the tests with the Go **race detector** enabled, which can help detect some of the data races that can occur in real life. For example:

```bash
go test -race ./...
```

### Test Coverage Report

Another interesting point is code coverage. You can enable **coverage analysis** when running tests by using the **-coverprofile** option. This will generate a **coverage report** that can be used by Sonarqube or others, like this:

```bash
go test -coverprofile=profile.out ./...
```

{% include alerts/info.html content="This is a must-have to run it in your <strong>CI</strong> especially if you have a code scanner that accepts this type of report!" %}

You can also view it in your web browser to see the **coverage per file in detail**, using the **go tool cover -html** command as follows:

```bash
go tool cover -html=profile.out
```

### Managing your Dependencies

Before making any changes to your code, I recommend that you run the following commands to **update** your dependencies and get them in a **tidy** state:

```bash
go get -u ./...
go mod tidy
```

{% include alerts/info.html content="When you start a development try to always keep your project up to date." %}

The **go get -u ./...** command will **update all dependencies** at once for a given module, just run the following from the root directory of your module.

The **go mod tidy** command will **remove all unused dependencies** from your **go.mod** and **go.sum** files, and update the files to include dependencies for all possible combinations of build/OS/architecture tags (note: go run, go test, go build etc. are 'lazy' and will only fetch the packages needed for the current build/OS/architecture tags). By doing this before each commit, it will be easier to determine which changes to your code are responsible for adding or removing which dependencies when you look at the version control history.

{% include alerts/info.html content="<strong>go mod tidy</strong> is a must-have to run and compare the changes in your <strong>CI</strong>!<br/>
You can check the differences using <strong>git diff --exit-code</strong>." %}

## [Golangci-Lint](https://golangci-lint.run/)

**Golangci-lint** is a **linting aggregator** for Go that runs **linters in parallel**, reuses Go's build cache, and caches analysis results to greatly improve performance on subsequent runs.

This is the best way to implement linting in Go projects.

![golangci-lint example](https://golangci-lint.run/demo.svg)

**Golangci-lint** is [configurable via yaml](https://golangci-lint.run/usage/configuration/) which can allow you to **version your rules**. Try to use as **many linters as possible**, especially if you are **starting in Go**. This will help you to understand the good practices of this language.

{% include alerts/info.html content="This is a must-have to install in your <strong>CI</strong> !" %}

{% include alerts/info.html content='If you want to enable it in your vscode, check <a href="https://golangci-lint.run/usage/integrations/#editor-integration">this documentation</a>.' %}

## References

Golangci-lint website: <https://golangci-lint.run/><br/>
An Overview of Go's Tooling: <https://www.alexedwards.net/blog/an-overview-of-go-tooling><br/>
A go tooling cheat sheet: <https://github.com/fedir/go-tooling-cheat-sheet/blob/master/go-tooling-cheat-sheet.pdf>
