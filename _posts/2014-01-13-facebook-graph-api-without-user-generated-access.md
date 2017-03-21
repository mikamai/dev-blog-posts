---
ID: 299
post_title: >
  Facebook Graph API without user
  generated access token
author: Stefano Guglielmetti
post_date: 2014-01-13 11:34:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/13/facebook-graph-api-without-user-generated-access/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/73197541656/facebook-graph-api-without-user-generated-access
tumblr_mikamayhem_id:
  - "73197541656"
---
<p><strong>[EDIT] This method is meant to be used only by server side applications, in general it&rsquo;s not a good practice to include the app_id and app_secret in an API call!</strong></p>

<p>Since I started using the Facebook Graph API, the Access Token generation has always been by biggest problem: they are not easy to generate, and they expire!</p>

<p>So you need to make calls to the Facebook Graph API from a Facebook APP and you don&rsquo;t want to use a user generated token? There is an easy way!</p>

<p>There is an alternative method to call the Facebook Graph API that doesn&rsquo;t require a user generated token:</p>

<pre><a href="https://graph.facebook.com/endpoint?key=value&amp;access_token=app_id%7Capp_secret">https://graph.facebook.com/endpoint?key=value&amp;access_token=app_id|app_secret</a></pre>

<p>And&hellip; That&rsquo;s it!! Just give to your app the right permissions, and you&rsquo;ll never need to generate an Access Token again!!</p>