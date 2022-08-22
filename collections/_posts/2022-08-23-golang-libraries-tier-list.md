---
title: "Top List of Libraries for CRUD"
date: 2022-08-23T07:49:03+01:00
layout: post
authors: ["Sidorenko Konstantin"]
categories: ["Golang", "Development"]
description: You start a new CRUD project but you don't know which library to choose, you will find here the top libraries.
thumbnail: "assets/images/thumbnail/2022-08-23-golang-libraries-tier-list.png"
comments: true
---

This post was written on {{ page.date | date: "%-d %B %Y"}}, the libraries can change, be in maintenance mode etc.

When you choose a library in Go, **pay attention to several points**:

- look if the library is in **maintenance mode**
- look at the number of **stars**
- look at the **date** of the **last commit**

With these points you will have **more or less** an opinion on the **reliability** of this library.

## <ins>HTTP Server</ins>

### Generator

### [go-swagger](https://github.com/go-swagger/go-swagger)

Brings to the go community a complete suite of fully-featured, high-performance, API components to work with a Swagger API: **server**, **client** and **data model**.

- Generates a **server** from a swagger specification
- Generates a **client** from a swagger specification
- Generates a **CLI** (command line tool) from a swagger specification (alpha stage)
- Supports most **features** offered by **jsonschema** and **swagger**, including polymorphism
- Generates a **swagger specification** from annotated go code
- Additional tools to work with a **swagger spec**
- Great customization features, with vendor extensions and customizable templates

The **go-swagger** focus with code generation is to produce **idiomatic**, **fast go code**, which plays **nice with golint**, go vet etc.

It will also do the **parameter checks** for you, such as **enums**, **minimum for numbers** etc.

{% include alerts/warning.html content='It works <strong>only</strong> with <strong>Swagger 2.0</strong>.' %}

{% include alerts/info.html content='This is a must have if you have a <strong>Swagger 2.0</strong> specification to <strong>generate</strong> your <strong>client</strong> or <strong>server</strong>.' %}

### [oapi-codegen](https://github.com/deepmap/oapi-codegen)

Is a code generator for **OpenAPI 3.0**, it can **generate** **server**, **client**, **models** etc.
For the **server generator** it can generate code for **[echo](#echo)**, **chi**, **[Gin](#Gin)** and **net/http**.

{% include alerts/info.html content='This is a must have if you have an <strong>OpenAPI 3.0</strong> specification to <strong>generate</strong> your <strong>client</strong> or <strong>server</strong>.' %}

### Manual Coding

### [fiber](https://github.com/gofiber/fiber)

Is an **Express inspired** web framework built on **top of [Fasthttp](https://github.com/valyala/fasthttp)**, the **fastest HTTP engine** for Go.
Designed to ease things up for fast development with **zero memory allocation** and **performance in mind**.

{% include alerts/info.html content='Focus on <strong>performance</strong> and manage <strong>JSON</strong>, <strong>JWT</strong> etc.<br/>With this, your <strong>APIs will rox</strong>!' %}

### [echo](https://github.com/labstack/echo)

**Simplifies the creation** of web applications and restful APIs. If you have Python experience, you may find the framework similar to Flask.

With the **rise of [fiber](#fiber)** which does almost the same thing with better performance, **echo** has a **slight drop in popularity**.
But it's still a **classic** in Go.

{% include alerts/info.html content='Echo handles <strong>JSON</strong>, <strong>JWT</strong> etc.<br/>Everything you need to build a REST API.' %}

### [gin](https://github.com/gin-gonic/gin)

Is a web framework written in Go. It features a [martini-like](https://github.com/go-martini/martini) API with performance that is up to 40 times faster thanks to [httprouter](https://github.com/julienschmidt/httprouter).

With the **rise of [fiber](#fiber)** which does almost the same thing with better performance, **gin** has a **slight drop in popularity**.
But it's still a **classic** in Go.

## <ins>Database</ins>

## Postgres

### Generator

### [sql-to-go](https://github.com/thecampagnards/sql-to-go)

**Generates** Go **structures** and **functions** based on an **SQL files** containing **create tables** and **alter**.

{% include alerts/info.html content='Can be useful to <strong>generate your models</strong> if you have to <strong>write your SQL scripts</strong>.' %}

### Manual Coding

### [bun](https://bun.uptrace.dev/)

Is a **lightweight** Go **ORM** for PostgreSQL, MySQL, MSSQL, and SQLite.
Bun is a rewrite of [go-pg](https://github.com/go-pg/pg).

**[go-pg](https://github.com/go-pg/pg)** is still **maintained** and there is **no urgency** in **rewriting go-pg** apps in Bun, but **new projects** should **prefer Bun** over go-pg. And once you are familiar with the updated API, you should be able to migrate a 80-100k lines go-pg app to Bun within a single day.

It is usually **easier** to **write complex queries** with Bun rather **than [GORM](https://github.com/go-gorm/gorm)**. Out of the box, Bun has **better integration** with **database-specific functionality**, for example, PostgreSQL **arrays**. Bun is also **faster**, partly because **Bun** is **smaller in size and scope**.

Bun does **not support** such **popular [GORM](https://github.com/go-gorm/gorm) features** like **automatic migrations**, **optimizer/index/comment hints**, and database resolver.

{% include alerts/info.html content="It's a <strong>classic</strong> in Go when using <strong>Postgres</strong>." %}

## Mongo

### [mongo-go-driver](https://github.com/mongodb/mongo-go-driver)

Is the **official** MongoDB supported **driver** for Go.

## Logger

## [zerolog](https://github.com/rs/zerolog)

The zerolog package provides a fast and simple logger dedicated to JSON output.

Zerolog's API is designed to provide both a great developer experience and stunning [performance](https://github.com/rs/zerolog#benchmarks). Its unique chaining API allows zerolog to write JSON (or CBOR) log events by avoiding allocations and reflection.

Uber's [zap](https://godoc.org/go.uber.org/zap) library pioneered this approach. Zerolog is taking this concept to the next level with a simpler to use API and even better performance.

{% include alerts/info.html content="It provide <strong>HTTP handlers</strong> to build your logger with some request informations." %}

## [zap](https://github.com/uber-go/zap)

Blazing fast, structured, leveled logging in Go.

## Config

## [cleanenv](https://github.com/ilyakaznacheev/cleanenv)

Is a **minimalistic** **configuration reader**.

This is a simple configuration reading tool. It just does the following:

- **reads** and parses configuration structure **from the file**
- **reads** and overwrites configuration structure **from environment variables**
- **writes** a **detailed variable** list to help output

## [viper](https://github.com/spf13/viper)

Viper is a **complete configuration solution** for Go applications including 12-Factor apps.
It is designed to work within an application, and can handle **all types of configuration** needs and formats. It supports:

- setting defaults
- reading from JSON, TOML, YAML, HCL, envfile and Java properties config files
- live watching and re-reading of config files (optional)
- reading from environment variables
- reading from remote config systems (etcd or Consul), and watching changes
- reading from command line flags
- reading from buffer
- setting explicit values

Viper can be thought of as a registry for all of your applications configuration needs.

{% include alerts/info.html content='viper is a <strong>heavy solution</strong>, I usually use it more for building CLI mixed with <a href="https://github.com/spf13/cobra">cobra</a>.' %}

## References

List of awesome go tools: <https://github.com/avelino/awesome-go>
