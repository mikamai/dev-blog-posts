---
ID: 307
post_title: Integrate bootstrap 3 without gem
author: Emanuele Serafini
post_date: 2013-12-09 13:32:21
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/09/integrate-bootstrap-3-without-gem/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69482494851/integrate-bootstrap-3-without-gem
tumblr_mikamayhem_id:
  - "69482494851"
---
<p>Twitter Bootstrap is based on Less.js, the popular dynamic CSS scripting language. In your Rails project you may prefer to use Sass instead of Less, because it’s already supported by Rails out of the box, or because your application already contains a large amount of Sass code, or possibly just because you’re more familiar with it.</p>
<p>At first glance, this seems to be a serious problem for Twitter Bootstrap: it’s not appropriate for a large portion of the Rails community that prefers Sass, since it was implemented with the Javascript-centric Less.js technology.</p>
<p>In addition, I recently ran into <a href="https://github.com/yabawock/bootstrap-sass-rails/issues/62">this problem</a> using bootstrap with gem. In Rails 4.0, precompiling assets no longer automatically copies non-JS/CSS assets from <code>vendor/assets</code> and <code>lib/assets</code> so glyphicons font does not work out of the box.</p>
<h3>Stylesheets</h3>
<p><a href="https://github.com/jlong/sass-bootstrap">Sass-twitter-bootstrap</a> is not a gem, but instead is just a github repo containing Twitter’s translated code. To use it in your Rails app, just clone the repo and copy Sass source files right into your application like this:</p>
<pre><code>$ git clone <a href="https://github.com/jlong/sass-twitter-bootstrap.git">https://github.com/jlong/sass-twitter-bootstrap.git</a>

$ cp -r sass-twitter-bootstrap/lib your_app/assets/stylesheets/twitter
</code></pre>
<p>And now since the Rails asset pipeline supports Sass out of the box, we’re good to go… almost! If you run your app now, you’ll see an error:</p>
<pre><code>ActionView::Template::Error (Undefined variable: "$baseline".
(in /path/to/app/assets/stylesheets/twitter/forms.scss))
</code></pre>
<p>To fix this, there’s a simple solution remove line <code>*= require_tree .</code> from <code>application.css</code> and require bootstrap.scss directly.</p>
<pre><code>/*
 * This is a manifest file that'll automatically include all the stylesheets available in this directory
 * and any sub-directories. You're free to add application-wide styles to this file and they'll appear at
 * the top of the compiled file, but it's generally better to create a new file per style scope.
 *= require_self
 *= require twitter/bootstrap
*/
</code></pre>
<p>The problem is caused because the translated Sass version was designed to be included once using the bootstrap.scss file, which in turns includes all of the other files.</p>
<h3>Javascripts and font</h3>
<p>Simply, <a href="http://getbootstrap.com/">download Twitter Bootstrap 3 source</a>. In the downloaded archive you will see a <code>js</code> and <code>fonts</code> folders. For js files copy content like this:</p>
<pre><code>$ mkdir your_app/assets/javascripts/twitter
$ cp -r bootstrap-3.0.3/js/* your_app/assets/javascripts/twitter
</code></pre>
<p>Include the font:</p>
<pre><code>$ mkdir your_app/assets/fonts
$ cp -r bootstrap-3.0.3/fonts/* your_app/assets/fonts
</code></pre>
<p>To get the <a href="http://glyphicons.com/">glyphicons</a> font working I had to add a line to the <code>config/application.rb</code> file. Add the following within class Application &lt; Rails::Application.</p>
<pre><code>config.assets.paths &lt;&lt; Rails.root.join('app', 'assets', 'fonts')
</code></pre>
<p>Now we need to update the <code>_glyphicons.scss</code> and change the paths to the font urls. So you have to replace:</p>
<pre><code>@font-face {
  font-family: 'Glyphicons Halflings';
  src: url('#{$icon-font-path}#{$icon-font-name}.eot');
  src: url('#{$icon-font-path}#{$icon-font-name}.eot?#iefix') format('embedded-opentype'),
       url('#{$icon-font-path}#{$icon-font-name}.woff') format('woff'),
       url('#{$icon-font-path}#{$icon-font-name}.ttf') format('truetype'),
       url('#{$icon-font-path}#{$icon-font-name}.svg#glyphicons_halflingsregular') format('svg');
}
</code></pre>
<p>With:</p>
<pre><code>@font-face {
font-family: 'Glyphicons Halflings';
src: url(font-path('glyphicons-halflings-regular.eot') + "?#iefix") format('embedded-opentype'),
     url(font-path('glyphicons-halflings-regular.woff')) format('woff'), 
     url(font-path('glyphicons-halflings-regular.ttf'))  format('truetype'),
     url(font-path('glyphicons-halflings-regular.svg') + "#glyphicons_halflingsregular") format('svg');
}
</code></pre>