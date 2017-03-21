---
ID: 178
post_title: 'ThreeWP Broadcast: S3 support'
author: Matteo Zobbi
post_date: 2014-12-03 17:33:29
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/03/threewp-broadcast-s3-support/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/104254477594/threewp-broadcast-s3-support
tumblr_mikamayhem_id:
  - "104254477594"
---
<p>If you have a Wordpress Multisite (or Wordpress Network) with the ThreeWP Broadcast plugin and you would like to move your assets on Amazon S3 you certanly have to deal with <span>with one annoying problem</span>.</p>
<p>The problem is, whenever you broadcast a new blog entry on other sites, the copy_attachment() method doesn&rsquo;t work, no attachments are copied on the broadcasted posts.</p>
<p>This gist has a solution: <a href="https://gist.github.com/jbckmn/9652508">https://gist.github.com/jbckmn/9652508</a></p>
<p>With this little hack S3 hosted images are broadcasted <span>correctly</span>.</p>