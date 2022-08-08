---
title: "Application & Containers"
date: 2022-08-10T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Container", "CI"]
description: How to optimize the containerization of your application.
thumbnail: "assets/images/thumbnail/2022-08-10-container-application-&-containers.jpg"
comments: true
---

{% include alerts/green.html content='<strong>Optimizing your docker images</strong> to avoid importing the <strong>whole "planet"</strong> can be a good thing.' %}

In this post I will take the example of a Go application but it can be well applied to other languages.

## [Scratch](https://hub.docker.com/_/scratch)

**Scratch** is a reserved image of **size 0** with nothing in it.
Using a scratch image **reduces the size** of a final Docker image by **~50%** compare to an [alpine image](https://hub.docker.com/_/alpine)!
Not bad, right?

Except that the image can't do SSL certificate verification because of the **missing SSL certificates**!

Writing a simple Go application that request <https://google.com>, and package it inside **scratch**:

```Dockerfile
FROM golang:1.19-alpine as builder

WORKDIR /go/src/app
RUN go mod init example
RUN echo -e 'package main\n\
import "net/http"\n\
func main() {\n\
	if _, err := http.Get("https://google.com"); err != nil {\n\
		panic(err)\n\
	}\n\
}' > main.go

RUN CGO_ENABLED=0 go build -a -ldflags "-s -w" -o /go/bin/app

FROM scratch
COPY --from=builder /go/bin/app /
CMD ["/app"]
```

Go playground: <https://go.dev/play/p/U9_V7aoIiaZ>.

```console
panic: Get "https://google.com": x509: certificate signed by unknown authority

goroutine 1 [running]:
main.main()
        /go/src/app/main.go:5 +0x4c
```

As you can see, it is impossible to request an https URL.
You'll have to do this to make it work:

```Dockerfile
FROM alpine:3.16 as builder-cert

RUN apk add --no-cache ca-certificates

FROM golang:1.19-alpine as builder-go

WORKDIR /go/src/app
RUN go mod init example
RUN echo -e 'package main\n\
import "net/http"\n\
func main() {\n\
	if _, err := http.Get("https://google.com"); err != nil {\n\
		panic(err)\n\
	}\n\
}' > main.go

RUN CGO_ENABLED=0 go build -a -ldflags "-s -w" -o /go/bin/app

FROM scratch
# import the certs to make them available in scratch
COPY --from=builder-cert /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder-go /go/bin/app /
CMD ["/app"]
```

Personally, you can opt for **scratch** if you don't need the SSL certificates.
But I think it's **better** to use a ~2 MiB **distroless images**, which we'll see now.

## Distroless

**Distroless** images contain only your **application and its runtime dependencies**. They do not contain package managers, shells or other programs that you would expect to find in a standard Linux distribution.

**Limiting the contents** of your runtime container to precisely what is needed for your application is a **good practice** used. It **improves the signal-to-noise** ratio of scanners (e.g. CVE) and reduces the burden of establishing provenance to what you need.

{% include alerts/green.html content='<strong>Distroless images</strong> contain only the minimum requirements for an application so they are very small.' %}

### [Google Images](https://github.com/GoogleContainerTools/distroless)

The smallest distroless image, [gcr.io/distroless/static-debian11](https://github.com/GoogleContainerTools/distroless/blob/main/base/README.md), is about 2 MiB. This is about **50% of the size of Alpine (~5 MiB)**, and **less than 2% of the size of Debian (124 MiB)**.

Dockerfie example, you just need to copy your binary:

```Dockerfile
FROM golang:1.19-alpine as builder

WORKDIR /go/src/app
RUN go mod init example
RUN echo -e 'package main\n\
import "net/http"\n\
func main() {\n\
	if _, err := http.Get("https://google.com"); err != nil {\n\
		panic(err)\n\
	}\n\
}' > main.go

RUN CGO_ENABLED=0 go build -a -ldflags "-s -w" -o /go/bin/app

FROM gcr.io/distroless/static-debian11
COPY --from=builder /go/bin/app /
CMD ["/app"]
```

## Results

Overview of different sizes you can get depending on the base images:

```console
$ docker image ls
test-alpine        latest      fae3b8af8e1b   3 seconds ago        10.1MB # alpine image
test-distroless    latest      98117ef14761   About a minute ago   6.93MB # distroless
test-scratch-certs latest      b015a7cd8a59   14 minutes ago       4.78MB # scrtach with certs
test-scratch       latest      2aee7e47308f   About an hour ago    4.57MB # scratch
```
