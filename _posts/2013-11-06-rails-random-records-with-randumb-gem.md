---
ID: 322
post_title: RAILS RANDOM RECORDS WITH RANDUMB GEM
author: Andrea Longhi
post_date: 2013-11-06 09:04:01
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/06/rails-random-records-with-randumb-gem/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/66173865197/rails-random-records-with-randumb-gem
tumblr_mikamayhem_id:
  - "66173865197"
---
<p>If you ever needed to pick some random record inside a rails app, you probably ended up writing some named scope like the following:</p>
<pre>  class Product &lt; ActiveRecord::Base
    scope :random, lambda { |n| scoped.limit(n).order('RAND()') }
  end

  Product.random(3)
</pre>
<p>which picks 3 random products from the database.</p>
<p>Of course this works fine, but it has a major drawback: the SQL random function is hardcoded in the model and if in the future you ever happen to change the database (ie you decide to host your app on Heroku, where postgres is the default RDBMS) your code will not work anymore, because it was developed according to MySQL APIs.</p>
<p>Here it comes <em>randumb</em> to rescue: just add in your gemfile <code><strong>gem 'randumb'</strong></code> and you&rsquo;re done, you can query your db for random products with the same interface and the gem will apply the correct random function according to your current RDBMS. Happy coding!</p>