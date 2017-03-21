---
ID: 148
post_title: 'Oauth2 on Rails: the client application'
author: Andrea Longhi
post_date: 2015-03-02 12:55:23
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/03/02/oauth2-on-rails-the-client-application/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/112508735689/oauth2-on-rails-the-client-application
tumblr_mikamayhem_id:
  - "112508735689"
---
<p>In <a href="http://dev.mikamai.com/post/110722727899/oauth2-on-rails">part 1</a> we completed the Oauth server application. We now need to build the client app:</p>

<pre><code>
rails new oauth-client
</code></pre>

<p>Then add the <a href="https://github.com/intridea/omniauth-oauth2">omniauth-oauth2</a> gem, which allows us to use Oauth2 by just creating a simple configuration (strategy) file:</p>

<pre><code>
gem 'omniauth-oauth2'
</code></pre>

<p>And now we need to to build the strategy.</p>

<h2>The doorkeeper Omniauth strategy</h2>

<p>Omniauth is a versatile gem, allowing programmers to use it with variety of oauth providers, each one with its own different configuration.</p>
<p>This is achieved by using <i>strategies</i>, there are plenty already built for you, but in order to authenticate to our oauth-server application we need to write a custom one. We are naming it &ldquo;doorkeeper&rdquo; after the name of the gem used in the server app that handles Oauth authentication.</p><p>Let&rsquo;s create the file lib/doorkeeper.rb and put the following code in it:</p>

<pre><code>
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    class Doorkeeper &lt; OmniAuth::Strategies::OAuth2
      option :name, 'doorkeeper'
      option :client_options, {
        site:          'http://localhost:3000',
        authorize_url: 'http://localhost:3000/oauth/authorize'
      }

      uid {
        raw_info['id']
      }

      info do
        {
          email: raw_info['email'],
        }
      end

      extra do
        { raw_info: raw_info }
      end

      def raw_info
        @raw_info ||= access_token.get('/me').parsed
      end
    end
  end
end
</code></pre>

<p>Here we define a class, we give the strategy the name &ldquo;doorkeeper&rdquo;, we specify some urls where omniauth will go to manage the server authentication. The <code>#raw_info</code> method instead is used to fetch the information about the user logged on the server application (again, in the <a href="http://dev.mikamai.com/post/110722727899/oauth2-on-rails">previous post</a> we wrote the code, it&rsquo;s inside application_controller.rb). The bulk of this class is just omniauth boilerplate, to see an example and some more details on building strategies you can visit the <a href="https://github.com/intridea/omniauth-oauth2">gem readme</a>.</p>

<p>Now we need to generate the omniauth initializer:</p>

<pre><code>
touch config/initializers/omniauth.rb
</code></pre>

and put this code in it:

<pre><code>
require 'doorkeeper'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :doorkeeper, &lt;application_id&gt;, &lt;application_secret&gt;
end
</code></pre>

<p>We require the strategy we just defined, then we tell omniauth to add it to its current strategies (and eventually to the application middlewares).
Of course we must replace <code>&lt;application_id&gt;</code> and <code>&lt;application_secret&gt;</code> with the values we got when creating the client application configuration on the server (when in the previous post we visited <code>http://localhost:3000/oauth/applications/new</code> &hellip; remember?)</p>

<p>The omniauth configuration is not over yet: we need to add some omniauth routes in config/routes.rb:</p>

<pre><code>
Rails.application.routes.draw do
  root to: redirect('/auth/doorkeeper')

  get '/auth/:provider/callback' =&gt; 'application#authentication_callback'
end
</code></pre>

<p>The first line redirects to the path &rsquo;<code>/auth/doorkeeper</code>&rsquo;, which will be handled by omniauth. The second is the callback url, used when returning from the server authentication process. Let&rsquo;s write the code for its action, directly in <code>application_controller.rb</code>:</p><pre><code>
class ApplicationController &lt; ActionController::Base
  auth = request.env['omniauth.auth']
  render json: auth.to_json
end
</code></pre>

<p>OmniAuth will always return a hash of information after authenticating with an external provider in the Rack environment under the key omniauth.auth. We access that object and render it as json.</p>

<h2>Try it</h2>
<p>It&rsquo;s time to start both applications: oauth-server must run on port 3000, while oauth-client must run on 3001. When you visit localhost:3001 this is what should happen:</p>
<ul><li>you are redirected to <code>/auth/doorkeeper</code>, a url that will start the Oauth authentication using the &ldquo;doorkeeper&rdquo; omniauth strategy</li>
  <li>You are furher redirected to the server-app</li>
  <li>If you aren&rsquo;t authenticated on the server-app you will be redirected to <code>localhost:3000/users/sign_in</code>.</li>
  <li>After successful sign in on the server, you will be redirected on the client app at <code>/auth/doorkeeper/callback</code>. Here you will see the information we got from the server app in json format.</li>
</ul><figure class=""><img src="http://68.media.tumblr.com/f9c2983c1b1d446dc587685058f336ec/tumblr_inline_nj5itn5RzH1s337n9.png" alt="image" /></figure><p>At this point you can save the user information received from the server app in the client app database and build your user data and resources from there. </p>