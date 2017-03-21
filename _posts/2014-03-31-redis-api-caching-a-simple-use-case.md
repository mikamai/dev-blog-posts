---
ID: 265
post_title: 'Redis API caching: A simple use case'
author: Giovanni Intini
post_date: 2014-03-31 11:25:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/31/redis-api-caching-a-simple-use-case/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/81281769803/redis-api-caching-a-simple-use-case
tumblr_mikamayhem_id:
  - "81281769803"
---
<p>Last saturday MIKAMAI hosted an interesting meetup, the <a href="http://www.meetup.com/Open-Source-Saturday-Milano/events/171093972/" title="Open Source Saturday Milano" target="_blank">Open Source Saturday Milano</a>.</p>
<p>The format for the OSS is simple, people form groups and work on contributing to existing open source projects, or start new ones.</p>
<p>I couldn&rsquo;t resist, and pitched my idea of adding an optional layer cache to <a href="https://github.com/mikamai/ruby-lol/" title="ruby-lol" target="_blank">ruby-lol</a>, an open source wrapper to the <a href="http://developer.riotgames.com/" target="_blank">Riot Games API</a> that I contributed writing.</p>
<p>I got just one person interested in the project, but we were ready to go, and nothing would have stopped us, ruby-lol needed some cache, and boy, we wanted to cache that calls!</p>
<p>A layer cache for an API wrapper is usually a good idea, and something most people using said wrapper have to implement anyway. Being in a love relationship with <a href="http://redis.io/" title="Redis" target="_blank">Redis</a>, we decided to go that way.</p>
<p>Now, everyone does caching in Redis, but everyone does it their own way, so feel free to complain about the way we did it, and feel even freer to contribute better solutions to our problem!</p>
<p>Our starting point was this:</p>
<pre><code>client = Lol::Client.new "my_api_key"</code></pre>
<p>We wanted to change it like this:</p>
<pre><code>client = Lol::Client.new "my_api_key", :redis =&gt; "redis://whatever.local:6379", :ttl =&gt; 900</code></pre>
<p>Our initialization already supported an option hash, so we just had to delegate redis initialization to a new method:</p>
<pre><code>    def initialize api_key, options = {}
      @api_key = api_key
      @region = options.delete(:region) || "euw"
      set_up_cache(options.delete(:redis), options.delete(:ttl))
    end

    def set_up_cache(redis_url, ttl)
      return @cached = false unless redis_url

      @ttl = ttl || 900
      @cached = true
      @redis = Redis.new :url =&gt; redis_url
    end
</code></pre>
<p>Don&rsquo;t be scared by all those instance variables, they are all supported by proper written accessor methods!</p>
<p>After having changed the initialization we hit our first problem. You call the API methods like this:</p>
<pre><code>summoner = client.summoner.by_name "intinig"
team = client.team.get summoner_id</code></pre>
<p><em>client.summoner</em> and <em>client.team</em> are respectively instances of <em>SummonerRequest</em> and <em>TeamRequest</em> (both subclasses of the more general <em>Request</em>) and they have no access to client. So we had to add the capability, when instantiating said classes, to pass them caching data.</p>
<p>To do that we did this:</p>
<pre><code>    # Returns an options hash with cache keys
    # @return [Hash]
    def cache_store
      {
        redis:  @redis,
        ttl:    @ttl,
        cached: @cached,
      }
    end
    
    # Now we can get a new Request like this
    SummonerRequest.new(api_key, region, cache_store)
</code></pre>
<p>With all the supporting work done we could move into caching itself. All <em>FooRequest</em> classes do the dirty work with <em>Request#perform_request,</em> the method that gets the real data from the API with the help of <a href="https://github.com/jnunemaker/httparty" target="_blank">httparty</a>.</p>
<p>This is the pre-cache version of <em>Request#perform_request</em>:</p>
<pre><code>    # Calls the API via HTTParty and handles errors
    # @param url [String] the url to call
    # @return [String] raw response of the call
    def perform_request url
      response = self.class.get(url)
      raise NotFound.new("404 Not Found") if response.respond_to?(:code) &amp;&amp; response.not_found?
      raise InvalidAPIResponse.new(response["status"]["message"]) if response.is_a?(Hash) &amp;&amp; response["status"]

      response
    end
</code></pre>
<p>The cached version just adds two if blocks that handle the caching logic. Please don&rsquo;t bash my usage of if here :)</p>
<pre><code>    # Calls the API via HTTParty and handles errors
    # @param url [String] the url to call
    # @return [String] raw response of the call
    def perform_request url
      if cached? &amp;&amp; result = store.get(clean_url(url))
        return JSON.parse(result)
      end

      response = self.class.get(url)
      raise NotFound.new("404 Not Found") if response.respond_to?(:code) &amp;&amp; response.not_found?
      raise InvalidAPIResponse.new(response["status"]["message"]) if response.is_a?(Hash) &amp;&amp; response["status"]

      if cached?
        store.set clean_url(url), response.to_json
        store.expire url, ttl
      end

      response
    end
</code></pre>
<p>If you&rsquo;re smart, and I am pretty sure you are, you will have some questions: what is store? Why are you passing through JSON?</p>
<p>Store is just a method that returns the Redis instance we passed it during initialization. It&rsquo;s called store because in a future we might add support for more cache stores :)</p>
<p>JSON came in handy when we hit our second problem: httparty returns a hash, and we didn&rsquo;t want to destructure it into several Redis keys. The easiest way to proceed was serializing it using JSON.</p>
<p>WOAH! All this wall of text for less than ten new lines of code? You bet! The point of the article was that implementing a Redis cache on top of existing code is really easy :)</p>
<p>Last, but not least: <strong>TL;DR:</strong> Implementing a Redis cache is easy, do it, do it now!</p>
<p>PS. Check the whole (awesomely specced) code at <a href="https://github.com/mikamai/ruby-lol">https://github.com/mikamai/ruby-lol</a></p>