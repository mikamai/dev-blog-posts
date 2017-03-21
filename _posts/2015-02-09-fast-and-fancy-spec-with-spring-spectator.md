---
ID: 158
post_title: >
  Fast and fancy spec with spring +
  spectator
author: Emanuele Serafini
post_date: 2015-02-09 14:58:54
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/09/fast-and-fancy-spec-with-spring-spectator/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/110541312319/fast-and-fancy-spec-with-spring-spectator
tumblr_mikamayhem_id:
  - "110541312319"
---
<p>Several months ago I&rsquo;ve started using the <a href="https://github.com/elia/spectator">spectator</a> gem. I’m using the Rspec testing framework with the spectator event watcher as the test runner. Spring allows us to watch the test run almost immediately after hitting save of a file in the editor.</p>

<p>In my experience spectator is easy to to use, you don&rsquo;t need to configure anything, it just works out of the box. Tools like <a href="https://github.com/ranmocy/guard-rails">guard</a> on the other hand have long list of options to run with.</p>

<p>Using spectator is pretty simple, just install the spectator gem <code>gem install spectator</code> and then run <code>spectator</code> in your root project. Now every time you modify a file in your project, <strong>spectator</strong> searches and launches the associated spec.</p>

<p>Our goal is to integrate spring in test environment in order to improve startup performance and thus speed up the entire process.</p>

<p>My current gemfile setup icludes:</p>

<pre><code># Gemfile

group :development, :test do
  gem 'rspec-rails', '~&gt; 3.0'
  gem 'spring'
end

group :development
    gem 'spring-commands-rspec'
    gem 'spectator'
end
</code></pre>

<p><a href="https://github.com/jonleighton/spring-commands-rspec">spring-commands-rspec</a> implements the rspec command for Spring. As you can see in spring-commands-rspec readme, this gem generates a  <code>bin/rspec</code> file like this:</p>

<pre><code># bin/rspec

#!/usr/bin/env ruby
begin
  load File.expand_path("../spring", __FILE__)
rescue LoadError
end
require 'bundler/setup'
load Gem.bin_path('rspec-core', 'rspec')
</code></pre>

<p>To verify that the installation of spring-commands-rspec is successful you can run <code>./bin/rspec</code> and then verify that spring is running with <code>spring status</code>.
The first startup will take a moment. Now that spring has started the app server for you, subsequent runs will use a forked process from this server instead and return faster.</p>

<p>To integrate faster <code>./bin/rspec</code> command in spectator just create a <code>.spectator.rb</code> file in your root project as above:</p>

<pre><code># .spectator.rb

ENV['RSPEC_COMMAND'] = './bin/rspec'
</code></pre>

<p>Have a nice and fast TDD!</p>