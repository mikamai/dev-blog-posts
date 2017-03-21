---
ID: 70
post_title: >
  Piping colors with xargs in the unix
  terminal
author: Andrea Longhi
post_date: 2015-10-23 12:44:36
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/23/piping-colors-with-xargs-in-the-unix-terminal/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/131744714649/piping-colors-with-xargs-in-the-unix-terminal
tumblr_mikamayhem_id:
  - "131744714649"
---
<p>
Unix terminals are cool, no doubt: you can even pipe commands. But sometimes something goes wrong, and your&rsquo;re prompted with strange errors.</p>
<p>Last week I needed to free some space on a few remote servers so I started poking with <code>du</code>. I found out the information I needed by piping <code>ls</code> with <code>du</code> with the command <code>ls | xargs du -sh</code>. When I got to the second server, I started to get crazy errors:
</p>

<pre>
$ ls | xargs du -sh
du: cannot access `33[0m33[01;34mapp_link33[0m': No such file or directory
du: cannot access `33[01;34mbackups33[0m': No such file or directory
du: cannot access `33[01;34mbin33[0m': No such file or directory
</pre>

<p>
The directory structure was pretty simple here and I initally saw no reason for the errors:
</p>

<pre>
$ ls 
app_link  backups   bin
</pre>

<p>What you cannot see here in my post is color&hellip; in fact the terminal output on that server was in glorious rainbow colors. Those crazy codes that showed on the du errors were actually the shell representation of those colors. I could switch them off (maybe unaliasing ls?) but they looked good and humans generally prefer colored text rather than dull black characters, especially in the terminal, so I removed them by adding in the pipe chain a clever <code>sed</code> command:<br /></p><pre>
$ ls | sed -r "s/x1B[([0-9]{1,2}(;[0-9]{1,2})?)?[m|K]//g" | xargs du -sh
4,00K all_link
7,9M  backups
3,2M  bin
</pre>

<p>and that fixed the issue.</p>