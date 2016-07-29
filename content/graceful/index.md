---
date: 2016-03-09T00:11:02+01:00
title: Graceful
weight: 330
---

[This is a package](https://github.com/iris-contrib/graceful).


Enables graceful shutdown.

```go

package main

import (
	"time"
        "github.com/kataras/iris"
	"github.com/iris-contrib/graceful"
)

func main() {
	api := iris.New()
	api.Get("/", func(c *iris.Context) {
		c.Write("Welcome to the home page!")
	})

	graceful.Run(":3001", time.Duration(10)*time.Second, api)
}


```