---
ID: 336
post_title: >
  Stop writing JavaScript and get back to
  Ruby!
author: Elia Schito
post_date: 2012-05-31 08:30:01
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2012/05/31/stop-writing-javascript-and-get-back-to-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/24120251386/stop-writing-javascript-and-get-back-to-ruby
tumblr_mikamayhem_id:
  - "24120251386"
---
<p><em>Cross posted from: <a href="http://elia.schito.me/post/24087273859/stop-writing-javascript-and-get-back-to-ruby">Eliæ</a></em></p>
<p>Remeber that feeling that you had after having tried <a href="http://coffeescript.org">CoffeeScript</a> getting back to a project with plain <a href="http://developer.yahoo.com/yui/theater/video.php?v=eich-yuiconf2009-harmony">JavaScript</a> and felt so constricted?</p>
<p>Well, after re-writing a couple of coffee classes with <a href="http://opalrb.org">OpalRuby</a> I felt exactly that way, and with 14.9KB of footprint (min+gz, jQuery is 32KB) is a complete no brainer.</p>
<p>So let&rsquo;s gobble up our first chunk of code, that will salute from the browser console:</p>
<pre class="ruby">puts 'hi buddies!'
</pre>
<p>You wonder how that gets translated?</p>
<p>Fear that it will look like this?</p>
<pre class="js">'egin',cb='bootstrap',u='clear.cache.gif',z='content',bc='end',lb='gecko',mb='g
cko1_8',yb='gwt.hybrid',xb='gwt/standard/standard.css',E='gwt:onLoadErrorFn',B=
'gwt:onPropertyErrorFn',y='gwt:property',Db='head',qb='hosted.html?hello',Cb='h
ref',kb='ie6',ab='iframe',t='img',bb="javascript:''",zb='link',pb='loadExternal
Refs',v='meta',eb='moduleRequested',ac='moduleStartup',jb='msie',w='name',gb='o
pera',db='position:absolute;width:0;height:0;border:none',Ab='rel',ib='safari',
rb='selectingPermutation',x='startup',m='hello',Bb='stylesheet',ob='unknown',fb
='user.agent',hb='webkit';var fc=window,k=document,ec=fc.__gwtStatsEvent?functi
on(a){return fc.__gwtStatsEvent(a)}:null,zc,pc,kc,jc=l,sc={},Cc=[],yc=[],ic=[],
vc,xc;ec&amp;&amp;ec({moduleName:m,subSystem:x,evtGroup:cb,millis:(new Date()).getTime(
),type:nb});if(!fc.__gwt_stylesLoaded){fc.__gwt_stylesLoaded={}}if(!fc.__gwt_sc
riptsLoaded){fc.__gwt_scriptsLoaded={}}function oc(){var b=false;try{b=fc.exter
</pre>
<p>Nope! <a href="http://youtu.be/LJP1DphOWPs">it&rsquo;s Chuck Testa</a></p>
<pre class="js">(function() {
  return this.$puts("hi buddies!")
}).call(Opal.top);
</pre>
<p><strong>But it will be obviously a mess to integrate it with existing javascript!!1</strong></p>
<p>Let&rsquo;s how to add <code>#next</code> and <code>#prev</code> to the <code>Element</code> class:</p>
<pre class="ruby">class Element
  def next(selector = '')
    element = nil
    `element = jQuery(this.el).next(selector)[0] || nil;`
    Element.new element if element
  end
end
</pre>
<p><strong>But I&rsquo;m on Rails!</strong></p>
<p>Here <a href="http://elia.github.com/opal-rails/">you served</a>:</p>
<pre class="ruby">gem 'opal-rails'
</pre>
<p>Which il provide you a requirable libraries for <code>application.js</code></p>
<pre class="js">//= require opal
//= require rquery
</pre>
<p>and the <code>.opal</code> extension for your files</p>
<pre class="ruby"># app/assets/javascripts/hi-world.js.opal

puts "G'day world!"
</pre>
<p>A template handler:</p>
<pre class="ruby"># app/views/posts/show.js.opal.erb

Element.id('&lt;%= dom_id @post %&gt;').show
</pre>
<p>and a <a href="http://haml-lang.com">Haml</a> filter!</p>
<pre class="haml">-# app/views/posts/show.html.haml

%article= post.body

%a#show-comments Display Comments!

.comments(style="display:none;")
  - post.comments.each do |comment|
    .comment= comment.body

:opal
  Document.ready? do
    Element.id('show-comments').on :click do
      Element.find('.comments').first.show
      false # aka preventDefault
    end
  end
</pre>