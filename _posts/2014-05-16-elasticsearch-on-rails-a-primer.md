---
ID: 250
post_title: Elasticsearch on Rails, a primer
author: Andrea Longhi
post_date: 2014-05-16 09:35:48
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/16/elasticsearch-on-rails-a-primer/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/85901585484/elasticsearch-on-rails-a-primer
tumblr_mikamayhem_id:
  - "85901585484"
---
<p><a href="http://www.elasticsearch.org/overview/elasticsearch">Elasticsearch</a> is a full text search server powered by <a href="http://lucene.apache.org/core/">Lucene</a>, the Apache Foundation text-search engine used also by <a href="http://lucene.apache.org/solr/">Solr</a>.</p>
<p>Elasticsearch is basically a a distributed real-time document store where every field is indexed and searchable, was built to be distributed from the ground up, can be used for real-time analytics, has multi-tenacy, REST API and uses JSON for its query DSL, features that make it quite attractive to developers.</p>
<h3>Rails example application</h3>
<p>Let&rsquo;s install the Elasticsearch server on our machine: the <a href="http://www.elasticsearch.org/overview/elkdownloads/">project download page</a> has both code and instructions organized in an attractive way, you won&rsquo;t have any problem with it. If you&rsquo;re on OSX you can install via homebrew using the following command: <code>brew install elasticsearch</code></p>
<p>By default the server will run on port 9200, so after starting the service make sure everything is fine running <code>curl -X GET http://localhost:9200/</code> in the terminal.</p>
<p>If everything works as expected we can now create the app and add the gems:</p>
<pre><code>
rails new demoapp
cd demoapp
echo "gem 'elasticsearch-rails'" &gt;&gt; Gemfile
echo "gem 'elasticsearch-model'" &gt;&gt; Gemfile
bundle
</code></pre>
<p>Create the Article model and populate the db with some data that we will search against later:</p>
<pre><code>
rails g model article title:string body:text
rake db:migrate
rails c

hello_world  = Article.create title: 'Hello World', body: 'Lovely day!'
power_search = Article.create title: 'Power Search', body: 'Elasticsearch rules the world!'
</code></pre>
<p>(Those variable names will be used later in the examples, so keep them noted).</p>
<p>Let&rsquo;s include some modules into the Article class to make it searchable:</p>
<pre><code>
class Article &lt; ActiveRecord::Base
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
end
</code></pre>
<p>Add this line to the top of the app <code>Rakefile</code> to include the handy elasticsearch rake tasks:</p>
<pre><code>require 'elasticsearch/rails/tasks/import'</code></pre>
<p>The new tasks can be listed with <code>bundle exec rake -D elasticsearch</code>.</p>
<p>Remember that in order for old records to be found by elasticsearch we need to import them:</p>
<pre><code>bundle exec rake environment elasticsearch:import:all</code></pre>
<p>We can do the very same thing in the rails console:</p>
<pre><code>Article.import</code></pre>
<h3>The search DSL</h3>
<p>Let&rsquo;s play in the rails console making the simplest possible search:</p>
<pre><code>search = Article.search 'world'</code></pre>
<p>This will return an object with class <code>Elasticsearch::Model::Response::Response</code></p>
<p>If you just need to show data without going through the actual models in the database you can use the <code>results</code> method, which conveniently wraps the JSON returned by Elasticsearch in ruby objects:</p>
<pre><code>
search.results.map { |res| res.title }
# =&gt; ["Hello World", "Power Search"]
</code></pre>
<p>If you want to see the actual found records, then you just need to call <code>records</code> on that object:</p>
<pre><code>search.records</code></pre>
<p>The above query could be rewritten using Elasticsearch JSON DSL syntax like follows:</p>
<pre><code>Article.search('{"query": {"match": {"_all": "world"}}}').records</code></pre>
<p>Luckily the gem allows us to use both JSON or ruby hash syntax and of course we prefer the latter, don&rsquo;t we? All the following examples will use ruby hashes.</p>

<h4>Boosting</h4>
<p>The following example uses the <em>caret</em> to give a boost to the <code>title</code> field:</p>
<pre><code>
records = Article.search(query: {multi_match: {query: 'world', fields: ['title^10', 'body']}}).records
records.size # =&gt; 2
records.first == hello_world # =&gt; true
</code></pre>
<p>Let&rsquo;s change the records order boosting the <code>body</code> field instead:</p>
<pre><code>
records = Article.search(query: {multi_match: {query: 'world', fields: ['title', 'body^10']}}).records
records.first == power_search # =&gt; true
</code></pre>

<h4>Fuzzy search</h4>
<p>When it comes to fulltext search the ability to match mispelled words is mandatory. We can accomplish that easily using the <code>fuzziness</code> param:</p>
<pre><code>
records = Article.search(query: {match: {_all: {query: 'wold', fuzziness: 2}}}).records
records.first.title #  =&gt; "Hello World"
</code></pre>
<p><code>fuzziness</code> represents the maximum allowed <a href="http://en.wikipedia.org/wiki/Levenshtein_distance">Levenshtein distance</a>, it accepts an integer between 0 and 2 (where 2 means the fuzziest search possible) or the string &ldquo;AUTO&rdquo; which will generate an edit distance based on the charachers length of the terms in the query.</p>
<h4>Conclusions</h4>
<p>So far we&rsquo;ve just scratched the surface of Elasticsearch capabilities and complexity, there is so much more to learn: tokenizers, analyzers, indexes, facets, but that&rsquo;s something far beyond the scope of this short article. I suggest you to go deeper reading the Elasticsearch <a href="http://www.elasticsearch.org/guide/">documentation</a> and the elasticsearch-rails <a href="https://github.com/elasticsearch/elasticsearch-rails">gem readme</a>.</p>