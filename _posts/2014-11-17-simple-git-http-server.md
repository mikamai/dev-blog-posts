---
ID: 185
post_title: Simple Git HTTP server
author: Elia Schito
post_date: 2014-11-17 14:28:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/17/simple-git-http-server/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/102873984224/simple-git-http-server
tumblr_mikamayhem_id:
  - "102873984224"
---
<p>The other day I had to share a repo to a colleague without him having access to the online repo.</p>

<p>Conscious that git can serve repositories through HTTP I set myself to discover the simplest way to do it without a full blown git hosting app like GitLab or Gitorious.</p>

<p>At that point I took out my google-fu and came up with the following:</p>
    <pre><code class="bash">git update-server-info # this will prepare your repo to be served
ruby -run -ehttpd -- . -p 5000
git clone http://localhost:5000/.git repo_name
</code></pre>

<p>Now by just knowing your IP anyone will be able to clone that repo.</p>

<p><strong>BONUS</strong></p>

<p>If you&rsquo;re a PHP nostalgic OSX 10.10 comes with the <code>php</code> command which is able to serve a directory and interpret any PHP file in it (like <code>mod_php</code> would):</p>
    <pre><code class="bash">git update-server-info # this will prepare your repo to be served
php -S 0.0.0.0:5000 -t .
git clone http://localhost:5000/.git repo_name
</code></pre>

<p><em>Stay tuned for more PHP <strong>and</strong> Ruby awesomeness!</em></p>