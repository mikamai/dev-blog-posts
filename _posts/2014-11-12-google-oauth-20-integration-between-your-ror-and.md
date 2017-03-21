---
ID: 187
post_title: >
  Google OAuth 2.0 integration between
  your RoR and mobile applications
author: Emanuel Carnevale
post_date: 2014-11-12 18:29:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/12/google-oauth-20-integration-between-your-ror-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/102461925589/google-oauth-20-integration-between-your-ror-and
tumblr_mikamayhem_id:
  - "102461925589"
---
<p>We have a working web application that uses Google+ OAuth 2.0 authentication to allow access only to owners of our own email address domain.
The solution is out of the box or close to it: put <code>devise</code>, <code>omniauth</code> and <code>omniauth-google-oauth2</code> in your Gemfile, set it up, generate the keys <a href="https://console.developers.google.com">here</a> and you are ready to go.</p>

<p>But having decided to create a native mobile application to access the web application DB, we had to implement the same login method. In the past we could’ve used <code>token_authenticable</code> of Devise fame, but since it has been (rightfully) deprecated for security reasons we have two options:
1. We could use <a href="https://developers.google.com/accounts/docs/CrossClientAuth">Cross Client Authentication</a> and you can check <a href="http://dev.mikamai.com/post/101852140929/google-device-authentication-in-your-rails-app">here</a> this approach. Unfortunately the documentation seems to be heavily Android centric, but we’ll delve into the iOS approach in the future.
2. We could ask for a short lived access_token on the native app and then exchange it with the web application to provide authentication.</p>

<p>In this article we focus on how to adapt the web application to accept the access_token and grant access when it has been validated.</p>

<p>We have to add a before_filter to ApplicationController:</p>

<p>We are passing store: false, so the user is not stored in the session and a valid access_token is needed for every request.</p>

<p>Now we just need to add a method to the User class and that&rsquo;s it.</p>

<p>Here we had to validate the token against the Google API to ensure the token is still valid and to retrieve the email address and check if it&rsquo;s in our database.</p>

<p>Obviously the first Cross Client Authentication is preferable since we are not passing an access_token (albeit short lived), but this approach has the benefit of not tying our application to any SDK ;)</p>