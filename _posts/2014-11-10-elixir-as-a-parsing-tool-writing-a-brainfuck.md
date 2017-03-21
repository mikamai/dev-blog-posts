---
ID: 188
post_title: 'Elixir as a parsing tool: writing a Brainfuck interpreter, part two'
author: Massimo Ronca
post_date: 2014-11-10 16:55:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/10/elixir-as-a-parsing-tool-writing-a-brainfuck/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/102283561929/elixir-as-a-parsing-tool-writing-a-brainfuck
tumblr_mikamayhem_id:
  - "102283561929"
---
<blockquote>This is the second in a series of articles on building a brainfuck interpreter in Elixir</blockquote>
In the <a href="http://dev.mikamai.com/post/100075543414/elixir-as-a-parsing-tool-writing-a-brainfuck">first part</a> we built a minimal brainfuck interpreter that can already run some basic program.
For example
<pre><code>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.
# prints A

,-.
# prints the ASCII character preceding the one taken as input
# in "B" -&gt; out "A" 
</code></pre>
But honestly we can’t do anything more with it.

The first missing feature is <em>memory management</em>. We have implemented the functions that move the pointer to memory cells left and right, but we’re still stuck with a non expanding memory tape of one cell only.

Let’s implement memory auto expansion, turns out it is gonna be very easy.

<!--more-->
<h3>Memory management</h3>
The functions that handle the <code>&gt;</code> and <code>&lt;</code> operators are
<pre><code class="elixir">defp run(@op_pinc &lt;&gt; rest, addr, mem, output) do
  run(rest, addr+1, mem, output)
end

defp run(@op_pdec &lt;&gt; rest, addr, mem, output) do
  run(rest, addr-1, mem, output)
end
</code></pre>
One easy way to try to implement an auto expanding memory would be to check if we are accessing a cell past the end of the tape or before the initial one and handling those edge cases.
Something like this
<pre><code class="elixir">defp run(@op_pinc &lt;&gt; rest, addr, mem, output) do
  if length(mem) == addr+1 do
    mem = mem ++ [0]
  end
  run(rest, addr+1, mem, output)
end

defp run(@op_pdec &lt;&gt; rest, addr, mem, output) do
  if addr == 0 do
    mem = [0] ++ mem
  end
  run(rest, addr-1, mem, output)
end
</code></pre>
This solution has several problems:
<ul>
 	<li>the second function does not work, it keeps decrementing the memory pointer and can go negative, which is an unwanted behaviour (it should be <code>zero</code>). We should add another <code>if</code> to make it work, not so good.</li>
 	<li>We are shadowing the original <code>mem</code> variable by reassigning its value.</li>
 	<li>we are modifying code that already works (we’ll talk about testing Elixir code in the next article) introducing branching and making it harder to mantain</li>
 	<li>we’re not <em>taking advantage of Elixir features</em></li>
</ul>
One of the main selling point of Elixir is pattern matching in function declaration.

But pattern matching alone could not be enough, as we’ve seen in this example.

In Elixir you can add conditions to function declarations that act in tandem with pattern matching and limit the range of values a function accept. They are called <code>guard clauses</code>, I’ll rewrite those two functions to use them:
<blockquote>note the <code>when</code> after function declaration</blockquote>
<pre><code class="elixir">
# we are moving past the end of the memory tape
defp run(@op_pinc &lt;&gt; rest, addr, mem, output) when length(mem) == addr + 1 do
  # append a new cell, initialize its value to zero, return the next address
  run(rest, addr+1, mem ++ [0], output)
end

# we are moving to the left of the first cell of the memory tape
defp run(@op_pdec &lt;&gt; rest, addr, mem, output) when addr == 0 do
  # prepend a new empty cell, initialize its value to zero, return zero as address
  run(rest, 0, [0] ++ mem, output)
end
</code></pre>
This is much better, requires no branching, can be added without modifying the existing code and by looking at the definition we can easily guess when the function is going to be called.

The only limitation is that Erlang VM only allows <a href="http://elixir-lang.org/getting_started/5.html#5.2-expressions-in-guard-clauses.">a limited set of expressions in guards</a>.

With this simple addition, we have created a complete memory management system, that can automatically expand on both sides, in a virtually unlimited way.
<h3>Loops and jumps</h3>
Implementing brainfuck operators has been quite linear until now.
We just had to follow the language specs and we obtained a working implementation.

Loops are a bit harder task though.

The following are the representations of the two ways to define loops

the <code>while</code> loop
<img src="http://i.imgur.com/IiIEPo8.jpg" alt="flow chart of while loop" />

the <code>do until</code> loop
<img src="http://i.imgur.com/Joke2ar.jpg" alt="do until loop" />

In Elixir specs they are defined as
<ul>
 	<li><code>[</code> if the current cell value is zero, jump to the next matching <code>]</code></li>
 	<li><code>]</code> if the current cell value is non-zero jump back to the matching <code>[</code></li>
</ul>
Looks like brainfuck author overengineered the loops, making it possible to have both.
But when it’s time to implement them, we can choose to check the loop condition only one time (at the beginning or at the end) and treat the other end of the loop as an <em>unconditional</em> jump.
I’ve chosen to implement the <code>while</code> loop.

To implement the loop in brainfuck, we need a function that matches <em>balanced couples</em> of <code>[</code> and <code>]</code> first (did someone say s-expressions?).

We cannot simply match <code>]</code> when we find a <code>[</code> and the reason is fairly obvious: we could not have nested loops (<code>[[-]-]</code> would not work).

The algorithm we are using is the following:
<ol>
 	<li>when we match a <code>[</code> we check the value at the current memory address, if it is zero, we jump past the end of the loop</li>
 	<li>if it is not zero, we extract the loop’s body and execute it, like it is a stand alone program, then collect the results and jump (<em>unconditionally</em>) to the beginning of the loop</li>
 	<li>go back to 1.</li>
</ol>
In Elixir code it is
<pre><code class="elixir">
defp run(@op_lbeg &lt;&gt; rest, addr, mem, output) do
  case mem |&gt; byte_at addr do
    0 -&gt;
      run(rest |&gt; jump_to_lend, addr,  mem, output)
    _ -&gt;
      {a, m, o} = run(rest |&gt; loop_body, addr,  mem, output)
      # prepend [ to the input, to make sure we call this function again
      run(@op_lbeg &lt;&gt; rest, a, m, o)
  end
end
</code></pre>
remember that in Elixir <code>_</code> means <em>match everything</em>.

To match the balanced couples of square brackets, I used this algorithm
<ol>
 	<li>when a <code>[</code> is found, pass the rest to a matcher function with a <code>depth</code> parameter with value <code>1</code> and a parameter <code>acc</code>, to hold the length of the loop body, with a value of <code>zero</code></li>
 	<li>every character we find will increment <code>acc</code></li>
 	<li>if we find a <code>[</code>, increment <code>depth</code> too</li>
 	<li>if we find a <code>]</code> decrement <code>depth</code></li>
 	<li>if <code>depth</code> is zero, we have found the end of loop, return the length of its body</li>
 	<li>if we reach the end of the input and <code>depth</code> is not zero, square brackets are unbalanced, <code>raise</code> an error</li>
</ol>
Let’s translate this to Elixir
<pre><code class="elixir">
# start the matching loop
defp match_lend(source), do: match_lend(source, 1, 0)

# if depth is zero, we have reached the other end of the loop
# return the body length
defp match_lend(_, 0, acc), do: acc

# if we reached the end of the input, but depth is not zero, the
# sequence is unbalanced, raise an error
defp match_lend(@empty, _, _), do: raise "unbalanced loop"

# [ increment the depth
defp match_lend(@op_lbeg &lt;&gt; rest, depth, acc), do: match_lend(rest, depth+1, acc+1)
# ] decrement the depth
defp match_lend(@op_lend &lt;&gt; rest, depth, acc), do: match_lend(rest, depth-1, acc+1)
# every other character just increment acc (loop body length)
defp match_lend(&lt;&lt;_&gt;&gt; &lt;&gt; rest, depth, acc), do: match_lend(rest, depth, acc+1)

# returns the slice of the input program starting from the end of the loop after ]
defp jump_to_lend(source), do: source |&gt; String.slice (source |&gt; match_lend)..-1
# return the slice of the input that represent the loop's body 
# between 0 and the body length-1 (everything but the last ])
defp loop_body(source), do: source |&gt; String.slice 0..(source |&gt; match_lend)-1

</code></pre>
This implementation automatically works for nested loops of any depth.
Every time a <code>[</code> command is found, the program is split in a smaller one and executed until the loop condition is met (this does not save you from infinite loops).

We have now a complete implementation of a brainfuck interpreter that can run any brainfuck program.

To test it let’s run it inside <code>iex</code> the Elixir shell

<img src="http://i.imgur.com/1lTQqee.gif" alt="iex brainfuck session" />

In the <a href="http://dev.mikamai.com/post/109477559604/brainfuck-in-elixir-part-three-compiling">next post</a> I’ll talk about <span style="text-decoration: line-through;">testing the code, creating a project and compiling down to an executable and the command line tools</span> creating a Brainfuck compiler.
<blockquote><a href="https://gist.github.com/wstucco/3064b6d01f1f8cf1292c">As usual, I’ve created a gist with all the code presented in this post</a></blockquote>