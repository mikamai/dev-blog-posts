---
ID: 245
post_title: Mocking Rails, for SPEED
author: Elia Schito
post_date: 2014-05-28 16:25:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/28/mocking-rails-for-speed/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/87110678174/mocking-rails-for-speed
tumblr_mikamayhem_id:
  - "87110678174"
---
<p><strong>DISCLAIMER:</strong> This article is by no means related to the recent quarrel about TDD&rsquo;s death (be it presumed or actual), nor to <a href="http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html">this article</a> or <a href="http://martinfowler.com/articles/is-tdd-dead/">these hangouts</a>.</p>

<p><img src="http://cl.ly/1b1K1y390v2b/Trollface.svg" alt=":trollface:" /></p>

<p>Now that we&rsquo;re clear let us get to the real title:</p>

<h2>How I test Rails stuff without loading Rails and learned to use <em>mkdir</em></h2>

<h3>Chapter 1: <em>mkdir</em><sup id="fnref1"><a href="#fn1" rel="footnote">1</a></sup></h3>

<p><em>this chapter is quite brief and straightforward…</em></p>

<p>Starting from Rails 3 every folder you create under <code>app</code> will be used for <strong>autoloading</strong>, this means that you can group files <em>by concern</em><sup id="fnref2"><a href="#fn2" rel="footnote">2</a></sup> instead of grouping <em>them by MVC role</em>, or you maybe want to add new &ldquo;roles&rdquo;.</p>

<p>For example the app I&rsquo;m currently working on contains looks like this:</p>
    <pre><code class="text">app/
├── admin
├── assets
├── authentication
├── authorization
├── controllers
├── forms
├── helpers
├── journaling
├── mailers
├── models
├── proposals
├── utils
└── views
</code></pre>

<p>The <code>journaling</code> folder contains <code>journal.rb</code> (a simple Ruby class) and <code>journal_entry</code> (an ActiveRecord model).</p>

<h3>Chapter 2: Rails not required</h3>

<p>Talking about that <code>Journal</code>, it doesn&rsquo;t really need Rails to be used, it&rsquo;s ok with any <code>JournalEntry</code>-ish object that exposes this API:</p>
    <pre><code class="ruby">class FauxJournalEntryClass &lt; Struct.new(:difference, :user, :model_type, :model_id, :action)
  def save!
    @persisted = true
  end

  def persisted?
    @persisted
  end
end
</code></pre>

<p>So it can be used and tested without loading <code>rails</code>, that gives opportunity to have the corresponding spec to give us really fast feedback. Enter a <em>lighter</em> version of our <code>spec_helper</code>:</p>
    <pre><code class="ruby"># spec/spec_helper.rb
RSpec.configure do |config|
  # yada, yada, rspec default config from `rspec --init`
end
</code></pre>

<p>along with a speedy spec:</p>
    <pre><code class="ruby">require 'light_spec_helper'
require File.expand_path('../../app/journal/journal', __FILE__)

describe Journal do
  # examples here...
end
</code></pre>

<p>and have the normal <code>spec_helper</code> to load the <em>light</em> one:</p>
    <pre><code class="ruby"># spec/spec_helper.rb
require 'light_spec_helper'
# stuff coming from `rails generate rspec:install`...
</code></pre>

<h3>Chapter 3: Mocking Rails</h3>

<p><em>…without making fun of it</em></p>

<p>With time i often found myself in the need to add a logger, to check current environment or to know the app&rsquo;s root from this kind of plain classes.</p>

<p>In a Rails app the obvious choices are <code>Rails.logger</code>, <code>Rails.env</code> and <code>Rails.root</code>. In addition I really can&rsquo;t stand that <code>File.expand_path</code> and <code>light_spec_helper</code>, they&rsquo;re just ugly.</p>

<p>My solution was to:</p>

<ul><li>have a <em>faux</em> <code>Rails</code> constant that quacks like Rails in all those handy methods</li>
<li>add the most probable load paths into the <em>light</em> <code>spec_helper</code></li>
<li>rename the the <em>rails-ready</em> <code>spec_helper</code> to <code>rails_helper</code> and keep the <em>light</em> helper as the default one</li>
</ul><p>Here&rsquo;s how it looks:</p>
    <pre><code class="ruby"># spec/spec_helper.rb
require 'bundler/setup'
require 'pathname'

unless defined? Rails
  module Rails
    def self.root
      Pathname.new(File.expand_path("#{__dir__}/.."))
    end

    def self.env
      'test'
    end

    def self.logger
      @logger ||= begin
        require 'logger'
        Logger.new(root.join("log/#{env}.log"))
      end
    end

    def self.cache
      nil
    end

    def self.fake?
      true
    end

    def self.implode!
      class &lt;&lt; self
        %i[fake? cache logger env root implode!].each do |m|
          remove_method m
        end
      end
    end
  end
end

$:.unshift *Dir[File.expand_path("#{Rails.root}/{app/*,lib}")]

RSpec.configure do |config|
  # yada, yada, rspec default config from `rspec --init`
end
</code></pre>

<p>Notice anything of interest?<br />
I bet you already guessed what <code>Rails.implode!</code> does…</p>

<p>Here&rsquo;s the shiny new <code>rails_helper</code>:</p>
    <pre><code class="ruby">ENV["RAILS_ENV"] ||= 'test'
require 'spec_helper'
Rails.implode! if Rails.respond_to? :implode!

require File.expand_path('../../config/environment', __FILE__)
require 'rspec/rails'
# Requires supporting ruby files with custom matchers and macros, etc,
# in spec/support/ and its subdirectories.
Dir[Rails.root.join('spec/support/**/*.rb')].each do |f|
  f = Pathname(f).relative_path_from(Rails.root.join('spec'))
  require f
end

ActiveRecord::Migration.maintain_test_schema! # rails 4.1 only

RSpec.configure do |config|
  # stuff coming from `rails generate rspec:install`...
end
</code></pre>

<h3>Conclusion</h3>

<p>The introduction of <a href="https://github.com/rails/spring#usage"><code>Spring</code></a> in Rails 4 has actually made all this stuff useless, running RSpec through it makes it blazingly fast.</p>

<p><em>Sorry if I wasted your time and I didn&rsquo;t even make you smile.</em></p>

<div class="footnotes">
<hr><ol><li id="fn1">
<p>I don&rsquo;t actually use <code>mkdir</code>, I know how to use <a href="http://dev.mikamai.com/post/77705936763/pimp-my-textmate2-ruby-edition">my editor</a> <a href="#fnref1" rev="footnote">↩</a></p>
</li>

<li id="fn2">
<p>Not <a href="http://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns">those concerns</a> <a href="#fnref2" rev="footnote">↩</a></p>
</li>

</ol></div>