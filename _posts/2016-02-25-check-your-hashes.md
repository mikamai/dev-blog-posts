---
ID: 47
post_title: Check your hashes
author: Emanuele Serafini
post_date: 2016-02-25 14:53:53
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/02/25/check-your-hashes/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/139971017694/check-your-hashes
tumblr_mikamayhem_id:
  - "139971017694"
---
<p>In this brief post I want to report a brief experiece regarding a Ruby 2.0+ feature.</p>

<p>A few weeks ago I wanted to enforce the name of the arguments accepted when initializing a simple PORO.</p>

<p>The object itself was pretty simple and its purpouse was to abstract the access and add some behavior to another one. Guess what, the encapsulated object was an Hash.</p>

<p>Thanks to the fact that I was on Ruby 2.0+ (specifically the last stable, i.e. 2.3.0) I immediately looked at <strong>keyword arguments</strong>.</p>

<p>This feature is already well known and there are actually many blog posts that describe how to use it. My favorite article is probably <a href="http://www.justinweiss.com/articles/fun-with-keyword-arguments/">Justin&rsquo;s post</a>.</p>

<p>I&rsquo;ll introduce some code I&rsquo;ll explain later:</p>

<pre><code>class Resource
  attr_reader :type, :id

  def initialize(resource_type:, resource_id:)
    @type = resource_type
    @id = resource_id
  end

  %i[name description].each do |method|
    define_method(method) { instance.send method }
  end

  def instance
    @instance ||= type.constantize.find(id)
  end

  def permitted?
    permitted_types.include? type
  end

  def not_permitted?
    !permitted?
  end

  def permitted_types
    self.class.permitted_types
  end

  def self.permitted_types
    %w[NewsItem Gate Course Document Gallery Video Survey Content]
  end
end
</code></pre>

<p>In this example I&rsquo;ve created a simple PORO which can be used to interact with different ActiveRecord <code>resources</code> such as NewsItem, Gate, Course, Document Gallery, Video, Survey and Content.</p>

<p>I wrote the code with some specs and so I felt I had my back somehow guarded.</p>

<p>Despite that, when I put the object in use, I stumbled in a banal <code>ArgumentError</code>. The code was actually a simple initialization of my object: <code>Resource.new home_block_params</code></p>

<p>I knew that <code>home_block_params</code> was an <code>Hash</code> and I also knew that it was composed by the expected key and value pairs, i.e. <code>resource_type</code> and <code>resource_id</code>.
You will probably have already realized what I clumsily missed: the nature of the <code>Hash</code> keys.
They were indeed strings while they had to be symbols!</p>

<p>I wrote some specs (just for fun ;)) to clear the things up:</p>

<pre><code>describe '#initialize' do
  it 'raises an ArgumentError if no resource_type and resource_id arguments has been specified' do
    expect do
      described_class.new
    end.to raise_error(ArgumentError)
  end

  it 'raises an ArgumentError if resource_type and resource_id arguments has been specified as strings' do
    expect do
      described_class.new('resource_type' =&gt; 'News', 'resource_id' =&gt; 1)
    end.to raise_error(ArgumentError)
  end

  it 'does not raise any error if resource_type and resource_id arguments has been specified as strings' do
    expect do
      permitted_home_block_resource
    end.not_to raise_error
  end
end
</code></pre>

<p>The quick solution in my case was to rely on <code>Hash#symbolize_keys</code>. Unfortunately this is something given by <a href="http://api.rubyonrails.org/classes/Hash.html#method-i-symbolize_keys">Ruby On Rails</a>.</p>

<p>Anyway the take home message here is to properly refresh your memory when dealing with some midly new language features and to always consider the nature of <code>Hash</code> objects keys!</p>