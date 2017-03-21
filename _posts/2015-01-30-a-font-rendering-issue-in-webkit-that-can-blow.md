---
ID: 162
post_title: >
  A font rendering issue in webkit that
  can blow your head
author: Nicola Racco
post_date: 2015-01-30 16:33:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/01/30/a-font-rendering-issue-in-webkit-that-can-blow/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/109587693559/a-font-rendering-issue-in-webkit-that-can-blow
tumblr_mikamayhem_id:
  - "109587693559"
---
Yesterday I encountered a really weird issue about font rendering on Safari.<!--more--> I had a web page completely styled and working on all browsers/devices. I was missing only a css animation to fadein/fadeout some images.

With the animations running, Safari started to change font aliasing in unrelated parts of the page from ‘initial’ to 'antialiased’ on his own, and than back to 'initial’ at the end of the animation.

Luckily for me, [someone already had got this problem](http://stackoverflow.com/questions/9733011/safari-changing-font-weights-when-unrelated-animations-are-running).

As explained in one of the answers on stackoverflow:

> _When you trigger GPU compositing (eg, through CSS animation), the browser sends that element to the GPU, but also anything that would appear on top of that element if its top/left properties were changed. This includes any `position:relative` elements that appear after the animating one._

> _The solution is to give the animating element `position:relative` and a `z-index` that puts it above everything else. That way you get your animation but keep the (superior IMO) sub-pixel font rendering on unrelated elements._