---
ID: 55
post_title: Secure your Rails API
author: Emanuele Serafini
post_date: 2016-01-26 15:22:07
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/01/26/secure-your-rails-api/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/138088508719/secure-your-rails-api
tumblr_mikamayhem_id:
  - "138088508719"
---
<p>There are many approaches to locking down an API. In this post I will you some best practise  aimed to secure your data and your service.</p>

<h3>API key</h3>

<p>Use API keys, not passwords to protect your API from unauthorized access. To authenticate request you can generate an unique token which can be passed through HTTP header. This token shoud be created upfront and made public only to authorized user.</p>

<pre><code>class User &lt; ActiveRecord::Base
  before_validation :generate_access_token, unless: :access_token?


  private

  def generate_access_token
    begin
      self.access_token = SecureRandom.hex
    end while self.class.exists?(access_token: access_token)
  end
end
</code></pre>

<p>Rails offers <code>authenticate_with_http_token method</code>, which automatically checks the Authorization request header for a token and passes it as an argument to the given block. Here&rsquo;s an example used by controller.</p>

<pre><code>module API
  class ResourcesController &lt; ActionController::Base
    before_action :get_resource
    append_before_action :authenticate

    private


    def resource
      @resource ||= Resource.find_by id: params[:id]
    end

    def render_no_record_found
      render json: 'Not Found', status: 404
    end

    def get_resource
      resource || render_no_record_found
    end

    def authenticate_token
      authenticate_with_http_token do |token, options|
        resource.access_token == token
      end
    end

    def render_unauthorized
      self.headers['WWW-Authenticate'] = 'Token realm="Application"'
      render json: 'Invalid Token', status: 401
    end

    def authenticate
      authenticate_token || render_unauthorized
    end
  end
end
</code></pre>

<h3>Identity token</h3>

<p>Another solution to keep private your resource is to allow access to your data using a token instead of <code>id</code>. While id resource is easy to guess a 16 characters random token is hard to imagine. This token should be stored on resource itself and used by controller as parameter to get resource, here&rsquo;s an example:</p>

<pre><code>class Resource &lt; ActiveRecord::Base
  before_validation :generate_identity_token, unless: :identity_token?


  private

  def generate_identity_token
    begin
      self.identity_token = SecureRandom.urlsafe_base64
    end while self.class.exists?(identity_token: identity_token)
  end
end
</code></pre>

<p>In ths case I&rsquo;ve used <code>SecureRandom</code> with <code>urlsafe_base64</code> to be sure that generate token can be used in a url. In your controller you can should change:</p>

<pre><code>def resource
  @resource ||= Feed.find_by identity_token: params[:identity_token]
end
</code></pre>

<h3>Rate limit</h3>

<p>There are several ways to limit API usage. Depending on service you&rsquo;ll need to protect the quality and reliability of your application. For example, rapidly creating content, polling aggressively instead of using webhooks, making API calls with a high concurrency, or repeatedly requesting data that is computationally expensive may result in abuse rate limiting.</p>

<p>If you&rsquo;re not sure what I suggest you to try <a href="https://github.com/kickstarter/rack-attack">rack-attack</a>. Rack::Attack is a rack middleware aimed to protect your web app from bad clients. It allows whitelisting, blacklisting, throttling, and tracking based on arbitrary properties of the request.</p>