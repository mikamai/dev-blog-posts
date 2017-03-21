---
ID: 249
post_title: Slow AJAX request in WordPress
author: Gianluca
post_date: 2014-05-19 16:39:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/19/slow-ajax-request-in-wordpress/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/86220980814/slow-ajax-request-in-wordpress
tumblr_mikamayhem_id:
  - "86220980814"
---
Ok, you have set up a wonderful WordPress (WP) theme with a lot of AJAX data fetching and animations.

You have also followed the known best practice to route the AJAX request to the admin-ajax WP backend.

Nice eh!…but wait…why it seems so slooooooow?

<!--more-->

Well the answer to this question may vary but in case the problem resides in a long server response then, the first thing you should do is to check which WP plugin are you using.

It may be that one, or worse some of them, are doing unneeded operations that are slowing down the server response.

An example? Posts ordering.

WP has a native functionality to order posts in its backend but there are also some plugins that let you order your content in a simpler way by giving you a drag and drop functionality.

Well I found that one of these was actually giving me the over mentioned problem.

How did I found that? Simple. I firstly checked the response time from my browser network analyzer and identified the problem (i.e. the slow server response for admin-ajax.php). Then I took a look at the plugins I was using and disabled one by one the ones that were giving me AJAX functionality either in the back or the front end.

Actually I was pretty lucky because I identified the bad plugin at the first try… :P

Anyway, moral of the story: if you have problems with AJAX check the plugins you’re using…you may have some surprises! ;)