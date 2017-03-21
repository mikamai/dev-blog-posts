---
ID: 206
post_title: >
  Mongoid and awesome_print, a simple
  patch!
author: Gianluca
post_date: 2014-09-22 15:04:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/22/mongoid-and-awesomeprint-a-simple-patch/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/98147578909/mongoid-and-awesomeprint-a-simple-patch
tumblr_mikamayhem_id:
  - "98147578909"
---
Just a couple of days ago I had the need to print out some Ruby objects on my terminal in a nice and well formatted way.

The default <a href="http://www.ruby-doc.org/core-2.1.3/Kernel.html#method-i-p" target="_blank"><code>p</code></a> wasn’t enough for me and so, with just a couple of keystrokes, I end up on the github page of the well known <a href="https://github.com/michaeldv/awesome_print">awesome_print</a> gem.

After adding the gem to my gem <code>.gemspec</code> (yep I need the pretty printing feature for my first gem :)) and requiring it inside the code I stumbled upon a little problem.

Here it is:
<pre><code>
uninitialized constant Moped::BSON (NameError)
</code>
</pre>
<!--more-->

Moped, as written down <a href="http://mongoid.org/en/moped/">here</a>, is the supported MongoDB driver for the Ruby ODM library <a href="http://mongoid.org/en/mongoid/index.html" target="_blank">Mongoid</a> (from version 3 and higher).

As a matter of fact I didn’t tell you that the objects I needed to print out weren’t just PORO (Plain Old Ruby Objects) but rather objects augmented with the <code>Mongoid::Document</code> module.

After a few minutes of tinkering with it I asked Google about the error and found that it was actually a known issue and the workaround was to simply define the missing constant.

But defining it with which value?

After a little bit more tinkering I found that (for the version of the libraries I was using i.e. <code>mongoid 4.0.0</code>, <code>moped 2.0.0</code> and <code>awesome_print 1.2.0</code>) the <code>BSON</code> constant is correctly defined in contrast with the previous one.<code>
</code>

So the solution of my problem was simply:
<pre><code>
Moped::BSON = BSON
</code>
</pre>
I know, this is <strong>r</strong><strong>eally nothing special</strong>.

Anyway there was something bothering me about this “nothing special patch assignment”. It was floating in my code freely with a little comment on it stating something like:
<pre><code># this is to patch the error regarding blah blah blah...</code></pre>
Even to me it wasn’t feeling right.

So by thinking of a possible solution it came to my mind a nice resume I read some time ago about modules. <a href="https://raw.githubusercontent.com/elm-city-craftworks/pr-monthly/gh-pages/b5e5a89847701c4aa7c170cf/sept-2012-modules.pdf" target="_blank">Here</a> it is.

Modules could be used to accomplish various different goals and maybe the most generic one is indeed to encapsulate “stuff”.

This perfectly fits my need and so I build up a little one in which I’m requiring the awesome_print gem and together with it I define the missing constant.

Here it is:
<pre><code>
module PatchedAwesomePrint
  require 'awesome_print'
  ::Moped::BSON = ::BSON
end
</code>
</pre>
Note that here I referred to both constants absolutely with the <code>::</code> operator to properly address them.

In this way my patch is really encapsulated and all I needed to do to get rid of the error is to include the “patching” module i.e:
<pre><code>
include PatchedAwesomePrint
</code>
</pre>
As always I hope this can be helpful to everyone will face the problem I faced myself.

Cheers!