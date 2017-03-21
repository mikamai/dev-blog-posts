---
ID: 214
post_title: 'Chrome why don&#8217;t u default?!'
author: Gianluca
post_date: 2014-09-03 08:49:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/03/chrome-why-dont-u-default/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/96526531869/chrome-why-dont-u-default
tumblr_mikamayhem_id:
  - "96526531869"
---
Every distro hopper ends up from time to time on Ubuntu or on its derivatives to check how things goes.

Actually I’m a little bit of a distro hopper, at least in this period, and the distro on which I “hopped” on this time is Xubuntu 14.04 (yep, the LTS).

Now, there are many blog posts and articles that focus on the after installation steps and explain how to setup a productive desktop with all the bells and whistles.

I really like those articles but there are some little things that they don’t (can’t) consider (explain).

One of these things is a strange behavior of XFCE regarding Google Chrome.

<!--more-->

Indeed the Desktop Environment (DE) doesn’t seem to love the browser, in particular when it gets set up as the default one.

This is how you may end up:

<img src="http://68.media.tumblr.com/c316357bba822edcc1c329bea54dadb5/tumblr_inline_n9fzobAboM1ss63kf.png" alt="image" />

Seems like XFCE messed up something here right?

WRONG!

After a lot of googling to understand how the DE handles the preferred applications I found the problem by comparing the other entries of the selectbox and the one that should be present but it’s not.

The latter (i.e. Google’s Chrome one) should be handled by the file <code>google-chrome.desktop</code> located in <code>~/.local/share/xfce4/helpers/</code>.

By opening it and comparing with, for example, with the <code>firefox.desktop</code> located in <code>/usr/share/xfce4/helpers/</code> I found many differences.

So I tried to add, step by step, the missing entries from the latter in the former and look at the results. What I wanted was to have Google Chrome appear in the select box so that I can select it and make it the default browser.

My goal was to stop it (i.e. Google Chrome) from complaining about not being the default one!

By following a trial and error technique I ended up with the following additions:
<pre><code> 
...
Type=X-XFCE-Helper
X-XFCE-Binaries=google-chrome;google-chrome-stable;
X-XFCE-Category=WebBrowser
X-XFCE-Commands=/usr/bin/google-chrome-stable
X-XFCE-CommandsWithParameter=/usr/bin/google-chrome-stable "%s"

[NewWindow Shortcut Group]
</code></pre>
I also changed the lasts lines under the <code>[NewIncognito Shortcut Group]</code> section as following:
<pre><code>
X-XFCE-Commands=/usr/bin/google-chrome-stable --incognito    
X-XFCE-CommandsWithParameter=/usr/bin/google-chrome-stable --incognito "%s"
</code>
</pre>
The result:

<img src="http://68.media.tumblr.com/bb280eed9e27df94a8017e0e44c01bb9/tumblr_inline_n9g0qqKCOf1ss63kf.png" alt="image" />

Now this seems to be the fix I was searching for!

At the end of the story it wasn’t XFCE that messed up the things but Chrome with its flawed <code>google-chrome.desktop</code>.

I hope this can be helpful for someone else other than me ;)

Cheers!