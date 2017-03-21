---
ID: 146
post_title: Lightweight line charts with Pista.js
author: Francesco Marassi
post_date: 2015-03-06 14:31:37
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/03/06/lightweight-line-charts-with-pistajs/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/112877844559/lightweight-line-charts-with-pistajs
tumblr_mikamayhem_id:
  - "112877844559"
---
<figure><img src="http://68.media.tumblr.com/e89782d14bb7916c8dd2928821c1d54f/tumblr_inline_nkrdlq2COF1qb1umt.png" alt="image" /></figure><p>Some months ago, while I was working on a new version of <a href="http://trackthisfor.me">trackthisfor.me</a>, I tried to find the best library for timeseries line charts, but even if there are dozens of great libraries (<a href="http://c3js.org/">c3.js</a>, <a href="http://www.flotcharts.org/">flot</a>, <a href="http://www.chartjs.org/">chartjs</a> just to name a few) none of these was perfect for my needs and I ended up building another one.<br /></p><p>The goal was to have a &lt;10kb library (5.6kb as of now), easy to download also on mobile and that works well both with jquery and zepto. <br />Although the dimension is small, it supports some customizations such as:<br />- straight/curve lines;<br />- fillable lines;<br />- multiple lines in the same chart;<br />- tooltip callback support;<br />- custom max/min X Y values;<br />- goal value;</p><figure><img src="http://68.media.tumblr.com/5f9bc1e1b52ab662c1b3bf4f5521f416/tumblr_inline_nkrdt6MYOE1qb1umt.png" alt="image" /></figure><p>There is a lot of stuff missing (such as legends, and a non-time x-axis), but i didn&rsquo;t want to add a lot of bloat. If you want to have a full chart library, you&rsquo;d rather want to use flot or d3.js. :)</p><p>The usage is really easy:</p><pre><code>//index.html<br />&lt;div id="chart"&gt;&lt;/div&gt;</code></pre><pre><code>//script.js<br />$(document).ready(function(){<br />var data= [<br />  { name:"first",<br />    data:[<br />        {value:12, date:"12/14/2014"},<br />        {value:32, date:"12/15/2014"},<br />        {value:56, date:"12/17/2014"},<br />        {value:45, date:"12/19/2014"}<br />    ]<br />  }<br />]<br />options={<br />  height: 150,<br />  width: 600,<br />}<br />$("#chart").pista(data, options);<br /></code></pre><p>Check it out on <a href="https://github.com/urcoilbisurco/pista.js">Github</a> or on this <a href="http://codepen.io/anon/pen/myGbea">CodePen</a> demo!<br /></p>