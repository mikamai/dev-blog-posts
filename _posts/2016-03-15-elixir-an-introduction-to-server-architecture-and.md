---
ID: 44
post_title: 'Elixir: an introduction to server architecture and GenServer behaviour'
author: Andrea Longhi
post_date: 2016-03-15 13:21:02
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/03/15/elixir-an-introduction-to-server-architecture-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/141087465149/elixir-an-introduction-to-server-architecture-and
tumblr_mikamayhem_id:
  - "141087465149"
---
<p>Erlang (and Elixir) strongly advocates concurrency, scalability, and fault tolerance via multiprocessing and the use of specific design patterns based on it.
An Erlang process is not an operative system process: it is faster, lighter, with much smaller memory footprint.
By spawning hundreds, thousands or even millions of these processes you can achieve great scalability, as 
<a href="http://joearms.github.io/2016/03/13/Managing-two-million-webservers.html">this recent post</a> 
explains.</p> 
<p>The server/client architecture builds on such lightweight processes, can handle state, and is one of the keys to the 
great results that can be achieved using Elixir.</p>
<!--more-->

<p>Let&rsquo;s build a server starting from this simple calculator module code:</p>
<pre><code>
defmodule Calculator do
  def calc(value, operator, n) do
    case operator do
      :+ -&gt; value + n
      :- -&gt; value - n
      :* -&gt; value * n
      :/ -&gt; value / n
    end
  end  
end

Calculator.calc 50, :-, 8
# 42
</code></pre>
<p>
This code doesn&rsquo;t handle state, just as you would expect when using a functional language: for multiple operations you 
need to pipe the functions:</p>
<pre><code>Calculator.calc(50, :-, 8) 
|&gt; Calculator.calc(:-, 21) 
|&gt; Calculator.calc(:*,2)</code></pre>

<p>Spawning an Erlang process is very easy, fast and cheap:</p>
 <code> pid = spawn fn -&gt; end</code>
 <p>The variable <code>pid</code> is a reference to the process and can be used to communicate with it. 
 A server is basically a never-ending process
that recursively handles some kind of requests. Let&rsquo;s build some looping process first, with the ability to 
handle state as well. Copy and paste this code in your <code>iex</code> shell:</p>

<pre><code>
defmodule Server do
  def start do
    spawn fn -&gt; loop(1) end
  end

  defp loop(n) do
    IO.puts "looping #{n} times"
    :timer.sleep 1000
    loop n+1
  end
end

Server.start
# looping 1 times
# looping 2 times
# ...
</code></pre>

<p>Every second a new message is printed on the screen: these letters come from the spawned server process, 
which is <em>non blocking</em> (you can still use the repl) and kind of <em>stateful</em> 
(the number is incremented after each iteration).
</p><p>Let&rsquo;s go back to our calculator example. Let&rsquo;s write the <code>CalcServer</code> module that complements the
<code>Calculator</code> core module. Here the <code>loop</code> function will at first handle the incoming 
 messages, then recursively call itself with the new updated state:

</p><pre><code>
defmodule CalcServer do
  import Calculator

  def start do
    spawn fn -&gt; loop(0) end
  end

  def loop value do
    new_value = receive do
      {:value, caller} -&gt; 
        send caller, value
        value
      {operator, n} -&gt; calc(value, operator, n)
    end
    loop new_value
  end
end
</code></pre>
<p>Let&rsquo;s play a little with it:</p>
<pre><code>
calc = CalcServer.start
send  calc, {:+, 23}
send  calc, {:-, 2}
send  calc, {:*, 2}
</code></pre>
<p>The variable <code>calc</code> references the calculator server process (returned by <code>spawn</code>), 
so we can send messages to it. These messages then get pattern-matched in the loop
 <code>receive</code> block, precisely here:</p>
<code>{operator, n} -&gt; calc(value, operator, n)</code>
<p>The subsequent loop function will be called with the calculation result, 
courtesy of the <code>Calculator</code> module and its function <code>calc</code> that has
 been imported using the <code>import Calculator</code> directive.</p>
 <p>The messages we sent up until this point were fire-and-forget, meaning we didn&rsquo;t receive any response from the server.</p>
<p>But now we need to read the total, so we can resort to bidirectional communication. First we need to send
 the appropriate message to the server, which replies to our process with another message that we can read in our
  main process:</p>
<pre><code>
send calc, {:value, self}
receive do
  total -&gt; IO.puts "Current total is #{total}"
end
# Current total is 42
</code></pre>

<p>We have successfully built from scratch a simple server that relies on processes and asyncronous communication.
Of course this is buggy and trivial code, but it was useful to analyze some details behind the GenServer architecture.</p>

<p>Now that we have a grasp of how things works, we can convert the existing code to use the GenServer behaviour. GenServer stands for &ldquo;generic server&rdquo;,
an abstraction/design-pattern that helps build servers with consistent and predictable interface. Here is the new server code:</p><pre><code>
defmodule CalcGenServer do
  use GenServer

  import Calculator

  def start do
    GenServer.start __MODULE__, 0
  end

  def handle_call :value, _, value do
    {:reply, value, value}
  end
 
  def handle_cast {operator, n}, value do
    {:noreply, calc(value, operator, n)}
  end
end
</code></pre>
<p>In order to use the GenServer behaviour we only need to define a few functions: <code>start</code>, with the 
module name to delegate calls and casts (here the current module with <code>__MODULE__</code>), and the start value <code>0</code>.
</p><p>Casts are requests that don&rsquo;t need an answer, fire-and-forget. Calls are requests that require an answer, and here we have
 one example of each: reading the total value is a call, while operations with the calculator are handled with a cast.</p>
<p>Now, here is how we can use the new server:</p>
<pre><code>
{:ok, calc} = CalcGenServer.start
GenServer.cast calc, {:+, 84}
GenServer.cast calc, {:/, 2}
GenServer.call calc, :value 
# 42.0
</code></pre>

<p>This is fairly straightforward, and we don&rsquo;t need to listen explicitly for responses anymore, as we did with the 
 previous example.</p>
<p>In this post we just scratched the surface of GenServer, for a detailed introduction and explanation I strongly recommend reading the official Elixir
<a href="http://elixir-lang.org/docs/stable/elixir/GenServer.html">documentation</a>.<br /> Happy coding!</p>