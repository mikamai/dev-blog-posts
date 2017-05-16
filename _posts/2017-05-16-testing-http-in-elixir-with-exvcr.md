---
ID: 1435
post_title: Testing HTTP in Elixir with ExVCR
author: Emanuel Carnevale
post_date: 2017-05-16 13:28:20
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/05/16/testing-http-in-elixir-with-exvcr/
published: true
---
Studying Elixir, I decided to write a client for an existing API and I wanted to do it in the correct way™: writing a complete test coverage.
My choice was to use <a href="https://github.com/edgurgel/httpoison" target="_blank" rel="noopener noreferrer">HTTPoison</a> (since I like Poison for JSON parsing) and <a href="https://github.com/parroty/exvcr" target="_blank" rel="noopener noreferrer">ExVCR</a> to write the API tests.

So, what is ExVCR? If you come from the Rails land, it's the Elixir version of VCR, it:
"Record and replay HTTP interactions library for elixir. It’s inspired by Ruby’s VCR, and trying to provide similar functionalities."
It allows you to make API calls and record those on "cassettes", allowing you to replay the recordings at every test run, decoupling the test phase from the API server and permitting offline tests.

<!--more-->To start using it, we have to add in `config/mix.exs` the correct dependency:

<script src="https://gist.github.com/ecarnevale/3a3a10b5bbae694d725de5a956d04d81.js?file=mix.exs"></script>

Since we chose HTTPoison that uses Hackney as HTTP client, we have to specify the correct adapter, so our test file will look like:

<script src="https://gist.github.com/ecarnevale/3a3a10b5bbae694d725de5a956d04d81.js?file=api_client.empty.exs"></script>

The API we are writing the client for requires a login through basic_auth and if successful yields a cookie you can use for all the subsequent communications, until it expires.
Our first test is then:

<script src="https://gist.github.com/ecarnevale/3a3a10b5bbae694d725de5a956d04d81.js?file=api_client.exs"></script>

Having everything (tests and cassettes included) under source control, we don't want to put any sensitive data inside the repo. So we have username and password in ENV variables.

In <code>fixtures/vcr_cassettes/api_client</code> we find a <code>login.json</code> where our request and the response have been recorded. If we look into it we have:

<script src="https://gist.github.com/ecarnevale/3a3a10b5bbae694d725de5a956d04d81.js?file=login.json"></script>

We can see that the cassette has recorded the username and password we sent and the Cookie we got as response. Wanting to put everything in git we want to avoid these information to be written on disk. ExVCR gave you the opportunity to filter parameters based on a regular expression or simply censor request and/or response headers.

The whole list available was the following:

<code>
filter_sensitive_data(body)
filter_request_header(header, value)
filter_url_params(url)
strip_query_params(url)
</code>

Unfortunately, using hackney, basic_auth data were placed in the option part of the request and no function to filter data in the option were given. So I wrote a patch to add this missing functionality and I added <code>filter_request_options</code> to the bunch. I then submitted a <a href="https://github.com/parroty/exvcr/pull/102" target="_blank" rel="noopener noreferrer">Pull Request</a> that was merged to the benefit of everyone.

With this new little tool available, we can now go in <code>config/test.exs</code> and we can write:

<script src="https://gist.github.com/ecarnevale/3a3a10b5bbae694d725de5a956d04d81.js?file=test.exs"></script>

If we run <code>mix test</code> again (remember to delete the cassette beforehand, otherwise ExVCR will use the saved one without contacting the API server), our newly censored cassette will be:

<script src="https://gist.github.com/ecarnevale/3a3a10b5bbae694d725de5a956d04d81.js?file=login.censored.json"></script>

Now we can write all the tests we want without the fear to put sensitive data in our repos. I hope that this new functionality albeit small is going to be helpful to someone else too.