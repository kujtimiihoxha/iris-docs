---
date: 2016-07-29T03:13:00+02:00
title: Render
weight: 170
---

Think the 'Render'  as an action which sends\/responses with a rich content to the client.

The render actions, are separated in two iris-theoretical 'categories'

* Response content using Response Engines, by 'Content-Type\/ or Key', you will understand what key is, later.

* Templates using Template Engines, by 'filename'.


### [Response Engines](/response-engines.md)

Easy and fast way to render any type of data. **JSON, JSONP, XML, Text, Data, Markdown** .or any custom type. 

- examples are located [here](https://github.com/iris-contrib/examples/tree/master/response_engines/)

### [Template Engines](/template-engines.md)

Iris gives you the freedom to render templates through 6+ built'n template engines, you can create your own and 'inject' that to the iris station, you can also use more than one template engines at the same time \(when the file extension is different from the other\). 

- examples are located [here](https://github.com/iris-contrib/examples/tree/master/template_engines/) 

## Response Engine
### Install

Install one response engine and all will be installed.

```sh
$ go get -u github.com/iris-contrib/response/json
```

### Iris' Station configuration

Remember, when 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL` 

```go
iris.Config.Gzip = true // compressed gzip contents to the client, the same for Template Engines also, defaults to false
iris.Config.Charset = "UTF-8" // defaults to "UTF-8", the same for Template Engines also
```

They can be overriden for specific `Render` action: 
```go
func(ctx *iris.Context){
 ctx.Render("any/contentType", anyValue{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

### How to use

First of all don't be scary about the 'big' article here, a response engine works very simple and is easy to understand how.

Let's see what are the built'n response by content-type context's methods using the defaults only, unchanged, response engines.


```go

package main

import (
	"encoding/xml"

	"github.com/kataras/iris"
)

type ExampleXml struct {
	XMLName xml.Name `xml:"example"`
	One     string   `xml:"one,attr"`
	Two     string   `xml:"two,attr"`
}

func main() {
	iris.Get("/data", func(ctx *iris.Context) {
		ctx.Data(iris.StatusOK, []byte("Some binary data here."))
	})

	iris.Get("/text", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Plain text here")
	})

	iris.Get("/json", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, map[string]string{"hello": "json"}) // or myjsonStruct{hello:"json}
	})

	iris.Get("/jsonp", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", map[string]string{"hello": "jsonp"})
	})

	iris.Get("/xml", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, ExampleXml{One: "hello", Two: "xml"}) // or iris.Map{"One":"hello"...}
	})

	iris.Get("/markdown", func(ctx *iris.Context) {
		ctx.Markdown(iris.StatusOK, "# Hello Dynamic Markdown Iris")
	})

	iris.Listen(":8080")
}


```


Bellow you will, propably, see how 'good' are my english (joke...), but at the end we're coders and some of us programmers too, so I hope you will be able to understand at least, the code snippets ( a lot of them, you will be tired from this simplicity ).




**Text Response Engine**

```go

package main

import "github.com/kataras/iris"

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, you don't have to set it manually

	myString := "this is just a simple string which you can already render with ctx.Write"

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, myString)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.Render("text/plain", myString)
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/plain", myString)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.Render("text/plain", myString, iris.RenderOptions{"charset": "UTF-8"}) // default & global charset is UTF-8
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/plain", myString)
	})

	iris.Listen(":8080")
}

```

**Custom response engine**

You can create a custom response engine using a func or an interface which implements the
` iris.ResponseEngine`  which contains a simple function: ` Response(val interface{}, options ...map[string]interface{}) ([]byte, error)
` 

A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType  

You can imagine its useful, I will show you one right now.

Let's do a 'trick' here, which works for all other response engines, custom or not:

say for example, that you want a static'footer/suffix' on your content.

IF a response engine has the same key and the same content type then the contents are appended and the final result will be rendered to the client
.

Let's do this with ` text/plain` content type, because you can see its results easly, the first engine will use this "text/plain" as key, the second & third will use the same, as firsts, key, which is the ContentType also.
```go

package main

import (
	"github.com/iris-contrib/response/text"
	"github.com/kataras/iris"
)

func main() {
	// here we are registering the default text/plain,  and after we will register the 'appender' only
	// we have to register the default because we will 
    // add more response engines with the same content,
	// iris will not register this by-default if 
    // other response engine with the corresponding ContentType already exists

	iris.UseResponse(text.New(), text.ContentType) // it's the key which happens to be a valid content-type also, "text/plain" so this will be used as the ContentType header

	// register a response engine: iris.ResponseEngine 
	iris.UseResponse(&CustomTextEngine{}, text.ContentType)
	
    // register a response engine with func
	iris.UseResponse(iris.ResponseEngineFunc(func(val interface{}, options ...map[string]interface{}) ([]byte, error) {
		return []byte("\nThis is the static SECOND AND LAST suffix!"), nil
	}), text.ContentType)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Hello!") // or ctx.Render(text.ContentType," Hello!")
	})

	iris.Listen(":8080")
}

// This is the way you create one with raw iris.ResponseEngine implementation:

// CustomTextEngine the response engine which appends a simple string on the default's text engine
type CustomTextEngine struct{}

// Implement the iris.ResponseEngine
func (e *CustomTextEngine) Response(val interface{}, options ...map[string]interface{}) ([]byte, error) {
	// we don't need the val, because we want only to append, so what we should do?
	// just return the []byte we want to be appended after the first registered text/plain engine

	return []byte("\nThis is the static FIRST suffix!"), nil
}

```

**iris.ResponseString**


ResponseString gives you the result of the response engine's work, it doesn't renders to the client but you can use
this function to collect the end result and send it via e-mail to the user, or anything you can imagine.


```go
package main

import "github.com/kataras/iris"

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.
	
	[this is a link](https://github.com/kataras/iris) `

	iris.Get("/", func(ctx *iris.Context) {
		// let's see
		// convert markdown string to html and print it to the logger
		// THIS WORKS WITH ALL RESPONSE ENGINES, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.TemplateString)
		htmlContents := iris.ResponseString("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"}) // default is the iris.Config.Charset, which is UTF-8

		ctx.Log(htmlContents)
		ctx.Write("The Raw HTML is:\n%s", htmlContents)
	})

	iris.Listen(":8080")
}


```


Now we can continue to the rest of the default & built'n response engines


**JSON Response Engine**


```go

package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, iris.Map{"name": "iris"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("application/json", myjson{Name: "iris"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/json", myjson{Name: "iris"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	"github.com/iris-contrib/response/json"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// use the json's Config, we need the import of the json response engine in order to change its internal configs
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			Indent        bool
			UnEscapeHTML  bool
			Prefix        []byte
			StreamingJSON bool
		}
	*/
	iris.UseResponse(json.New(json.Config{
		Prefix: []byte("MYPREFIX"),
	}), json.ContentType) // you can use anything as the second parameter, the json.ContentType is the string "application/json", the context.JSON renders with this engine's key.

	jsonHandlerSimple := func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	}

	jsonHandlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// imagine that we need the context.JSON to be listening to our "application/json" response engine with a custom prefix (we did that before)
	// but we also want a different renderer, but again application/json content type, with Indent option setted to true:
	iris.UseResponse(json.New(json.Config{Indent: true}), "json2")("application/json")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	json2Handler := func(ctx *iris.Context) {
		ctx.Render("json2", myjson{Name: "My iris"})
	}

	iris.Get("/", jsonHandlerSimple)

	iris.Get("/render", jsonHandlerWithRender)

	iris.Get("/json2", json2Handler)

	iris.Listen(":8080")
}

```


**JSONP Response Engine**

```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", iris.Map{"name": "iris"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", myjson{Name: "iris"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go

package main

import (
	"github.com/iris-contrib/response/jsonp"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			Indent   bool
			Callback string // the callback can be override by the context's options or parameter on context.JSONP
		}
	*/
	iris.UseResponse(jsonp.New(jsonp.Config{
		Indent: true,
	}), jsonp.ContentType)
	// you can use anything as the second parameter,
	// the jsonp.ContentType is the string "application/javascript",
	// the context.JSONP renders with this engine's key.

	handlerSimple := func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", myjson{Name: "iris"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "8859-1"})
	}

	//second example,
	// but we also want a different renderer, but again "application/javascript" as content type, with Callback option setted globaly:
	iris.UseResponse(jsonp.New(jsonp.Config{Callback: "callbackName"}), "jsonp2")("application/javascript")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	handlerJsonp2 := func(ctx *iris.Context) {
		ctx.Render("jsonp2", myjson{Name: "My iris"})
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/jsonp2", handlerJsonp2)

	iris.Listen(":8080")
}


```



**XML Response Engine**


```go
package main

import "github.com/kataras/iris"

type myxml struct {
	XMLName xml.Name `xml:"xml_example"`
	First   string   `xml:"first,attr"`
	Second  string   `xml:"second,attr"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, iris.Map{"first": "first attr ", "second": "second attr"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/xml", myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"})
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	encodingXML "encoding/xml"

	"github.com/iris-contrib/response/xml"
	"github.com/kataras/iris"
)

type myxml struct {
	XMLName encodingXML.Name `xml:"xml_example"`
	First   string           `xml:"first,attr"`
	Second  string           `xml:"second,attr"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			Indent bool
			Prefix []byte
		}
	*/
	iris.UseResponse(xml.New(xml.Config{
		Indent: true,
	}), xml.ContentType)
	// you can use anything as the second parameter,
	// the jsonp.ContentType is the string "text/xml",
	// the context.XML renders with this engine's key.

	handlerSimple := func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// but we also want a different renderer, but again "text/xml" as content type, with prefix option setted by configuration:
	iris.UseResponse(xml.New(xml.Config{Prefix: []byte("")}), "xml2")("text/xml") // if you really use a PREFIX it will be not valid xml, use it only for special cases
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	handlerXML2 := func(ctx *iris.Context) {
		ctx.Render("xml2", myxml{First: "first attr", Second: "second attr"})
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/xml2", handlerXML2)

	iris.Listen(":8080")
}


```


**Markdown Response Engine**


```go

package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.
	
	[this is a link](https://github.com/kataras/iris) `

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Markdown(iris.StatusOK, markdownContents)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		htmlContents := ctx.MarkdownString(markdownContents)
		ctx.HTML(iris.StatusOK, htmlContents)
	})

	// text/markdown is just the key which the markdown response engine and ctx.Markdown communicate,
	// it's real content type is text/html
	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("text/markdown", markdownContents)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/markdown", markdownContents)
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	"github.com/iris-contrib/response/markdown"
	"github.com/kataras/iris"
)

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.
	
	[this is a link](https://github.com/kataras/iris) `

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			MarkdownSanitize bool
		}
	*/
	iris.UseResponse(markdown.New(), markdown.ContentType)
	// you can use anything as the second parameter,
	// the markdown.ContentType is the string "text/markdown",
	// the context.Markdown renders with this engine's key.

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// but we also want a different renderer, but again "text/markdown" as 'content type' (this is converted to text/html behind the scenes), with MarkdownSanitize option setted to true:
	iris.UseResponse(markdown.New(markdown.Config{MarkdownSanitize: true}), "markdown2")("text/markdown")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	handlerMarkdown2 := func(ctx *iris.Context) {
		ctx.Render("markdown2", markdownContents, iris.RenderOptions{"gzip": true})
	}

	iris.Get("/", handlerWithRender)

	iris.Get("/markdown2", handlerMarkdown2)

	iris.Listen(":8080")
}


```

** Data(Binary) Response Engine **


```go

package main

import "github.com/kataras/iris"

func main() {
	myData := []byte("some binary data or a program here which will not be a simple string at the production")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Data(iris.StatusOK, myData)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.Render("application/octet-stream", myData)
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/octet-stream", myData)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.Render("application/octet-stream", myData, iris.RenderOptions{"gzip": true}) // gzip is false by default
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("application/octet-stream", myData)
	})

	iris.Listen(":8080")
}

```


 ----- 

 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/response_engines/) 

- You can contribute to create more response engines for Iris, click [here](https://github.com/iris-contrib/response) to navigate to the reository.

## Template Engines

### Install

Install one template engine and all will be installed.

```sh
$ go get -u github.com/iris-contrib/template/html
```

### Iris' Station configuration 

Remember, when 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL`

```go
iris.Config.IsDevelopment = true // reloads the templates on each request, defaults to false
iris.Config.Gzip  = true // compressed gzip contents to the client, the same for Response Engines also, defaults to false
iris.Config.Charset = "UTF-8" // defaults to "UTF-8", the same for Response Engines also
```

The last two options (Gzip, Charset) can be overriden for specific 'Render' action:

```go
func(ctx *iris.Context){
    ctx.Render("templateFile.html", anyBindingStruct{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

### How to use

Most examples are written for the HTML Template Engine(default and built'n template engine for iris) but works for the rest of the engines also.


You will see first the template file's code, after the main.go code


** HTML Template Engine, and general **


```html
<!-- ./templates/hi.html -->

<html>
<head>
<title>Hi Iris [THE TITLE]</title>
</head>
<body>
	<h1>Hi {{.Name}}
</body>
</html>

```

```go
// ./main.go
package main

import "github.com/kataras/iris"

// nothing to do, defaults to ./templates and .html extension, no need to import any template engine because HTML engine is the default
// if anything else has been registered
func main() {
	iris.Config.IsDevelopment = true // this will reload the templates on each request, defaults to false
	iris.Get("/hi", hi)
	iris.Listen(":8080")
}

func hi(ctx *iris.Context) {
	ctx.MustRender("hi.html", struct{ Name string }{Name: "iris"})
}


```

```html
<!-- ./templates/layout.html -->
<html>
<head>
<title>My Layout</title>

</head>
<body>
	<h1>Body is:</h1>
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>
```

```html
 <!-- ./templates/mypage.html --> 
<h1>
	Title: {{.Title}}
</h1>
<h3>Message : {{.Message}} </h3>
```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/html"
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {

	iris.UseTemplate(html.New(html.Config{
		Layout: "layout.html",
	})).Directory("./templates", ".html") // the .Directory() is optional also, defaults to ./templates, .html
	// Note for html: this is the default iris' templaet engine, if zero engines added, then the template/html will be used automatically
	// These lines are here to show you how you can change its default configuration

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage.html", mypage{"My Page title", "Hello world!"}, iris.RenderOptions{"gzip": true})
		// Note that: you can pass "layout" : "otherLayout.html" to bypass the config's Layout property or iris.NoLayout to disable layout on this render action.
		// RenderOptions is an optional parameter
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/layouts/layout.html -->
<html>
<head>
<title>Layout</title>

</head>
<body>
	<h1>This is the global layout</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```

```html
<!-- ./templates/layouts/mylayout.html -->
<html>
<head>
<title>my Layout</title>

</head>
<body>
	<h1>This is the layout for the /my/ and /my/other routes only</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```

```html
<!-- ./templates/partials/page1_partial1.html -->
<div style="background-color: white; color: red">
	<h1>Page 1's Partial 1</h1>
</div>


```

```html
<!-- ./templates/page1.html -->
<div style="background-color: black; color: blue">

	<h1>Page 1</h1>

	{{ render "partials/page1_partial1.html"}}

</div>


```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/html"
	"github.com/kataras/iris"
)

func main() {
	// directory and extensions defaults to ./templates, .html for all template engines
	iris.UseTemplate(html.New(html.Config{Layout: "layouts/layout.html"}))
	//iris.Config.Render.Template.Gzip = true
	iris.Get("/", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil); err != nil {
			println(err.Error())
		}
	})

	// remove the layout for a specific route
	iris.Get("/nolayout", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil, iris.RenderOptions{"layout": iris.NoLayout}); err != nil {
			println(err.Error())
		}
	})

	// set a layout for a party, .Layout should be BEFORE any Get or other Handle party's method
	my := iris.Party("/my").Layout("layouts/mylayout.html")
	{
		my.Get("/", func(ctx *iris.Context) {
			ctx.MustRender("page1.html", nil)
		})
		my.Get("/other", func(ctx *iris.Context) {
			ctx.MustRender("page1.html", nil)
		})
	}

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/layouts/layout.html -->

<html>
<head>
<title>My Layout</title>

</head>
<body>
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>

```

```html
<!-- ./templates/partials/page1_partial1.html -->
<div style="background-color: white; color: red">
	<h1>Page 1's Partial 1</h1>
</div>


```

```html
<!-- ./templates/page1.html -->
<div style="background-color: black; color: blue">

	<h1>Page 1</h1>

	{{ render "partials/page1_partial1.html"}}

</div>


```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/html"
	"github.com/kataras/iris"
)

func main() {
	// directory and extensions defaults to ./templates, .html for all template engines
	iris.UseTemplate(html.New(html.Config{Layout: "layouts/layout.html"}))

	iris.Get("/", func(ctx *iris.Context) {
		s := iris.TemplateString("page1.html", nil)
		ctx.Write("The plain content of the template is: %s", s)
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/page.html -->
<a href="{{url "my-page1"}}">http://127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{url "my-page2" "theParam1" "theParam2"}}">http://127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{url "my-page3" "theParam1" "theParam2AfterStatic"}}">http://127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{url "my-page4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}">http://127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />
<a href="{{url "my-page5" "theParam1" "theParam2AfterStatic" "otherParam" "matchAnythingAfterStatic"}}">http://127.0.0.1:8080/mypath5/:param1/statichere/:param2/:otherparam/anything/*anything</a>
<br />
<br />
<a href="{{url "my-page6" .ParamsAsArray }}">http://127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic</a>


```

```go
// ./main.go
// Package main an example on how to naming your routes & use the custom 'url' HTML Template Engine, same for other template engines
// we don't need to import the iris-contrib/template/html because iris uses this as the default engine if no other template engine has been registered.
package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Get("/mypath", emptyHandler)("my-page1")
	iris.Get("/mypath2/:param1/:param2", emptyHandler)("my-page2")
	iris.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("my-page3")
	iris.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("my-page4")

	// same with Handle/Func
	iris.HandleFunc("GET", "/mypath5/:param1/statichere/:param2/:otherparam/anything/*anything", emptyHandler)("my-page5")

	iris.Get("/mypath6/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("my-page6")

	iris.Get("/", func(ctx *iris.Context) {
		// for /mypath6...
		paramsAsArray := []string{"theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.html", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")

		println("The full uri of " + routeName + "is: " + iris.URL(routeName))
		// if routeName == "my-page1"
		// prints: The full uri of my-page1 is: http://127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName)
		// http://127.0.0.1:8080/redirect/my-page1 will redirect to -> http://127.0.0.1:8080/mypath
	})

	iris.Listen(":8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("Hello from %s.", ctx.PathString())

}


```


```html
<!-- ./templates/page.html -->
<!-- the only difference between normal named routes and dynamic subdomains named routes is that the first argument of  url
is the subdomain part instead of named parameter-->

<a href="{{url "dynamic-subdomain1" "username1"}}">username1.127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{url "dynamic-subdomain2" "username2" "theParam1" "theParam2"}}">username2.127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{url "dynamic-subdomain3" "username3" "theParam1" "theParam2AfterStatic"}}">username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{url "dynamic-subdomain4" "username4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}">username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />
<a href="{{url "dynamic-subdomain5" .ParamsAsArray }}" >username5.127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic</a>


```
I will add hosts files contens only once, here, you can imagine the rest.

**File location is Windows: Drive:/Windows/system32/drivers/etc/hosts, on Linux: /etc/hosts**
```sh
# localhost name resolution is handled within DNS itself.
127.0.0.1       localhost
::1             localhost
#-IRIS-For development machine, you have to configure your dns also for online, search google how to do it if you don't know

127.0.0.1		username1.127.0.0.1
127.0.0.1		username2.127.0.0.1
127.0.0.1		username3.127.0.0.1
127.0.0.1		username4.127.0.0.1
127.0.0.1		username5.127.0.0.1
# note that you can always use custom subdomains
#-END IRIS-

```
```go
// ./main.go
// Package main same example as template_html_4 but with wildcard subdomains
package main

import (
	"github.com/kataras/iris"
)

func main() {

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
		wildcard.Get("/mypath5/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("dynamic-subdomain5")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		paramsAsArray := []string{"username5", "theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.html", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}


```


**Django Template Engine**


```html
<!-- ./templates/mypage.html -->
<html>
<head>
<title>Hello Django from Iris</title>

</head>
<body>
	{% if is_admin %}
	<p>{{username}} is an admin!</p>
	{% endif %}
</body>
</html>

```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/django"
	"github.com/kataras/iris"
)

func main() {

	iris.UseTemplate(django.New()).Directory("./templates", ".html")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage.html", map[string]interface{}{"username": "iris", "is_admin": true}, iris.RenderOptions{"gzip": true})
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/page.html -->
<!-- the only difference between normal named routes and dynamic subdomains named routes is that the first argument of  url
is the subdomain part instead of named parameter-->
<a href="{{ url("dynamic-subdomain1","username1") }}">username1.127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain2","username2","theParam1","theParam2") }}">username2.127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain3","username3","theParam1","theParam2AfterStatic") }}" >username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain4","username4","theParam1","theparam2AfterStatic","otherParam","matchAnything") }}" >username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />



```

```go
// ./main.go
// Package main same example as template_html_5 but for django/pongo2
package main

import (
	"github.com/iris-contrib/template/django"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(django.New())

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part

		if err := ctx.Render("page.html", nil); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}


```

> Note that, you can see more django examples syntax by navigating [here](https://github.com/flosch/pongo2)


**Handlebars Template Engine**

```html
<!-- ./templates/layouts/layout.html -->

<html>
<head>
<title>Layout</title>

</head>
<body>
	<h1>This is the global layout</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>

```

```html
<!-- ./templates/layouts/mylayout.html -->
<html>
<head>
<title>my Layout</title>

</head>
<body>
	<h1>This is the layout for the /my/ and /my/other routes only</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```


```html
<!-- ./templates/partials/home_partial.html -->
<div style="background-color: white; color: red">
	<h1>Home's' Partial here!!</h1>
</div>


```

```html
<!-- ./templates/home.html -->
<div style="background-color: black; color: white">

	Name: {{boldme Name}} <br /> Type: {{boldme Type}} <br /> Path:
	{{boldme Path}} <br />
	<hr />

	The partial is: {{ render "partials/home_partial.html"}}

</div>

```


```go
// ./main.go
package main

import (
	"github.com/aymerick/raymond"
	"github.com/iris-contrib/template/handlebars"
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {
	// set the configuration for this template engine  (all template engines has its configuration)
	config := handlebars.DefaultConfig()
	config.Layout = "layouts/layout.html"
	config.Helpers["boldme"] = func(input string) raymond.SafeString {
		return raymond.SafeString("<b> " + input + "</b>")
	}

	// set the template engine
	iris.UseTemplate(handlebars.New(config)).Directory("./templates", ".html") // or .hbs , whatever you want

	iris.Get("/", func(ctx *iris.Context) {
		// optionally, set a context  for the template
		ctx.Render("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/"})

	})

	// remove the layout for a specific route using iris.NoLayout
	iris.Get("/nolayout", func(ctx *iris.Context) {
		if err := ctx.Render("home.html", nil, iris.RenderOptions{"layout": iris.NoLayout}); err != nil {
			ctx.Write(err.Error())
		}
	})

	// set a layout for a party, .Layout should be BEFORE any Get or other Handle party's method
	my := iris.Party("/my").Layout("layouts/mylayout.html")
	{
		my.Get("/", func(ctx *iris.Context) {
			// .MustRender -> same as .Render but logs the error if any and return status 500 on client
			ctx.MustRender("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/my/"})
		})
		my.Get("/other", func(ctx *iris.Context) {
			ctx.MustRender("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/my/other"})
		})
	}

	iris.Listen(":8080")
}

// Note than you can see more handlebars examples syntax by navigating to https://github.com/aymerick/raymond


```

>  Note than you can see more handlebars examples syntax by navigating [here](https://github.com/aymerick/raymond)


**Pug/Jade Template Engine**

```html
<!-- ./templates/partials/page1_partial1.jade -->
#footer
  p Copyright (c) foobar


```

```html
<!-- ./templates/page.jade -->
doctype html
html(lang=en)
	head
		meta(charset=utf-8)
		title Title
	body
		p ads
		ul
			li The name is {{bold .Name}}.
			li The age is {{.Age}}.

		range .Emails
			div An email is {{.}}

		with .Jobs
			range .
				div.
				 An employer is {{.Employer}}
				 and the role is {{.Role}}

		{{ render "partials/page1_partial1.jade"}}


```

```go
// ./main.go
package main

import (
	"html/template"

	"github.com/iris-contrib/template/pug"
	"github.com/kataras/iris"
)

type Person struct {
	Name   string
	Age    int
	Emails []string
	Jobs   []*Job
}

type Job struct {
	Employer string
	Role     string
}

func main() {
	// set the configuration for this template engine  (all template engines has its configuration)
	cfg := pug.DefaultConfig()
	cfg.Funcs["bold"] = func(content string) (template.HTML, error) {
		return template.HTML("<b>" + content + "</b>"), nil
	}

	iris.UseTemplate(pug.New(cfg)).
		Directory("./templates", ".jade")

	iris.Get("/", func(ctx *iris.Context) {

		job1 := Job{Employer: "Super Employer", Role: "Team leader"}
		job2 := Job{Employer: "Fast Employer", Role: "Project managment"}

		person := Person{
			Name:   "name1",
			Age:    50,
			Emails: []string{"email1@something.gr", "email2.anything@gmail.com"},
			Jobs:   []*Job{&job1, &job2},
		}
		ctx.MustRender("page.jade", person)

	})

	iris.Listen(":8080")
}


```



```html
<!-- ./templates/page.jade -->
a(href='{{url "dynamic-subdomain1" "username1"}}') username1.127.0.0.1:8080/mypath
p.
 a(href='{{url "dynamic-subdomain2" "username2" "theParam1" "theParam2"}}') username2.127.0.0.1:8080/mypath2/:param1/:param2

p.
 a(href='{{url "dynamic-subdomain3" "username3" "theParam1" "theParam2AfterStatic"}}') username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2

p.
 a(href='{{url "dynamic-subdomain4" "username4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}') username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something

p.
 a(href='{{url "dynamic-subdomain5" .ParamsAsArray }}') username5.127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic


```

```go
// ./main.go
// Package main same example as template_html_5 but for pug/jade
package main

import (
	"github.com/iris-contrib/template/pug"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(pug.New()).Directory("./templates", ".jade")

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
		wildcard.Get("/mypath5/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("dynamic-subdomain5")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		paramsAsArray := []string{"username5", "theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.jade", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}

// Note than you can see more Pug/Jade syntax examples by navigating to https://github.com/Joker/jade


```

>  Note than you can see more Pug/Jade syntax examples by navigating [here](https://github.com/Joker/jade)

```html
<!-- ./templates/basic.amber -->
!!! 5
html
    head
        title Hello Amber from Iris

        meta[name="description"][value="This is a sample"]

        script[type="text/javascript"]
            var hw = "Hello #{Name}!"
            alert(hw)

        style[type="text/css"]
            body {
                background: maroon;
                color: white
            }

    body
        header#mainHeader
            ul
                li.active
                    a[href="/"] Main Page
                        [title="Main Page"]
            h1
                 | Hi #{Name}

        footer
            | Hey
            br
            | There


```


```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/amber"
	"github.com/kataras/iris"
)

type mypage struct {
	Name string
}

func main() {

	iris.UseTemplate(amber.New()).Directory("./templates", ".amber")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("basic.amber", mypage{"iris"}, iris.RenderOptions{"gzip": true})
	})

	iris.Listen(":8080")
}


```


**Custom template engine** 

Simply, you have to implement only **3  functions**, for load and execute the templates. One optionally (**Funcs() map[string]interface{}**) which is used to register the iris' helpers funcs like `{{ url }}` and `{{ urlpath }}`.

```go

type (
	// TemplateEngine the interface that all template engines must implement
	TemplateEngine interface {
		// LoadDirectory builds the templates, usually by directory and extension but these are engine's decisions
		LoadDirectory(directory string, extension string) error
		// LoadAssets loads the templates by binary
		// assetFn is a func which returns bytes, use it to load the templates by binary
		// namesFn returns the template filenames
		LoadAssets(virtualDirectory string, virtualExtension string, assetFn func(name string) ([]byte, error), namesFn func() []string) error

		// ExecuteWriter finds, execute a template and write its result to the out writer
		// options are the optional runtime options can be passed by user
		// an example of this is the "layout" or "gzip" option
		ExecuteWriter(out io.Writer, name string, binding interface{}, options ...map[string]interface{}) error
	}

	// TemplateEngineFuncs is optional interface for the TemplateEngine
	// used to insert the Iris' standard funcs, see var 'usedFuncs'
	TemplateEngineFuncs interface {
		// Funcs should returns the context or the funcs,
		// this property is used in order to register the iris' helper funcs
		Funcs() map[string]interface{}
	}
)

```

The simplest implementation, which you can look as example, is the Markdown Engine, which is located [here](https://github.com/iris-contrib/template/tree/master/markdown/markdown.go).



**iris.TemplateString**


Executes and parses the template but instead of rendering to the client, it returns the contents. Useful when you want to send a template via e-mail or anything you can imagine.

```html
<!-- ./templates/mypage.html -->
<html>
<head>
<title>Hello Django from Iris</title>

</head>
<body>
	{% if is_admin %}
	<p>{{username}} is an admin!</p>
	{% endif %}
</body>
</html>

```


```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/django"
	"github.com/kataras/iris"
)

func main() {

	iris.UseTemplate(django.New()).Directory("./templates", ".html")

	iris.Get("/", func(ctx *iris.Context) {
		// THIS WORKS WITH ALL TEMPLATE ENGINES, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.ResponseString)
		rawHtmlContents := iris.TemplateString("mypage.html", map[string]interface{}{"username": "iris", "is_admin": true}, iris.RenderOptions{"charset": "UTF-8"}) // defaults to UTF-8 already
		ctx.Log(rawHtmlContents)
		ctx.Write("The Raw HTML is:\n%s", rawHtmlContents)
	})

	iris.Listen(":8080")
}


```

 > Note that: iris.TemplateString can be called outside of the context also 




-----


 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/template_engines/) 

- You can contribute to create more template engines for Iris, click [here](https://github.com/iris-contrib/template) to navigate to the reository. 