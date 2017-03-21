---
ID: 283
post_title: >
  Easy full-text search with PostgreSQL
  and Rails
author: Andrea Longhi
post_date: 2014-02-19 13:35:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/19/easy-full-text-search-with-postgresql-and-rails/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/77171462056/easy-full-text-search-with-postgresql-and-rails
tumblr_mikamayhem_id:
  - "77171462056"
---
<p>Solr, Sphinx, ElasticSearch are some of the big players when it comes to full-text search on the web: they&rsquo;re great products for sure, but you need to add yet another server to your stack if you plan to use one of them.</p>
<p>But if you&rsquo;re already using (or willing to use) <strong>PostgreSQL</strong> as your rails app RDBMS then you probably already have all you need: in fact among the many features of PostgreSQL there is full-text search capability as well.</p>
<p>Today I&rsquo;m going to use the gem <a href="https://github.com/Casecommons/pg_search">pg_search</a> to show how easy is to add a full-text search capability to a basic rails application.</p>
<p>First things first, let&rsquo;s create a new rails app that uses PostgreSQL:</p>
<pre><code>rails new search_example -d postgresql</code></pre>
<p>Remember to edit the database.yml file with the right username and password values.</p>
<p>We&rsquo;re now going to add some nifty modules to postgres that allow some matching based on the english dictionary and some fuzzy logic, this can be achieved by creating a migration with this body:</p>
<pre><code> rails g migration add_contrib_extensions </code></pre>
<pre><code>
  class AddContribExtensions &lt; ActiveRecord::Migration
    def up
      execute 'CREATE EXTENSION pg_trgm;'
      execute 'CREATE EXTENSION fuzzystrmatch;'
    end
  end
</code></pre>
<p>We&rsquo;re now ready to create a couple of models to be searched:</p>
<pre><code> rails g model article title:string body:text
rails g model gallery title:string </code></pre>
<p>We want to search both in the body and in the title fields of the tables. Add the gem to the Gemfile:</p>
<pre><code> gem 'pg_search' </code></pre>
<p>then of course run</p>
<pre><code> bundle </code></pre>
<p>Create the migration for the multi-table search:</p>
<pre><code> rails g pg_search:migration:multisearch </code></pre>
<p>and the migration for the dmetaphone support:</p>
<pre><code>
rails g pg_search:migration:dmetaphone
rake db:create &amp;&amp; db:migrate
</code></pre>
<p>At this point we&rsquo;re ready to configure the models:</p>
<pre><code>
class Article
  include PgSearch
  multisearchable against: [:title, :body]
end
</code></pre>
<pre><code>
class Gallery
  include PgSearch
  multisearchable against: :title
end
</code></pre>
<p>and we&rsquo;re almost done. Let&rsquo;s write an initializer that configures PgSearch to use the postgres modules we added:</p>
<pre><code> touch config/initializers/pg_search.rb </code></pre>
<p>and add this code:</p>
<pre><code>
PgSearch.multisearch_options = {
  using: {
    tsearch:    {dictionary: 'english'},
    trigram:    {threshold:  0.1},
    dmetaphone: {}
  }
}
</code></pre>
<p>For further explaination please check postgres documentation for tsearch, trigram and dmetaphone modules.</p>
<p>Add some seed data in seed.rb:</p>
<pre><code>
Gallery.create title: 'Lolcats'
Article.create title: 'Lolcatz are funny', body: "yes they're indeed"
</code></pre>
<p>Active record callbacks will transparently handle the indexing of your models. Ready for a test? Launch the rails console and type:</p>
<pre><code>
search = PgSearch.multisearch 'lolcats'
search.map { |s| s.searchable.title }.inspect
#=&gt; ["Lolcatz", "Locats are funny "]

search = PgSearch.multisearch 'lolcts'
search.map { |s| s.searchable.title }.inspect
#=&gt; ["Lolcatz", "Locats are funny "]
</code></pre>
<p>Both records were found even if the word &ldquo;locats&rdquo; is mispelled in one of the titles, thanks to the optional dictionary statement we passed to the gem configuration in the initializer file.</p>
<p>PostgreSQL full text search modules and the gem pg_search work so well together that we really did need to write just a little code to make it happen. </p>