---
ID: 282
post_title: >
  jQuery injection for profit and profit,
  cause fun is overrated
author: Giovanni Intini
post_date: 2014-02-21 14:17:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/21/jquery-injection-for-profit-and-profit-cause-fun/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/77379031823/jquery-injection-for-profit-and-profit-cause-fun
tumblr_mikamayhem_id:
  - "77379031823"
---
<p>Working on a project for an important client (more on that in the next months), we came up with the need to inject jQuery on a page, making it available to your functions.</p>

<p>In addition to that we wanted to be sure it didn’t conflict with other jQueries the page was already using, and, if a new version of jQuery was already available, use that instead of injecting a new one.</p>

<p>This whole injection thing is not very complicated per se. You just have to be sure you’re not messing up with what’s already in the page.</p>

<p>Let’s walk through our solution to see how we implemented it. All the code you see here will be <a href="http://coffeescript.org">CoffeeScript</a>, the javascript translation is really straightforward and won’t be provided :)</p>

<p>We start with an empty html document, something like this:</p>

<pre>&lt;!DOCTYPE html&gt;
&lt;head&gt;
  &lt;meta charset="utf-8" /&gt;
  &lt;script src="jquery-injection.js"&gt;&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;/body&gt;</pre>

<p>jQuery-injection.js is the compiled version of our coffeescript source.</p>

<p>The simplest way to inject jQuery would be something like this:</p>

<pre>@Injector = 
    injectjQuery: -&gt;
        script = document.createElement('script')
        script.src = "//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"
        head = document.getElementsByTagName('head')[0]
        
        head.appendChild(script)
</pre>

<p>We’re creating a globally available Injector object with an injectjQuery function that creates a script element, sets its src attribute and adds it to head.</p>

<p>This is all fine and dandy, but what if we already have jQuery? This would load jQuery twice and there would be namespace conflicts. What we want to do is check for an existing jQuery and use jQuery.noconflict() to make sure they don’t clash.</p>

<pre>checkAndInjectjQuery: -&gt;
    if jQuery?
      if @versionCompare(jQuery.fn.jquery, "2.0.3") &lt; 0
        @injectjQuery(true)
      else
        @jq = window.jQuery
    else
      @injectjQuery(false)   
</pre>

<p>What’s going on in here? We check if jQuery exists, and that its version is at least 2.0.3. The version check is done via versionCompare, a coffeescript simplified port of <a href="http://stackoverflow.com/questions/6832596/how-to-compare-software-version-number-using-js-only-number">this code</a> I found on StackOverflow.</p>

<p>If jQuery exists and is sufficiently new, we just map this.jq to window.jQuery. Our local jq variable is what we’re gonna use inside our functions instead of using $.</p>

<p>If you paid enough attention, you will have noticed we’re calling injectjQuery with a parameter.</p>

<pre>  injectjQuery: (saveOriginal) -&gt;
    script = document.createElement('script')
    script.src = "http://ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"
    head = document.getElementsByTagName('head')[0]

    if saveOriginal
      @jq = $.noConflict(true)

    head.appendChild(script)
</pre>

<p>injectjQuery(true) will assume that there’s another jQuery in the page, so it saves it (temporarily) in this.jq, and then cleans after it, making sure it won’t be there when the new jQuery loads.</p>

<p>Wait, did I say “when the new jquery loads”? You can bet I did! Everything we’re doing is asyncronous, so we can’t be sure that the new jQuery is already available.</p>

<p>Avoiding potential issues with that requires some callback magicks. First we edit injectjQuery, adding, immediately after the last line, the following code</p>

<pre>@checkLoaded(saveOriginal)</pre>

<p>checkLoaded is a function that keeps calling itself until jQuery is effectively loaded.</p>

<pre> checkLoaded: (saveOriginal) -&gt;
    window.setTimeout( =&gt;
      if !jQuery?
        @checkLoaded(saveOriginal)
      else
        if saveOriginal
          temp = @jq
          @jq = $.noConflict(true)
          window.jQuery = temp
          window.$ = jQuery
        else
          @jq = jQuery

        @continueInitialization()
    , 500)
</pre>

<p>It recognizes the same saveOriginal parameter we used before, restoring the original jQuery where it belongs, and leaving us with our new jQuery, available through this.jq.</p>

<p>continueInitializazion is the function that should handle all of your post-jquery-loaded needs.</p>

<p>In <a href="https://gist.github.com/intinig/9072053">this gist</a> you’ll find the complete Injector class, with some bonus console.log calls to better understand what’s going on.</p>

<p><img src="http://68.media.tumblr.com/d7cdf5bc4a819ab83787903006e4115e/tumblr_inline_n174c9rO851r11frw.png" /></p>

<p>Thank you for reading up to this last paragraph, and have fun injecting jQuery where you please.</p>