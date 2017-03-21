---
ID: 163
post_title: 'Brainfuck in Elixir, part three: compiling'
author: Massimo Ronca
post_date: 2015-01-29 11:13:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/01/29/brainfuck-in-elixir-part-three-compiling/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/109477559604/brainfuck-in-elixir-part-three-compiling
tumblr_mikamayhem_id:
  - "109477559604"
---
<blockquote><p>This is the third in a series of articles on building a brainfuck interpreter in Elixir.<br />
In the <a href="http://dev.mikamai.com/post/100075543414/elixir-as-a-parsing-tool-writing-a-brainfuck">first one</a> we built a minimal brainfuck interpreter that could understand the basic instructions.<br />
In <a href="http://dev.mikamai.com/post/102283561929/elixir-as-a-parsing-tool-writing-a-brainfuck">the second</a>, we completed it implementing loops.<br />
In this third episode we&rsquo;ll write a simple compiler to translate Brainfuck instructions to a machine readable intermediate format (AST) and a VM that executes it.</p></blockquote>

<p>This post was supposed to be about testing and the command line tools, I changed my mind and I will talk about improving our interpreter and turning it into a compiler.</p>

<p>Writing a performant compiler is probably one of the most challenging task for a programmer, but the theory behind it is actually quite simple. <br />
Compilers just <em>transform</em> a source code written in a programming language to some other code, usually a different programming language (including intermediate languages and machine language). <br />
Most of the time, they are built following a common design, this one</p>

<p><img src="http://twimgs.com/ddj/images/article/2012/0512/latfig1.gif" alt="Compiler design" /></p>

<blockquote><p>copyright <a href="http://www.drdobbs.com/architecture-and-design/the-design-of-llvm/240001128">Dr. Dobb&rsquo;s</a></p></blockquote>

<p>Our compiler will be much simpler: we will completely skip the optimizer (for now) and will directly execute the output of the fronted (AST or abstract syntax tree, from now on).</p>

<p>Technically, we are writing the frontend of the compiler, which is the starting point for everything else to come.</p>

<!--more-->

<h4>The intermediate language</h4>

<p>Brainfuck is already an intermediate language, very similar to assembly. Each symbol is an <code>opcode</code>: for example <code>&gt;</code> and <code>&lt;</code> can be easily mapped to a relative <code>JMP</code>, <code>+</code> maps to <code>INC</code>, <code>-</code> to <code>DEC</code>, <code>[</code> and <code>]</code> are <code>JZ</code> (jump if zero) and <code>JNZ</code> (jump if not zero). <br /><code>.</code> and <code>,</code> are more complex, there&rsquo;s no single instruction in assembly for reading and writing chars to the screen, but basically they are the <code>C</code> equivalent of <code>putchar</code> and <code>getchar</code>.</p>

<p>Everybody love assembly, but we will not use those <code>opcodes</code>, we will use something more similar to labels, something mnemonic, because, right now, we just need to know which block to execute, we have very simple instructions, with no parameters, that do just one thing. <br />
Things will change when we&rsquo;ll get our hands on the optimizer, but for now we&rsquo;ll keep things simple, and map Brainfuck instructions to <a href="http://elixir-lang.org/getting_started/2.html#2.3-atoms">Elixir atoms</a> (think about them as Ruby&rsquo;s symbols).</p>

<p>Our instructions set will be the following:</p>

<p><code>+</code> -&gt; <code>:inc_d</code><br /><code>-</code> -&gt; <code>:dec_d</code><br /><code>&gt;</code> -&gt; <code>:inc_p</code><br /><code>&lt;</code> -&gt; <code>:dec_p</code><br /><code>.</code> -&gt; <code>:put_c</code><br /><code>,</code> -&gt; <code>:get_c</code></p>

<p>Loops are mapped to <a href="http://elixir-lang.org/getting_started/7.html#7.1-keyword-lists">Elixir keywords</a>, we already ignore the end loop instruction <code>]</code>, because we unconditionally jump back to <code>[</code> when we find one.<br />
That leaves <code>[</code> as the only complex instruction in the set, the only that carries a parameter (the body of the loop). <br />
So loops are defined as</p>

<p><code>[</code> -&gt; <code class="elixir">{:loop, [loop body]}</code></p>

<p>or in the condensed form</p>

<p><code>[</code> -&gt; <code class="elixir">[loop: [loop body]]</code></p>

<p>Loop body is always a list of instructions.</p>

<h4>The compiler implementation</h4>

<p>To write the interpreter, we already wrote a Brainfuck scanner, tokenizer and parser. We&rsquo;ll take advantage of it to emit our <code>AST</code>, turns out it can be written in a very compact way</p>


<p>Pretty short, pretty easy to follow.<br />
First we create a <code>Brainfuck.Compiler</code> module, that is our namespace for the compiler, we then put a few conditions: if the input is a string, we convert it to a <a href="http://elixir-lang.org/getting_started/6.html#6.3-char-lists">char list</a> that is easier to traverse, and we declare that when <code>compile</code> is called with an empty list, there are no more instructions to translate, we are done and return the <code>AST</code>.<br />
Unless the stack is not empty, which is an error condition, we&rsquo;ll se why in a moment.</p>

<p>Every instruction found, is appended to the <code>AST</code> list, every not recognized symbol, is ignored and discarded.</p>

<p>What it does is take this input</p>

<pre><code>+-&gt;&lt;[.,]
</code></pre>

<p>and translate it to</p>

<pre><code>[:inc_d, :dec_d, :inc_p, :dec_p, loop: [:put_c, :get_c]]
</code></pre>

<p>Loops are handled recursively: once we find a <code>[</code>, we save in a <code>stack</code> the current <code>AST</code>.<br />
We use the stack as a FIFO queue, we prepend the <code>AST</code> to the actual value of the stack, because the loop could be nested, we then execute the body of the loop like it was a standalone program.<br />
When we find a <code>]</code>, we pop from the head of the stack and prepend it to the loop <code>AST</code> and keep popping until we have emptied the stack. <br />
This way we can track unbalanced pairs of <code>[</code> and <code>]</code>.<br />
If we pop and the stack is empty, we popped once too often. <br />
If we get to the end, where no instruction is left to be picked up, and the stack is not empty, we haven&rsquo;t popped enough.</p>

<h3>The Virtual Machine</h3>

<p>Our virtual machine is not really a virtual machine, in the strict sense of the term, it is more a runtime that knows our bytecode and how to execute it.<br />
It is really not much different from the interpreter, it reads a list of inputs and decide what to do with them. <br />
But, it has some advantages. <br />
The first one is that the compiler ensures correctness of our code: we can&rsquo;t be sure that the code does what it is supposed to do or that there won&rsquo;t be an nfinite loop, but we can assume it is formally correct (no unbalanced loops, for example).<br />
The second one is that having a bytecode, enable us to optimize the code.<br />
The simpler optimization is that we don&rsquo;t have to scan the code back and forth to find the boundaries of the loops, the are already expresse in the <code>AST</code>.<br />
Infact to <code>run</code> loops the VM we just executes them</p>


<blockquote><p><code>keywords</code> in Elixir are matched with <code>{:key, value}</code></p></blockquote>

<p>This optimization alone make our programs run up to six times faster. <br />
Conclusions are based on higly non-scientifical benchmarks</p>

<pre><code>
$ time ./brainfuck test/fixtures/bench_ok.bf
  OK

  real  1m10.399s
  user  1m9.483s
  sys 0m0.391s

$ time ./brainfuck -i test/fixtures/bench_ok.bf
  OK

  real  6m2.453s
  user  5m57.601s
  sys 0m1.704s 
</code></pre>

<p>The more loops there are in the Brainfuck code, the more it should benefit from the compilation.</p>

<p>You can find all the code <a href="https://github.com/wstucco/elixir-brainfuck/">on github</a>, to create the brainfuck executable run <code>mix escript.build</code>, if you run it with the <code>-i</code> flag, it will use the interpreter written in the previous two articles, otherwise it will use the compiler.<br />
To run the tests use <code>mix test</code>.</p>

<p>And if you find the reason why the interpreter get stuck in an infinite loop running <a href="https://github.com/wstucco/elixir-brainfuck/blob/master/test/fixtures/bench.bf">this brainfuck program</a>, please, <a href="https://github.com/wstucco/elixir-brainfuck/issues/1">let me know</a>.</p>