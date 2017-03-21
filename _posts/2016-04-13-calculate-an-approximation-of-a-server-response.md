---
ID: 39
post_title: >
  Calculate an approximation of a server
  response time with Rails and Apache
author: Ivan Lasorsa
post_date: 2016-04-13 08:32:10
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/04/13/calculate-an-approximation-of-a-server-response/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/142730453714/calculate-an-approximation-of-a-server-response
tumblr_mikamayhem_id:
  - "142730453714"
---
<p>This post was born from the need of calculating some kind of server response time during a page request. Although this is not perfect, because doesn’t account for the time that the response takes to reach the user, it&rsquo;s pretty accurate on the server side.</p>

<p>In order to have the request start time I added this header in the Apache configuration server:</p>

<p><code>RequestHeader append Request-Started "%t"</code></p>

<p>I didn&rsquo;t use a simple <code>before_filter</code> because it would have not considered the time spent in rack middlewares, for example.</p>

<p>Now in my action I can count on a request header like this:</p>

<p><code>Request-Started: t=1460473552166899</code></p>

<p>Note that this timestamp is in microseconds.</p>

<p>Now in the page view template I can get the total request time doing some simple math:</p>

<pre><code>
server_response_time = current_time - time_from_header

def current_time
  Time.now.utc
end

def time_from_header
  Time.at(request.headers['Request-Started'].split("=").last.to_i / 1_000_000).utc
end
</code></pre>