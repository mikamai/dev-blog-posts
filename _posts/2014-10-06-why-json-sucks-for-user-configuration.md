---
ID: 201
post_title: Why JSON sucks for user configuration
author: Elia Schito
post_date: 2014-10-06 14:40:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/06/why-json-sucks-for-user-configuration/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/99319315864/why-json-sucks-for-user-configuration
tumblr_mikamayhem_id:
  - "99319315864"
---
<p><em>In the current <em>era of JavaScript</em> <a href="http://json.org">JSON</a> has become super common. In this article I want to make a point about it not being suitable for user configuration.</em></p>

<h2>The good</h2>

<p>Let&rsquo;s start with <em>the good parts</em>. It&rsquo;s not XML. Being born as a data interchange format from the mind of <a href="https://en.wikipedia.org/wiki/Douglas_Crockford">Douglas Crockford</a> it shines for its simplicity in both the definition and easiness in parsing. That&rsquo;s pretty much it. As a configuration tool the only good thing I can say is that many people are accustomed to it, and <em>usually</em> already know its rules.</p>

<h2>The bad</h2>

<p>As popular as it is the JSON format has started to be overused, especially for letting users to configure stuff (that&rsquo;s the point of the article). The first thing to notice here is that JSON is not intended to be used that way, instead we can quote from the official page:</p>

<blockquote>
<p>JSON (JavaScript Object Notation) is a lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate.</p>
</blockquote>

<p><em>source: <a href="http://json.org">http://json.org</a></em></p>

<p>What I&rsquo;d like to highlight is that JSON is a compromise between write/readability <strong>for both humans and machines</strong>. A data interchange format can easily be almost unreadable to humans, that happens all the time with binary formats <strong>but</strong> a configuration format should instead be biased towards humans.</p>

<h2>The ugly</h2>

<p><em>But what&rsquo;s wrong with JSON?</em></p>

<p>I see three things that make JSON fail as a configuration format:</p>

<h3>Strictness</h3>

<p>The format is really demanding. Contrary to the ordinary JavaScript literal object format (which is where it comes from) JSON mandates <strong>double quotes around keys</strong> and <strong>strict absence of commas before closed parentheses</strong>.</p>

<h3>Lack of comments</h3>

<p>The absence of comments is perfectly fine in a data interchange formats where the payload will probably travel on wires and needs to be as tiny as possible. In a configuration format conversely it&rsquo;s plain non-sense as they can be used to help the user explaining each configuration option or for the user to explain why he chose a particular option.</p>

<h3>Readability</h3>

<p>While JSON is quite readable when properly formatted, this good practice is not enforced by the format itself, so it&rsquo;s up to coders good will to add returns, tabs and spaces in the right places. Lucky enough they usually do.</p>

<h2>Conclusions</h2>

<p><em>JSON has become very popular thanks to JavaScript to the point that is also a <a href="http://www.sublimetext.com/docs/3/settings.html">super</a> <a href="http://leopard.in.ua/2013/01/07/chef-solo-getting-started-part-3/">common</a> <a href="https://www.expeditedssl.com/heroku-button-maker">format</a> for user configuration. It wasn&rsquo;t designed to do so and in fact it does an awful job.</em></p>

<p>Almost anything is a good alternative, from <a href="https://en.wikipedia.org/wiki/INI_file">INI</a> to <a href="https://github.com/bevry/cson#readme">CSON</a>. The one I like the most tho is YAML (which in turn has been erroneously used as a data interchange format). Here is the YAML tagline:</p>

<blockquote>
<p>YAML is a human friendly data serialization standard for all programming languages.</p>
</blockquote>

<p><em>source: <a href="http://yaml.org">http://yaml.org</a></em></p>