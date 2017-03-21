---
ID: 46
post_title: Ruby builtin Hash
author: Nicola Racco
post_date: 2016-03-03 20:19:21
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/03/03/ruby-builtin-hash/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/140404196794/ruby-builtin-hash
tumblr_mikamayhem_id:
  - "140404196794"
---
Today I was really debated whether to publish another post about SQL (it was a reflective post on how ORM may have ruined us) and a post about the introspection of a ruby builtin feature.

I hate to be repetitive (the SQL post would have been my third post in a row) and I always find enlightening to study language internals because there you can see the real programming magic. So, why not to post on the Ruby Hash and its lookup complexity of `O(1)`?

Then I realized there's plenty of blog posts about the Hash object. And [one of these is written so well that I could never compete](https://blog.engineyard.com/2013/hash-lookup-in-ruby-why-is-it-so-fast).

What I really liked of this post is that the author is not only explaining the theory behind Hash, but he also try to reimplement it.