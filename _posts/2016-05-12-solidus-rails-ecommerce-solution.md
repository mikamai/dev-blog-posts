---
ID: 31
post_title: Solidus – Rails eCommerce Solution
author: Matteo Revelli
post_date: 2016-05-12 14:50:26
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/05/12/solidus-rails-ecommerce-solution/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/144250186344/solidus-rails-ecommerce-solution
tumblr_mikamayhem_id:
  - "144250186344"
---
Solidus is a fully open source project, written in Ruby on Rails and it is a continuation (a fork) of Spree Commerce. This platform is designed for retailers who have many products and it is completely scalable, stable and highly customizable. The project is composed of several gems, which are built to work together. These gems are:
<ul>
 	<li>solidus_api (RESTful API)</li>
 	<li>solidus_frontend (Cart and storefront)</li>
 	<li>solidus_backend (Admin area)</li>
 	<li>solidus_core (Essential models, mailers, and classes)</li>
 	<li>solidus_sample (Sample data)</li>
</ul>
<!--more-->

The setup of a new Solidus app is very simple and quick. It is necessary to create a plain Rails 4.2 App and then install the various Solidus gems. To do this, we have to add the following gems to the Gemfile and run, as usual, “bundle install”:
<ul>
 	<li>gem ‘solidus’</li>
 	<li>gem 'solidus_auth_devise’</li>
</ul>
These gems can be installed all together or you can only use the gems you need. For instance, once you have installed the core of the platform, you can combine it with an external frontend (perhaps the one you’ve written). After that, you have to generate the configuration files and migrations:

<code>bundle exec rails g spree:install
bundle exec rake railties:install:migrations</code>

In the next step, we have to run the migrations in order to create the new models in the DB: <code>bundle exec rake db:migrate</code>

After these few passages, we are ready to launch the local server and try the e-store, that will be available at localhost:3000 (to reach the administrator section just add …/admin). In order to test its functionality, Solidus provides a lot of sample data in order to play immediately with the application.
<img src="http://68.media.tumblr.com/9421009f575a272b224efe0ab4908b97/tumblr_inline_o7220mW0DH1u6zrof_540.png" alt="image" />

To start customizing our brand new store, one way is to make a copy form the original source of Solidus (you can download it from GitHub), modify it and paste it into the code of our application. You are able to do this, because the gems are installed globally on your machine.

In that way, the new piece of code will override the old one. We also can do the same thing with the assets, CSS, JS and even methods according to our needs. As we can see, the power of this platform is immense and limitless. This is why I love it! My opinion on the first impact is actually positive, because I think this is a great alternative to other more famous ecommerce development methods.

This is just a getting started for beginner. If you want to try Solidus and know more information, please visit the website <a href="https://solidus.io/">solidus.io/</a>.