---
ID: 197
post_title: 'Elixir as a parsing tool: writing a Brainfuck interpreter, part one'
author: Massimo Ronca
post_date: 2014-10-15 13:33:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/15/elixir-as-a-parsing-tool-writing-a-brainfuck-2/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/100075543414/elixir-as-a-parsing-tool-writing-a-brainfuck
tumblr_mikamayhem_id:
  - "100075543414"
---
<blockquote>
<p>For instructions on how to install the Elixir environment you can take a look at the <a href="http://elixir-lang.org/getting_started/1.html#1.1-installers">getting started guide</a>.</p>
</blockquote>

<p>To get used to the language and try some of the code in this post, you can start <code>iex</code>, the Elixir shell.<br />
One of its best features are autocompletion of module and function names and the integrated documentation accessible with the command <code>h</code>.<br />
This is an example of an <code>iex</code> session:</p>

<p><img src="http://i.imgur.com/lKirKhq.gif" alt="image" /></p>

<h3>Brainfuck, the language</h3>

<p>Brainfuck is an esoteric programming language with a very small set of instructions and it&rsquo;s the perfect language to write compilers or interpreters for.<br />
Brainfuck is Turing complete and there are only 8 instructions.<br />
A typical implementation requires a memory of at least 30 thousand cells, but ideally infinite on both sides (we&rsquo;ll see this is very easy to implement in Elixir), each initially set to zero and a data pointer that points to the first memory cell.<br />
The available commands are:</p>

<ul><li><code>&gt;</code> increment the data pointer (point to the cell on the right)</li>
<li><code>&lt;</code> decrement the data pointer (point to the cell on the left)</li>
<li><code>+</code> increment the value at the data pointer location</li>
<li><code>-</code> decrement the value at the data pointer location</li>
<li><code>.</code> output the value at the data pointer as byte</li>
<li><code>,</code> read a byte into the cell at pointer location</li>
<li><code>[</code> if the current cell value is zero, jump to the next matching <code>]</code></li>
<li><code>]</code> if the current cell value is non-zero jump back to the matching <code>[</code></li>
</ul><p>All other characters are considered comments, hence ignored.</p>

<!--more-->

<h3>Implementation details</h3>

<p>We are going to write an Elixir module that only export one public function <code>run</code> that accept a brainfuck program as string and scan it, character by character, until we reach the end.<br />
As a result it returns a triplet containing the final data pointer address, the memory state and the output generated.<br />
I assumed that each memory cell is an unsigned byte, that overlap on overflow (255+1 becomes zero again).<br />
Input and output operations work on bytes too.</p>

<p>In Elixir pattern matching is a fundamental feature for controlling the program flow, there are no loop instructions, so we are forced to use recursion.<br />
The condition of our loops are expressed in the function definition.</p>

<p>The logic of our interpreter is very simple: we are going to consume the program string char by char by using the pattern below</p>

<blockquote>
<p><code>&lt;&gt;</code> is the operator for string concatenation, it work on bitstring, but since strings in Elixir are binaries, it works on strings too</p>
</blockquote>
    <pre><code class="hls php">run(first_char &lt;&gt; rest_of_the_program, ... ) do
  ...
  run(rest_of_the_program, ...)
end
</code></pre>

<p>As an example, the simple program <code>++.</code>, which increment the location zero two times, and then output the result will flow like this</p>
    <pre><code class="hls php"># step 1
run("+" &lt;&gt; "+.", 0, [0], "")
  # inc the value at zero
  run("+.", 0, [0+1], "")

# step 2
run("+" &lt;&gt; ".", 0, [1], "")
  # inc the value at zero
  run(".", 0, [1+1], "")

# step 3
run("." &lt;&gt; "", 0, [2], "")
  # append the value at zero to output
  run("", 0, [2], "" &lt;&gt; &lt;&lt;2&gt;&gt;)

# final step
run("", 0, [2], &lt;&lt;2&gt;&gt;)
  # return {addr, memory, output} -&gt; {0, [2], &lt;&lt;2&gt;&gt;}


</code></pre>

<h3>First steps</h3>

<p>Let&rsquo;s begin with the definition of our module, we are going to define our instruction set and the run function</p>
    <pre><code class="hls php">defmodule Brainfuck do

    # opcodes
    @op_vinc "+" # increment value at memory address
    @op_vdec "-" # decrement value at memory address
    @op_pinc "&gt;" # increment memory address
    @op_pdec "&lt;" # decrement memory address
    @op_putc "." # output byte at memory address
    @op_getc "," # input byte into memory address
    @op_lbeg "[" # loop begin
    @op_lend "]" # loop end

    def run(program), do: run(program, 0, [0], "")

</code></pre>

<p>You may have noticed that run doesn&rsquo;t really do anything, except call another run function with different parameters.<br />
This is a common pattern in Elixir programs: functions are defined as name/arity (the arity is the number of parameters a function takes), so for example the first <code>run</code> function is <code>run/1</code> and the one we are calling is <code>run/4</code>, for Elixir they are two completely different functions, even if they share the same name.<br />
Elixir have strictly immutable types, we need to carry the state around by passing it in parameters, we&rsquo;ll see in a minute what the parameters are for.</p>

<h3>With a little help from my friends</h3>

<p>Before getting our hands dirty (they&rsquo;re not gonna be that dirty, I promise) I&rsquo;m going to show you some helper functions I created to keep code as clean as possible.</p>
    <pre><code class="hls php">defp inc_at(list, addr), do: List.update_at(list, addr, &amp;(&amp;1+1 |&gt; rem 255))
defp dec_at(list, addr), do: List.update_at(list, addr, &amp;(&amp;1-1 |&gt; rem 255))
</code></pre>

<p>First thing to notice is that they start with a <code>defp</code> not <code>def</code>.<br />
It means they are private to the module and not visible from the outside.<br />
Both of them take two parameters, a list and an address that represent the position inside the list, and update the corresponding value.<br />
It&rsquo;s easy to guess by their name what they do: <code>inc_at</code> increment the value of <code>list</code> at position <code>adrr</code> and <code>dec_at</code> decrement it.<br />
We use the facilities provided by the core <code>List</code> module by calling <code>update_at</code>, that takes a list, an address and a function that update the value.<br />
As we said in Elixir data types are immutable, so <code>update_at</code> is returning a copy of the list with the modified value, it is not modifying the value in place.<br />
The only tricky part is understanding the third parameter, the update function.<br />
The <code>&amp;()</code> is called <a href="http://elixir-lang.org/getting_started/8.html#8.4-function-capturing">capture syntax</a> in Elixir and is basically a shorthand for creating anonymous functions.<br />
Rewriting the same function without it would look like this <code>fn(a) -&gt; a+1 |&gt; rem 255 end</code>, the capture syntax is more concise and allows us to get rid of the function parameter and use params placeholders (<code>&amp;1</code> represent the first parameter, <code>&amp;2</code> the second and so on).</p>

<p>The other thing you might have noticed, if you are new to Elixir, is the <code>|&gt;</code> symbol. That&rsquo;s called the pipe operator and act much like a unix pipe, it <code>cat</code>s the argument(s) on left as <strong>first parameter</strong> of the function on the right.<br />
So <code>rem &amp;1+1, 255</code> can be rewritten as <code>&amp;1+1 |&gt; rem 255</code>.<br />
It has no advantage in this case as number of characters typed, but it makes clearer what we are doing: we are taking the value of the first parameter, adding (or subtracting) 1 to it and then piping the result on the function <code>rem</code> with the parameter 255.<br /><code>rem</code> returns the remainder of the int division, that&rsquo;s how we keep memory values in the byte size range, by going back to zero when the value overflows 255.</p>

<p>In the same family, but with a different purpose, I created <code>put_at</code>, that completely replace a value in a list at a speicified address.<br />
Of course this function takes a third parameter, the new value we are putting into the list.</p>
    <pre><code class="hls php">defp put_at(list, addr, val), do: List.replace_at(list, addr, val)
</code></pre>

<h3>The real <code>run</code> function</h3>

<p>So what are those parameters we pass to our internal function for?<br />
I&rsquo;ll explain by showing you the final step of our interpreter</p>
    <pre><code class="hls php">defp run("", addr, mem, output), do: {addr, mem, output}
</code></pre>

<p>The first parameter is the program string <em>at this point</em> of the execution, second is the pointer to the memory cell currently active, third is the current state of the memory tape and the last is the output string we are going to return.<br />
We know our program has ended when the run function is called with and empty string as first parameter.<br />
We then return the triplet containing the current data pointer, the memory cells and the output string.</p>

<p>Without the previous function our program would not end and it will give us an error like this</p>
    <pre><code class="hls php">** (FunctionClauseError) no function clause matching in Brainfuck.run/4
    iex:25: Brainfuck.run("", 0, [0], "")
</code></pre>

<p>because there is no function matching the pattern with the empty string as first param.</p>

<p>The second basic function is the generic one that matches a string starting with some character, no matter which,  skips it, and calls run again with the rest of the string.</p>
    <pre><code class="hls php">defp run(&lt;&lt;_&gt;&gt; &lt;&gt; rest, addr, mem, output), do: run(rest, addr, mem, output)
</code></pre>

<p>With this two functions in place we already have a scanner, a completely useless scanner, that skips everything and then returns the initial state.The complete code for this is just a few lines long</p>
    <pre><code class="hls php">defmodule Brainfuck do
    def run(program), do: run(program, 0, [0], "")
    # exit program
    defp run("", addr, mem, output), do: {addr, mem, output}
    # skip everything
    defp run(&lt;&lt;_&gt;&gt; &lt;&gt; rest, addr, mem, output), do: run(rest, addr, mem, output)
end

Brainfuck.run("hello world")
# output: {0, [0], ""}
</code></pre>

<blockquote>
<p>Note that Elixir matches patterns from top to bottom, so we need to put the function that skips unrecognized commands at the end, otherwise more specific patterns would be ignored.</p>
</blockquote>

<h3>Basics, strings and the I/O</h3>

<p>I&rsquo;m gonna start implementing commands, by defining the functions that handle the <code>IO</code> operations, basically they output a byte and read a byte from input (in our case <code>stdin</code>).<br />
To do that, first we need to introduce two more helper functions</p>
    <pre><code class="hls php">defp byte_at(list, addr), do: list |&gt; Enum.at addr
defp char_at(list, addr), do: [list |&gt; byte_at addr] |&gt; to_string
</code></pre>

<p><code>byte_at</code> extracts the byte at position <code>addr</code> in <code>list</code> (AKA our memory cells), while <code>char_at</code> returns the same byte as string value.<br />
In Elixir strings are binaries, or, in other words, strings of bits.<br />
To convert a byte value to a string, we cannot simply use <code>to_string</code> function, because it will convert the byte to its string representation, not to the character rapresented by its value, so we need to wrap it inside [] and make it a byte list (the internal representartion of ASCII strings).<br />
As an experiment, you can start <code>iex</code> and try this code</p>
    <pre><code class="hls php">65 |&gt; to_string # print "65"
[65] |&gt; to_string # print "A"
[65, 66, 67] |&gt; to_string # print "ABC"
</code></pre>

<p>Handle @op_putc opcode, that appends one byte to <code>output</code></p>
    <pre><code class="hls php">defp run(@op_putc &lt;&gt; rest, addr, mem, output) do
    run(rest, addr, mem, output &lt;&gt; (mem |&gt; char_at addr))
end
</code></pre>

<p>When @op_putc is at the beginning of the program, this function call <code>run</code> with the new output formed by appending the character at the current memory location to the old output.<br />
Rest becomes the new program, while address and memory are unchanged.</p>

<p>Next is @op_getc, which reads a byte from <code>stdin</code> and puts it in the current memory location.</p>
    <pre><code class="hls php">defp run(@op_getc &lt;&gt; rest, addr, mem, output) do
    val = case IO.getn("Inputn", 1) do
        :eof -&gt; 0
        c    -&gt; c
    end
    run(rest, addr, mem |&gt; put_at(addr, val), output)
end
</code></pre>

<p>It&rsquo;s a bit trickier than the previous, but gives us the opportunity to introduce the <code>case</code> statement.<br />
In Elixir everything is an expression, and returns a value.<br />
We use this feature to assign to <code>val</code> the result of the <code>case</code> expression.<br />
Inside the case we use pattern matching to match the return value of <code>IO.getn</code>, which, straight from the Elixir interactive help</p>
    <pre><code class="hls php">Gets a number of bytes from the io device. If the io device is a unicode
device, count implies the number of unicode codepoints to be retrieved.
Otherwise, count is the number of raw bytes to be retrieved. It returns:

  • data - the input characters
  • :eof - end of file was encountered
  • {:error, reason} - other (rare) error condition; for instance, {:error,
    :estale} if reading from an NFS volume
</code></pre>

<p>We read one byte from the input, if it returns <code>:eof</code>, return 0, if it returns some <code>data</code>, we return it (it is guaranteed to be one byte long).<br />
We ignore error conditions, since they are very rare, especially in our simple case.<br />
The new memory will have <code>val</code> value at <code>addr</code> position.</p>

<p>Not hard at all until now.</p>

<p><img src="http://i.imgur.com/6oA7eED.png" alt="very easy" /></p>

<h3>Let&rsquo;s talk about memory</h3>

<p>There are two opcodes in brainfuck that operates on memory values, <code>+</code> and <code>-</code>.<br />
The implementation is very straightforward</p>
    <pre><code class="hls php">defp run(@op_vinc &lt;&gt; rest, addr, mem, output) do
    run(rest, addr, mem |&gt; inc_at(addr), output)
end

defp run(@op_vdec &lt;&gt; rest, addr, mem, output) do
    run(rest, addr,  mem |&gt; dec_at(addr), output)
end
</code></pre>

<p>Now that we (hopefully) grasped the basics of the Elixir syntax and how pattern matching is used, it should be pretty easy to understand how these two functions work.</p>

<p>Last two functions we meet today handle the data pointer.<br />
I&rsquo;ll just show you the two basic cases, when the pointer moves inside the memory length, we&rsquo;ll keep handling the auto expansion of the tape to the left and right for the next part.</p>
    <pre><code class="hls php">defp run(@op_pinc &lt;&gt; rest, addr, mem, output) do
    run(rest, addr+1, mem, output)
end

defp run(@op_pdec &lt;&gt; rest, addr, mem, output) do
    run(rest, addr-1, mem, output)
end
</code></pre>

<p>Almost no need to explain what it is going on, the data pointer is simply incremented or decremented and the new value is passed to <code>run</code>.</p>

<p>In the next post I&rsquo;ll talk about how to handle expanding the memory tape when needed and, the most fun part, where Elixir capabilities really shine, handling loops and jumps in a very easy way.</p>

<blockquote>
<p><a href="https://gist.github.com/wstucco/bc6a5037fe8b1fbf1cf0">I&rsquo;ve created a gist with all the code presented in this post</a>, of course it misses loops, but you can use it as a starting point for your own experiments</p>
</blockquote>

If you enjoyed this, <a href="http://dev.mikamai.com/post/102283561929/elixir-as-a-parsing-tool-writing-a-brainfuck">GOTO part 2</a>