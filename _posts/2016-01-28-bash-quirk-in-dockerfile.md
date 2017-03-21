---
ID: 54
post_title: Bash quirk in Dockerfile
author: Gianluca
post_date: 2016-01-28 14:00:50
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/01/28/bash-quirk-in-dockerfile/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/138214854889/bash-quirk-in-dockerfile
tumblr_mikamayhem_id:
  - "138214854889"
---
<p>In my last article I wrote about my initial experience with <a href="http://dev.mikamai.com/post/137748096409/ecs-and-kiss-dockerization-of-wordpress">ECS, Docker and Wordpress</a>.</p>

<p>While already practical with the core concepts of Docker, after learning the basics of ECS and creating a working Dockerfile I encountered the need to dive deeper in many aspects of the &ldquo;platform&rdquo; and the technology under discussion.</p>

<!--more-->

<p>One of the aspects that I wanted to investigate more about the technology, i.e. Docker, was related in particular to the Dockerfile optimizations and refactorings.</p>

<p>In the context of refactoring, one of the simple thing I wanted to do was to dry out the creation of some directories needed by NGINX.</p>

<p>To do that I relied on a bit of Bash and tried the following <code>RUN</code>:</p>

<pre><code>RUN mkdir -p /var/cache/nginx/{client_temp,proxy_temp,fastcgi_temp,uwsgi_temp,scgi_temp}
</code></pre>
<p>Unfortunately it didn&rsquo;t worked :(</p>

<p>After a bit of <a href="http://www.urbandictionary.com/define.php?term=Googling&amp;defid=318864">googling</a> around I found the explanation of my problem right <a href="https://github.com/docker/docker/issues/13011">here</a> among the Docker issues.</p>

<p>As stated/reported by <a href="https://github.com/docker/docker/issues/13011#issuecomment-99347948">Vincent</a> (thank you for that ;)) it seems that the problem was related in particular to my relying on Bash.</p>

<p>The fix?</p>

<p>Well it is already presented by Vincent ;)</p>

<pre><code>RUN bash -c 'mkdir -pv /tmp/{a,b,c}'
</code></pre>

<p>I hope this can be helpful for everyone facing the same problem I ecountered ;)</p>

<p>Cheers!</p>