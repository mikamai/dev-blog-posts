---
ID: 263
post_title: 'Showing bulletpoints in an inline unordered list [CSS, HTML]'
author: Andrew Coates
post_date: 2014-04-04 15:28:01
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/04/showing-bulletpoints-in-an-inline-unordered-list/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/81684147803/showing-bulletpoints-in-an-inline-unordered-list
tumblr_mikamayhem_id:
  - "81684147803"
---
When displaying an unordered list () as inline, you will lose the bullpoints. For times when you want to create a horizontal list with bullets, you can use a non-repeated background image, and positioning them next to your list items using CSS. I&rsquo;ve included a JSFiddle at the end of this post.

<h3>Creating the HTML list</h3>
<pre>
&lt;ul&gt;<br />&lt;li&gt;&lt;a href="#one"&gt;One&lt;/a&gt;&lt;/li&gt;<br />&lt;li&gt;&lt;a href="#two"&gt;Two&lt;/a&gt;&lt;/li&gt;<br />&lt;li&gt;&lt;a href="#three"&gt;Three&lt;/a&gt;&lt;/li&gt;<br />&lt;/ul&gt;
</pre>


<h3>CSS</h3>
<p>Set the unordered list&rsquo;s margin and padding to 0, and remove any bullet points that might appear.</p>
<pre>
ul {
margin: 0;
padding: 0;
list-style-type: 0;
}
</pre>

<p>To create the horizontal list, set the list items to display inline. Add the bullet image using background-image (I&rsquo;m hosting an example bullet on Imgur for now), set to no-repeat, position it using background-position, and give it a left margin:</p>
<pre>
li { 
    display: inline;
    background-image: url(<a href="http://i.imgur.com/qWICm4M.jpg">http://i.imgur.com/qWICm4M.jpg</a>);
    background-repeat: no-repeat;
    background-position: 0 .4em;
    padding-left: .6em;
}
</pre>

Click <a href="http://jsfiddle.net/4Dcbr/">here</a> for a JSFiddle demo