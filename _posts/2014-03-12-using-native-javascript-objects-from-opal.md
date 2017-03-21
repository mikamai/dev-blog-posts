---
ID: 273
post_title: >
  Using native JavaScript objects from
  Opal
author: Elia Schito
post_date: 2014-03-12 22:23:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/12/using-native-javascript-objects-from-opal/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/79398725537/using-native-javascript-objects-from-opal
tumblr_mikamayhem_id:
  - "79398725537"
---
<blockquote>
  <p><strong>Question:</strong> can I call JS functions from Opal?
  <strong>Answer:</strong> totally!</p>
</blockquote>

<p>Opal standard lib (stdlib) includes a <code>native</code> module, let&rsquo;s see how it works and wrap <code>window</code>:</p>

<pre><code class="ruby">require 'native'

window = Native(`window`) # equivalent to Native::Object.new(`window`)
</code></pre>

<p>Now what if we want to access one of its properties?</p>

<pre><code class="ruby">window[:location][:href]                         # =&gt; "http://dev.mikamai.com/"
window[:location][:href] = "http://mikamai.com/" # will bring you to mikamai.com
</code></pre>

<p>And what about methods?</p>

<pre><code class="ruby">window.alert('hey there!')
</code></pre>

<p>So let&rsquo;s do something more interesting:</p>

<pre><code>class &lt;&lt; window
  # A cross-browser window close method (works in IE!)
  def close!
    %x{
      return (#@native.open('', '_self', '') &amp;&amp; #@native.close()) ||
             (#@native.opener = null &amp;&amp; #@native.close()) ||
             (#@native.opener = '' &amp;&amp; #@native.close());
    }
  end

  # let's assign href directly
  def href= url
    self[:location][:href] = url
  end
end
</code></pre>

<p>That&rsquo;s all for now, bye!</p>

<pre><code class="ruby">window.close!
</code></pre>