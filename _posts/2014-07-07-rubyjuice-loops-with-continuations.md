---
ID: 229
post_title: '[RubyJuice] Loops with continuations'
author: Andrea Longhi
post_date: 2014-07-07 15:18:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/07/rubyjuice-loops-with-continuations/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/91051483419/rubyjuice-loops-with-continuations
tumblr_mikamayhem_id:
  - "91051483419"
---
<p>Ruby continuations are a mistery for most of the rubists, including me. Personally I never had the chance to use them in production code, but I think that&rsquo;s mainly because I never correctly understood how they work, plus for every scenario when continuations could be used I was able to find a more conventional solution.</p>
<p>Continuations were removed from the core library (probably due to lack of appreciation) so you need to <code>require 'continuation'</code> to use them in ruby &gt; 1.8</p>
<h3>What are continuations anyway?</h3>
<p>Ruby <a href="http://www.ruby-doc.org/core-2.1.2/Continuation.html">documentation</a> about continuations says:</p>
<p>«Continuations are somewhat analogous to a structured version of C&rsquo;s <strong>setjmp/longjmp</strong> (although they contain more state, so you might consider them closer to threads).»</p>
<p>So if you already know some C you probably have a rough idea of what we&rsquo;re talking about. If you don&rsquo;t and you happen to have some knowledge of the BASIC programming language, then you can consider them like <strong>GOTO</strong> statements.</p>
<p>Today I&rsquo;m going to show how to create loops with continuations, but you can use them for other purposes as well.</p>
<h3>The Basics</h3>
<p>During the early 80s everybody used to write endless loops like this:</p>
<pre><code>10 PRINT "HELLO WORLD"
20 GOTO 10</code></pre>
<p>The code above should be easy to understand for anybody, also for who had no previous experience with BASIC. So, how do we express the same behavior with ruby continuations?</p>
<pre><code>continuation = callcc {|cc| cc }
p 'Hello world'
continuation.call(continuation)</code></pre>
<p>Remember to save the code into a file, as continuation loops don&rsquo;t work correctly inside irb.</p>
<p>What&rsquo;s happening here? The first line creates a <code>Continuation</code> instance and stores it into the <code>continuation</code> variable. The second line is the body of the loop. The third line works like the good old GOTO statement, invoking the continuation; the program continues from the end of the <code>callcc</code> block.</p>
<p>The code inside the block of the <code>callcc</code> method is executed only once (I&rsquo;m actually not doing much there in this example, but you can do). The block returns the continuation object.</p>
<p>Now, let&rsquo;s write some code that prints integers up to a given number:</p>
<pre><code>def integers_upto(n)
  i = 0
  continuation = callcc {|cc| cc }
  p i
  i += 1
  continuation.call(continuation) if i &lt;= n
end

integers_upto 3
# 0 1 2 3
</code></pre>
<p>The code above can be slightly improved: we can avoid to explicitly set the variable <code>i</code> to its incremented value and let the continuation do it:</p>
<pre><code>def integers_upto(n)
  continuation, i = callcc {|cc| [cc, 0]}
  p i
  continuation.call(continuation, i+1) if i &lt;= n
end
</code></pre>
<p>Do you like it? I don&rsquo;t really much. There are far too many ways to have the same behavior with simpler code, for example:</p>
<pre><code>def integers_upto(n)
  0.upto(n) {|i| p i}
end</code></pre>
<h3>Fibonacci with continuations</h3>
<p>Sometimes continuations allow to write some very coincise code, for example look at this Fibonacci numbers implementation:</p>
<pre><code>def fibonacci(n)
  cc, fib, prev, i = callcc {|cc| [cc, 1, 0, 0]}
  p fib
  cc.call(cc, fib+prev, fib, i+1) if i &lt; n
end

fibonacci 10

# 1 1 2 3 5 8 13 21 34 55 89</code></pre>
<p>I like the fact that the code makesit  self-evident that we&rsquo;re passing the sum of the current number (fib) and its previous (prev) as the new fibonacci number, just like is stated by the fibonacci definition.</p>
<p>I hope you had fun with continuations loops so far, and keep experimenting with them.</p>