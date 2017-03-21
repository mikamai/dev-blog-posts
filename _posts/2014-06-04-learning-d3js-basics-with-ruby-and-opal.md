---
ID: 243
post_title: >
  Learning D3.js basics with Ruby (and
  Opal)
author: Elia Schito
post_date: 2014-06-04 16:51:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/04/learning-d3js-basics-with-ruby-and-opal/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/87806500959/learning-d3js-basics-with-ruby-and-opal
tumblr_mikamayhem_id:
  - "87806500959"
---
<p><em>I always wanted to learn D3.js, problem is JavaScript is too awesome and that kept turning me off… till now!</em></p>

<p>Let&rsquo;s take a random tutorial that we will attempt to translate into Ruby, for example this: <a href="http://bl.ocks.org/mbostock/3883245">http://bl.ocks.org/mbostock/3883245</a>.</p>

<p>A couple of thing we need to bear in mind are these: D3 is not object-oriented and it&rsquo;s a pretty complex library. We&rsquo;re expecting some problems.</p>

<p>To have something to compare it to we may note, for example, that JQuery is basically a class (<code>$</code>) that exposes methods that returns other instances of the same class. That makes things easy enough when we try to use it from an OO language like Ruby.</p>

<h2>Part I — The Setup</h2>

<p>Following the tutorial, let&rsquo;s setup the HTML page:</p>
    <pre><code class="html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;meta charset="utf-8"&gt;
&lt;style&gt;
/* …uninteresting CSS from the tutorial here… */
&lt;/style&gt;
&lt;body&gt;
&lt;script src="http://d3js.org/d3.v3.min.js" charset="utf-8"&gt;&lt;/script&gt;

&lt;!-- OUR ADDITIONS HERE --&gt;
&lt;script src="http://cdn.opalrb.org/opal/current/opal.js"&gt;&lt;/script&gt;
&lt;script src="http://cdn.opalrb.org/opal/current/native.js"&gt;&lt;/script&gt;
&lt;script src="http://cdn.opalrb.org/opal/current/opal-parser.js"&gt;&lt;/script&gt;

&lt;script type="text/ruby" src="./app.rb"&gt;&lt;/script&gt;

&lt;/body&gt;
&lt;/html&gt;
</code></pre>

<p>As you probably have noted we added a couple of libraries from the <a href="http://cdn.opalrb.org/">Opal CDN</a>.</p>

<p>First we added <code>opal.js</code>, that is the runtime and core library that are necessary to run code compiled with Opal.</p>

<p>Then there&rsquo;s <code>native.js</code> that we&rsquo;ll use to interact with native objects (<a href="http://dev.mikamai.com/post/79398725537/using-native-javascript-objects-from-opal">more details in this other post</a>).</p>

<p>And last we have <code>opal-parser.js</code> and <code>app.rb</code> (that is declared as <code>type="text/ruby"</code>). The parser will look for all script tags marked as <code>text/ruby</code> and will load, parse and run them.</p>

<p>In additon we&rsquo;re also providing a <code>data.tsv</code> file as described in the tutorial.</p>

<h2>Part II — Code Ungarbling</h2>

<p><strong>About the process of translating</strong></p>

<p>The approach I used to translate this code is quite simple, I copy/pasted all the JavaScript from the tutorial into <code>app.rb</code> and commented it out. Then I translated one line (or sentence) at a time tackling errors as they came.</p>

<h3>Using Native</h3>

<p>The first thing we need to do is to wrap the <code>d3</code> global object with <code>Native</code>, so that is ready for Ruby consumption.</p>
    <pre><code class="ruby">d3 = Native(`window.d3`)
</code></pre>

<p>Now let&rsquo;s look at the first chunk of code</p>

<p><strong>original code:</strong></p>
    <pre><code class="js">var parseDate = d3.time.format("%d-%b-%y").parse;

var x = d3.time.scale()
    .range([0, width]);

var y = d3.scale.linear()
    .range([height, 0]);

var xAxis = d3.svg.axis()
    .scale(x)
    .orient("bottom");

var yAxis = d3.svg.axis()
    .scale(y)
    .orient("left");

</code></pre>

<p><strong>converted code</strong></p>
    <pre><code class="ruby">d3 = Native(`window.d3`)
time  = d3[:time]
scale = d3[:scale]
svg   = d3[:svg]

date_parser = time.format("%d-%b-%y")

x = time.scale.range([0, width])
y = scale.linear.range([height, 0])

x_axis = svg.axis.scale(x).orient(:bottom)
y_axis = svg.axis.scale(y).orient(:left)
</code></pre>

<p>By trying to run this code we discover early on that somehow <code>Native</code> is failing to deliver calls via method missing.</p>

<p>The reason is that bridged classes are a kind of their own and turns out that D3 tends to return augmented anonymous functions and arrays. Now both <code>Array</code> and <code>Proc</code> are bridged classes. That means that they are the same as their JS counterparts: <code>Array</code> and <code>Function</code>.</p>

<p><code>Native</code> will have no effect on bridged classes (no wrapping) and therefore all calls to properties added by D3 will end in calls on <code>undefined</code>.</p>

<p>To solve this we&rsquo;ll manually expose the methods on those classes, the final code looks like this:</p>
    <pre><code class="ruby">module Native::Exposer
  def expose(*methods)
    methods.each do |name|
      define_method name do |*args,&amp;block|
        args &lt;&lt; block if block_given?
        `return #{self}[#{name}].apply(#{self},#{args.to_n})`
      end
    end
  end
end

Proc.extend Native::Exposer
Array.extend Native::Exposer

Proc.expose :range, :axis, :scale, :orient, :line, :x, :y, :parse, :domain
Array.expose :append, :attr, :call, :style, :text, :datum
</code></pre>

<p>The complete code can be found in <a href="https://gist.github.com/elia/88a4554653df42a6f7c3">this gist</a>.</p>

<p><strong>UPDATE:</strong> <a href="http://ju-jist.herokuapp.com/elia/88a4554653df42a6f7c3/index.html">see the code in action on Ju-Jist</a>, and <a href="http://dev.mikamai.com/post/87984833354/enter-ju-jist-the-gist-runner">read about building ju-jist in this article</a></p>

<h3>A better solution</h3>

<p>Let&rsquo;s hope that in the future the Opal team (me included) will add method missing stubs<sup id="fnref1"><a href="#fn1" rel="footnote">1</a></sup> to all bridged classes (instead of just adding them <code>BasicObject</code>).</p>

<h2>Conclusion</h2>

<p>We probably didn&rsquo;t learn very much about how to use D3.js, but we discovered a bit of its internals, exposing an interesting style of JavaScript.</p>

<div class="footnotes">
<hr><ol><li id="fn1">
<p>(which is the underlying mechanism that powers <code>method_missing</code> support in Opal) <a href="#fnref1" rev="footnote">↩</a></p>
</li>

</ol></div>