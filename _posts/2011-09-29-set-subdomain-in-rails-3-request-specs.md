---
ID: 358
post_title: Set subdomain in Rails 3 request specs
author: Elia Schito
post_date: 2011-09-29 13:58:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2011/09/29/set-subdomain-in-rails-3-request-specs/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/10805870538/set-subdomain-in-rails-3-request-specs
tumblr_mikamayhem_id:
  - "10805870538"
---
<p>Every request spec runs inside an <a href="http://railsapi.com/doc/rails-v3.0.8rc1/classes/ActionDispatch/Integration/Session.html" title="Open Integration::Session from railsapi.com">Integration::Session</a> hence just add:</p>
<pre><code class="ruby">before { host! 'mysubdomain.example.com' }</code></pre>
<p>This of course works for Rails3+Rspec2.</p>