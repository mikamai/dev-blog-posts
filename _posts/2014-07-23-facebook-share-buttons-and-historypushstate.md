---
ID: 222
post_title: >
  Facebook share buttons and
  history.pushState()
author: Ivan Prignano
post_date: 2014-07-23 16:47:24
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/23/facebook-share-buttons-and-historypushstate/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/92643124079/facebook-share-buttons-and-historypushstate
tumblr_mikamayhem_id:
  - "92643124079"
---
<p>If you&rsquo;ve ever tried to use Facebook <a href="https://developers.facebook.com/docs/plugins" target="_blank">social plugins</a>&lsquo; latest version in your website, you might have noticed how smooth and streamlined is now the <a href="https://developers.facebook.com/docs/plugins/share-button" target="_blank">embedding process</a>.</p>
<p>Select a button type, include the SDK, copy-paste a snippet of code and&hellip; you&rsquo;re done! You can also leave the <em>data-href</em> attribute blank, so the plugin will like/share the current page. Awesome!</p>
<p>At this point you might ask yourself, &ldquo;Well, will this nice little button work with my super cool AJAX-powered app?&rdquo;. The answer is <strong>not really</strong>.</p>
<p><img src="http://68.media.tumblr.com/03d30826b699a45daf506f20b22bc336/tumblr_inline_n96avqasCH1riz3e2.png" /></p>

<h2>The problem</h2>
<p>Facebook social plugins won&rsquo;t work properly when your application routing is based on Javascript&rsquo;s <strong>history.pushState()</strong> API – this includes a large number of <strong>Rails</strong> applications that rely on <a href="https://github.com/rails/turbolinks" target="_blank">Turbolinks</a> for their navigation.</p>
<p>This is because the page metadata is parsed by Facebook SDK only when a full page load is triggered. So, if I navigate from <em>example.com/page-one</em> to <em>example.com/page-two</em> and try to click on my Facebook share button, it <strong>will prompt a share for /page-one</strong>.</p>
<h2>The solution</h2>
<p>How to solve this? It&rsquo;s actually quite easy. A small snippet (here in CoffeeScript, but easily convertible in plain JavaScript) that will update our button and retrigger a parse will be enough:</p>
<pre><code>updateFbButtonHref = (el) -&gt;
  $fbButton = $(el).find('.fb-share-button')
  $fbButton.attr('data-href', window.location.href)

  FB.XFBML.parse()
</code></pre>
<p>Now, we can bind this function to the &ldquo;page:load&rdquo; event (in the case of a Turbolinks application), or generally after every pushState():</p>
<pre><code>
$(document).on 'page:load', -&gt;
  updateFbButtonHref('.my-fb-button-container')
</code></pre>
<p>And we&rsquo;re cool! Our button will now share the correct page.</p>