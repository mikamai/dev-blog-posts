---
ID: 257
post_title: >
  How to build a super-simple, clean CSS
  sticky footer
author: Ivan Prignano
post_date: 2014-04-18 13:49:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/18/how-to-build-a-super-simple-clean-css-sticky/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/83095438090/how-to-build-a-super-simple-clean-css-sticky
tumblr_mikamayhem_id:
  - "83095438090"
---
<p><a href="https://stackoverflow.com/tags/sticky-footer/info" target="_blank">Sticky footers</a>, although being a simple concept, have always been a pain to implement in plain CSS.</p>
<p>Lately I had to build one for a project here in Mikamai, and I found myself using a technique I realized not everyone knows.</p>
<p>It boils down to these few lines of CSS code:</p>
<pre>html
    min-height: 100%
    position: relative
    
body
    margin-bottom: 50px
    
footer
    width: 100%
    height: 50px
    position: absolute
    left: 0
    bottom: 0
</pre>
<p>And a basic page markup:</p>
<pre>body
    Meaningful content.
    
    footer
        Yay, I'm sticky!
</pre>
<p>Neat, right?</p>
<p>You&rsquo;re basically giving the body some extra &ldquo;space&rdquo; (via a bottom margin) in which you&rsquo;ll later position your footer absolutely. So, as you can see, the bottom margin on the body equals to your footer&rsquo;s height. </p>
<p>If you&rsquo;re using any CSS preprocessors, you could further improve this piece of code, storing your footer height in a variable. This way, when you&rsquo;ll need to edit your footer you just need to change a single value (ideally in a separated <em>var.scss</em> file).</p>
<p>Have fun!</p>