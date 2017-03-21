---
ID: 190
post_title: >
  Google device authentication in your
  Rails app
author: Nicola Racco
post_date: 2014-11-05 17:01:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/05/google-device-authentication-in-your-rails-app/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/101852140929/google-device-authentication-in-your-rails-app
tumblr_mikamayhem_id:
  - "101852140929"
---
Authentication is one of the most common issues with mobile development.
<!--more-->

**Note: this post assumes you are developing an android app, though the logic should be the same for iOS apps with Google SDK.**

Let’s say we have a regular web app that authenticates people via Google OAuth 2 in order to let the user complete some action (e.g. turning on/off a light).

At a certain point we want to add a mobile app that lets the user log in via google and operate on the web app. How can we solve the authentication issue?

A common solution, a tricky one indeed, is to use a WebView component inside the app to simulate the browser behavior. You can read a guide on how to achieve this solution [here](http://www.learn2crack.com/2014/01/android-oauth2-webview.html).

I don’t like tricky solutions so I started searching for a better one.

As every guide will tell you, on Android you can fetch an oauth2 token from google using GoogleAuthUtil, like [in this gist](https://gist.github.com/ianbarber/9607551).

You can use the received token only to interact with google services. If you have to interact with your custom server, there is no way to let it know that you are really you, or to let it fetch informations about you.

Now, to achieve these features, first of all create an Android certificate in your Google Developer Console, as explained in [this guide](http://developer.android.com/google/auth/http-auth.html).

### Sign your messages

[Official doc](https://developers.google.com/accounts/docs/CrossClientAuth#androidIdTokens) to the rescue, if you specify a particular scope when requesting a token (`audience:server:client_id:<your_web_client_id>`) google will return you an _ID token_. This token is intended to be used in post requests over an https channel to sign your messages. It is in fact a json object (containing the account email and a bunch of other information) cryptographically signed with a certificate.

So you could use this token to sign your messages. And what about our Rails app? There we'll use [this fork of the google-id-token gem](https://github.com/Nerian/google-id-token) (because the original one is outdated). Usage is as easy as a pie. In our `APIController` we’ll add the following code:

```ruby
before_filter :authenticate_android_user

private

def user_email
  id_token.try(:email)
end

def authenticate_android_user
  if params[:token].blank? || id_token.nil?
    head 401
    false
  end
end

def id_token
  @id_token ||= begin
    validator = GoogleIDToken::Validator.new
    validator.check params[:token], &#039;my_web_client_id&#039;, &#039;my_android_client_id&#039;
  end
end
```

If an API request has no token param or an invalid one, the rails app will return a 401 status code. Otherwise we can use the user_email method to fetch the user email.

### Offline Access

Another approach is to grant _offline access_ to the server. [Official doc](https://developers.google.com/accounts/docs/CrossClientAuth#offlineAccess) to the rescue again, if you specify a particular scope when requesting a token (`oauth2:server:client_id:<your_web_client_id>:api_scope:resource1 resource2`) google will return you a short-lived authorization code.

The mobile app can send this code to the server app that can exchange it for an access token and a refresh token that will allow the server app to make requests for the specified resources (_resource1_ and _resource2_ in this example).

In our Rails app we have to add the [google-api-client](https://github.com/google/google-api-ruby-client) gem. The following method would exchange the auth token for an access token and a refresh token:

```ruby
def exchange_token auth_token
  client = Google::API::Client.new application_name: &#039;MyApp&#039;, application_version: &#039;1.0.0&#039;
  client.authorization.client_id = &#039;123&#039;
  client.authorization.client_secret = &#039;asd&#039;
  client.authorization.code = params[:code]
  client.fetch_access_token!
  [client.authorization.access_token, client.authorization.refresh_token]
end
```

### Conclusion

What approach should you follow? You need the ID token for sure, because it lets the user identify himself. But if you also want the server to retrieve informations about the user you need both the ID token and the auth token.

When the mobile app starts it has to require two tokens: the ID token (to let the server identify the user) and the auth token (to let the server operate as if it was the user). Then the app should send both of them to the server calling a sort of 'login' API.

Now the server validates the ID Token and, if the user is logging in for the first time, it exchanges the auth token with google for an access token and a refresh token, storing both of them on the DB.

From now on the mobile app should embed the ID token every time it calls the server, to identify itself. And it's done!