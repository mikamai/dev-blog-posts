---
ID: 169
post_title: 'PayPal and the broken PrestaShop module [SOLVED] ;)'
author: Gianluca
post_date: 2014-12-22 19:31:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/22/paypal-and-the-broken-prestashop-module-solved/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/105889991234/paypal-and-the-broken-prestashop-module-solved
tumblr_mikamayhem_id:
  - "105889991234"
---
<p>Perfection doesn&rsquo;t exist but we can strive for it.</p>
<p>Refactoring activities can be considered as the embodiment of this striving. In the context of these activities code gets modified and revamped to properly respond to new and/or future possible features.</p>
<p>The results of this process are nothing more than software updates. </p>
<p>But software updates aren&rsquo;t always the outcome of the refactoring process: they are very often (mainly) released to fix bugs and security flaws that otherwise could lead to serious problems.</p>
<p>Sometimes software updates can also be disruptive and bring changes that can cause themselves breakages inside the applications where they are applied.</p>
<p>This is exactly what happened to us just a few days ago when we updated the PrestaShop (PS) Paypal module to address the SSL v3 protocol vulnerability named <a href="https://www.openssl.org/~bodo/ssl-poodle.pdf" target="_blank">POODLE</a>.</p>
<p>As a result we ended up with a broken checkout process and the following error:</p>
<pre><code>
Please try to contact the merchant:
PayPal response:
TIMESTAMP -&gt; 2013-04-23T01:25:10Z
L_ERRORCODE0 -&gt; 10472
L_SHORTMESSAGE0 -&gt; Transaction refused because of an invalid argument. See additional error messages for details.
L_LONGMESSAGE0 -&gt; CancelURL is invalid.
L_SEVERITYCODE0 -&gt; Error
</code>
</pre>
<p><a href="http://www.prestashop.com/forums/topic/241857-paypal-error-10472/" target="_blank">Here</a> there is a thread on the official PS forum concerning the problem.</p>
<p>The important thing is that in this case the module wasn&rsquo;t entirely to blame. </p>
<p>Indeed the problem was connected to the missing update of the files overriding the module templates.</p>
<p>PS modules templates represents the &ldquo;V&rdquo; (i.e. View) in the MVC architectural pattern on which PS modules founds their own roots. These views can be customized by overriding them and put their override files inside the PS theme in use.</p>
<p>This grants a good level of customization but at the same time it also injects some consistent dependencies between the modules and the theme. This can lead to problems like the one over mentioned. </p>
<p>In our case the breakage was related to the change of a variable name inside one of the module templates. This was <span>in particular one of the views </span>used in the context of the payment hook (i.e. express_checkout_payment.tpl.)</p>
<p>Without any apparent reason the module developers changed the smarty variable used inside the value of the input field express_checkout from $PayPal_current_shop_url to $PayPal_current_page.</p>
<p>In our case the variable we were using wasn&rsquo;t correctly valued and so we ended up with the over mentioned breakage.</p>
<p><span>The key takeaways of this post are mainly two.</span></p>
<p><span>The first one is that the use of the MVC architectural pattern doesn&rsquo;t automatically ensures a proper separation from the business logic (i.e. the main functionality) and the view layer. To achieve that the pattern must be correctly implemented.</span></p>
<p><span>The second one is directly bound to the first and could be summarized in:</span></p>
<p>always remember to check which layers gets updated when you release an update or when someone else does.</p>
<p>As always I hope this can help to solve the over mentioned problem.</p>
<p>:)</p>
<p><span>Cheers!</span></p>