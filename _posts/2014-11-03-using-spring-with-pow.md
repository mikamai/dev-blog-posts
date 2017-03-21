---
ID: 191
post_title: Using spring with pow!
author: Elia Schito
post_date: 2014-11-03 16:39:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/03/using-spring-with-pow/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/101680976209/using-spring-with-pow
tumblr_mikamayhem_id:
  - "101680976209"
---
<p>The other day I was a bit sad for I was aware that <a href="//pow.cx)">pow</a> (the web server) wasn&rsquo;t leveraging <a href="https://github.com/rails/spring">spring</a> (the Rails preloader) with its fast load times. </p>

<h2>Fixing config.ru</h2>

<p>Luckily this deficiency is easily fixed by adding the spring snippet to your <code>config.ru</code>:</p>
    <pre><code class="ruby"># This file is used by Rack-based servers to start the application.

begin
  load File.expand_path('../bin/spring', __FILE__)
rescue LoadError
end

require ::File.expand_path('../config/environment',  __FILE__)
run Rails.application
</code></pre>