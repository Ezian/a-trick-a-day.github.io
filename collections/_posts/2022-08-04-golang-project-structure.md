---
title: "Project structure"
date: 2022-08-04T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: By the end of this guide, you will be all about the Go project structure.
thumbnail: "assets/images/thumbnail/2022-08-04-golang-project-structure.jpg"
comments: true
---

## Packages

A package is made of **.go** files that are in the same directory and have the same package declaration at the beginning of the file (except for test files), this is the entry point of the Go code.

- Best Practice to use multiple files
  - Feel free to separate your code into as many files as possible
  - Make sure it is easy to navigate
  - Freely couple the sections of the service or application
- Keep types close
  - It is often a good practice to keep the main types grouped together at the top of a file. Or in a global package file called **types.go** ([example here](https://github.com/golang/crypto/blob/master/acme/types.go))
- Organise by responsibility
  - In other languages, we organize types by patterns or types, but in go, we organize the code by their functional responsibilities
- Start to [Use Godoc](https://pkg.go.dev/golang.org/x/tools/cmd/godoc)
  - Godoc extracts and generates documentation for Go programs. It works as a web server and presents the documentation as a web page
- Do not write your business logic in **main.go**

## Naming Convention

- Package names should be lowercase. Don't use kebab-case or camelCase and try to _avoid_ snake_case.
- _Avoid_ overly use terms like util, common, script etc
- Rename should follow the same rules:

  ```go
  import (
  	gotypes "go/types"
  	apitypes "myApp/api/types"
  )
  ```

## Common package and file

- **/cmd**

  This folder contains the **main files** of the application's entry point for the project, the name of the directory corresponding to the name of the binary.

- **/external**

  This folder contains **code that can be used by other** services. These may be API clients or utility functions that may be useful to other projects but do not warrant their own project.

- **/internal**

  This package contains the **private library code** used in your service, it is specific to the service function and is not shared with other services. One thing to note is that this privacy is enforced by the compiler itself, see [the Go 1.4 release notes for more details](https://go.dev/doc/go1.4#internalpackages).

- **go.mod**

  The **go.mod** file defines the **module path**, which is also the import path used for the root directory, and its **dependency requirements**, which are the other modules needed for a successful build.

- **go.sum**

  The **go.sum** file contains all the **checksums of the dependencies**, and is managed by the go tools. The checksum in the go.sum file is used to validate the checksum of each of the direct and indirect dependencies to confirm that none of them have been modified.

## Exemple of a project structure

```tree
├── cmd # folder with your binaries
│   └── my-api
│   │   └── main.go
│   └── my-worker-1
│       └── main.go
├── external
│   └── myappclient
│       └── client.go
├── internal
│   ├── api/ # folder with your controllers
│   ├── db/ # folder with your db logic
│   │   ├── ...
│   │   ├── db.go
│   │   └── db_test.go
│   └── transform/ # folder with the transforms (example db to api)
├─── Dockerfile
├─── go.mod
├─── go.sum
└─── README.md
```

You can also find a community layout there: <https://github.com/golang-standards/project-layout>
