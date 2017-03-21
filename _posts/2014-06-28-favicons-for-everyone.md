---
ID: 233
post_title: Favicons for everyone!
author: Gianluca
post_date: 2014-06-28 12:26:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/28/favicons-for-everyone/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/90150967939/favicons-for-everyone
tumblr_mikamayhem_id:
  - "90150967939"
---
I think every Web dev know this.

Web development, in particular front-end stuff, can be really though.

There aren’t many standards you can follow when dealing with the evil entities called browsers and so you must often rely on quirks to make your products look good on each one of the over mentioned evil entities.

One example? Favicons.

<!--more-->

As often happens in our field, every browsers handle them differently based on their presence and nature (i.e. .ico or .png and dimensions).

The situation gets even worse if we consider the terrible land of mobile…

Trying to get some knowledge about this topic I asked Google and so I found many articles, guides and blog posts that were really helpful.

However the revelation came from a reference I found in the header.php of a WordPress boilerplate theme called <a href="http://themble.com/bones/">Bones</a>.

The <a href="http://www.jonathantneal.com/blog/understand-the-favicon/">reference</a> (together with <a href="http://mathiasbynens.be/notes/touch-icons">this</a> one that focuses on touch icons) explains me a lot about the topic under discussion and that’s the reason why I want to share it with all of you. :)

I’m not gonna repeat what is written inside the over mentioned references but as a TL;DR I want to present here a snippet that should enable a generic WordPress theme to correctly handle favicons on the majority of the browsers both desktop and mobile:
<pre><code>
    &lt;!-- For iPad with high-resolution Retina display running iOS ≥ 7: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="152x152" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-152x152-precomposed.png"&gt;
    &lt;!-- For iPad with high-resolution Retina display running iOS ≤ 6: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="144x144" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-144x144-precomposed.png"&gt;
    &lt;!-- For iPhone with high-resolution Retina display running iOS ≥ 7: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="120x120" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-120x120-precomposed.png"&gt;
    &lt;!-- For iPhone with high-resolution Retina display running iOS ≤ 6: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="114x114" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-114x114-precomposed.png"&gt;
    &lt;!-- For the iPad mini and the first- and second-generation iPad on iOS ≥ 7: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="76x76" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-76x76-precomposed.png"&gt;
    &lt;!-- For the iPad mini and the first- and second-generation iPad on iOS ≤ 6: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="72x72" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-72x72-precomposed.png"&gt;
    &lt;!-- For non-Retina iPhone, iPod Touch, and Android 2.1+ devices: --&gt;
    &lt;link rel="apple-touch-icon-precomposed" href="&lt;?php echo get_template_directory_uri(); ?&gt;/apple-touch-icon-precomposed-57x57-precomposed.png"&gt;
    &lt;link rel="icon" href="&lt;?php echo get_template_directory_uri(); ?&gt;/favicon.png"&gt;
    &lt;!--[if IE]&gt;
    &lt;link rel="shortcut icon" href="&lt;?php echo get_template_directory_uri(); ?&gt;/favicon.ico"&gt;
    &lt;![endif]--&gt;
    &lt;?php // or, set /favicon.ico for IE10 win ?&gt;
    &lt;meta name="application-name" content="&lt;?php wp_title('|', true, 'left'); ?&gt;"&gt;
    &lt;meta name="msapplication-TileColor" content="#000000"&gt;
    &lt;meta name="msapplication-square70x70logo" content="tiny.png"/&gt;
    &lt;meta name="msapplication-square150x150logo" content="square.png"/&gt;
    &lt;meta name="msapplication-wide310x150logo" content="wide.png"/&gt;
    &lt;meta name="msapplication-square310x310logo" content="large.png"/&gt;
</code></pre>
Anyone who find something wrong with the snippet is obviously invited to let us (or directly me (:) know so that we can correct it.

Cheers! :D