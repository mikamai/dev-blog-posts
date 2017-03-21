---
ID: 325
post_title: Multiple Authors for Ruhoh
author: Nicola Racco
post_date: 2013-10-30 13:35:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/10/30/multiple-authors-for-ruhoh/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/65521436180/multiple-authors-for-ruhoh
tumblr_mikamayhem_id:
  - "65521436180"
---
Recently we spent a bit of time trying to setup a better, faster and nerdier alternative for our developers blog. If we could host our blog on github it would be really nerd like.

[Ruhoh](http://ruhoh.com/) seems to be a [better alternative](http://dickfeynman.ruhoh.com/technical/playing-with-ruhoh/) over [Jekyll](http://jekyllrb.com/), so we tried to setup a static blog using it, but it requires to replicate the post author info on each post, and this is not DRY!

We ended up writing a small plugin for this called: [Ruhoh Multiple Authors](https://github.com/mikamai/ruhoh-multiple_authors)