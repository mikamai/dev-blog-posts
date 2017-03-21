---
ID: 311
post_title: >
  Avoiding Safari exit confirmation dialog
  for dirty forms
author: Elia Schito
post_date: 2013-12-03 15:08:53
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/03/avoiding-safari-exit-confirmation-dialog-for-dirty/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/68881543190/avoiding-safari-exit-confirmation-dialog-for-dirty
tumblr_mikamayhem_id:
  - "68881543190"
---
<p><img src="http://68.media.tumblr.com/500cd1cffe8bfe5ff91d18391664fb25/tumblr_inline_mwysk2N7SI1qzz1dw.png" alt="Safari asks close confirmation if a form field isn't clean" /></p>

<p>Safari asks for your confirmation if you accidentally try to close a page if you filled a form but never submitted. This saved my day more than once and it&rsquo;s undoubtedly helpful.</p>

<p>The problem is that if you&rsquo;re sending the form via ajax Safari still thinks you&rsquo;re loosing some data. Luckily the fix is easy:</p>

<pre><code>var postBody = document.querySelector('textarea.post-body')
postBody.defaultValue = postBody.value
</code></pre>