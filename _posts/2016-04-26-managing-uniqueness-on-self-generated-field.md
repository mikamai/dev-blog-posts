---
ID: 36
post_title: >
  Managing uniqueness on self generated
  field
author: Emanuele Serafini
post_date: 2016-04-26 12:39:34
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/04/26/managing-uniqueness-on-self-generated-field/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/143426712989/managing-uniqueness-on-self-generated-field
tumblr_mikamayhem_id:
  - "143426712989"
---
<p>Recently I ran into a common problem for a Rails developer: how to manage uniqueness on a self generated field.
The problem was pretty simple, in my Rails app I&rsquo;ve got a resource that contains two different attributes generated during resource creation. These attributes <code>uuid, access_token</code> are provided to the user in order to interact with an API. As tokens I&rsquo;ve chosen to use UUID standard format.
<code>SecureRandom.uuid</code>, integrated with Ruby, aims to generate UUID but does not guarantee complete uniqueness so my goal was to avoid uniqueness collision at resource creation.</p>

<p>Here&rsquo;s resource (Feed) migration:</p>

<pre><code>class CreateFeeds &lt; ActiveRecord::Migration
  def change
    create_table :feeds do |t|
      t.uuid :uuid, null: false
      t.uuid :access_token, null: false
      t.string :name, null: false
      t.string :time_zone, default: 'UTC', null: false

      t.timestamps null: false
    end

    add_index :feeds, :uuid, unique: true
    add_index :feeds, :access_token, unique: true
  end
end
</code></pre>

<p>My first approach to solve the problem was to manage uniqueness using Rails validations, loops and hooks.</p>

<pre><code>class Feed &lt; ActiveRecord::Base
  validates :uuid, presence: true
  validates :access_token, presence: true

  before_validation :generate_uuid, unless: :uuid?
  before_validation :generate_access_token, unless: :access_token?


  private

  def generate_uuid
    begin
      self.uuid = SecureRandom.uuid
    end while self.class.exists?(uuid: uuid)
  end

  def generate_access_token
    begin
      self.access_token = SecureRandom.uuid
    end while self.class.exists?(access_token: access_token)
  end
end
</code></pre>

<p>As you can see in the log below, this solution brings the disadvantage of multple query on the database to check the presence of the generated UUIDs.</p>

<pre><code>2.3.0 :001 &gt; Feed.create(name: 'Example')

(0.1ms)  BEGIN
Feed Exists (14.7ms)  SELECT  1 AS one FROM "feeds" WHERE "feeds"."uuid" = $1 LIMIT 1  [["uuid", "1be12f2a-23f0-49f9-bac8-456c09fcede6"]]
Feed Exists (5.7ms)  SELECT  1 AS one FROM "feeds" WHERE "feeds"."access_token" = $1 LIMIT 1  [["access_token", "9b773024-741f-4904-8718-a6b68eb8b389"]]
SQL (3.6ms)  INSERT INTO "feeds" ("name", "uuid", "access_token", "time_zone", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5, $6) RETURNING "id"  [["name", "Example"], ["uuid", "1be12f2a-23f0-49f9-bac8-456c09fcede6"], ["access_token", "9b773024-741f-4904-8718-a6b68eb8b389"], ["time_zone", "UTC"], ["created_at", "2016-04-25 09:57:34.845915"], ["updated_at", "2016-04-25 09:57:34.845915"]]
(135.0ms)  COMMIT
</code></pre>

<p>After several attepts I&rsquo;ve found out that the best solution was to let broke resource creation and manage <code>ActiveRecord::RecordNotUnique</code>.</p>

<pre><code>class Feed &lt; ActiveRecord::Base
  UNIQUE_FIELDS = %w[uuid access_token]

  before_save :assign_uuid, :assign_access_token
  around_save :save_managing_uniqueness

  private

  def save_managing_uniqueness
    self.class.connection.execute('SAVEPOINT before_record_create;')
    yield
    self.class.connection.execute('RELEASE SAVEPOINT before_record_create;')
  rescue ActiveRecord::RecordNotUnique =&gt; error
    @attempts = @attempts.to_i + 1
    if @attempts &lt; 3
      duplicated_field = error.message[/Key ((w+))/, 1]
      send("assign_#{duplicated_field}", true) if UNIQUE_FIELDS.include?(duplicated_field)
      self.class.connection.execute('ROLLBACK TO SAVEPOINT before_record_create;')
      retry
    else
      raise error, 'Retries exhausted'
    end
  end

  def assign_uuid(force = false)
    self.uuid = SecureRandom.uuid if uuid.nil? || force
  end

  def assign_access_token(force = false)
    self.access_token = SecureRandom.uuid if access_token.nil? || force
  end
end
</code></pre>

<p>Be aware to change <code>duplicated_field = error.message[/Key ((w+))/, 1]</code> if you&rsquo;re not working with Postgres.</p>

<p>I&rsquo;m not fully convinced of using a regex to find what attribute raises duplication error, feel free to improve this part. The idea was to give a working example of rescue in a Rails transaction.</p>