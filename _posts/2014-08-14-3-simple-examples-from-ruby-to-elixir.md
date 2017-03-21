---
ID: 216
post_title: 3 Simple examples from Ruby to Elixir
author: Elia Schito
post_date: 2014-08-14 11:43:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/08/14/3-simple-examples-from-ruby-to-elixir/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/94717466094/3-simple-examples-from-ruby-to-elixir
tumblr_mikamayhem_id:
  - "94717466094"
---
<p><em>In this post we&rsquo;re gonna see how to transform a simple script from Ruby to <a href="http://elixir-lang.org">Elixir</a>.</em></p>

<h2>Installing Elixir</h2>

<p>The first thing you need is to have Elixir installed on your box, the instructions are dead simple and you can <br />
find them in the official <a href="http://elixir-lang.org/getting_started/1.html#1.1-installers">Getting Started</a> page.<br />
For example on OS X is as simple as <code>brew update; brew install elixir</code>.</p>

<h2>The Ruby Script</h2>

<p>The script is the one I use to fire up <a href="http://dev.mikamai.com/post/77705936763/pimp-my-textmate2-ruby-edition">my editor</a> adding support for the <code>file:line:column</code><br />
format that is often found in error stacktraces. I keep this script in <code>~/bin/e</code>.</p>
    <pre><code class="ruby">#!/usr/bin/env ruby

command = ['mate']

if ARGV.first
  file, line_and_column = ARGV.first.split(':', 2)

  command &lt;&lt; file
  command += ['-l', line_and_column] if line_and_column
end
command &lt;&lt; '.' if command.size == 1
exec *command
</code></pre>

<h2>Take 1: Imperative Elixir</h2>

<p>As we all know, no matter the language, you can keep your old style. In this first example we&rsquo;ll see the same</p>
    <pre><code class="elixir">#!/usr/bin/env elixir

if System.argv != [] do
  [file| line_and_column] = String.split(hd(System.argv), ":", parts: 2)
  args = [file]

  if line_and_column != [] do
    args = args ++ ["-l"| line_and_column]
  end
else
  args = ["."]
end
System.cmd("mate", args)
</code></pre>

<h3>The “Guillotine” operator</h3>

<p>The first thing we notice the change in syntax for the splat assignment:</p>
    <pre><code class="ruby"># Ruby
a, b = [1,2,3]
a # =&gt; 1
b # =&gt; 2
</code></pre>
    <pre><code class="elixir"># Elixir
[a| b] = [1,2,3]
a # =&gt; 1
b # =&gt; [2,3]
</code></pre>

<p>The <code>|</code> operator in Elixir will in fact take out the head of the list and leave the rest on its right. <br />
It can be used multiple times:</p>
    <pre><code class="elixir">[a| [b| c]] = [1,2,3]
a # =&gt; 1
b # =&gt; 2
c # =&gt; [3]
</code></pre>

<p>what happens here is that the list that in the first example was <code>b</code> is now beheaded again. <br />
If instead we wanted <code>c</code> to equal <code>3</code> the assignment would look like this:</p>
    <pre><code class="elixir">[a| [b| [c]]] = [1,2,3]
a # =&gt; 1
b # =&gt; 2
c # =&gt; 3
</code></pre>

<p>As we can see Elixir matches the form of the two sides of the assignments and extracts values and variables accordingly.</p>

<h3>Other notes</h3>

<p>Let&rsquo;s see a couple of other things that we can learn in this simple example</p>

<h4>List concatenation: <code>++</code></h4>

<p>The <code>++</code> operator simply concatenates two lists:</p>
    <pre><code class="elixir">a = [1,2] ++ [3,4]
a # =&gt; [1,2,3,4]
</code></pre>

<h3>Double quoted <code>"strings"</code></h3>

<p>All strings need to be double quoted in Elixir, as single quotes are reserved for other uses. <br />
I make the mistake of using single quotes all the time. Probably that&rsquo;s the price for being a <br /><a href="https://twitter.com/RoflscaleTips/status/46241818270109698">ROFLScale expert</a>.</p>

<h2>Take 2: First steps in Pattern Matching</h2>

<p>With this second version we&rsquo;re gonna see the pattern matched <code>case</code>. </p>

<p>Notice anything?</p>

<p>Yes. All <code>if</code>s are gone.</p>
    <pre><code class="elixir">#!/usr/bin/env elixir

args = System.argv
args = case args do
  [] -&gt; []
  [""] -&gt; []
  [path] -&gt; String.split(path, ":", parts: 2)
end

args = case args do
  [] -&gt; ["."]
  [file] -&gt; [file]
  [file, ""] -&gt; [file]
  [file, line_and_column] -&gt; [file, "-l", line_and_column]
end

System.cmd("mate", args)
</code></pre>

<p>We now reduced the whole program to a couple of switches that will <em>route</em> the input and <em>transform</em> it <br />
towards the intended result.</p>

<p>That&rsquo;s it. No highlights for this implementation. Just a LOLCAT.</p>

<p><img src="http://cl.ly/image/2p0Y0z023L3f/nuWh9.gif" alt="cat getting scared for no reason" /></p>

<h2>Take 3: Modules and pipes</h2>
    <pre><code class="elixir">#!/usr/bin/env elixir

defmodule Mate do
  def open(argv), do: System.cmd("mate", argv |&gt; parse_argv)

  def parse_argv([]), do: ["."]
  def parse_argv([options]) do
    [file| line_and_column] = String.split(options, ":", parts: 2)
    [file| line_and_column |&gt; line_option]
  end

  def line_option([]),                do: []
  def line_option([""]),              do: []
  def line_option([line_and_column]), do: ["-l", line_and_column]
end

Mate.open System.argv
</code></pre>

<h3>Module and defs</h3>

<p>As you have seen we have now organized out code into a module and moved stuff to defined module <br />
functions. The same function can be defined multiple times, Elixir will take care of matching the arguments <br />
you pass to a function to the right.</p>

<p>Let&rsquo;s review the two forms of function definition:</p>
    <pre><code class="elixir">defmodule Greetings do
  # extended
  def hello(name) do
    IO.inspect("hello #{name}")
  end

  # onliner
  def hello(), do: IO.inspect("hello world!")
end

Greetings.hello "ppl" # =&gt; "hello ppl"
Greetings.hello       # =&gt; "hello world!"
</code></pre>

<p>Be sure to remember the comma before <code>do:</code> otherwise Elixir will complaint.</p>

<h3>The <code>|&gt;</code> pipe operator</h3>

<p>If you&rsquo;re familiar with commandline piping you&rsquo;ll fell like at home with the pipe operator. <br />
Basically it will take the result of each expression and pass it as <strong>the first</strong> argument of the next one.</p>

<p>Let&rsquo;s see an example:</p>
    <pre><code class="elixir">"hello world" |&gt; String.capitalize |&gt; IO.inspect # =&gt; "Hello world"
</code></pre>

<p>That is just the same of:</p>
    <pre><code class="elixir">s = "hello world"
s = String.capitalize(s)
s = IO.inspect(s)
s # =&gt; "Hello world"
</code></pre>

<p>or </p>
    <pre><code class="elixir">IO.inspect(String.capitalize("hello world")) # =&gt; "Hello world"
</code></pre>

<p>Where the latter is probably the least comprehensible to human eyes</p>

<h2>What’s next?</h2>

<h3>Playing</h3>

<ul><li><a href="http://elixir-lang.org/getting_started/1.html#1.6-interactive-mode">Play around with the interactive console <code>iex</code></a></li>
<li><a href="http://elixir-lang.org/getting_started/2.html">Read the Getting Started guide from the Official site</a></li>
</ul><h3>Studying</h3>

<ul><li><a href="https://twitter.com/mikamai">Look out for my next article on Elixir by following our twitter account</a></li>
<li><a href="http://pragprog.com/book/elixir/programming-elixir">Read Programming Elixir book by the PragProgs</a></li>
</ul><h3>Watching</h3>

<ul><li><a href="https://www.youtube.com/watch?v=aZXc11eOEpI&amp;list=PLE7tQUdRKcyakbmyFcmznq2iNtL80mCsT&amp;index=3">Watch Elixir&rsquo;s creator keynote from ElixirConf2014</a></li>
<li><a href="https://www.youtube.com/playlist?list=PLE7tQUdRKcyakbmyFcmznq2iNtL80mCsT">Watch some other presentations from ElixirConf2014</a></li>
</ul>