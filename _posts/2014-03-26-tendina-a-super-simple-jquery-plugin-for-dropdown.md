---
ID: 267
post_title: >
  Tendina, a super-simple jQuery plugin
  for dropdown side menus
author: Ivan Prignano
post_date: 2014-03-26 14:37:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/26/tendina-a-super-simple-jquery-plugin-for-dropdown/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/80779300192/tendina-a-super-simple-jquery-plugin-for-dropdown
tumblr_mikamayhem_id:
  - "80779300192"
---
<p>I have just released the first beta version of my jQuery plugin <strong>Tendina</strong>.</p>
<p><img alt="image" src="https://github.com/iprignano/tendina/raw/master/demo/tendina.gif" /></p>

<p><span>Tendina is a easy-to-use jQuery plugin to rapidly build dropdown side menus. </span></p>
<p><span>Its usage is very simple and straightforward: you just need a basic unordered list menu markup and a link to jQuery and the plugin. For example:</span></p>
<pre>    
&lt;body&gt;<br />    &lt;ul class="dropdown"&gt;<br />      &lt;li&gt;<br />        &lt;a href="#"&gt;Menu 1&lt;/a&gt;<br />        &lt;ul&gt;<br />          &lt;li&gt;&lt;a href="#"&gt;Submenu 1&lt;/a&gt;&lt;/li&gt;<br />        &lt;/ul&gt;<br />      &lt;/li&gt;<br />      &lt;li&gt;<br />        &lt;a href="#"&gt;Menu 2&lt;/a&gt;<br />        &lt;ul&gt;<br />          &lt;li&gt;&lt;a href="#"&gt;Submenu 2&lt;/a&gt;&lt;/li&gt;<br />          &lt;li&gt;&lt;a href="#"&gt;Submenu 2&lt;/a&gt;&lt;/li&gt;<br />          &lt;li&gt;&lt;a href="#"&gt;Submenu 2&lt;/a&gt;&lt;/li&gt;<br />        &lt;/ul&gt;<br />      &lt;/li&gt;<br />      &lt;li&gt;<br />        &lt;a href="#"&gt;Menu 3&lt;/a&gt;<br />        &lt;ul&gt;<br />          &lt;li&gt;&lt;a href="#"&gt;Submenu 3&lt;/a&gt;&lt;/li&gt;<br />          &lt;li&gt;&lt;a href="#"&gt;Submenu 3&lt;/a&gt;&lt;/li&gt;<br />        &lt;/ul&gt;<br />      &lt;/li&gt;<br />    &lt;/ul&gt;<br /><br />    &lt;script src="http://code.jquery.com/jquery-1.11.0.min.js"&gt;&lt;/script&gt;<br />    &lt;script src="../tendina.js"&gt;&lt;/script&gt;<br />    &lt;script&gt;<br />      $('.dropdown').tendina()<br />    &lt;/script&gt;<br />  &lt;/body&gt;
</pre>
<p>As you can see, you just need to call the function on your elements and&hellip; That&rsquo;s it! <span>The plugin will handle all the interactions and leave all the styling to you, so you don&rsquo;t have to override useless CSS classes.</span></p>
<p>Tendina will also work on dinamically appended elements, and additionally add a &ldquo;selected&rdquo; class to expanded elements.</p>
<p>Check it out on <a href="https://github.com/iprignano/tendina" title="Github" target="_blank">Github</a> or on this demo <a href="http://codepen.io/iprignano/full/tjoua" title="Demo CodePen">CodePen</a>!</p>