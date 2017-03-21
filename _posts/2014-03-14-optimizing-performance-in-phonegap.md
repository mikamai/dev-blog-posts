---
ID: 272
post_title: Optimizing performance in PhoneGap
author: Andrew Coates
post_date: 2014-03-14 09:50:21
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/14/optimizing-performance-in-phonegap/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/79543744489/optimizing-performance-in-phonegap
tumblr_mikamayhem_id:
  - "79543744489"
---
<a href="http://www.phonegap.com">PhoneGap</a> is an open-source application framework designed to bridge the gap between multiple mobile development devices. The result is an efficient and simple way to create hybrid mobile apps with a common codebase. 
<br /><br />
Developers create the HTML, CSS, and JavaScript files, and PhoneGap (powered by Apache Cordova) creates the rest.
<br />
One common issue that arises with PhoneGap (and other broad application frameworks) is related to its performance. 
<br />
Some tips to increase performance include:

<h3>Removing (JQuery) animations</h3>

Transistions and animiations have a considerable effect an app&rsquo;s performance. One alternative to JQuery animations is pure CSS animations. By default, 3D CSS animations will be hardware accelerated, but you can force hardware acceleration on 2D animations by adding this CSS rule:
<br /><pre>
 -webkit-transform: translate3d(0,0,0);
transform: translate3d(0,0,0);
</pre>
<h3>Ajax loading</h3>
Favouring Ajax loading will reduce the stress on the processor. If you prefer otherwise, I suggest using the great HTML5 feature &lsquo;prefetch&rsquo; to silently load pages before a user requests them:
<br /><pre>
link rel="prefetch" href="http://dev.mikamai.com/post/79398725537/using-native-javascript-objects-from-opal" /
</pre>

<h3>Consider some external libraries, but avoid JQueryMobile</h3>
JQueryMobile has a bad reputation at the moment. It&rsquo;s large, bloated and slow, and the documentation isn&rsquo;t great. An alternative to JQM is <a href="http://www.sencha.com/products/touch">Sencha Touch</a>. Unlike JQM, Sencha is not designed to work on both desktop and mobile, which makes the library smaller and more optimized for mobile devices. One thing to note about Sencha is that it is not free, so you should expect initial expenses when developing your app with Sencha.
<br /><br />
Note: For a great Sencha + Phonegap guide, see <a href="http://phonegap.com/blog/2013/11/20/SenchaPhoneGap/">here</a>.
<br /><br />
Another useful library I can recommend is <a href="https://github.com/ftlabs/fastclick">FastClick</a>. This will remove the horrible 300ms delay (used to detect double tabs) when pressing a button, following a link, changing pages etc. It will make the application feel a lot smoother.