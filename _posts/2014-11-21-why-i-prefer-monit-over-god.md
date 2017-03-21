---
ID: 183
post_title: Why I prefer monit over god
author: Emanuele Serafini
post_date: 2014-11-21 10:29:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/21/why-i-prefer-monit-over-god/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/103193594049/why-i-prefer-monit-over-god
tumblr_mikamayhem_id:
  - "103193594049"
---
<p><a href="http://mmonit.com/">monit</a> and <a href="http://godrb.com/">god</a> are two powerful process monitoring tools. If you&rsquo;re developing a rails application running on vps, you&rsquo;ll need to monitor your active processes/tasks (web server, database and application server &hellip; ) so to be advised when an error occurs.</p>

<p>Both tools use a specific DSL for defining processes, the main difference being that god uses ruby as configuration language.<br />
May think &ldquo;Cool! I can write in my beloved ruby&rdquo;, but in my opinion monit configuration files are much clearer. At first sight monit configuration files are easily readable while god files use specific labels whose structure is not immediately understandble. Here&rsquo;s an example of the same task written in monit and god:</p>

<pre><code>### MONIT ###

check process httpd
  with pidfile /etc/httpd/run/httpd.pid
  start program = "/etc/init.d/httpd start"
  stop program = "/etc/init.d/httpd stop
  if failed port 80 protocol http then restart
  if totalmem &gt; 1024 MB for 2 cycles then restart
  if 5 restarts within 5 cycles then alert
  set alert foo@bar


### GOD ###

 God.watch do |w|
  w.name = "httpd"
  w.pid_file = "/var/run/httpd.pid"
  w.start = "/etc/init.d/httpd start"
  w.stop = "/etc/init.d/httpd stop"

  w.transition(:up, :start) do |on|
    on.condition(:process_running) do |c|
      c.running = false
    end
  end

  w.restart_if do |restart|
    restart.condition(:memory_usage) do |c|
      c.above = 1024.megabytes
      c.times = 2
    end
  end

  w.transition([:start, :restart], :up) do |on|
    on.condition(:tries) do |c|
      c.times = 5
      c.notify = 'foo@bar'
    end
  end
</code></pre>

<p>Another important feature that god lack is a web interface. Monit with few settings gives you a basic web interface, from which you can watch and control all your active processes.</p>

<img src="http://68.media.tumblr.com/f37fe803d4279847f774760d027284dc/tumblr_inline_nfdwnlfW891qfdn20.png" /><p>Last but not least, monit&rsquo;s website <a href="http://mmonit.com/monit/">home page</a> is really well done ;)</p>