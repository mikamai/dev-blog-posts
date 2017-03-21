---
ID: 290
post_title: >
  Sharing session between your rails 4.x
  app and discourse
author: Andrea Longhi
post_date: 2014-02-03 13:00:06
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/03/sharing-session-between-your-rails-4x-app-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/75476602797/sharing-session-between-your-rails-4x-app-and
tumblr_mikamayhem_id:
  - "75476602797"
---
<p>Discourse is rails application specifically built as a forum application which has gained a lot of momentum recently. Usually you want to integrate discourse with your main rails app, namely on a subdomain of its own, so one of the hottest topics is sharing session between the two apps.</p>
<p>Sharing session between rails 4 apps is rather simple, basically you have to set the same session cookie name and domain scope in <code>session_store.rb</code>:</p>
<pre>    <code>
        YourApp::Application.config.session_store :cookie_store, key: '_your_app', domain: '.yourdomain.com'
    </code>
</pre>
<p>and you have to set the session secret to the same value in <code>secret_token.rb</code>, for example:</p>
<pre>    <code>
        YourApp::Application.config.secret_key_base = 'some_very_long_random_string'
    </code>
</pre>
<p>The caveat with discourse is that <code>secret_token.rb</code> file by default uses <code>Discourse::Application.config.secret_token = </code> so make sure you replace the <code>.secret_token =</code> part with <code><span>.secret_key_base =</span></code>, failing that it will not work properly ;-)</p>