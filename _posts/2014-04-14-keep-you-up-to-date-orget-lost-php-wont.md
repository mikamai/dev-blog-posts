---
ID: 259
post_title: 'Keep you up-to-date or&#8230;get lost! (PHP won&#8217;t forgive!)'
author: Gianluca
post_date: 2014-04-14 08:51:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/14/keep-you-up-to-date-orget-lost-php-wont/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/82673022501/keep-you-up-to-date-orget-lost-php-wont
tumblr_mikamayhem_id:
  - "82673022501"
---
<p>PHP is getting some pretty nice feature on each new release.</p>
<p>However most of the versions provided by the hosting providers don&rsquo;t perfectly align with the evolution of the language.</p>
<p>In this situation you should really take care on how you write your code!</p>
<p>One example?</p>
<p>Function array dereferencing or if you prefer:</p>
<pre>foo()[0]</pre>
<p>You like it eh ;)</p>
<p>Unfortunately this is available only up from version 5.4.0.</p>
<p>If you&rsquo;re handling some projects that are hosted on servers shipping an older version of the one over mentioned 5.4.0 think twice before start doing what you like :P</p>
<p>P.S: self experienced ;)</p>