---
ID: 37
post_title: Test RDBMS should == production RDBMS
author: Andrea Longhi
post_date: 2016-04-21 11:48:20
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/04/21/test-rdbms-should-production-rdbms/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/143160661659/test-rdbms-should-production-rdbms
tumblr_mikamayhem_id:
  - "143160661659"
---
<p>In Mikamai we developed <a href="http://svel.to/"><abbr title="Italian words that means `quick`">svelto</abbr></a>, our own url shortener application in 2008, but at that time the web was still on IPv4 (and it mostly is today, for what it matters). When registering visits we store also the visitor’s remote ip, and a few days ago we received a visit from an IPv6 number which ended up in an application error:</p>

<!--more--> 

<pre><code>Mysql2::Error: Data too long for column 'remote_ip'
at row 1: INSERT INTO `visits` (
  `created_at`, `remote_ip`, `shorty_id`, `updated_at`, `user_agent`
) VALUES (
  '2016-04-10 19:08:44', '2600:1011:b126:3242:9f8c:89ba:c2cd:1299'
  ...
)
</code></pre>
<p>The issue was caused by a column size limit on the remote_ip field. The limit was set at 15 chars, enough only for IPv4.</p>

<p>I immediately wrote a regression test in order to expose the issue but&hellip; no red sign after running it, it passed right away!<br />The reason was that the test environment was using <strong>Sqlite3</strong> RDBMS instead of <strong>Mysql</strong>, as all other environments did.</p>

<p>After fixing this issue the test was now red, with the problem exposed and ready for fix (easy fix, of course&hellip; I added a migration and alter the limit of the coulumn to 45 chars in order to address the IPV6 format).</p>

<p><strong>TL;DR</strong>: you should use the same RDBMS setup in all your application environments, as long as there is no real reason to make them different, or you could get a bite!</p>