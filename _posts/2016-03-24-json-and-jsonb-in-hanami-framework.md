---
ID: 41
post_title: Json and Jsonb in Hanami framework
author: Emanuele Serafini
post_date: 2016-03-24 14:42:33
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/03/24/json-and-jsonb-in-hanami-framework/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/141604787724/json-and-jsonb-in-hanami-framework
tumblr_mikamayhem_id:
  - "141604787724"
---
<p>In the past week I&rsquo;ve tried Hanami, formerly Lotus. Hanami is a web framework based on Ruby, it promises fast response time, security and simplicity.</p>

<p>After following the <a href="http://hanamirb.org/guides/getting-started/">getting started guide</a> I&rsquo;ve started to reproduce a simple web API written in Rails. My goal was to understand Hanami pros and cons. At some point I came across a model with a <code>jsonb</code> attribute. Rails supports <code>json</code> and <code>jsonb</code> out of the box if you&rsquo;re using Postgres 9.2 or greater.
Hanami data mapper supports the most common Ruby data type such as String, Integer, or DateTime, but if you need to store <code>json</code> and <code>jsonb</code> types you need to customize your setup using Postgres builtin type.</p>

<p>Here you can find some code that exaplains how to store this type of data:</p>

<pre><code># lib/ext/pg_jsonb.rb
require 'hanami/model/coercer'
require 'sequel'
require 'sequel/extensions/pg_json'

class PGJsonb &lt; Hanami::Model::Coercer
  def self.dump value
    ::Sequel.pg_jsonb value
  end

  def self.load value
    ::Kernel.Hash(value) unless value.nil?
  end
end
</code></pre>

<pre><code># lib/ext/pg_json.rb
require 'hanami/model/coercer'
require 'sequel'
require 'sequel/extensions/pg_json'

class PGJson &lt; Hanami::Model::Coercer
  def self.dump value
    ::Sequel.pg_json value
  end

  def self.load value
    ::Kernel.Hash(value) unless value.nil?
  end
end
</code></pre>

<pre><code># lib/bookshelf.rb
require_relative './ext/pg_jsonb'
require_relative './ext/pg_json'
...

Hanami::Model.configure do
  # ...
  mapping do
    # ...
    collection :user do
      attribute :id,   Integer
      attribute :options_in_jsonb, PGJsonb
      attribute :options_in_json, PGJson
    end
  end
end.load!
</code></pre>

<p>Let&rsquo;s give Hanami a chance and try to embrace Hanami&rsquo;s model separation (entity / repository).</p>