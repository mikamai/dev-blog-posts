---
ID: 276
post_title: IE Best Standards support and Rails 4.1
author: Elia Schito
post_date: 2014-03-05 18:25:04
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/05/ie-best-standards-support-and-rails-41/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/78664776898/ie-best-standards-support-and-rails-41
tumblr_mikamayhem_id:
  - "78664776898"
---
<p><code>config.action_dispatch.best_standards_support</code> has been removed from Rails 4.1, to force IE to do it&rsquo;s best just add this line to your <code>config/application.rb</code>:</p>

<pre><code class="ruby">config.action_dispatch.default_headers['X-UA-Compatible'] = 'IE=Edge,chrome=1'
</code></pre>