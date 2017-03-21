---
ID: 57
post_title: >
  PostgreSQL Transaction and Rails
  callbacks
author: Diomede Tripicchio
post_date: 2016-01-19 14:42:36
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/01/19/postgresql-transaction-and-rails-callbacks/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/137621422759/postgresql-transaction-and-rails-callbacks
tumblr_mikamayhem_id:
  - "137621422759"
---
<p>Hi all, in theese days I am working on a new little feature on a Rails project. I have to generate a random and unique token for every new record of a model.</p>

<p>Unfortunately I have encountered a strange error <strong>ActiveRecord::StatetementInvalid</strong>. Let’s see some code and understand why this happens.</p>
<!--more-->

<p>I have created a simple project (<a href="http://github.com/oeN/after_create_callback">Git Repo</a>) with a single model, User, that has <em>email</em> and <em>token</em> as columns. I want to generate automatically a unique random token for each user, so in my db migration I have added an index on the token column:</p>

<pre><code>...
add_index :users, :token, unique: true
...
</code></pre>

<p>And in my model I have added an <strong>after_create</strong> callback:</p>

<pre><code>class User &lt; ActiveRecord::Base
  after_create :generate_token

  private

  def generate_token
    update_column :token, SecureRandom.hex(8)
  end
end
</code></pre>

<p>This works great and generates a random token, but at the moment there is no control on its uniqueness; Rails simply raises an <strong>ActiveRecord::RecordNotUnique</strong> error if the token is not unique.</p>

<p>Well, I can rescue this error and retry to generate a new token.</p>

<pre><code>...
  def generate_token
    update_column :token, SecureRandom.hex(8)
  rescue ActiveRecord::RecordNotUnique
    retry
  end
...
</code></pre>

<p>Now I have a model that retries generating a new token until it is unique, Great!</p>

<p>Oh yeah, works great; until you switch from the little cute SQLite to the super powered PostgreSQL!</p>

<p>With PostgreSQL this code ends in another error <strong>ActiveRecord::StatetementInvalid</strong> raised after a PostgreSQL error <strong>PG:InFailedSqlTransaction</strong>.</p>

<p>With PostgreSQL I can’t run a new SQL query if a previous query, in the same transaction, fails.</p>

<p>So if I want to preserve the behavior of my model I have to use a different callback.</p>

<p>The problem is that I am making an invalid <strong>update_column</strong> during a transaction, to solve this I can use the <em>after_commit</em> callback. When the record is saved I regenerate the token until it is unique, and this happens out of the previous transaction.</p>

<pre><code>class User &lt; ActiveRecord::Base
  after_commit :generate_token, on: :create

  private

  def generate_token
    update_column :token, SecureRandom.hex(8)
  rescue ActiveRecord::RecordNotUnique
    retry
  end
end
</code></pre>

<p>The only one cons of this solution are the tests, to make it pass I need to use the <a href="https://github.com/DatabaseCleaner/database_cleaner">database_cleaner</a> gem with <strong>:truncation</strong> as strategy. Otherwise in tests Rails does not fire the <em>after_commit</em> trigger.</p>

<pre><code>RSpec.configure do |config|

  config.before(:suite) do
    DatabaseCleaner.strategy = :truncation
    DatabaseCleaner.clean_with(:truncation)
  end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end

end
</code></pre>

<p>There is another couple of different solutions to archive this result, maybe a trigger at the database level or something else, but for now this one works.</p>

<p>To avoid an almost impossible, but possible, infinite loop I’ve added a retry limit, that you can find in the <a href="http://github.com/oeN/after_create_callback">Repo</a>.</p>

<p>You can find also an another branch <em>sqlite</em> with the SQLite version of the code.</p>

<p>I hope this article has been helpful, bye!</p>