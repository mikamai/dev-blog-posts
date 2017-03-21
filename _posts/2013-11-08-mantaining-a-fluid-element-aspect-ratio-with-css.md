---
ID: 320
post_title: >
  Mantaining a fluid element aspect ratio
  with CSS pseudo-elements
author: Ivan Prignano
post_date: 2013-11-08 09:23:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/08/mantaining-a-fluid-element-aspect-ratio-with-css/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/66358880210/mantaining-a-fluid-element-aspect-ratio-with-css
tumblr_mikamayhem_id:
  - "66358880210"
---
<p>Do you ever wondered how to scale a div within a fluid layout, keeping its aspect ratio?</p>
<p>The solution is pretty easy, and - amazingly - you <strong>don&rsquo;t need</strong> <strong>Javascript</strong>! </p>
<p>Just nest your element in a fluid container - this will be our <em>ratio-killer</em> - and give it these rules:</p>
<pre>  
.container {
  position: relative;
  display: inline-block;
  width: 50%;<br />}</pre>
<p>These properties will let our container div to scale accordingly to the parent element width. Then, we&rsquo;ll use <strong>CSS3 pseudo elements on the container</strong> to do the magic:</p>
<pre>&amp;:before {<br />  content: "";<br />  display: block;<br />  padding-top: 145%;<br />}</pre>
<p>The <strong>padding-top</strong> will represent the height of our div. So tweak the percentage accordingly to the desired aspect ratio!</p>
<p>Finally, we&rsquo;ll place the element for the contents and give it absolute positioning:</p>
<pre>.inside-content {<br />  position: absolute;<br />  top: 0;<br />  bottom: 0;<br />  left: 0;<br />  right: 0;<br />}</pre>
<p>And <em>voilà</em>! The trick is done: we now have a <strong>fluid element that scales keeping its aspect ratio</strong>.</p>
<p>Want to give it a ride? Here&rsquo;s an example: <a href="http://cdpn.io/GIJmz" title="an example">http://cdpn.io/GIJmz</a></p>