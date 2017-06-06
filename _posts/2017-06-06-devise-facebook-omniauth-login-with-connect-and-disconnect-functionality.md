---
ID: 1010
post_title: >
  Devise Facebook Omniauth login with
  connect and disconnect functionality
author: Gianluca
post_date: 2017-06-06 07:00:30
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/06/06/devise-facebook-omniauth-login-with-connect-and-disconnect-functionality/
published: true
---
When talking about users authentication in Rails land there is one name that generally stands above all the other available gems. This name is <a href="https://github.com/plataformatec/devise">Devise</a>.

I would not be so wrong to call it the de facto standard of Rails authentication considering the time has been around and the vast documentation it has under its belt.

Regarding for example the OAuth2 functionality there is a well documented <a href="https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview">page</a> inside the wiki that describes how to implement it for Facebook.

Unfortunately what presented inside the documentation doesn't always blend well with the other functionalities of an application.

<!--more-->

In Devise documentation however there is a fundamental part regarding the implementation of the <code>facebook</code> callback that is required to register/retrieve the user and handle his sign in. This is done, as you can see, by overriding the <code>Devise::OmniauthCallbacksController</code>.
<pre><code>
class Users::OmniauthCallbacksController &lt; Devise::OmniauthCallbacksController 
  def facebook # You need to implement the method below in your model (e.g. app/models/user.rb) 
    @user = User.from_omniauth(request.env["omniauth.auth"]) 
    if @user.persisted? 
      sign_in_and_redirect @user, :event =&gt; :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind =&gt; "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
</code></pre>
After overriding the controller code the documentation dives into the implementation of the <code>from_omniauth</code> method inside the <code>User</code> model.  Here’s the implementation of the method at the time I'm writing this article:
<pre><code>
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.email = auth.info.email
    user.password = Devise.friendly_token[0,20]
    user.name = auth.info.name   # assuming the user model has a name
    user.image = auth.info.image # assuming the user model has an image
    # If you are using confirmable and the provider(s) you use validate emails, 
    # uncomment the line below to skip the confirmation emails.
    # user.skip_confirmation!
  end
end
</code></pre>
The main logic behind it is the retrieval or creation of a user (i.e. <a href="http://apidock.com/rails/ActiveRecord/Relation/first_or_create">`first_or_create`</a>) by the name of the <code>provider</code> and <code>uid</code> returned by the authentication service (i.e. <code>auth</code>). In case of creation the user gets set the <code>email</code>, <code>password</code>, <code>name</code> and <code>image</code>. If there is the need to store additional data for the user, e.g. the surname, you can simply set the attribute right after the last one present in the previous snippet of code.  If the user is already registered with the given <code>provider</code> and <code>uid</code> it is simply retrieved and returned.

Unfortunately this kind of implementation doesn't let you handle some additional functionality  you may need to add to your application, for example the possibility to manually connect and disconnect the user from his Facebook account.

Recently I was notified about of a bug related exactly to the over mentioned functionality. After a user disconnects his account from Facebook he can not login again through Facebook Oauth.

The problem was indeed related to over mentioned <code>User::from_omniauth</code> implementation. In particular it wasn't considering all the possible user states that the application could generate, like for example a user with a valid account that had previously logged in with Facebook and then disconnected his account from it.

In order to handle the over mentioned user states I should have added some more logic to the <code>User::from_omniauth</code> suggested by the Devise documentation.

To avoid the pollution of this method, however, I decided to extract the logic (i.e. the handling of users connecting and disconnecting from their Facebook account) from it. As a result I ended up implementing a simple PORO (...service object?...) that could be used inside the override of the custom <code>Devise::OmniauthCallbacksController</code>. Here there is the PORO:
<pre><code>
class UserFromFacebookAuth
  attr_reader :auth

  def initialize auth
    @auth = auth
  end

  def user
    if user = User.find_by(provider: auth.provider, uid: auth.uid)
      user
    elsif user = User.find_by(email: auth.info.email, provider: nil, uid: nil)
      user.tap { |o| o.update attributes_from_auth }
    else
      User.create attributes_from_auth
    end
  end

  private

  def attributes_from_auth
    {
      provider:      auth.provider,
      uid:           auth.uid,
      facebook_name: auth.info.name,
      name:          auth.info.first_name,
      surname:       auth.info.last_name,
      email:         auth.info.email,
      password:      Devise.friendly_token[0, 20],
      privacy_a:     false,
      privacy_b:     false
    }
  end
end
</code></pre>
and here the updated version of the <code>Devise::OmniauthCallbacksController</code>:
<pre><code>
class Users::OmniauthCallbacksController &lt; Devise::OmniauthCallbacksController def facebook # The new PORO! @user = UserFromFacebookAuth.new(auth).user if @user.persisted? sign_in_and_redirect @user, :event =&gt; :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind =&gt; "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
</code></pre>
I find the logic quite simple to follow and understand so I will not dive into its explanation.

The point that I want to stress out in this article may sound silly but I think it deserves some attention: online documentation is useful (in case of Devise even more!) but remains a "documentation". It should not be simply "used" but "understood" and adapted to each specific case.

To put it simple: copy&amp;paste? Yes but please, try to understand what you're doing! 😉

Cheers!