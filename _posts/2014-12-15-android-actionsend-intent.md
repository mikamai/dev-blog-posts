---
ID: 174
post_title: Android ACTION_SEND Intent
author: Andrea Venturelli
post_date: 2014-12-15 15:07:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/15/android-actionsend-intent/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/105269596944/android-actionsend-intent
tumblr_mikamayhem_id:
  - "105269596944"
---
When we develop an application we want users to be able to share contents using services and applications already installed on their device such as Facebook, Twitter, SMS, Mail, etc.

This possibilities are infinite so it will be great to find a way to present to users only the applications that can share the content type required; in Android it’s available the <strong>ACTION_SEND Intent</strong>.

<!--more-->

The use of this Intent is very simple:
<ul>
 	<li>Sharing text</li>
</ul>
<div>
<blockquote>
<pre>Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain");
startActivity(Intent.createChooser(sendIntent, "Share using"));</pre>
</blockquote>
</div>
<ul>
 	<li>Sharing binary objects (images, videos,..)</li>
</ul>
<div>
<blockquote>
<pre>Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_STREAM, uriToImage);
shareIntent.setType("image/jpeg");
startActivity(Intent.createChooser(shareIntent, "Share using"));</pre>
</blockquote>
</div>
It’s possible to make available our application in the list of applications that Intent call to share content adding the following code inside manifest.xml:
<blockquote>
<pre>&lt;intent-filter&gt; 
   &lt;action android:name="android.intent.action.SEND" /&gt;
   &lt;category android:name="android.intent.category.DEFAULT" /&gt;
   &lt;data android:mimeType="image/*" /&gt;
&lt;/intent-filter&gt;</pre>
</blockquote>
<em>android:mimeType</em> specifies the mime type which you are interested in listening.