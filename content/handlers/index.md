---
date: 2016-03-09T00:11:02+01:00
title: Handlers
weight: 60
---

Handlers, as the name implies, handle requests. Each of the handler registration methods described in the following subchapters returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

Handlers must implement the Handler interface:

```go
type Handler interface {
	Serve(*Context)
}
```

Once the handler is registered, we can use the returned [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type (which is actually a `func` type) to give name to the this handler registration for easier lookup in code or in templates. For more information, checkout the [Routing and reverse lookups](routing.md) section.

## Using Handlers

```go

type myHandlerGet struct {
}

func (m myHandlerGet) Serve(c *iris.Context) {
    c.Write("From %s", c.PathString())
}

//and so on


iris.Handle("GET", "/get", myHandlerGet{})
iris.Handle("POST", "/post", post)
iris.Handle("PUT", "/put", put)
iris.Handle("DELETE", "/delete", del)
```

## Using HandlerFuncs

HandlerFuncs should implement the Serve\(\*Context\) func.
HandlerFunc is most simple method to register a route or a middleware, but under the hood it acts like a Handler. It implements the Handler interface as well:

```go
type HandlerFunc func(*Context)

func (h HandlerFunc) Serve(c *Context) {
    h(c)
}

```

HandlerFuncs shoud have this function signature:

```go
func handlerFunc(c *iris.Context)  {
    c.Write("Hello")
}


iris.HandleFunc("GET","/letsgetit",handlerFunc)
//OR
iris.Get("/letsgetit", handlerFunc)
iris.Post("/letspostit", handlerFunc)
iris.Put("/letputit", handlerFunc)
iris.Delete("/letsdeleteit", handlerFunc)
```

## Using Handler API

HandlerAPI is any custom struct which has an `*iris.Context` field and known methods signatures.

Before continue I will liked to notice you that this method is slower than `iris.Get, Post..., Handle, HandleFunc`.

I know maybe sounds awful but I, my self not using it, I did it because developers used to use frameworks with the 'MVC' pattern, so think it like the 'C\|Controller'. If you don't care about routing performance\(~ms\) and you like to spent some code time, you're free to use it.

Instead of writing Handlers\/HandlerFuncs for eachone API routes, you can use the `iris.API` function.

```go
API(path string, api HandlerAPI, middleware ...HandlerFunc) error
```

**For example**, for a user API you need some of these routes:

* GET `/users` , for selecting all
* GET`/users/:id` , for selecting specific
* PUT `/users` , for inserting
* POST `/users/:id` , for updating
* DELETE `/users/:id` , for deleting

Normally, with HandlerFuncs you should do something like this:

```go
iris.Get("/users", func(ctx *iris.Context){})
iris.Get("/users/:id", func(ctx *iris.Context){ id := ctx.Param("id) })

iris.Put("/users",...)

iris.Post("/users/:id", ...)

iris.Delete("/users/:id", ...)
```

**But** with API you can do this instead:

```go
package main

import (
    "github.com/kataras/iris"
)

type UserAPI struct {
    *iris.Context
}

// GET /users
func (u UserAPI) Get() {
    u.Write("Get from /users")
    // u.JSON(iris.StatusOK,myDb.AllUsers())
}

// GET /:param1 which its value passed to the id argument
func (u UserAPI) GetBy(id string) { // id equals to u.Param("param1")
    u.Write("Get from /users/%s", id)
    // u.JSON(iris.StatusOK, myDb.GetUserById(id))

}

// PUT /users
func (u UserAPI) Put() {
    name := u.FormValue("name")
    // myDb.InsertUser(...)
    println(string(name))
    println("Put from /users")
}

// POST /users/:param1
func (u UserAPI) PostBy(id string) {
    name := u.FormValue("name") // you can still use the whole Context's features!
    // myDb.UpdateUser(...)
    println(string(name))
    println("Post from /users/" + id)
}

// DELETE /users/:param1
func (u UserAPI) DeleteBy(id string) {
    // myDb.DeleteUser(id)
    println("Delete from /" + id)
}

func main() {
    iris.API("/users", UserAPI{})
    iris.Listen(":8080")
}

```

As you saw you can still get other request values via the \*iris.Context, API has all the  flexibility of handler\/handlerfunc.

If you want to use **more than one named parameter**, simply do this:

```go
// users/:param1/:param2
func (u UserAPI) GetBy(id string, otherParameter string) {}
```

API receives a third parameter which are the middlewares, is optional parameter:

```go
func main() {
    iris.API("/users", UserAPI{}, myUsersMiddleware1, myUsersMiddleware2)
    iris.Listen(":8080")
}

func myUsersMiddleware1(ctx *iris.Context) {
    println("From users middleware 1 ")
    ctx.Next()
}
func myUsersMiddleware2(ctx *iris.Context) {
    println("From users middleware 2 ")
    ctx.Next()
}

```

Available methods: "GET", "POST", "PUT", "DELETE", "CONNECT", "HEAD", "PATCH", "OPTIONS", "TRACE" should use this **naming conversion**:  **Get\/GetBy, Post\/PostBy, Put\/PutBy** and so on...

## Using native http.Handler

> Not recommended and I will not help you if any issue comes up, it is just there for your first conversional steps.
> Note also that using native http handler you cannot access url params.

```go

type nativehandler struct {}

func (_ nativehandler) ServeHTTP(res http.ResponseWriter, req *http.Request) {

}

func main() {
    iris.Handle("", "/path", iris.ToHandler(nativehandler{}))
    //"" means ANY(GET,POST,PUT,DELETE and so on)
}


```

## Using native http.Handler via iris.ToHandlerFunc()

```go
iris.Get("/letsget", iris.ToHandlerFunc(nativehandler{}))
iris.Post("/letspost", iris.ToHandlerFunc(nativehandler{}))
iris.Put("/letsput", iris.ToHandlerFunc(nativehandler{}))
iris.Delete("/letsdelete", iris.ToHandlerFunc(nativehandler{}))

```
## Routing

As mentioned in the [Handlers](handlers.md) chapter, Iris provides several handler registration methods, each of which returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

### Route naming

Route naming is easy, since we just call the returned `RouteNameFunc` with a string parameter to define a name:

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {

	// define a function
	render := func(ctx *iris.Context) {
		ctx.Render("index.html", nil)
	}

	// handler registration and naming
	iris.Get("/", render)("home")
	iris.Get("/about", render)("about")
	iris.Get("/page/:id", render)("page")

	iris.Listen(":8080")
}
```

### Route reversing AKA generating URLs from the route name

When we register the handlers for a specific path, we get the ability to create URLs based on the structured data we pass to Iris. In the example above, we've named three routers, one of which even takes parameters. If we're using the default `html/template` templating engine, we can use a simple action to reverse the routes (and generae actual URLs):

```
Home: {{ url "home" }}
About: {{ url "about" }}
Page 17: {{ url "page" "17" }}
```

Above code would generate the following output:

```
Home: http://0.0.0.0:8080/ 
About: http://0.0.0.0:8080/about
Page 17: http://0.0.0.0:8080/page/17
```

### Using route names in code

We can use the following methods/functions to work with named routes (and their parameters):

* global [`Lookups`](https://godoc.org/github.com/kataras/iris#Lookups) function to get all registered routes
* [`Lookup(routeName string)`](https://godoc.org/github.com/kataras/iris#Framework.Lookup) framework method to retrieve a route by name
* [`URL(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Framework.URL) framework method to generate url string based on supplied parameters
* [`Path(routeName string, args ...interface{}`](https://godoc.org/github.com/kataras/iris#Framework.Path) framework method to generate just the path (without host and protocol) portion of the URL based on provided values
* [`RedirectTo(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Context.RedirectTo) context method to return a redirect response to a URL defined by the named route and optional parameters

### Examples

Check out the [`template_engines/template_html_4`](https://github.com/iris-contrib/examples/blob/master/template_engines/template_html_4/) example in the `iris-contrib/examples` repository.