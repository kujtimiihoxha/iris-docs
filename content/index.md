---
date: 2016-07-29T00:00:02+01:00
title: Introduction
type: index
weight: 0
---
<a href ="https://github.com/kataras/iris"> <img src="http://iris-go.com/assets/book/cover_4.jpg" /> </a>


## Table of Contents

* [Introduction](/)
    * [Why](#why)
* [Features](features/)
* [Versioning](versioning/)
* [Install](install/)
* [Hi](hi/)
* [Transport Layer Security](tls/)
* [Handlers](handlers/)
   * [Using Handlers](handlers/#using-handlers)
   * [Using HandlerFuncs](handlers/#using-handlerfuncs)
   * [Using custom struct for a complete API](handlers/#using-handlerapi)
   * [Using native http.Handler](handlers/#using-native-httphandler)
       * [Using native http.Handler via iris.ToHandlerFunc()](handlers/#using-native-httphandler-via-tohandlerfunc)
   * [Routing and reverse lookups](handlers/#routing)
* [Middleware](middleware/)
* [API](api/)
* [Declaration](declaration/)
* [Configuration](configuration/)
* [Party](party/)
* [Subdomains](subdomains/)
* [Named Parameters](named_parameters/)
* [Static files](static_files/)
* [Send files](send_files/)
* [Send e-mails](send_emails/)
* [Render](render/)
   * [Response Engines](render/#response-engines)
   * [Template Engines](render/#template-engines)
* [Gzip](gzip/)
* [Streaming](streaming/)
* [Flash messages](flash_messages/)
* [Body binder](body_binder/)
* [Custom HTTP Errors](custom_http_errors/)
* [Context](context/)
* [Logger](logger/)
* [HTTP access control](http_access_control/)
* [Basic Authentication](basic_authentication/)
* [OAuth, OAuth2](oauth_oauth2/)
* [JSON Web Tokens(JWT)](jwt/)
* [Secure](secure/)
* [Sessions](sessions/)
* [Websockets](websockets/)
* [Graceful](graceful/)
* [Recovery](recovery/)
* [Plugins](plugins/)
* [Internationalization and Localization](internationalization_and_localization/)
* [Easy Typescript](typescript/)
* [Browser based Editor](browser_based_editor/)
* [Control panel](control_panel/)
* [Examples](examples/)

## Why

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, does your mind go to the standard net/http package?
Then you have to deal with some common situations like dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many other issues that net/http doesn't solve. 

The net/http package is not complete enough to quickly build well-designed back-end web systems. When you realize this, you might be thinking along these lines:

- Ok, the net/http package doesn't suit me, but there are so many frameworks, which one will work for me?!
- Each one of them tells me that it is the best. I don't know what to do!

##### The truth

I did some deep research and benchmarks with 'wrk' and 'ab' in order to choose which framework would suit me and my new project. The results, sadly, were really disappointing to me.

I started wondering if golang wasn't as fast on the web as I had read... but, before I let Golang go and continued to develop with nodejs, I told myself:

> '**Makis, don't lose hope, give at least a chance to Golang. Try to build something totally new without basing it off the "slow" code you saw earlier; learn the secrets of this language and make *others* follow your steps!**'.

These are the words I told myself that day [**13 March 2016**]. 

The same day, later the night, I was reading a book about Greek mythology. I saw an ancient goddess' name and was inspired immediately to give a name to this new web framework (which I had already started writing) - **Iris**.

**Two months later**, I'm writing this intro. 

 I'm still here [because Iris has succeed in being the fastest go web framework](https://github.com/kataras/iris#benchmarks)

![](comment1.png)
![](comment2.png)
![](comment3.png)
![](comment4.png)
![](comment5.png)
![](comment6.png)
![](comment7.png)
![](comment8.png)

![](comment10.png)

![](comment12.png)

![](comment13.png)

![](comment14.png)

![](comment17.png)


![](comment21.png)

![](comment22.png)

![](comment24.png)

![](comment23.png)