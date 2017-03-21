---
ID: 129
post_title: Picking a gem for fun and learning
author: Andrea Longhi
post_date: 2015-05-04 13:17:59
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/04/picking-a-gem-for-fun-and-learning/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/118112480144/picking-a-gem-for-fun-and-learning
tumblr_mikamayhem_id:
  - "118112480144"
---
<p>Recently I&rsquo;ve been working on a small rails project that handles uploads and imports of Microsoft Excel files. <br />File upload is a very common scenario and, as most rails developers already know, there are plenty of gems to ease the job. Some of them like Paperclip, Carrierwave or Dragonfly are so  well established that you really can&rsquo;t go wrong with any of them, they&rsquo;re complete and easy to use, with great documentation and a huge user base; in short, they&rsquo;re a perfect tool for the job.</p>
<p>Parsing MS Excel files is not as common but I have already done it a few times through the years and there are many gems for this task as well, new and old. The good old <a href="https://github.com/zdavatz/spreadsheet">spreadsheet</a> gem is the one I used most in the past, but the most promising and interesting one is <a href="https://github.com/randym/axlsx">axlsx</a>, to be paired with <a href="https://github.com/straydogstudio/axlsx_rails">axlsx_rails</a> if you plan to use it inside a rails application.</p>
<p>
I eventually decided to use a different one, a small unpolished gem named <a href="https://github.com/pythonicrubyist/creek">creek</a>. Why? One reason is that, unlike any of its competitors, it builds an hash structure for each document row instead of using an array, so you get something like this:</p>

<pre><code>
[
  {"A1" =&gt; "First Row, first col", "B1" =&gt; "First row, second col", "C1" =&gt; nil}, 
  {"A2" =&gt; "Second Row, first col", "B2" =&gt; "Second Row, second col", "C2" =&gt; nil}
]

instead of:
[
  ["First Row, first col", "First row, second col", nil],
  ["Second Row, first col", "Second Row, second col", nil]
]
</code></pre>

<p>I think this is a nice feature for speeding up the debugging process because having explicit cell coordinates makes the mapping to the original document so much easier, at least for me, especially when the XLS file has many columns and data is repeated or very similar among them.
</p>
<p>
Anyway, there are many other reasons I can think of, for example:</p>
<ul><li>the gem is two years old, which means that it&rsquo;s not in its early development stage so I can assume that most of the relevant bugs have already been worked out</li>
<li>github shows recent activities for this gem, which means that it&rsquo;s not abadonware and the author is still taking care of the maintenance, answering promptly to programmers issues and merge requests</li>
<li>the codebase is simple and small, so modifications and updates should be easy when needed</li>
<li>there are a few tests written with rspec, which means that the gem developer follows the ruby community best practices and I can easily add new ones following the existing patterns</li>
<li>it&rsquo;s easier to get involved when the project is very small: you can easily think about new features to add or make some simple refactoring activities which will probably be welcomed by the author</li>
</ul><p>I think the last point is one of the nicest facets of open source programming: it&rsquo;s easy to get involved and contribute with some small modifications that can improve the original codebase, make new acquantaintances in the process, and start a learning experience for some of the involved programmers, may that person be you or the original developer. If you&rsquo;re more skilled you probably can teach some nifty pattern, trick or refactoring to the gem creator, while if he&rsquo;s more skilled than you he can guide you in the process of improving the gem or show you why your modifications cannot be accepted (i.e. did you write tests for them? Are your changes too much coupled to your specific needs?).</p>
<p>Of course your mileage may vary, but I think that sometimes choosing a rather unknown gem instead of the most famous and celebrated one can be a great choice, especially when you&rsquo;re on a small project and you feel confident that you can fix issues when they arise. It will surely be a more interesting and rewarding experience; of course we&rsquo;re in the business because we are fast in delivering code that works, but nobody mandated that we should not have fun while doing it ;-)</p><p>Happy coding!</p>