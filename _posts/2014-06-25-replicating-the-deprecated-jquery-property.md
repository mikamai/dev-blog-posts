---
ID: 234
post_title: >
  Replicating the deprecated jQuery
  property .selector in CoffeeScript
author: Ivan Prignano
post_date: 2014-06-25 13:36:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/25/replicating-the-deprecated-jquery-property/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/89858338769/replicating-the-deprecated-jquery-property
tumblr_mikamayhem_id:
  - "89858338769"
---
<p>Recently I found myself needing a way to get any (id or class) selector from any DOM element as a string in an easy way. </p>
<p>After a quick search I discovered that jQuery had an handy <a href="https://api.jquery.com/selector/" target="_blank"><em>.selector</em> property</a> that would have been a perfect fit for my needs.</p>
<p>Unluckily,<strong> it was deprecated in version 1.7.</strong></p>
<p><strong><img alt="image" src="http://68.media.tumblr.com/fd1cfacfa82282bc2a1a05c3765049e0/tumblr_inline_n7q76lNT6d1riz3e2.png" /></strong></p>

<p>I felt like the solution suggested by the guys at jQuery wasn&rsquo;t flexible enough and not very user-friendly – passing the element selector as a parameter of the plugin method call is definitely not an elegant way to deal with the problem.</p>
<p>So I tried to write a simple function to solve it&hellip; in CoffeeScript!</p>
<h3>The snippet</h3>
<pre><code>getSelector = (el) -&gt;
  firstClass = $(el).attr('class')?.split(' ')[0]
  elId       = $(el).attr('id')

  if (elId isnt undefined) then "##{elId}" else ".#{firstClass}"
</code></pre>
<p>As you can see, it&rsquo;s really easy to understand: it just checks if there&rsquo;s an id attribute and returns it as a string. If it&rsquo;s undefined, it returns the first class instead.</p>
<p>Here you can see it in action in a <a href="http://codepen.io/iprignano/pen/mkbyu" target="_blank">CodePen</a> example.</p>
<p>I could write it in a single line without the two variables declaration, but I wanted to get the most out of the beautiful <em>human-friendly </em>CoffeeScript syntax, and I think it actually makes the snippet a lot more readable.</p>
<h3>More jQuery pls</h3>
<p>If you wanted to, you could also extend jQuery itself adding this little function as a method in a very easy way:</p>
<pre><code>jQuery.fn.extend(
  getSelector: -&gt;
    firstClass = $(this).attr('class')?.split(' ')[0]
    elId       = $(this).attr('id')

    if (elId isnt undefined) then "##{elId}" else ".#{firstClass}"
)
</code>
</pre>
<p>And then use it like that:</p>
<pre><code>
$ -&gt;
  mySelector = $('div').getSelector()
</code>
</pre>

<p>Hope that helped you!</p>