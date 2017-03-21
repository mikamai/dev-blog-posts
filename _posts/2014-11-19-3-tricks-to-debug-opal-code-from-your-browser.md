---
ID: 184
post_title: >
  3 Tricks to debug Opal code from your
  browser
author: Elia Schito
post_date: 2014-11-19 16:33:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/19/3-tricks-to-debug-opal-code-from-your-browser/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/103047475349/3-tricks-to-debug-opal-code-from-your-browser
tumblr_mikamayhem_id:
  - "103047475349"
---
<h2>1. Use <code>pp</code></h2>

<p><code>pp</code> stands for Pretty Print and is part of the CRuby standard library. Usually what it does is just reformatting your output to make it readable.</p>
    <pre><code class="ruby">require 'pp'
pp 10.times.map { 'hello' }
</code></pre>

<p>In CRuby would output:</p>
    <pre><code class="text">["hello",
 "hello",
 "hello",
 "hello",
 "hello",
 "hello",
 "hello",
 "hello",
 "hello",
 "hello"]
</code></pre>

<p>Opal instead will just pass the object to <code>console.log()</code>:</p>

<p><img src="http://cl.ly/image/3R0r0v2g2Q1t/Screen%20Shot%202014-11-19%20at%2017.17.09.png" alt="console.log output in Safari's console" /></p>

<h2>2. Call Ruby methods from the browser console</h2>

<p>Another good thing to keep in mind is how methods are mapped into JavaScript, the rule is really simple: <code>$</code> is prefixed to the method name.</p>
    <pre><code class="js">&gt; Opal.top.$puts('hello world');
=&gt; "hello world"

&gt; Opal.Object.$new().$object_id();
=&gt; 123

</code></pre>

<p>Once you know that, you can also learn how to call methods with and without blocks:</p>
    <pre><code class="js">&gt; Opal.send(Opal.top, 'puts', 'hello world');
=&gt; "hello world"

&gt; var object = Opal.send(Object, 'new');
&gt; Opal.send(object, 'object_id');
=&gt; 123

&gt; Opal.block_send([1,2,3], 'map', function(n) { return n*2 });
=&gt; [2,4,6]

&gt; Opal.block_send([1,2,3], 'map', function(n) { return Opal.send(n, '*', 2); });
=&gt; [2,4,6]

</code></pre>

<h2>3. Inspecting instance variables</h2>

<p>Last but not least remember that instance variables are mapped to simple <em>unprefixed</em> properties of the JavaScript object.</p>

<p>Opal code:</p>
    <pre><code class="ruby">class Person
  def initialize(name)
    @name = name
  end
end
</code></pre>

<p>In the console (with JavaScript):</p>
    <pre><code class="js">&gt; var person = Opal.Person.$new('Pippo');
=&gt; #&lt;Person:123 @name="Pippo"&gt;

&gt; person.name
=&gt; "Pippo"
</code></pre>

<h2>Conclusion</h2>

<p>That&rsquo;s all for now, happy hacking!</p>