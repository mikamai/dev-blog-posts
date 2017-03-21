---
ID: 221
post_title: 'Easy interstital pages with RoR &amp; meta tags'
author: Andrew Coates
post_date: 2014-07-25 15:20:22
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/25/easy-interstital-pages-with-ror-meta-tags/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/92832391129/easy-interstital-pages-with-ror-meta-tags
tumblr_mikamayhem_id:
  - "92832391129"
---
Last week I was asked to create a simple interstitial page. Basically, a page that a user would be redirected to between page navigation. Normally they are used for ads, but you can be more creative.
<br /><br />
I came across a simple way to implement the interstitial page into our link shortener app using HTML meta tags. First, a redirect page was created, &rsquo;<i>redirect.html.erb</i>&rsquo;. Inside this page, I used the meta attribute &rsquo;<i>http-equiv</i>&rsquo; to force a refresh after 5 seconds, which along with the <i>&lsquo;content&rsquo;</i> attribute would redirect to a given URL. For example:
<br /><br /><pre><code>&lt;meta http-equiv="refresh" content="5;URL=http://mikamai.com" /&gt;</code></pre>
In order to direct this page to the user&rsquo;s original URL, it was stored in a variable in the controller file, <i>@url</i>, in this case we will define it manually for readability:
<br /><br /><pre><code>@url = "http://mikamai.com"</code></pre>
<br /><br />
Now, the URL can be dynamically changed, and we can swap out the static URL in the &rsquo;<i>redirect.html.erb</i>&rsquo; page for the <i>@url</i> variable:
<br /><br /><pre><code>&lt;meta http-equiv="refresh" content="5;URL=&lt;%= @url %&gt;" /&gt;
</code></pre>
<br /><br />
The process of redirecting to the &rsquo;<i>redirect.html.erb</i>&rsquo; page is handled in the controller.
<br /><br /><pre><code>
def follow
      respond_to do |format|
        format.html { render '/redirect' }  ## Renders the redirect.html.erb page
      end
 end
</code></pre>
<br /><br />
And thus, once a user goes to a shortened link, for example <a href="http://svel.to/1">http://svel.to/1</a>,  an interstitial page will be rendered which after 5 seconds will redirect to <a href="http://mikamai.com">http://mikamai.com</a>.
<br /><br />
I also added a simple countdown using pure JavaScript and the <a href="http://www.w3schools.com/jsref/met_win_settimeout.asp">setTimeout function.</a>:
<br /><br /><pre><code>var timer = 5,
countdownSpan = document.getElementById('countdown');

(function timeDown() {
    countdownSpan.innerHTML = timer--;
    if (timer &gt;= 0) {
        setTimeout(function () {
            timeDown();
        }, 1000);
    }
}());</code></pre>