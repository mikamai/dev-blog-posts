---
ID: 75
post_title: 'Writing Ruby extensions in Go &#8211; an in depth review'
author: Massimo Ronca
post_date: 2015-10-12 00:51:53
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/12/writing-ruby-extensions-in-go-an-in-depth-review/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/130986121064/writing-ruby-extensions-in-go-an-in-depth-review
tumblr_mikamayhem_id:
  - "130986121064"
---
For a very long time the only way to extend Ruby with native estensions has been using <code>C</code>.
I don’t have the numbers with me, but I guess <code>C</code> is not the first choice for Ruby programmers when they have to pick up a secondary/complentary language.
So I started investigating the possibility of writing them in other languages, taking advantage of the favorable moment: in the past 3-4 years we had an explosion of new languages that compile down to native code and can be easily used to produce a <code>C</code> equivalent shared library that Ruby can pick up and load.

My main goal was to find languages that a Ruby programmer would understand effortlessly or with a minimum investment.
This first episode will focus on Go.

<!--more-->

<h2 id="toc_0">You will need Go 1.5 or above</h2>
Up until version 1.4, there was really no point in building a native extension in Go, you’d have to create a <code>C</code> proxy function for every Go function being called, at the point that there was literally no benefit compared to writing everything in pure <code>C</code>.
With version 1.5, Go made a step forward, introducing the support for building shared objects; this opens up a lot of new possibilities for writing shared code that gets executed outside the Go environment, including Ruby.
<h2 id="toc_1">Done in 60 seconds</h2>
This is all your <q>hello world</q> extension will just be:

Compile it with

Have I already told you that <a href="https://github.com/ffi/ffi/wiki">FFI</a> is awesome?

There are a few things you need to pay attention to:
<ul>
 	<li>It is better to put <code>import "C"</code> on its own line, separated from other imports, we’ll see why in a moment</li>
 	<li><code>//export</code> is a special <a href="https://golang.org/cmd/cgo/">Cgo</a> comment that instructs the compiler to emit a <code>C</code> function with the same name and parameters. The name of the exported <code>C</code> function must match the name of the Go function or it will fail. The comment must start exactly with <code>//export</code>, no spaces anywhere.</li>
 	<li>a <code>func main</code> is required, but then ignored</li>
</ul>
<h2 id="toc_2">Autarchy, or writing extensions the hard way</h2>
This could be all, but of course it is not.
Writing a Ruby extensions in a different language is one thing, writing it because the other language really offers some noticeable advantage, is entirely different.
As a base for this series of articles, I will port the <a href="https://github.com/SamSaffron/fast_blank"><code>fast_blank</code></a> <code>C</code> extension.
I’ve chosen it for very simple reasons:
<ul>
 	<li>it is deadly easy to port, even to languages that I’m not particularly familiar with</li>
 	<li>it bundles a benchmark suite, so we can measure the performance gains/losses</li>
 	<li>it is real life code that’s been downloaded a quarter million times so far</li>
</ul>
<blockquote>For the impatients: you can clone the <a href="https://github.com/mikamai/go_fast_blank">repo of the <code>go_fast_blank</code> gem on Github</a>.</blockquote>
Even if <code>FFI</code> is great, it is still a dependency, that needs to be installed and maintained.
<code>C</code> extensions are usually self contained and take advantage of the <code>MRI</code> <code>C</code> programming interface to build the necessary exported APIs.
Go has a very good support for interfacing to <code>C</code> as you can see in this file

There are some new tricks here, that need an explanation.
Just before <code>import "C"</code> we find what’s called <q>preamble</q> by Cgo, and it’s just <code>C</code> code that the compiler put at the beginning of the generated file, as it is, before starting the compilation.
<blockquote><strong>IMPORTANT</strong>: there must be no empty line between the end of the preamble and the <code>import "C"</code> directive or the compilation will fail.
that’s the reason why I told you to put <code>import "C"</code> on its own line.</blockquote>
In other words, the generated code will start with

You probably have guessed that the <code>C</code> prefix gives access to the <code>C</code> world directly from Go, with some added benefit: for example the <code>C.CString</code> converts Go native strings to <code>C</code> (<code>char*</code>) strings. This function allocates memory, so you <em>must</em> free the memory using <code>C.free</code>.

We do that in <code>defer C.free(unsafe.Pointer(cs))</code>, that tells Go to free the memory as soon as the surroinding function returns and is a very common pattern.
The pointer to the string is declared as <code>unsafe.Pointer</code> because it does not belong to the Go world.

Another thing you might have noticed is the reverse twin function <code>C.GoString</code> that takes a <code>C</code> string and returns a Go string. In this case no memory is allocated, the GC takes care of everything, so no freeing is required.

Some of the code just refers to the <code>MRI</code> programming interface, defined in <code>ruby.h</code> and related headers.
For example <code>C.VALUE</code> is a macro for various types of pointers to data structures (from strings to function pointers) and <code>C.rb_define_method</code> defines a new method.
It takes four parameters: the class to which the method belongs to (in this case <code>C.rb_cString</code> which is the Ruby equivalent of the builtin <code>String</code> class), the name of the method (in this case <code>blank?</code>) a callback and the number of arguments (zero in our case).

Basically we are writing something like
<pre class="line-numbers"><code class="language-ruby">class String
    def blank?
    ...
    end
end</code></pre>
The third argument of <code>C.rb_define_method</code> is the <code>C</code> function that gets executed when the method is invoked on the Ruby side.
The Go runtime and the <code>C</code> code are executed in different threads, with different stacks, it it prohibited to pass a pointer to Go code to <code>C</code>, so we can’t take a pointer to a Go function and simly pass it to <code>C</code>, because it won’t work.

There is a workaround, we can <code>//export</code> our Go function and pass the pointer to it instead, after casting it to <code>*[0]byte</code> (the Go equivalent of <code>void*</code>): <code>(*[0]byte)(C.go_fast_blank)</code>.
There is only a small problem: <code>C.go_fast_blank</code> does not exists until the <code>C</code> files are compiled, so we cannot implicitly refer to it.
We need to add a forward declaration to tell the compiler we know this function exists and it’s imlpemented somewhere else outside here.
That’s what the line <code>extern inline VALUE go_fast_blank(VALUE);</code> is for, and it’s a standar declaration for <code>rb_define_method</code> callbacks (<code>function_name(VALUE) -&gt; VALUE</code>).
The rest is quite straightforward:
<pre class="line-numbers"><code class="language-none">gs := C.GoString(C.rb_string_value_cstr(&amp;s))</code></pre>
Take a <code>VALUE</code> convert it to a <code>C</code> string and then convert the result to a Go string.
<pre class="line-numbers"><code class="language-none">if gs == "" || strings.TrimLeftFunc(gs, unicode.IsSpace) == "" {
    return C.Qtrue
}

return C.Qfalse</code></pre>
If the string is empty or after removing all the unicode spaces on the left side, it is still empty, we found a blank string. Otherwise we return false.
<code>Qtrue</code> and <code>Qfalse</code> are just two <code>C</code> #defines that map to a Ruby boolean.

Each extension has an <code>Init</code> function, and it’s automatically called when the extension is <code>require</code>d.
The name of the function must be <code>Init_#{extension_name}</code>, in our case <code>Init_go_fast_blank</code>.

Last but not least, to compile our self contained extension, we need to pass some flags to the compiler. We’ll do it using a specific Cgo comment: <code>#cgo</code>.
just before the <code>#include "ruby.h"</code> add these lines
<pre class="line-numbers"><code class="language-C">
#cgo: CFLAGS: -I#{RbConfig::CONFIG['rubyhdrdir']} -I#{RbConfig::CONFIG['rubyarchhdrdir']}
#cgo: LDFLAGS: -L#{RbConfig::CONFIG['libdir']} #{RbConfig::CONFIG['LIBRUBYARG']}
</code></pre>
The interpolation codes must be replaced with the actual value.
I’ve put it there for reference.
Hint: the output must have <code>.bundle</code> extension and not <code>.so</code> as we did before, othewise Ruby will refuse to laod it.
In the <a href="https://github.com/mikamai/go_fast_blank">repo of the <code>go_fast_blank</code> gem</a> you can find an ad hoc <code>extconf.rb</code> that will take care of everything.
<h2 id="toc_3"><a href="https://www.youtube.com/watch?v=bs56ygZplQA">Race for the prize</a></h2>
Now we have a compiled, native, Ruby extension, launch <code>irb</code> and type
<pre class="line-numbers"><code class="language-ruby">2.2.2 :001 &gt; require 'go_fast_blank'
go_fast_blank init
 =&gt; true </code></pre>
You should see our extension announcing itself by printing <code>go_fast_blank init</code>.
It’s time to measure the performances and comment the results.
After launching <code>benchmark</code>, the numbers are:

<img title="Ruby VS Go" src="http://i.imgur.com/4YddNT2.png" alt="Ruby VS Go" />
Go is between 2 and 4 times slower than the original Ruby implementation!

<img title="Y U SO SLOW?" src="http://i.imgur.com/48BOGlI.png" alt="GO, Y U SO SLOW" />

Well, first of all Go is not only slower than Ruby, but it’s plateuing, looks like the speed
of the Go extension is not influenced byt the length of the string, but it’s just going as fast as it can,
and that is the fastest speed possible.
A loss in performance was to be expected, Go <span style="text-decoration: line-through;">does not generate binary executable code, but code for its VM</span> generate executable code that have to deal with its runtime internals, it is somewhat in between Java and compiled languages.
But honestly actually running slower than Ruby code was a real surprise.
According to this <a href="https://groups.google.com/forum/#!msg/golang-nuts/RTtMsgZi88Q/61hgyGSkWiQJ">Russ Cox answer</a>, calling <code>C</code> from Go has an aoverhead
similar to calling ten Go functions, looks like Go is one of those languages that can run faster ported code, than calling
the <code>C</code> implementation.
If every function call counts for ten, it’s no wonder that calling it thousands of time in a tight loop, would cause such
a tremendous loss in performances.
To test this assumption, I moved the tight loop from Ruby to the Go extension: I ran the same comparison one thousand times
inside a Go loop and the same I did on the Ruby side.
These are the new results:

<img src="http://i.imgur.com/hCDdCfA.png" alt="Ruby VS Go updated" />

This time Go ran a bit faster, but with long strings the same slowness arises.
I suspect the conversions routines from Ruby VALUE to Go strings are responsible for most of the overhead.
Removing it from the equation gave me much better results (between 300 and 16 times faster than Ruby), but it’s a low level optimization that makes sense only for three lines functions that are called over and over again, like this one.
These numbers are not to be taken as a real benchmark, they are just the results of a micro benchmark and are not representetive of real performances in a real world application.
But they clearly show that running Go in a tight loop has a serious performance overhead, while if you delegate to
Go some heavy lifting work, it can give some performance boost.
<h2 id="toc_4">Conclusions</h2>
Writing Ruby extensions in Go, especially in conjunction with the great <code>FFI</code> library, can be real fun.
You got the feeling of <em><q>scripting Ruby</q></em> without any of the drawbacks of writing low level <code>C</code> code.
Writing completely auto contained extensions, it’s a lot more work, but it’s more tedious than hard.
The situation could improve vastly when someone will wrap the Ruby programming interface in a nice Go package to hide the <code>C</code> inheritance and maybe
write a <a href="https://blog.golang.org/generate"><code>go:generate</code></a> plugin to automate all the boilperplate code (for example
exporting the functions to <code>C</code>). But in the end it is still a lot easier than writing pure <code>C</code>.
Perfomance wise though, I’m doubtful that you could have some gain just by rewriting parts of you app in Go.
It is in fact quite possibly the opposite.
Go has a performance problem when intercating with <code>C</code> and it’s by design.

However, there could be patterns where Go could be really helpful.
I’m sure Go channles and concurrency are worth exploring.
Maybe in a next episode.