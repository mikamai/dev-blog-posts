---
ID: 355
post_title: How to remove ActiveRecord from Rails
author: olistik
post_date: 2012-04-03 15:41:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2012/04/03/howto-remove-activerecord-from-rails/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/20410751002/howto-remove-activerecord-from-rails
tumblr_mikamayhem_id:
  - "20410751002"
---
<p>If your application doesn&rsquo;t use the database you can apply the instructions contained in <a href="https://gist.github.com/2292864" title="How to remove the ActiveRecord dependency from a Rails application" target="_blank">this diff</a> and then issue a</p>
<pre>bundle
</pre>
<p>Tested on Rails 3.2.3</p>