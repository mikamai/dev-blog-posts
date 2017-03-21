---
ID: 164
post_title: API, better with spec
author: Emanuele Serafini
post_date: 2015-01-28 14:55:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/01/28/api-better-with-spec/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/109394132454/api-better-with-spec
tumblr_mikamayhem_id:
  - "109394132454"
---
<p>Rails is a perfect tool to create robust web APIs that can serve different client applications at the same time. Even if building a REST api is pretty simple you need to integrate a complete tests suite to be prepared to modify and maintain your code for future features.</p>

<p>In a previous post we show how to build a small api able to retrieve and store temperatures coming from an Arduino board. In the example below I want to illustrate some tecniche to spec your api controller in order to test expected behaviour.</p>

<p>This code explains how to test a controller shown <a href="http://dev.mikamai.com/post/105596485149/from-arduino-to-rails-through-api">here</a>.</p>

<pre><code>require 'rails_helper'

RSpec.describe Api::TemperaturesController, :type =&gt; :controller do
  describe 'GET #index' do
    context 'without data' do
      it 'returns no temperatures' do
        get :index
        expect(response.status).to eq 404
        expect(response.body).to eq('[]')
      end
    end

    context 'with some data' do
      before { create(:temperature) }
      it 'returns all temperatures' do
        get :index
        expect(response.status).to eq 200
        body = JSON.parse(response.body)
        expect(body).to be_kind_of(Array)

        temperature_value = body.first['value'].to_f
        expect(temperature_value).to eq 20.0
      end
    end
  end

  describe 'POST #create' do
    let(:valid_token) { ActionController::HttpAuthentication::Token.encode_credentials('valid_token') }
    let(:invalid_token) { ActionController::HttpAuthentication::Token.encode_credentials('invalid_token') }

    context 'without token' do
      it 'respond 401 unauthorized message' do
        post :create, { 'temperature': { 'value': 20 } }
        expect(response.status).to eq 401
      end
    end

    context 'with invalid token' do
      it 'respond 401 unauthorized message' do
        request.env['HTTP_AUTHORIZATION'] = invalid_token
        post :create, { 'temperature': { 'value': 20 } }
        expect(response.status).to eq 401
      end
    end

    context 'with valid token' do
      it 'respond 201 message' do
        temperature_value = '20.32'
        request.env['HTTP_AUTHORIZATION'] = valid_token
        post :create, { 'temperature': { 'value': temperature_value } }
        expect(response.status).to eq 201

        body = JSON.parse(response.body)
        expect(temperature_value).to eq temperature_value
      end
    end
  end
end
</code></pre>

<p>In this example we also test token based authentication. Rails offers the authenticate_with_http_token method, which automatically checks the Authorization request header for a token and passes it as an argument to the given block:</p>

<pre><code>authenticate_with_http_token do |token, options|
  ...
end
</code></pre>

<p>In our example Api::TemperaturesController inherith from BaseController which includes authentication process.</p>

<pre><code>class Api::TemperaturesController &lt; BaseController
  before_action :authenticate, only: :create

  ...

end
</code></pre>

<pre><code>class BaseController &lt; ActionController::Base

  def authenticate
    authenticate_token || render_unauthorized
  end

  def authenticate_token
    authenticate_with_http_token do |token, options|
      token == 'valid_token'
    end
  end

  def render_unauthorized
    self.headers['WWW-Authenticate'] = 'Token realm="Application"'
    render json: 'invalid token', status: 401
  end
end
</code></pre>