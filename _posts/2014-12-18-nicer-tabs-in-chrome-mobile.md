---
ID: 171
post_title: Nicer tabs in Chrome mobile
author: Andrea Venturelli
post_date: 2014-12-18 16:04:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/18/nicer-tabs-in-chrome-mobile/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/105529059974/nicer-tabs-in-chrome-mobile
tumblr_mikamayhem_id:
  - "105529059974"
---
Android users who have updated their system to<a href="http://www.android.com/versions/lollipop-5-0/">Lollipop</a> have surely noticed that each Chrome tab is now visible in the task manager, together with all the other running apps.
This is for sure a good news, but there’s more if you are a web developer: you may be interested in making tabs of your apps nicer and catchier.

<!--more-->

With Android Lollipop Goggle allows you to assign a custom color to the title bar and to the toolbar for each open Chrome tab. All you have to do is a simple modification to your page HTML. Here’s how it works:
<blockquote>
<div>Starting in version 39 of Chrome for Android on Lollipop, you’ll now be able to use the theme-color meta tag to set the toolbar color—this means no more Seattle gray toolbars! The syntax is pretty simple: add a meta tag to your page’s with the name=”theme-color”, and set the content to any valid CSS color.</div></blockquote>
And here’s some sample code:
<pre><code> 
&lt;html&gt;
  &lt;head&gt; 
    &lt;meta name="theme-color" content="orange"&gt; 
  &lt;/head&gt; 
  &lt;body&gt; 
    &lt;p&gt;Hello Lollipop!&lt;/p&gt; 
  &lt;/body&gt; 
&lt;/html&gt;</code></pre>
Now, let’s go and color them all!
<p align="center"><img class="aligncenter" src="http://68.media.tumblr.com/ac6cca72a4a806f9d3e988514acb5854/tumblr_inline_ngqhtf2QPZ1r9vg8d.png" alt="image" /></p>