---
ID: 192
post_title: >
  Understanding throttling and debouncing
  of functions in JavaScript
author: Ivan Prignano
post_date: 2014-10-29 15:46:07
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/29/understanding-throttling-and-debouncing-of/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/101261947854/understanding-throttling-and-debouncing-of
tumblr_mikamayhem_id:
  - "101261947854"
---
<p><strong>Throttling</strong> and <strong>debouncing</strong> are arguably two concepts too often overlooked in front end development, even though they&rsquo;re quite simple to grasp and bring major benefits when developing complex and/or intensive web applications.</p>
<h3>Theory</h3>
<p>Throttling and debouncing function calls basically prevent functions to be executed too many times in a specified time interval. More specifically:</p>
<ul><li><span>Throttle will queue and <strong>execute functions calls at most once per given milliseconds</strong>; </span></li>
<li><span>Debounce will execute the function once and <strong>filter all following calls</strong> until a given time has passed, then fire it again.</span></li>
</ul><p>If you&rsquo;re more a visual person, check out <a href="http://jsfiddle.net/amyseqmedia/dD99u/37/embedded/result/" target="_blank">this interactive example</a> to get how they work.</p>
<h3>Ok, but when should I use them?</h3>
<p>Throttling and debouncing are particularly useful when you need to listen (and react) to events that fire a lot of times, rapidly – think of a window resize, or a page scroll. If you try to attach a console.log() handler to one of those two events and fire them, you might have something like that:</p>
<p><img alt="image" src="http://68.media.tumblr.com/f184e376c8f58b2cef8301aeb47ccc76/tumblr_inline_ne62onUwQs1riz3e2.gif" /></p>

<p>That&rsquo;s a lot of logs, right? Imagine if our handler function had to calculate different elements&rsquo; positions and had to move stuff around at the same time. Now, put this in the context of a heavy-interactive web application and your lag is served!</p>
<h3>Enter throttle and debounce</h3>
<p>That&rsquo;s a perfect use case for some good ol&rsquo; throttling. </p>
<p>The most famous implementation of the two is probably the <a href="http://underscorejs.org/" target="_blank">Underscore.js</a> one, which we&rsquo;re going to use in our small example.</p>
<p>Its syntax is quite simple (from the <a href="http://underscorejs.org/#throttle" target="_blank">Underscore.js docs</a>): </p>
<pre><code>_.throttle(function, wait, [options])</code></pre>
<p>Let&rsquo;s put it in practice!</p>
<h3>A _.throttle example</h3>
<p>Our <span>console.log() </span>handler could be like this:</p>
<pre><code>var i = 0;
var handler = function() {
  console.log('Scroll event fired ' + i + ' times!');
  i++;
}</code></pre>
<p>We&rsquo;ll pass it as an argument to the _.throttle function, so that it will return our new, throttled handler: </p>
<pre><code>var throttledHandler = _.throttle(handler, 500);</code></pre>
<p>Now, some jQuery sugar to listen for the scroll event and fire it:</p>
<pre><code>$(window).on('scroll', throttledHandler);</code></pre>
<p>Et voilà, less lag and a better approach to intensive Javascript application development:</p>
<p><img alt="image" src="http://68.media.tumblr.com/0e7f2d45fed478d713a526b36e25ccc4/tumblr_inline_ne7pqyy8hc1riz3e2.gif" /></p>

<p>You can see how our throttled handler function now fires at most once every 500ms.</p>
<p>If you&rsquo;re interested in how throttling and debouncing work internally, take a look at the <a href="http://underscorejs.org/docs/underscore.html#section-70" target="_blank">annotated source code</a> provided by the Underscore.js pals!</p>