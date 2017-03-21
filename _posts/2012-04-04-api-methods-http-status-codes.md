---
ID: 353
post_title: Returning only an HTTP status code
author: olistik
post_date: 2012-04-04 14:34:55
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2012/04/04/api-methods-http-status-codes/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/20467218307/api-methods-http-status-codes
tumblr_mikamayhem_id:
  - "20467218307"
---
<p>If your making an API and some methods of your controller doesn&rsquo;t need to render anything peculiar, you could rely on the <a href="http://api.rubyonrails.org/classes/ActionController/Head.html#method-i-head" title="ActionController::Head" target="_blank">head method</a>.</p>
<pre>head :ok
</pre>
<p>For an exhaustive list of the rails mapping you can use the handy <em>cheat</em> gem, created by <a href="https://twitter.com/joshsusser" title="Josh Susser" target="_blank">Josh Susser</a>.</p>
<pre>$ gem install cheat
$ cheat status_codes
status_codes:
  Use these codes for the #head or #render methods.  ex:
    head :ok
    render :file =&gt; '404.html.erb', :status =&gt; :not_found
  
status_codes:
  Use these codes for the #head or #render methods.  ex:
    head :ok
    render :file =&gt; '404.html.erb', :status =&gt; :not_found
  
  1xx Informational
  
  100 =&gt; :continue
  101 =&gt; :switching_protocols
  102 =&gt; :processing
  
  2xx Success
  
  200 =&gt; :ok
...
</pre>