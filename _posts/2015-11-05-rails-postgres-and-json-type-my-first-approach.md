---
ID: 67
post_title: >
  Rails + Postgres and JSON type, my first
  approach
author: Emanuele Serafini
post_date: 2015-11-05 14:30:34
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/11/05/rails-postgres-and-json-type-my-first-approach/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/132601439084/rails-postgres-and-json-type-my-first-approach
tumblr_mikamayhem_id:
  - "132601439084"
---
<p>I know that Postgres JSON data type has been released more than a year ago, but I just recently had the opportunity to use this feature on a Rails project.</p>

<p>I&rsquo;ve been asked to develop an app that allow users to check into nearby event.</p>

<p>One of the requests was to track whether the user location was inaccurate or whether the user region differed from the event region. At the beginning I was not aware of exactly what information was required, so I decided to create a json column in my checkins table called <code>report</code>. Here&rsquo;s my checkins table structure:</p>

<pre><code>create_table "check_ins", force: :cascade do |t|
  t.integer  "user_id",                 null: false
  t.integer  "event_id",                null: false
  t.json     "report",     default: {}, null: false
  t.datetime "created_at",              null: false
  t.datetime "updated_at",              null: false
end

add_index "check_ins", ["event_id"], name: "index_check_ins_on_event_id", using: :btree
add_index "check_ins", ["user_id", "event_id"], name: "index_check_ins_on_user_id_and_event_id", using: :btree
add_index "check_ins", ["user_id"], name: "index_check_ins_on_user_id", using: :btree
</code></pre>

<p>The <code>report</code> column would contain a list of check-in status <code>{:status_type =&gt; 'Message for status type', :other_status_type =&gt; "Message for other status type"}</code> that are stored only when some constraints are met.</p>

<p>My decision to store report data as JSON turned out to be the right one when the client decided to include some new tracking information in the check-in record. With this dynamic attribute I was able to implement this new feature very quickly without problems.</p>

<p>JSON attributes type not only allow schemaless data structures but also can be queried with ease. For example:</p>

<pre><code>anna = User.create name: 'Anna'
paul = User.create name: 'Paul'
john = User.create name: 'John'
party = Event.create name: 'Big party', starts_at: 2.days.from_now, ends_at: 3.days.from_now

CheckIn.create user: anna, event: party, report: {with_soda: 'Participant brings soda'}
CheckIn.create user: paul, event: party, report: {without_mask: 'Participant is not masquerade', bring_friends: false}
CheckIn.create user: john, event: party, report: {without_mask: 'Participant is not masquerade', with_soda: 'Participant brings soda', bring_friends: true}

CheckIn.where("(report -&gt;&gt; 'with_soda')::text IS NOT NULL")

[#&lt;CheckIn id: 1, user_id: 1, event_id: 1, report: {"with_soda"=&gt;"Participant brings soda"}, created_at: "2015-11-05 13:29:04", updated_at: "2015-11-05 13:29:04"&gt;, #&lt;CheckIn id: 3, user_id: 3, event_id: 1, report: {"without_mask"=&gt;"Participant is not masquerade", "with_soda"=&gt;"Participant brings soda", "bring_friends"=&gt;true}, created_at: "2015-11-05 13:29:05", updated_at: "2015-11-05 13:29:05"&gt;]
</code></pre>

<p>Or you can search for a specific value of a key:</p>

<pre><code>CheckIn.where("report -&gt;&gt; 'bring_friends' = 'true'")

[#&lt;CheckIn id: 3, user_id: 3, event_id: 1, report: {"without_mask"=&gt;"Participant is not masquerade", "with_soda"=&gt;"Participant brings soda", "bring_friends"=&gt;true}, created_at: "2015-11-05 13:29:05", updated_at: "2015-11-05 13:29:05"&gt;]
</code></pre>

<p>It also possible to create a scope to filter JSON data:</p>

<pre><code>class User &lt; ActiveRecord::Base
  has_many :check_ins
  has_many :events, through: :check_ins

  scope :without_mask, -&gt; { joins(:check_ins).where("(check_ins.report -&gt;&gt; 'without_mask')::text IS NOT NULL") }
end
</code></pre>

<p>As far as I can tell, it has been a positive experience using JSON type. I&rsquo;ll definitely use this new feature again, Postgres with JSON type is now mature and offer a complete set of <a href="http://www.postgresql.org/docs/9.3/static/functions-json.html">functions and operators</a>.</p>