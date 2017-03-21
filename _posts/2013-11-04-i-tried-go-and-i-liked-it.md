---
ID: 323
post_title: I tried Go and I liked it
author: Massimo Ronca
post_date: 2013-11-04 09:25:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/04/i-tried-go-and-i-liked-it/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/65984358855/i-tried-go-and-i-liked-it
tumblr_mikamayhem_id:
  - "65984358855"
---
<h1>I tried Go and I liked it</h1>
<img title="Gopher" src="http://upload.wikimedia.org/wikipedia/commons/2/23/Golang.png" alt="Gopher in all of its glory" />

Say hello to <a href="http://golang.org/doc/gopher/">Gopher</a>.
Gopher is the mascotte of a new language from <a href="http://google.com">Google</a>, called <a href="http://golang.org">Go</a>, with the capital <strong>G</strong>.
Not a very clever name from a search engine company, if you ask me, but that’s probably the only bad thing you will hear about it.
<blockquote><strong>Hint:</strong> use the word golang to search on Google</blockquote>
<h2>A brief introduction</h2>
Created inside Google by <a title="Ken Thompson" href="http://en.wikipedia.org/wiki/Ken_Thompson">Ken Thompson</a> and <a title="Rob Pike" href="http://en.wikipedia.org/wiki/Rob_Pike">Rob Pike</a>, fathers of Unix and UTF-8, to overcome the limitations of C++ (compile times being the most annoying), Go is a <em>concurrent</em>, <em>garbage-collected</em> language with <em>fast compilation</em>.

Its real strength is just simplicity.
As Rob Pike once said, <a href="http://commandcenter.blogspot.it/2012/06/less-is-exponentially-more.html">less is exponentially more</a> and I strongly agree with him.
<h3>Features</h3>
<ul>
 	<li>blazing fast compilation speed</li>
 	<li>statically compiled binaries (the result is a single binary with no external dependencies)</li>
 	<li>type safe, statically typed with some type inference support. More errors get caught at compile time, less time is spent debugging</li>
 	<li><a href="http://talks.golang.org/2012/splash.article#TOC_14.">garbage collected</a> with support for pointers, but no pointer arithmetics (for safety and good health of programmers minds).</li>
 	<li>strict compiler: <a href="http://golang.org/doc/effective_go.html#blank_unused">you can’t declare a variable or import a package without using it</a></li>
 	<li>concurrency and parallelism through <a href="http://golang.org/doc/effective_go.html#goroutines">goroutines</a>. Goroutines are one of the peculiarities of Go, they are a cheap, lightweight construct built on top of threads, that run concurrently with other goroutines. If you have more than one core processor, they also run in parallel, in a completely transparent way for the programmer. Communication is managed sending messages through <a href="http://golang.org/doc/effective_go.html#channels">channels</a> which are basically type safe queues.</li>
 	<li>Object orientation but <strong>no</strong> classes. Any type can be an object.</li>
 	<li><strong>No type inheritance</strong> in favour of <em><a href="http://talks.golang.org/2012/splash.article#TOC_15.">composition</a></em> and <em><a href="http://en.wikipedia.org/wiki/Duck_typing">duck typing</a></em>. IS-A relationships are banned!</li>
 	<li><a href="http://golang.org/doc/effective_go.html#multiple-returns">multiple return values</a></li>
 	<li><a href="http://golang.org/pkg/">rich</a> standard library.</li>
 	<li>a powerful set of <a href="http://golang.org/doc/cmd">command line tools</a> including one to <a href="http://golang.org/cmd/gofmt/">enforce coding conventions</a> and one for <a href="http://blog.golang.org/godoc-documenting-go-code">automatic code documentation</a>.
Many IDE that support Go, launch gofmt just before save, to ensure that every Go file obey the rules.</li>
 	<li>last, but not least, cross compiling. Go compiler can create binaries for platforms/architectures different from the one it is running, provided <a href="http://golang.org/doc/install#requirements">the platform is supported</a>.</li>
</ul>

<!--more-->

<h3>Installing Go</h3>
You can download Go from the <a href="https://golang.org">golang.org website</a>.
There are packages for many different platforms (Linux, Windows, Mac OS, BSD) and architectures (x86, x64, ARM).
Detailed instructions can be found <a href="http://golang.org/doc/install">here</a>.

Basically you need to create two environment variables:
<ul>
 	<li>GOROOT which is the system wide Go root folder (it should be configured automatically by the installer)</li>
 	<li>GOPATH that will contain all your code and all the packages you are going to install</li>
 	<li>it is optional, but recommended, to add $GOPATH/bin to the $PATH variable</li>
</ul>
<h3>First steps and getting help</h3>
You should now have a working installation of the Go development environment, but if you come from Ruby or Python, you might get lost while reading someone else’s code or trying to figure out which is the idiomatic way to solve a problem in Go.
No worries, Go has an answer for that too.
Included in the package there is <a href="http://godoc.org/code.google.com/p/go.tools/cmd/godoc">godoc</a>.
When launched with the -http param, godoc act as a web server that present the documentation in form of web pages.
This is a typical godoc summoning ritual
<pre class="brush: bash">$ godoc -http=:60666
$ open http://localhost:60666
</pre>
My advice is to read carefully <a href="http://golang.org/doc/code.html">How to write Go code</a> and <a href="http://golang.org/doc/effective_go.html">Effective Go</a> before starting writing Go code.
Go does not provide REPL, but you can try your snippets in the <a href="http://play.golang.org/">Playground</a>.
<blockquote><strong>NOTE:</strong> every package you install with the go tool or write by yourself, will update the documentation as well.</blockquote>
<h2>SHUT UP! SHOW US THE CODE!</h2>
Ok!
<pre class="brush: go">package main

import (
    "fmt"
    "net/http"
)

func main() {

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello World!")
    })

    fmt.Println("Server running at http://localhost:8080/")
    fmt.Println("hit CTRL+C to shut it down")
    http.ListenAndServe(":8080", nil)
}
</pre>
This is our first Go program.
Every client that connects to localhost, port 8080, will receive the <em>“Hello World!”</em> message.
The example looks a lot like the <em>Node.js</em> <a href="http://nodejs.org/about/">hello world web serve example</a>, but unlike <em>Node.js</em>, this code is already multithreaded. Every new client is served by a goroutine, that the net/http starts behind the scenes, taking advantage of concurrency and multiple cores, if present.
Talking about simplicity, is there anything simpler than that?
<blockquote><strong>Note:</strong> To run the code you have to save it in a file and then execute go run &lt;filename&gt; from the console</blockquote>
Let’s explore the code above in more detail:
<h3>Packages and imports</h3>
The first statement in a Go program must always be
<pre class="brush: go">package name
</pre>
Executable packages have to reside in a package called <strong><em>main</em></strong>.
The entry point function is called <strong><em>main</em></strong> as well.

Next we find the import section, you can import packages by declaring your intentions with the <strong><em>import</em></strong> directive.
This is the first Go idiom we encounter: <em>statement grouping</em>.
You can import packages (or declare variables) one per line, like in
<pre class="brush: go">import "fmt"
import "net/http"
</pre>
or you can group them together, surrounding the imports with parenthesis, like in our example.
<h3>Variables and type inference</h3>
Variables are declared <em>name first, then type</em>.
If you <em>declare and assign</em>, you can let the compiler infer the type.
<pre class="brush: go">// when variables are declared without assignment, Go assign them a default value:
// zero for numbers, empty string for strings, nil for pointers and nullable types
var a string
var b int

// type inference
c := "Hello, World!"  // := operator is available only inside function body
var s = "a string"    // this pattern is available outside functions
</pre>
Of course <em>statement grouping</em> is available too
<pre class="brush: go">var (
    s string
    i int
    f float64
)
</pre>
Don’t worry about lining up the elements between the brackets, gofmt will take care of it for you.
<h3>Anonymous functions and OO</h3>
Go support <a href="http://golang.org/ref/spec#Function_literals">anonymous functions</a> and high order functions.
Functions are first class citizens in Go and can be assigned and carried around like regular variables.
<pre class="brush: go">log := func(s string) {
    fmt.Printf("[%s] %s", time.Now(), s)
}

log("Hello World!")

// sample output
// [2009-11-10 23:00:00 +0000 UTC] Hello World!

// a more complex example

type logLevel string 
type logger func(string) // create a new type for a function that takes a string as input 

getLogger := func(l logLevel) logger {
    return func(s string) {
        fmt.Printf("[%s] %s: %sn", time.Now(), l, s)
    }
}

err := getLogger("Err")
warn := getLogger("Warn")
info := getLogger("Info")

err("File not found")
warn("Timezone is not set")
info("loggin' some stuff")

// sample output
// [2009-11-10 23:00:00 +0000 UTC] Err: File not found
// [2009-11-10 23:00:00 +0000 UTC] Warn: Timezone is not set
// [2009-11-10 23:00:00 +0000 UTC] Info: loggin' some stuff
</pre>
Let’s now improve our web server with new functionalities.
We want to send out an X-Powered-By header with the name of our web server.
What we need to do is define a new <strong><em>type</em></strong> that act as handler for the requests and will append the new header to the default set of headers.
In Go, an http handler is any object that implements the Handler’s <em>interface</em>, and the Handler’s <em>interface</em> contains only one method: ServeHTTP.
So we are going to create a new object that implements the ServeHTTP method and pass it to ListenAndServe.
Instead of classes, Go uses <strong>structs</strong>.
In this case it is an empty struct, but it can contain any other type as member variables (more on this later).
<pre class="brush: go">// declare the new type Middleware.
// Note: the type can be literally **any** type
// type seq []int is a perfectly legal declaration
// it creates a new type (not an alias) seq that represents a slice of ints
type Middleware struct {
}

// implements the ServeHTTP method as requested by Handler's interface
// notice the syntax: 
// after the keyword func we declare the type that the method will be attached to 
// and then we pass a parameter m that represents our instance variable
// in a very similar way to Python's self
func (m *Middleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Printf("Request to %s is being handled by our middlewaren", r.URL.Path)
    w.Header().Set("X-Powered-By", "mikamai-web-server")
}

// initialize and return a new object literal of type Middleware
// in Go we also declare the return type, after the parameters of the function
// in this case a pointer, denoted by the *, to Middleware type
func NewMiddleware() *Middleware {
    return &amp;Middleware{}
}
</pre>
We now pass the new handler to the listener
<pre class="brush: go">http.ListenAndServe(":8080", NewMiddleware())
</pre>
Every time a client connects, it doesn’t receive any message back, except for the new header.
<img src="http://68.media.tumblr.com/83ea05b44971684313f8d6d1c535b2d9/tumblr_mvhrpwaNvX1rhmakfo1_r1_500.png" alt="X-Powered-By mikamai" />

Nice, but not very interesting, plus we lost the ability to serve the content to the client, cause the handler function we declared before is not getting called.
How can we fix that?
Before explaining how to forward the call to the default handler, we’re going to modify our middleware to do something different.
In addition to adding the header with the artist’s signature, we want to limit the ability to visit our web site through one and only one specified host.
<pre class="brush: go">// we expand our middleware to contain the definition of the single allowed host
type Middleware struct {
    allowedHost string
}

// we modify the initializer function (our constructor) as well
func NewMiddleware(host string) *Middleware {
    return &amp;Middleware{
        allowedHost: host, // trailing comma is required by Go compiler
    }
}
</pre>
Rewrite ServeHTTP to check for the host validity
<pre class="brush: go">func (m *Middleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {

    // strip the port from hostname
    host := strings.Split(r.Host, ":")[0] // import "strings" in order to use Split

    // the signature is sent anyway
    w.Header().Set("X-Powered-By", "mikamai-web-server")

    if host == m.allowedHost {
        fmt.Printf("Request for host %s is being handled by our middlewaren", host)

        // net/http has a default handler called DefaultServerMux
        // we feeded it with an handler for "/" in the first example
        // forward the call to the default handler and send "Hello World!" to the client
        http.DefaultServeMux.ServeHTTP(w, r)
    } else {
        // request is denied, wrong hostname
        fmt.Printf("Request for host %s is strictly forbidden by our middlewaren", host)

        // order is important.If we send data before the header, the server assumes the return code is 200 OK
        w.WriteHeader(403)
        fmt.Fprintf(w, "&lt;h1&gt;Forbidden&lt;/h1&gt;you don't have permission to access host %s", host)
    }
}
</pre>
update the handler’s initialization
<pre class="brush: go">http.ListenAndServe(":8080", NewMiddleware("localhost"))
</pre>
et voilà

<img src="http://68.media.tumblr.com/aedc0f5dd849aae18ef022f7d10f3dad/tumblr_mvhrpwaNvX1rhmakfo2_r1_1280.png" alt="access denied" />

<hr />

<img src="http://68.media.tumblr.com/d3cd5cabdc9bb2a83f12923093d6581f/tumblr_mvhrpwaNvX1rhmakfo3_r1_1280.png" alt="access granted" />

<hr />

<img src="http://68.media.tumblr.com/a472850a96238d71194c7fbb23909ae7/tumblr_mvhrpwaNvX1rhmakfo4_r2_1280.png" alt="console output" />

All the code presented in this article can be downlaoded from <a href="https://gist.github.com/wstucco/7248624">github</a>