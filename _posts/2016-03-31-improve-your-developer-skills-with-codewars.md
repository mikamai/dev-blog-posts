---
ID: 40
post_title: >
  Improve your developer skills with
  Codewars
author: Ivan Lasorsa
post_date: 2016-03-31 15:48:45
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/03/31/improve-your-developer-skills-with-codewars/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/142016943374/improve-your-developer-skills-with-codewars
tumblr_mikamayhem_id:
  - "142016943374"
---
<p>Some weeks ago in Mikamai we started using <a href="http://www.codewars.com/">codewars</a>, a website that aims to improve your programming skills via gamification with code challenges. 
Since I&rsquo;m a junior developer willing to improve my Ruby knowledge I have used it a lot lately, so let&rsquo;s see how it works.</p>

<p>After signing up and completing a simple code problem, you can start browsing some kata and boost your programming skills.</p>

<p>Why kata? Codewars has this oriental, martial arts flavor so you&rsquo;ll find words like <code>kata</code> and <code>train</code> for quizzes, <code>kyu</code> and <code>dan</code> to classify user rankings.</p>

<p>Personally I&rsquo;m really enjoying the site and the learning process. 
The number of Ruby katas is huge, and I find expecially nice the chance to see other users solutions after you complete a kata, so you can see super-compact solutions such as:</p>

<pre><code>def anagrams(word, words)
  words.select { |w| w.chars.sort == word.chars.sort }
end
</code></pre>

<p>or regular expressions usage to solve a wide range of problems:</p>

<pre><code>def pig_it text
  text.gsub(/(w)(w+)*/, '21ay')
end
</code></pre>

<p>and you can write your own tests while you&rsquo;re training a kata.</p>

<p>You can choose among many programming languages (I&rsquo;m looking forward to see Elixir or Rust in the future).</p>