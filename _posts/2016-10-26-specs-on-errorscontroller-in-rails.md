---
ID: 9
post_title: Specs on ErrorsController in Rails
author: Emanuele Serafini
post_date: 2016-10-26 05:59:21
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/10/26/specs-on-errorscontroller-in-rails/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/152326090409/specs-on-errorscontroller-in-rails
tumblr_mikamayhem_id:
  - "152326090409"
---
<p>There are different approaches to respond to failed requests in Rails.
By default Rails handles exceptions serving static HTML documents stored in the public directory. In particular, 500 (server error) and 404 (not found) responses result in the application’s rendering 500.html and 404.html, respectively.</p>

<p>In our application usually we build custom 404 and 500 error pages served trough <code>ErrorsController</code>. Several guides illustrate how to move your error pages from static html to dynamic views, here some examples:</p>

<ul><li><a href="https://mattbrictson.com/dynamic-rails-error-pages">Dynamic Rails Error Pages</a></li>
<li><a href="https://wearestac.com/blog/dynamic-error-pages-in-rails">Dynamic Error Pages In Rails</a></li>
</ul><p>This article wants to be a simple reminder for execption pages testing. Remember to test expected error pages behaviour inside a inside a <strong>request</strong> spec.</p> The request spec provides a couple of nice things not available in a controller spec. First, it actually hits your application’s HTTP endpoints, as opposed to calling controller methods directly. Running this test insisde a request spec allow us to check the entire functionality specially the options that tells Rails to serve error pages from routes <code>config.exceptions_app = self.routes</code>. 

<p>For example if you want to test a simple <code>ActiveRecord::RecordNotFound</code> for a specific resource:</p>

<div><pre><code class="language-none">require 'rails_helper'

RSpec.describe 'Show Post', type: :request do
  ...
    
  context 'when post is not present' do
    get '/post/not_existing'
    it { expect(response).to have_http_status :not_found }

    it 'returns an error message' do
      expect(response).to have_content('This is not the page you are looking for.')
    end
  end

  ...
end</code></pre></div>