---
ID: 157
post_title: Oauth2 on Rails
author: Andrea Longhi
post_date: 2015-02-11 14:33:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/11/oauth2-on-rails/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/110722727899/oauth2-on-rails
tumblr_mikamayhem_id:
  - "110722727899"
---
<p><a href="http://oauth.net/2/">Oauth2</a> has become the defacto standard for online authentication: Twitter, Facebook, Four Square, Amazon are some of the big players that rely on it for their APIs.</p>

<p>The Oauth authentication process requires an authentication <strong>server application</strong> where users login, and a <strong>client application</strong> that uses the server application for user authentication.</p>

<p>We&rsquo;re going to create both the client and server application from scratch with rails using <a href="https://github.com/plataformatec/devise">devise</a>, <a href="https://github.com/doorkeeper-gem/doorkeeper">doorkeeper</a> and <a href="https://github.com/intridea/omniauth-oauth2">omniauth-oauth2</a> gems to speed up the process, make them work together and analyze how the authentication works.</p>
<p>In this first post we&rsquo;re going to build the server application step by step.</p>

<h2>The server application</h2>
<p>Devise will provide authentication, while doorkeeper will allow the app to work as an Oauth2 server. Let&rsquo;s generate a new app:</p>

<pre><code>
rails new oauth-server
</code></pre>

<p>then add devise and doorkeeper gems to the Gemfile:</p>

<pre><code>
gem "devise"
gem "doorkeeper"
</code></pre>

<p>generate the gems boilerplate and migrate the db:</p>

<pre><code>
rails g devise:install
rails g devise User

rails g doorkeeper:install
rails generate doorkeeper:migration

rake db:migrate
</code></pre>

<p>A bunch of files are generated in the process, among them there&rsquo;s the config/initializers/doorkeeper.rb initializer. Let&rsquo;s open this file and modify it to allow doorkeeper to use devise for authentication. Change the settings on the top of it as follows, remembering to comment out the &ldquo;fail&rdquo; instruction:</p>

<pre><code>
resource_owner_authenticator do
  current_user || begin
    session[:user_return_to] = request.fullpath
    redirect_to new_user_session_url
  end
end

admin_authenticator do
  current_user || redirect_to(new_user_session_url)
end
</code></pre>

<p>The <code>resource_owner_authenticator</code> block allows devise authenticated users to access their resources via doorkeeper. If the user is not authenticated we store the url where he came from in the session, and we redirect him to the login page.<br />
The <code>admin_authenticator</code> bock limits the access to the doorkeeper admin area only to registered users. This admin area is used to manage (create, update, destroy) the client application tokens required for the Oauth authentication.</p>
<p>You may want to limit the access to restricted group of user, for example to admins. This code may work for you:</p>

<pre><code>
admin_authenticator do
  current_user &amp;&amp; current_user.admin? || redirect_to(new_user_session_url)
end
</code></pre>

<p>Then let&rsquo;s create a dummy user that we&rsquo;ll use for experimenting in db/seed.rb:</p>

<pre><code>
User.create! email: 'test@test.com', password: 'password', password_confirmation: 'password'

rake db:seed
</code></pre>

<p>Let&rsquo;s create the actual action that will provide the client app with the authenticated user information. We could build a separate controller, but let&rsquo;s use application controller for the sake of brevity:</p>

<pre><code>
class ApplicationController &lt; ActionController::Base
  before_action :doorkeeper_authorize!, only: :me

  def me
    render json: User.find(doorkeeper_token.resource_owner_id).as_json
  end
end
</code></pre>

and update the routes as well:

<pre><code>
Rails.application.routes.draw do
  use_doorkeeper
  devise_for :users

  get '/me' =&gt; 'application#me'
end
</code></pre>

<p>The directive <code>before_action :doorkeeper_authorize!, only: :me</code> applies only to the <code>me</code> action, and it basically looks for a <code>doorkeeper_token</code>, an object built starting from the oauth authorization code, if not found it will render with a <code>401 Unauthorized</code> header. The same token object is then used in the <code>me</code> action body to retrieve the user details, which will be rendered as JSON.</p>

<p>Now we need to define <code>root_path</code> as Devise requires (please read the gem <a href="https://github.com/plataformatec/devise">readme</a>), then we can start the server and see what happens if we try to access the <code>/me</code> page:</p>
<pre><code>
rails s

curl -I localhost:3000/me
  HTTP/1.1 401 Unauthorized
  X-Frame-Options: SAMEORIGIN
  X-Xss-Protection: 1; mode=block
  X-Content-Type-Options: nosniff
  Cache-Control: no-store
  Pragma: no-cache
  WWW-Authenticate: Bearer realm="Doorkeeper", error="invalid_token", error_description="The access token is invalid"
  Content-Type: text/html
  X-Request-Id: 109af7db-5c09-4175-b6f3-cbd902f4caff
  X-Runtime: 0.003485
  Server: WEBrick/1.3.1 (Ruby/2.1.0/2013-12-25)
  Date: Fri, 30 Jan 2015 10:33:46 GMT
  Content-Length: 0
  Connection: Keep-Alive
  Set-Cookie: request_method=HEAD; path=/
</code></pre>
<p>As you can see, since we&rsquo;re not authenticated and we didn&rsquo;t provide a valid token the application responds with a <code>401 Unauthorized</code> header.</p>

<p>Ideally the the client application will come to this url and get the user information, if authorized, and so the login process on the client app can complete.</p>

<p>So,everything is set for authentication, codewise, on the server application. The client application will need to be registered on the oauth-server app in order to be able to authenticate users. Let&rsquo;s go and register it, heading your browser to <strong>http://localhost:3000/oauth/applications/new</strong>. <br />
You will be redirected to the authentication page, fill in the form with the dummy user credentials in order to login (test@test.com, password) and you will be now presented a new form to create the new Oauth client application settings. Fill the form giving the name you prefer (I chose oauth-client) to the app and use this url as the redirect URI: <code>http://localhost:3001/doorkeeper/callback</code>.<br />
This url must follow omniauth (used on the client app) conventions, where <i>doorkeeper</i> stands for the authentication strategy name (we&rsquo;re going to build it later, the name could be anything meaningful for us) and <i>callback</i> is a path fixture for omniauth recognition. We will start the client app on localhost:3001, of course, and please consider switching to the more secure <i>https</i> protocol when you are production ready.<br />
After pressing submit you will be presented with a success page showing the client app authentication settings, which include the application id, the secret key and the callback url. we&rsquo;re going to use them later, so take a note.</p>

<figure style="margin: 0 auto" class=""><img src="http://68.media.tumblr.com/262e191a55b3f8eba4eea8de22c606af/tumblr_inline_nizpwzJFqw1s337n9.jpg" alt="image" /></figure><p>Now let&rsquo;s create the actual action that will provide the client app with the authenticated user information. We should build a separated controller, but let&rsquo;s use application controller for brevity:</p>

<pre><code>
class ApplicationController &lt; ActionController::Base
  before_action :doorkeeper_authorize!, only: :me

  def me
    render json: User.find(doorkeeper_token.resource_owner_id).as_json
  end
end
</code></pre>

<p>The line <code>before_action :doorkeeper_authorize!, only: :me</code> applies only to the <code>me</code> action, and it basically looks for a <code>doorkeeper_token</code>, an object built on the oauth authorization code, if not found it will render with a <code>401 Unauthorized</code> header. The same token object is then used in the <code>me</code> action to retrieve the user information, which will be rendered in json format. Ideally the client application will come to this url and get the user information, if authorized, and so the login process on the client app can complete.<br /> Let&rsquo;s update the routes as well:</p>

<pre><code>
Rails.application.routes.draw do
  use_doorkeeper
  devise_for :users

  get '/me' =&gt; 'application#me'
end
</code></pre>

<p>The server application is now ready. Next time we will build the client application that will fetch the user data from the server. See you soon!</p>