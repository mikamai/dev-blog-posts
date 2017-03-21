---
ID: 65
post_title: How to rotate Rails logs
author: Andrea Longhi
post_date: 2015-11-10 15:21:32
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/11/10/how-to-rotate-rails-logs/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/132938865789/how-to-rotate-rails-logs
tumblr_mikamayhem_id:
  - "132938865789"
---
<a href='http://yukas.by/rails/logs/rotate/logrotate/2015/11/05/how-to-rotate-rails-logs/'>How to rotate Rails logs</a><div class="link_description"><p>Sometimes disk space is just not enough on a server, so you have to be extra careful when you add a new service/application: if you forget to update your logrotate configuration file sooner or later you may have a problem.</p><p>Sometimes no configuration is the best configuration, so I think that handling log rotation automatically via rails can be a good way to prevent that problem. </p><p>This post describes how to handle rails apps log rotation via ruby code, but also how to do it via logrotate, so pick your choice. </p></div>