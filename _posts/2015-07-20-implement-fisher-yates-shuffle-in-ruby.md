---
ID: 96
post_title: Implement Fisher-Yates shuffle in Ruby
author: Diomede Tripicchio
post_date: 2015-07-20 11:53:39
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/20/implement-fisher-yates-shuffle-in-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/124568650434/implement-fisher-yates-shuffle-in-ruby
tumblr_mikamayhem_id:
  - "124568650434"
---
<p>Some time ago, for a little personal project, I needed to implement a shuffle algorithm. After some research I decided to use the <a href="https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle">Fisher-Yates</a> algorithm.</p>
<!--more-->


<p>Simply stated, this algorithm works like this:</p>

<ol><li>Given an array <strong><code>a = [1, 2, 3, 4, 5, 6, 7, 8]</code></strong></li>
<li>Get a random number <strong><code>i</code></strong> between <strong><code>0</code></strong> and <strong><code>n - 1</code></strong> where <strong><code>n = a.length</code></strong></li>
<li>At this point swap <strong><code>array[i]</code></strong> with <strong><code>array[n]</code></strong></li>
<li>Restart.</li>
<li>But, the next time the random <strong><code>i</code></strong> must be calculated between <strong><code>0</code></strong> and <strong><code>n - 2</code></strong> and so on until you get to <strong><code>n = 0</code></strong></li>
</ol><p>In this way an element from the back of the array is swapped with an element from the front and waits to be shuffled.</p>

<p>Here is a solution to implement it in ruby:</p>

<pre><code class="ruby">class Array
  def shuffle!
    n = length
    while n &gt; 0
      i = rand(n -= 1)
      self[i], self[n] = self[n], self[i]
    end
    self
  end
end

puts (1..8).to_a.shuffle!.join(‘,’)
puts (1..52).to_a.shuffle!.join(‘,’)
</code></pre>

<p>For the next step I want to implement a weighted-shuffle algorithm, but at the moment I’m still missing something. I hope to be ready for the next post!</p>

<p>Bye and see you soon!</p>