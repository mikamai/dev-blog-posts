---
ID: 73
post_title: Tune your factory
author: Emanuele Serafini
post_date: 2015-10-16 13:03:45
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/16/tune-your-factory/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/131281525519/tune-your-factory
tumblr_mikamayhem_id:
  - "131281525519"
---
<p>As a Rails developer you&rsquo;ve surely seen factory like this:</p>

<pre><code>FactoryGirl.define do
  factory :event do
    sequence(:id, 1000)
    sequence(:name) {|n| "Event #{n}" }
    description 'Lorem ipsum Elit proident Excepteur commodo aliqua ut tempor laborum sunt labore incididunt qui.'
    starts_at rand(3.days.ago..Time.now)
    stops_at rand(Time.now..3.days.from_now)
    lat rand(-90.0..90.0)
    long rand(-180.0..180.0)
    country_code 'IT'
    city 'Milan'
    address 'Via Venini 42'
    published false
  end
end
</code></pre>

<p><a href="https://github.com/stympy/faker">Faker</a> is a gem that allow you to generate data. With Faker you can build a large amount of different kind of data with a simple DSL, saving you al lot of time. The same factroy rewritten using Faker would look like this.</p>

<pre><code>FactoryGirl.define do
  factory :event do
    sequence(:id, 1000)
    sequence(:name) {|n| "Event #{n}" }
    description Faker::Lorem.paragraph
    starts_at Faker::Date.between(3.days.ago, Date.today)
    stops_at Faker::Date.forward(3)
    lat Faker::Address.latitude
    long Faker::Address.longitude
    country_code Faker::Address.country_code
    citi Faker::Address.city
    address Faker::Address.street_address
    published false
  end
end
</code></pre>

<p>To use Faker in your rails app simply add <code>gem 'faker'</code> in your gemfile.</p>

<p>Happy building!</p>