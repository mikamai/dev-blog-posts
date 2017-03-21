---
ID: 301
post_title: >
  Profiling web pages in Chrome DevTools
  for fun and profit
author: Ivan Prignano
post_date: 2013-12-18 10:29:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/18/profiling-web-pages-in-chrome-devtools-for-fun-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/70380138078/profiling-web-pages-in-chrome-devtools-for-fun-and
tumblr_mikamayhem_id:
  - "70380138078"
---
<p><span>As a front-end developer, an important part of your work should be making your users comfortable interacting with your super-hot parallax website &ndash; that means, you should be careful avoiding framerate drops or laggy interactions.</span></p>

<p><span>I’ll quickly show you how to detect front-end bottlenecks in your web application using <strong>Chrome Developer Tools</strong>.<br /><br /></span></p>
<blockquote>
<p><strong>TL;DR</strong></p>
<p>Chrome DevTools can inspect browser events and track your website framerate. Use it to debug your application and smoothen your site UX!</p>
</blockquote>
<h3><span><br />A little introduction</span></h3>

<p><span>First, you’ll obviously need to open Chrome DevTools (on Macs you can press </span><strong>Alt + Cmd + I</strong><span>) and switch to the </span><strong>Timeline</strong><span> tab. You’ll see something like this:<br /><br /></span><img alt="image" src="http://i.imgur.com/2j3oMVW.png" /><br /><br /></p>

<p><span>Here you can take a peek at the 3 fundamental aspects for the (client side) performance of a web application:</span></p>

<ul><li>
<p><strong>Events. </strong><span>These are the events the page fires and that the browser executes (i.e. </span><em>Build my layout!</em><span> or</span><em> Hide this div (and recalculate the width of the adjacent one)!</em><span>)</span></p>
</li>
<li>
<p><strong>Frames</strong><span><strong>.</strong> You can guess that. The more the FPS, the smoother the website will be.</span></p>
</li>
<li>
<p><strong>Memory</strong><span><strong>.</strong> The amount of RAM memory your page takes to run. Useful for detecting JS memory leaks!</span></p>
</li>
</ul>
<p><span>They’re obviously tied together very closely and you should keep an eye on each one.</span></p>

<h3><span>Stop talkin’ now, let’s get our hands dirty!</span></h3>
<p></p>
<p><span>To start profiling a page, you need to press the <em>“Record”</em> bullet icon in the bottom left menu:<br /><br /></span><img alt="image" src="http://i.imgur.com/CH6Ea4h.png" /></p>

<p><span>When the icon switches red, reload the page, so that the DevTools can record and profile your website. </span><strong>Remember to turn it off!</strong><span> You’ll probably want to analyze a small bit of interaction at a time, otherwise you’ll be likely overwhelmed by a lot of data.</span></p>

<p><span>At this point, you’ll see something like this:</span></p>
<p><span><span><span></span><img alt="image" src="http://i.imgur.com/TWplk71h.png" /><br /><span></span><br /><span></span></span></span></p>
<p><span><span>Each bar represents an event and its duration. </span></span>As you can guess, every color refers to a different event: <strong>blue</strong> is for assets loading, <strong>yellow</strong> for scripting, <strong>violet</strong> for rendering and <strong>green</strong> for painting.</p>
<p>Just above that, you can see a timeline which represents the same events vertically &ndash; the higher the bar, the lesser the frames per second. This is very useful when you&rsquo;re trying to detect lag spikes and you want to rapidly find the cause. Just look at the higher bar, pair it with its own event and.. debug! :)</p>
<p><span>You can select which time window you want to analyze moving the handles on the timeline highlighted section (or you can scroll with your mouse wheel on it to reduce or expand it).</span></p>

<p><span>Having a clear representation of every page event and its related consequences is cool, and can speed up a lot front-end debugging. But there’s more!</span></p>
<p><span>In this case we can see that a <em>paint event</em> is taking place and that it needs ~25ms to complete. Well, <strong>click on the</strong> <strong>arrow </strong>to the left of the event and you’ll see the best part!<br /><br /></span><img alt="image" src="http://i.imgur.com/aU1Z5JJ.png" /><br /><br /></p>

<p><span>Clicking on the arrow you can inspect the internal processes of the selected event, and see which are the required steps (and how much time they need) for the completion of it! Crazy, huh?</span></p>

<p><span>Plus, if you <strong>hover on an event</strong>, you’ll see a very useful resumé of its properties:</span></p>
<p><span><span><span></span><img alt="image" src="http://i.imgur.com/sCKAteih.png" /><span></span><br /><span></span></span></span></p>
<p><span><br />We just saw how the <strong>Frames</strong> section works, but the <strong>Events</strong> section is pretty much the same thing, so I&rsquo;ll let you discover that by your own. The <strong>Memory</strong> tab, additionally, gives you a visual representation of the RAM memory usage, with other useful info (<em>DOM Node count</em>, <em>Event Listener Count</em>):</span></p>
<p><span><span><span></span><img alt="image" src="http://i.imgur.com/IKKi9Prh.png" /><span></span></span></span></p>
<h3><br />In conclusion</h3>

<p><span>Chrome DevTools is a fantastic piece of software that can help you in many, many ways. We have just scratched the surface - if you want to dive in, you can read the official </span><strong><a href="https://developers.google.com/chrome-developer-tools/">Google Developers documentation</a></strong><span> or read one of the </span><strong><a href="http://www.html5rocks.com/en/search?q=devtools">many tutorials on <em>HTML5 Rocks</em> website</a></strong><span>.</span></p>
<p><span> </span></p>