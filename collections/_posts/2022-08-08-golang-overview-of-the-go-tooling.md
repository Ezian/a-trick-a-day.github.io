---
title: "Overview of the Go tooling"
date: 2022-08-08T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Tool"]
description: By the end of this guide, you will be all about the Go tools.
thumbnail: "assets/images/thumbnail/2022-08-08-golang-overview-of-the-go-tooling.png"
comments: true
---

## Go CLI

Guide using **go version go1.18.3 linux/amd64**.

### Running Code

During development, the **go run** tool is a convenient way to test your code. It is essentially a shortcut that **compiles** your code, creates an executable binary in your /tmp directory, and then **runs** that binary in one step.

```bash
go run .            # Run the package in the current directory
go run ./cmd/myapi  # Run the package in the ./cmd/myapi directory
```

## Using Compiler and Linker Flags

To compile a **main** package and create an **executable binary**, you can use the **go build** tool. You can use it in conjunction with the **-o** flag, which allows you to explicitly set the output directory and binary name, as follows:

```bash
go build -o=foo .         # Build the package in the current directory
go build -o=foo ./cmd/foo # Build the package in the ./cmd/foo directory
```

In these examples, **go build** will compile the specified package (and all dependent packages), and send it to **./foo**.

You may also be interested in using the **-s** and **-w** flags to **remove debugging information** from the binary. This usually **reduces** the final **size** by about 25%. For example:

```bash
go build -ldflags="-s -w" -o=foo .  # Remove debugging information from the binary
```

### Cleaning your $GO_PATH

The **downloaded dependencies** are stored in the module cache located at **$GOPATH/pkg/mod**. If you need to **clear the module cache**, you can use the **go clean** tool. But beware: this will remove the downloaded dependencies for all projects on your machine.

```bash
go clean -modcache
```

### [View documentation](https://pkg.go.dev/golang.org/x/tools/cmd/godoc)

You can check the documentation of the clone project using the **go doc** tool.
And you can also run it on your own project to check if everything exposed is well documented.
For online packages, you can browse <https://golang.org/pkg>, which is generated from the github package doc.

```bash
go doc strings
go doc -http=:6060 # Run http server to expose the doc
```

### Upgrade to a new version of Go

The **go fix** tool was originally released in 2011 (when regular changes were still being made to the Go API) to help users automatically update their older code to be compatible with the latest version of Go. Since then, [Go's promise of compatibility](https://golang.org/doc/go1compat) means that if you upgrade from a 1.x version of Go to a newer 1.x version, everything should work and using **go fix** should generally be unnecessary.

However, there are a handful of very specific problems that it addresses. You can see a summary of these by running **go tool fix -help**. If you decide that you want or need to use **go fix** after the upgrade, you should run the following command, then inspect a diff of the changes before committing them.

```bash
go fix ./...
```

### Formating and Refactoring Code

You are probably familiar with using the **gofmt** tool to automatically **format your code**. But it also supports _rewrite rules_ that you can use to help **refactor your code**. I'll give you a demonstration.

Let's say you have the following code and you want to change the variable foo to Foo so that it is exported:

```go
var foo int

func main() {
	foo = 1
	fmt.Println("foo")
}
```

To do this, you can use **gofmt** with the **-r** option to implement a rewrite rule, the **-d** option to display a differential of the changes, and the **-w** option to make the changes in place, like this:

```bash
$ gofmt -d -w -r 'foo -> Foo' .
-var foo int
+var Foo int

 func bar() {
-	foo = 1
+	Foo = 1
 	fmt.Println("foo")
 }
```

Notice how this is smarter than a search and replace? The variable **foo** has been changed, but the string **"foo"** in the **fmt.Println()** statement has remained unchanged. Another thing to note is that the **gofmt** command works recursively, so the above command will be executed on all **\*.go** files in your current directory and subdirectories.

### Installing GO Packages

TODO

<https://go.dev/doc/go-get-install-deprecation>

```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint
golangci-lint -v
```

### Retrieving Dependencies

When you use **go run** (or **go test** or **go build**), all external dependencies will be automatically (and recursively) downloaded to fill in the import statements in your code. By default, the latest tagged version of the dependency will be downloaded, or if no tagged version is available, then the dependency at last commit.

If you know in advance that you need a specific version of a dependency (instead of the one Go retrieves by default), you can use **go get** with the corresponding version number or commit hash. For example:

```bash
go get github.com/uptrace/bun@v1.1.7
go get github.com/uptrace/bun@f2f4149
```

### Testing

You can use the go test tool to run tests in your project as follows:

```bash
go test .          # Run all tests in the current directory
go test ./...      # Run all tests in the current directory and sub-directories
go test ./foo/bar  # Run all tests in the ./foo/bar directory
```

You can also run the tests with the Go race detector enabled, which can help detect some of the data races that can occur in real life. For example:

```bash
go test -race ./...
```

It is important to note that enabling the run detector will increase the overall execution time of your tests.

### Profiling Test Coverage

You can enable coverage analysis when running tests by using the **-cover** option. This will display the percentage of code covered by the tests in the output for each package, like this:

```bash
$ go test -cover ./...
ok  	github.com/thecampagnards/sql-to-go	0.467s	coverage: 78.6% of statements
```

You can also generate a coverage profile using the **-coverprofile** flag and view it in your web browser using the **go tool cover -html** command as follows:

```bash
go test -coverprofile=profile.out ./...
go tool cover -html=profile.out
```

### Tidying your Dependencies

Before making any changes to your code, I recommend that you run the following command to tidy up your dependencies:

```bash
go mod tidy
```

The **go mod tidy** command will remove all unused dependencies from your **go.mod** and **go.sum** files, and update the files to include dependencies for all possible combinations of build/OS/architecture tags (note: go run, go test, go build etc. are 'lazy' and will only fetch the packages needed for the current build/OS/architecture tags). By doing this before each commit, it will be easier to determine which changes to your code are responsible for adding or removing which dependencies when you look at the version control history.

You can also add it in the controls of your **CI** to certify that **your dependencies are always clean**.

## [GolangCI Lint](https://golangci-lint.run/)

Golangci-lint is a linting aggregator for Go that runs linters in parallel, reuses Go's build cache, and caches analysis results to dramatically improve performance on subsequent runs.

It is the preferred way to set up linting in Go projects.

```go
import "os"

func main() {}
```

```bash
$ golangci-lint run ./demo.go
demo.go:4:8: "os" imported but not used (typecheck)
import "os"
```

Golangci-lint is [configurable via yaml](https://golangci-lint.run/usage/configuration/) which can allow you to version your rules.

It is a must to install in your **CI**

## References

An Overview of Go's Tooling: <https://www.alexedwards.net/blog/an-overview-of-go-tooling>

A go tooling cheat sheet: <https://github.com/fedir/go-tooling-cheat-sheet/blob/master/go-tooling-cheat-sheet.pdf>
