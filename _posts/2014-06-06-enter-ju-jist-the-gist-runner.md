---
ID: 242
post_title: Enter ju-jist, the gist runner
author: Elia Schito
post_date: 2014-06-06 13:33:48
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/06/enter-ju-jist-the-gist-runner/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/87984833354/enter-ju-jist-the-gist-runner
tumblr_mikamayhem_id:
  - "87984833354"
---
<blockquote>
<p>Neo: Ju jitsu? I&rsquo;m gonna learn Ju jitsu?</p>
</blockquote>

<h2>Forewords</h2>

<p>In the <a href="http://dev.mikamai.com/post/87806500959/learning-d3-js-basics-with-ruby-and-opal">last article</a> we explored the internals of <a href="http://d3js.org">D3.js</a>, the data visualization library.<br />
In doing that we ended up with a number of files in <a href="https://gist.github.com/elia/88a4554653df42a6f7c3">a gist</a>.</p>

<p>This morning I thought it&rsquo;d be nice to put up the thing into one of those JavaScript <em>fiddle</em> sites, I looked up a bunch of them: <a href="http://codepen.io">CodePen</a>, <a href="http://jsfiddle.net">JSFiddle</a>, <a href="http://jsbin.com">JS Bin</a> but none of them allowed for arbitrary extensions or loading a code from a Gist<sup id="fnref1"><a href="#fn1" rel="footnote">1</a></sup>.</p>

<p><em>I had to build my own.</em></p>

<h3>The Plan</h3>

<ol><li>load and run gists by visiting URLs in this form: <code><a href="http://ju-jist.herokuapp.com/USER/GIST_ID/FILE_NAME">http://ju-jist.herokuapp.com/USER/GIST_ID/FILE_NAME</a></code></li>
<li>eventually add a home page that will build the right url from the <code>GIST_ID</code> defaulting to <code>index.html</code> as file name</li>
</ol><h2>Section 1: Rack Proxy</h2>

<p>The first thing I did is to preparing a simple proxy rack application that would extract the <em>gist id</em> and <br /><em>file name</em> from the URL:</p>
    <pre><code class="ruby">parts = %r{^/(?&lt;user&gt;[^/]+)/?(?&lt;gist_id&gt;w+)/(?&lt;file&gt;.*)(?:$|?(?&lt;query&gt;.*$))}.match(env['PATH_INFO'])
gist_id = parts[:gist_id]
file = parts[:file]
user = parts[:user]
</code></pre>

<p>Note here how handy are actually <a href="http://mikepackdev.com/blog_posts/4-tuesday-tricks-named-regex-groups">named Regexp groups</a> (introduced in Ruby 1.9).</p>

<p>Then let&rsquo;s be <em>ol&rsquo; school</em> and use <a href="http://elia.schito.me/railsapi.com/public/doc/ruby-v2.0/classes/OpenURI.html"><code>open-uri</code></a> to fetch urls:</p>
    <pre><code class="ruby">contents = open("https://gist.githubusercontent.com/#{user}/#{gist_id}/raw/#{file}")
</code></pre>

<p>Pass it over to Rack:</p>
    <pre><code class="ruby">[200, {}, [contents.read]]
</code></pre>

<p>And wrap everything in a rack app:</p>
    <pre><code class="ruby"># config.ru
def call(env)
  parts = %r{^/(?&lt;user&gt;[^/]+)/?(?&lt;gist_id&gt;w+)/(?&lt;file&gt;.*)(?:$|?(?&lt;query&gt;.*$))}.match(env['PATH_INFO'])
  gist_id = parts[:gist_id]
  file = parts[:file]
  user = parts[:user]
  contents = open("https://gist.githubusercontent.com/#{user}/#{gist_id}/raw/#{file}")
  [200, {}, [contents.read]]
end

run self
</code></pre>

<h2>Section 2: The URL builder</h2>

<p>Next I prepared a simple form:</p>
    <pre><code class="html">&lt;!-- inside index.html --&gt;
&lt;form&gt;
  &lt;input type="text" id="user_and_gist_id"&gt;
  &lt;input type="submit" value="open"&gt;
&lt;/form&gt;
</code></pre>

<p>And then I used <a href="http://opalrb.org">Opal</a> and <a href="http://dev.mikamai.com/post/79398725537/using-native-javascript-objects-from-opal">Native</a> with some vanilla DOM to build the URL and <br />
redirect the user.</p>
    <pre><code class="ruby"># gist-runner.rb
$doc  = $$[:document]
input = $doc.querySelector('input#user_and_gist_id')
form  = $doc.querySelector('form')

form[:onsubmit] = -&gt; submit {
  Native(submit).preventDefault
  user_and_gist_id = input[:value]
  $$[:location][:href] = "/#{user_and_gist_id}/index.html"
}
</code></pre>

<p>And let Rack serve static files:</p>
    <pre><code class="ruby">use Rack::Static, urls: %w[/gist-runner.rb /favicon.ico], index: 'index.html'
run self
</code></pre>

<h2>Conclusion</h2>

<p><a href="http://ju-jist.herokuapp.com">Ju-Jist is up and running</a>, you can see the code from the <a href="http://dev.mikamai.com/post/87806500959/learning-d3-js-basics-with-ruby-and-opal">last article gist</a> live <a href="http://ju-jist.herokuapp.com/elia/88a4554653df42a6f7c3/index.html">on ju-jist</a>.</p>

<p>The code is <a href="https://github.com/mikamai/ju-jist">available on GitHub</a>.</p>

<div class="footnotes">
<hr><ol><li id="fn1">
<p>Actually JSFiddle has some docs for loading Gists, but I wasn&rsquo;t able to make it work. CodePen and others allow for external resources, but GitHub <a href="https://github.com/blog/1482-heads-up-nosniff-header-support-coming-to-chrome-and-firefox">blocks the sourcing of raw contents from Gists</a>. <a href="#fnref1" rev="footnote">↩</a></p>
</li>

</ol></div>