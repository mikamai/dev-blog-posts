---
ID: 212
post_title: >
  Dead simple view layer with Opal and
  jQuery
author: Elia Schito
post_date: 2014-09-08 12:49:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/08/dead-simple-view-layer-with-opal-and-jquery/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/96967839924/dead-simple-view-layer-with-opal-and-jquery
tumblr_mikamayhem_id:
  - "96967839924"
---
<h1>Dead simple view layer with Opal and jQuery</h1>

<p>Lately I noticed that I started to port from project to project a simple class that implements a basic view layer with Opal and jQuery.</p>

<p><em>In this post we will rebuild it from scratch, you can consider it as an introduction for both Opal and Ruby, and maybe also a bit of OOP</em></p>

<h2>The <code>View</code> class</h2>

<p>If you still generate your HTML on the server (<a href="http://stackoverflow.com/questions/11409243/how-to-seo-a-site-generated-mostly-by-javascript">making happy search engines</a>) and progressively add functionality via CSS and JavaScript then this class is probably a good fit.</p>

<p>Let&rsquo;s start by defining our API. We want an object that:</p>

<ul><li>takes charge of a piece of HTML</li>
<li>can setup listeners and handlers for events on that HTML node</li>
<li>can be easily composed with other objects</li>
<li>exposes behavior, hiding implementation</li>
</ul><h2>Step 1: An object, representing a piece of HTML</h2>

<p>Say we have this HTML, representing a search bar:</p>
    <pre><code class="html">&lt;section class="search"&gt;
  &lt;form&gt;
    &lt;input type="search" placeholder="Type here"&gt;&lt;/input&gt;
    &lt;input type="submit" value="Search"&gt;
  &lt;/form&gt;
&lt;/section&gt;
</code></pre>

<p>This is how we want to instantiate our view:</p>
    <pre><code class="ruby">Document.ready? do
  element = Element.find('section.search')
  search_bar = SearchBar.new(element)
  search_bar.setup
end
</code></pre>

<p>The <code>SearchBar</code> class can look like this:</p>
    <pre><code class="ruby">class SearchBar
  def initialize(element)
    @element = element
  end

  attr_reader :element

  def setup
    # do your thing here…
  end
end
</code></pre>

<h2>Step 2: Adding behavior</h2>

<p>Now that we have a place let&rsquo;s add some behavior. For example we want to clear the field if the ESC key is pressed and to block the submission if the field is empty. We need to concentrate on the <code>#setup</code> method.</p>

<h3>feature 1: “clear the field on ESC”</h3>
    <pre><code class="ruby">class SearchBar
  # …

  ESC_KEY = 27

  def setup
    element.on :keypress do |event|
      clear if event.key_code == ESC_KEY
    end
  end


  private

  def clear
    input.value = ''
  end

  def input
    @input ||= element.find('input')
  end
end
</code></pre>

<p>As you may have noted we&rsquo;re memoizing <code>#input</code> and we&rsquo;re searching the element inside our current HTML subtree. The latter is quite important, especially if you&rsquo;re coming from jQuery and used to <code>$</code>-search everything every time. Sticking to this convention will avoid down the road those nasty bugs caused by selector ambiguities.</p>

<h3>feature 2: “prevent submit if the field&rsquo;s empty”</h3>
    <pre><code class="ruby">class SearchBar
  # …

  def setup
    element.on :keypress do |event|
      clear if event.key_code == ESC_KEY
    end

    form.on :submit do |event|
      event.prevent_default if input.value.empty?
    end
  end


  private

  def form
    @form ||= element.find('form')
  end
end
</code></pre>

<p>YAY! Sounds like we&rsquo;re almost done!</p>

<h2>Step 3: Extracting the <code>View</code> class</h2>

<p>Now seems a good time to extract our view layer, thus leaving the <code>SearchBar</code> class with just the business logic:</p>
    <pre><code class="ruby">class View
  def initialize(element)
    @element = element
  end

  attr_reader :element

  def setup
    # noop, implementation in subclasses
  end
end


class SearchBar &lt; View
  def setup
    # …
  end

  private

  # …
end
</code></pre>

<h2>Step 4: Topping with some “Usability”</h2>

<p>Now that the <code>View</code> class has come to life we can add some sugar to make out lives easier.</p>

<h3>Default Selector</h3>

<p>We already know that the class will always stick to some specific HTML and its selector, it&rsquo;s a good thing then to have some sensible defaults while still allowing to customize. We&rsquo;ll define a default selector at the class definition, this way:</p>
    <pre><code class="ruby">class SearchBar &lt; View
  self.selector = 'section.search'
end

Document.ready? { SearchBar.new.setup }
</code></pre>

<p>The implementation looks like this:</p>
    <pre><code class="ruby">class View
  class &lt;&lt; self
    attr_accessor :selector
  end

  def initialize(options)
    parent   = options[:parent] || Element
    @element = options[:element] || parent.find(self.class.selector)
  end
end
</code></pre>

<h3>ActiveRecord style creation</h3>

<p>I&rsquo;d also like to get rid of that <code>Document.ready?</code> that pops up every time. As always let&rsquo;s define the API first:</p>
    <pre><code class="ruby">class SearchBar &lt; View
  self.selector = 'section.search'
end

SearchBar.create
</code></pre>

<p>And then the implementation</p>
    <pre><code class="ruby">class View
  # …

  def self.create(*args)
    Document.ready? { create!(*args) }
    nil
  end

  def self.create!(*args)
    instance = new(*args)
    if instance.exist?
      instances &lt;&lt; instance
      instance.setup
    end
    instance
  end

  def exist?
    element.any?
  end

  def self.instances
    @instances ||= []
  end
end
</code></pre>

<p>While there we also added a check on element existence and a list of active instances so that we can play nice with JS garbage collection and instantiate the class even if the HTML it needs is missing (e.g. we&rsquo;re on a different page).</p>

<h2>Conclusion</h2>

<p>Hope you enjoyed and maybe learned a couple of things about Opal and opal-jquery, below are some links for you:</p>

<p>The full implementation is available in this gist: <a href="https://gist.github.com/elia/50a723e6133a645b4858">https://gist.github.com/elia/50a723e6133a645b4858</a></p>

<p>Opal&rsquo;s website: <a href="http://opalrb.org">http://opalrb.org</a><br />
Opal API documentation: <a href="http://opalrb.org/docs/api">http://opalrb.org/docs/api</a><br />
Opal-jQuery API documentation: <a href="http://opal.github.io/opal-jquery/doc/master/api">http://opal.github.io/opal-jquery/doc/master/api</a><br />
jQuery documentation: <a href="http://jqapi.com">http://jqapi.com</a> (unofficial but handy)<br />
Vienna, a complete MVC framework: <a href="https://github.com/opal/vienna">https://github.com/opal/vienna</a> (unofficial but handy)</p>