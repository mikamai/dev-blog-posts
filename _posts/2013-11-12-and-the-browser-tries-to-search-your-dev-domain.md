---
ID: 319
post_title: 'and the browser tries to search your .dev domain on the web&#8230;'
author: Matteo Zobbi
post_date: 2013-11-12 13:17:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/12/and-the-browser-tries-to-search-your-dev-domain/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/66774978850/and-the-browser-tries-to-search-your-dev-domain
tumblr_mikamayhem_id:
  - "66774978850"
---
When I use <strong>Google Chrome</strong> or <strong>Safari</strong> with <strong>Pow</strong> and type in a .dev URL (Pow top-level domains), there are some conditions why the browser tries to search the web for the text of the URL instead of going to the URL. <!--more-->

To get around this issue I’ve written a little extension that forces Chrome or Safari to <strong>treat .dev URLs as valid domains</strong>.

You can find the source on <strong><a title="PowDomains Chrome &amp; Safari Extension" href="https://github.com/mapteo/PowDomains">Github</a></strong> and the <strong><a title="PowDomains on Chrome Store" href="https://chrome.google.com/webstore/detail/pow-domains/iigmgfbfgogbfnkgemfbpbncldjlfeoa">extension page</a></strong> on the Chrome Store, but if you mind to install another extension on your browser just remember to type the trailing ’/’ at the end of the URL, then the search will be bypassed.