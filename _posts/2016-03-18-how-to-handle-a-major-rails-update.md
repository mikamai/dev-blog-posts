---
ID: 43
post_title: How to handle a major Rails update
author: Diomede Tripicchio
post_date: 2016-03-18 13:52:05
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/03/18/how-to-handle-a-major-rails-update/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/141254431644/how-to-handle-a-major-rails-update
tumblr_mikamayhem_id:
  - "141254431644"
---
<p>These days I am working on a frightening task, a major update of a Rails project (3.2 to 4.2).</p>

<p>I wanted to write a post with the little tricks I am adopting to keep this task as simple as possible; but I found a very good article that explains most of them: <a href="http://thomasleecopeland.com/2016/03/15/make-next-rails-upgrade-easier.html">How to make your next Rails upgrade easier</a></p>

<p>The only addition I can make to this post is the suggestion to use Docker, with docker-compose, to have a completely new environment where you can fearlessly play around with changes and updates.</p>

<p>Last but not least, in my case the target project doesn’t have a real test suite that can be reliable. So now, before doing the real upgrade, I am adding some functional tests that cover the main parts of the site. This way I can be more confident that all works well.</p>

<p>I hope you found the article helpful as I did.</p>

<p>See you soon!</p>