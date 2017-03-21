---
ID: 293
post_title: >
  Align vertically any element in CSS
  using translateY
author: Ivan Prignano
post_date: 2014-01-27 11:13:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/27/align-vertically-any-element-in-css-using/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/74714567569/align-vertically-any-element-in-css-using
tumblr_mikamayhem_id:
  - "74714567569"
---
<p><span>If you’re reading this, you probably had to face - at least once in your life - the <em>challenge of the vertical align in CSS</em>.</span></p>
<div><span><img alt="image" src="http://68.media.tumblr.com/6db2106b713df86e644dddbcdaf2cffd/tumblr_inline_n0244d2Bn21riz3e2.png" /></span></div>

<p><span>There are </span><a href="http://coding.smashingmagazine.com/2013/08/09/absolute-horizontal-vertical-centering-css/"><span>many</span></a><span>, </span><a href="http://www.vanseodesign.com/css/vertical-centering/"><span>many</span></a><span> solutions. One could argue about which one is cleaner, more elegant, more compatible, and so on. Today I’ll just give you another option &ndash; you decide which one you like the most!</span></p>

<p></p>
<h3>Yet another way to do it</h3>

<p><span>Lately I’m using the</span><span><strong> translateY</strong> method</span><span>. </span></p>
<p><span><span><br /><span></span></span></span></p>
<pre>element {
  position: relative;
  top: 50%;
  transform: translateY(-50%);
}
</pre>
<p><span><span><br /><span></span></span></span></p>
<p><span>The explanation is trivial: we position the element relatively and give it 50% top, pushing it down for the half of the container height. Then, we move it the half of its own height up. And.. That’s it!</span><span></span></p>

<div><span><img alt="image" src="http://68.media.tumblr.com/aec6a862f99077399ba5cbef353d637a/tumblr_inline_n024gk7i1k1riz3e2.png" /></span></div>
<p></p>
<p><span>You can find a live example <a href="http://cdpn.io/dkJzI" title="here" target="_blank">here</a>.</span></p>
<p><span>I like the simplicity and conciseness of this method, and I’m using it quite often. The only downside of this is that <strong>i</strong></span><strong>t only works on browsers that support CSS3 Transforms</strong><span> (basically &gt; IE9).</span></p>
<p><span>If you don&rsquo;t plan to support legacy browsers, give it a try!</span></p>