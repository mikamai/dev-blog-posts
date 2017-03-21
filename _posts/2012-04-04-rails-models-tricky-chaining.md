---
ID: 352
post_title: >
  Chaining the hell out of ActiveRecord
  models
author: olistik
post_date: 2012-04-04 21:49:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2012/04/04/rails-models-tricky-chaining/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/20486824951/rails-models-tricky-chaining
tumblr_mikamayhem_id:
  - "20486824951"
---
<p>So, you want a <strong>quick and dirty</strong> way to obtain all the models in your Rails app that contain relations targeting a collection (like an <em>has_many</em> relation)?</p>
<pre class="brush: ruby">Dir['app/models/**/*.rb']
  .map {|model| File.basename(model, '.rb')}
  .map(&amp;:camelize)
  .map(&amp;:constantize)
  .select {|section|
    section.reflections.find {|name, reflection|
      reflection.collection?
    }
  }.map(&amp;:to_s)
</pre>
<p><a href="http://en.wikipedia.org/wiki/Law_of_Demeter" title="Demeter law" target="_blank">Demeter law</a>?</p>
<p><a href="http://www.urbandictionary.com/define.php?term=you+got+served&amp;defid=705447" title="You got served" target="_blank"><img align="middle" alt="You got served" height="240" src="http://cache.abovethelaw.com/uploads/2012/02/You-Got-Served-youve-been-service-of-process.gif" width="360" /></a></p>
<p>kind of.. ^_^</p>