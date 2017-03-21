---
ID: 254
post_title: 'It&#8217;s all about the context&#8230;or better, $this-&gt;context!'
author: Gianluca
post_date: 2014-05-07 06:35:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/07/its-all-about-the-contextor-better/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/85002570504/its-all-about-the-contextor-better
tumblr_mikamayhem_id:
  - "85002570504"
---
It has great power and it should always be considered.

It’s the context.

No matter what, it always influences the topic under discussion, be it a specific dissertation, a discussion with other people or a PrestaShop (PS) module.

Well in the last case I learned it the hard way.

<!--more-->

A few days ago I was writing my first 1.6 PS module and I found that something was wrong. I was setting a Smarty variable in an hook but it wasn’t showing in the template where the result of the hook call should have been printed.

The problem? Here:
<pre>$this-&gt;smarty-&gt;assign(array(...));</pre>
I was setting (assigning) my variable to the Smarty instance related to a specific hook and not the global one.

As a result, when I was trying to access the over mentioned variable in the template in which the result of the hook call should have been printed I kept on getting NULL.

To summarize I was setting a variable in the Smarty instance visible in the specific template related to the hook I relied on and not in the template in which the over mentioned should have been printed.

The solution? Here:
<pre>$this-&gt;context-&gt;smarty-&gt;assign(array(...));</pre>
In this way you’re sure to set a variable that will be present in all the templates considered during the handling of the hook you’re relying on.