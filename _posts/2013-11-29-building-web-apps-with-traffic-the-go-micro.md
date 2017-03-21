---
ID: 313
post_title: >
  Building web apps with Traffic, the Go
  micro framework.
author: Massimo Ronca
post_date: 2013-11-29 11:54:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/29/building-web-apps-with-traffic-the-go-micro/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/68453619468/building-web-apps-with-traffic-the-go-micro
tumblr_mikamayhem_id:
  - "68453619468"
---
Written by long time Ruby developer and MIKAMAI partner <a href="http://gravityblast.com/">Andrea Franz</a>, <a href="https://github.com/pilu/traffic/"><strong>Traffic</strong></a> is a <a href="http://flask.pocoo.org/docs/foreword/#what-does-micro-mean"><em>micro framework</em></a> for Web development inspired by <a href="http://www.sinatrarb.com">Sinatra</a>.

In times when software development is moving towards creating services for other applications, micro frameworks serve very well this paradigm by providing an easy way to create the building blocks, such as a REST API, on top of which larger applications can rely.

Micro frameworks do not cover every single aspect of application building, most of them are left to the developer, for example you decide which database to use and how to access it or not to use a DB at all, but the codebase is so compact that a single person can master it in a matter of few days.

Like other popular frameworks, <em>Traffic</em> features include
<ul>
 	<li>regexp routing</li>
 	<li>chainable request filters</li>
 	<li>middlewares</li>
 	<li>templates</li>
 	<li>serving of static files</li>
 	<li>custom error handlers</li>
 	<li>facilities to render in various format (HTML, JSON and XML)</li>
 	<li>errors and stacktraces in the browser</li>
 	<li>command line utility to generate new projects (although very basic)</li>
</ul>
Documentation is a little bit scarce at the moment, but there are many <a href="https://github.com/pilu/traffic/blob/master/examples/">examples</a> that make it easy to figure out how to use the various components.

We are going to analyze each one of them, by writing a small demo application, a simple clone of the placehold.it service powered by the Grumpy cat, instead of gray boxes.
As usual, code is available on <a href="https://github.com/wstucco/purrraceholder">github</a>.

<!--more-->

<h3>Installing Traffic</h3>
Assuming you already have a working installation of Go, download Traffic
<pre class="brush: go java">go get github.com/pilu/traffic</pre>
and the command line tool
<pre class="brush: go java">go get github.com/pilu/traffic/traffic</pre>
create a new project
<pre class="brush: go java">traffic new demo_project</pre>
run it
<pre class="brush: go java">cd demo_project  
go build &amp;&amp; ./demo_project
</pre>
and point your browser to <a href="http://127.0.0.1:7000/">http://127.0.0.1:7000/</a>
<h3>Routing</h3>
The first thing we need for our application is a router.
Traffic routes are a pair of an HTTP method and a URL pattern that, if matched,
call a function that act as route-handler.

Our first route
<pre class="brush: go java">package main

import (
    "github.com/pilu/traffic"
)

var router *traffic.Router

func main() {
    router = traffic.New()
    router.Get("/", RootHandler)
    router.Run()
}   
</pre>
Routes are matched in the same order they are declared, the first that match is executed.

Routes can contain named parameters that are accesible using the Param function.
<pre class="brush: go java">router.Get(`/:width/:height/?`, ImageHandler)

func ImageHandler(w traffic.ResponseWriter, r *traffic.Request) {
    width := r.Param("width")
    height := r.Param("height")
}
</pre>
and parameters can be optional
<pre class="brush: go java">router.Get(`/:width)/(:height)?`, ImageHandler)
</pre>
Route patterns can also include wildcards and regular expressions
<pre class="brush: go java">router.Get(`/:width/*`, ImageHandler)

// match (width)x(height) format
router.Get(`/(?P&lt;width&gt;d+)(x(?P&lt;height&gt;d+)?)?/?`, ImageHandler)
</pre>
<h3>Before filters</h3>
Traffic allows to prepend the request handler with filters, which are like regular request handlers that get executed before the real handler.
Before fitlers can be chained and can be attached to all routes or just some of them.
If a before filters write something in the Response Body, the request chain is interrupted.
<pre class="brush: go java">// if route match, before executing ImageHandler, Traffic executes the two filters
// RequireValidImageParameters and GenerateImageCache in order  
// If one of them fails and write to the response body, the execution stops
// before reaching the actual handler
router.Get(`/:width/?(:height)?/?`, ImageHandler).
    AddBeforeFilter(RequireValidImageParameters).
    AddBeforeFilter(GenerateImageCache)

func RequireValidImageParameters(w traffic.ResponseWriter, r *traffic.Request) {
    width, err := strconv.Atoi(r.Param("width")) 
    if err != nil { // conversion error, either var is empty or not a number
        // cannot continue
        w.WriteHeader(http.StatusNotFound) 
        return
    }

    height, err := strconv.Atoi(r.Param("height"))
    if err != nil {
        // if height is omitted the image is gonna be a square
        height = width 
    }

    if (width &lt;= 2560 &amp;&amp; width &gt; 0) &amp;&amp; (height &lt;= 2560 &amp;&amp; height &gt; 0) {
        // set vars for the next filter
    } else {
        // bad request
        w.WriteHeader(http.StatusBadRequest)
        w.Render("400", nil)
    }
}   

func GenerateImageCache(w traffic.ResponseWriter, r *traffic.Request) {
    // pseudo code  
    if not cache_folder_exists and create_folder_fails
        throw error 

    write_image_file_according_to_parameters
}

func ImageHandler(w traffic.ResponseWriter, r *traffic.Request) {
    // output the image with the correct content-type
    w.Header().Set("Content-Type", "image/jpeg")

    // at this point we can safely assume that the image file already exists
}

// this filter is global to the router and is applied before each request
router.AddBeforeFilter(PoweredByHandler)

func PoweredByHandler(w traffic.ResponseWriter, r *traffic.Request) {
    w.Header().Set("X-Powered-By", "Grumpy cat")
}
</pre>
<h3>Templates and static files</h3>
Traffic supports templates in the standard Go format.
Template library documentation can be found <a href="http://golang.org/pkg/html/template/">here</a>.
Traffic Response Writer has a method to render templates called Render, that takes the template name (without the extension) and an optional param that contains the data to be rendered.
By default templates are placed in the /view folder.
Templates can be nested one isnide the other like in our 404 example
<pre class="brush: go java">{{ template "includes/header" }}
    &lt;div class="error error-404"&gt;&lt;/div&gt;
{{ template "includes/footer" }}
</pre>
If you are writing an API you might find the methods WriteJSON and WriteXML useful too.

Traffic also support serving static assets: every file placed in the /public folder is directly accessible.
For example if we put a css inside /public/css/app.css it will be automatically accessible as http://&lt;address&gt;/css/app.css.

<strong>Update:  </strong>static files are served through StaticMiddleware that is added automatically only if environment is ‘development’.
Environment is set using the <strong><em>TRAFFIC_ENV</em></strong> variable, so if you set <em><strong>TRAFFIC_ENV=production, </strong></em>you have to manually add the StaticMiddleware
<pre class="brush: go java">   
if traffic.Env() == "production" {
    router.Use(traffic.NewStaticMiddleware(traffic.PublicPath()))
}
</pre>
The Traffic router has builtin handlers for 404 and 500 erros that can be customized.
<pre class="brush: go java">// Custom not found handler
router.NotFoundHandler = NotFoundHandler

func NotFoundHandler(w traffic.ResponseWriter, r *traffic.Request) {
    w.Render("404")
}

// Custom error handler
router.ErrorHandler = ErrorHandler

func ErrorHandler(w traffic.ResponseWriter, r *traffic.Request) {
    w.Render("500")
}
</pre>
<h3>Conclusions</h3>
<a href="https://github.com/pilu/traffic/">Traffic</a> is a young framework specifically crafted for small to medium applications.
I was able to create the demo app <a href="https://github.com/wstucco/purrraceholder">Purrraceholder</a> (read it with a japanese accent) in a couple of hours, without previous knowledge of Traffic internals.
I know there are people that can write a blog in 15 minutes, but I think hours is a more realistic time frame and, most of all, you are really in control of what’s happening.
If you wanna play with <a href="https://github.com/pilu/traffic/">Traffic</a>, you can start by forking <a href="https://github.com/wstucco/purrraceholder">Purraceholder</a> and adding some features.
These are the firsts that come to mind:
<ul>
 	<li>add more cats, there are never enough cats on the internet</li>
 	<li>treat special cases with special cats: longcat for vertical images, monorail cat for horizontal ones</li>
 	<li>add support for text overlay</li>
</ul>
happy coding with <a href="https://github.com/pilu/traffic/">Traffic</a>.