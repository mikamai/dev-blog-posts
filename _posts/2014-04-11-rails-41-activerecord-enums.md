---
ID: 260
post_title: Rails 4.1 ActiveRecord enums
author: Andrea Longhi
post_date: 2014-04-11 04:00:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/11/rails-41-activerecord-enums/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/82355998967/rails-41-activerecord-enums
tumblr_mikamayhem_id:
  - "82355998967"
---
<p>Rails 4.1 has recently been released and it came out with the usual bag of goodies, one of the most notable being <strong>ActiveRecord enums,</strong> a handy new feature that simplifies the creation and use of state attributes for your models.</p>
<p>Consider the following use case: our app needs users, and each of them has a state that can be <code>registered</code>, <code>active</code> or <code>blocked</code>.</p>
<p>How would have we addressed the task in the past? Well, probably by adding a string or integer field to the <code>users</code> table in order to hold the state value, and by writing a few scope methods to query the table. Nowadays it&rsquo;s much simpler: you just need to write a migration that adds the field to the table:</p>
<pre><code>
class AddStatus &lt; ActiveRecord::Migration
  def change
    add_column :users, :state, :integer
  end
end
</code></pre>
<p>and add the <strong>enum</strong> macro to the User class:</p>
<pre><code>
class User
  enum state: [:registered, :active, :blocked]
end
</code></pre>
<p>Let&rsquo;s see in details what we&rsquo;re getting. The user state is <code>nil</code> by default; we can query each state label to see if it matches the current user&rsquo;s state:</p>
<pre><code>
user = User.new
user.state
 # =&gt; nil

user.registered?
 # =&gt; false

user.state = :registered
user.registered?
 # =&gt; true
</code></pre>
<p>Want to update the state and save the record in one command? Easy enough:</p>
<pre><code>
user.registered!
user.persisted?
 # =&gt; true
user.registered?
 # =&gt; true
</code></pre>
<p>We also get handy scope methods for each state:</p>
<pre><code>
User.active
 # =&gt; #&lt;ActiveRecord::Relation []&gt;
User.registered
 # =&gt; #&lt;ActiveRecord::Relation [#&lt;User id: 7, status: 0...]&gt;
</code></pre>
<p>We can even create users with a specific state using the enum scopes:</p>
<pre><code>
User.registered.create
 # =&gt; #&lt;User id: 6, status: 1, ...&gt;
</code></pre>
<p>Can we inspect the actual state column value, before typecast? Of course we can, but <code>#state_before_type_cast</code> returns the enum label. So we&rsquo;re going to use the <code>[]</code> method:</p>
<pre><code>
user.state
 # =&gt; "registered"
user.state_before_type_cast
 # =&gt; "registered"
user[:state]
 # =&gt; 0
</code></pre>
<p>So, now we can fully understand how it works: ActiveRecord actually stores in the database the integer corresponding to the enum label position inside the array that we provided to the &lt;code&gt;enum&lt;/code&gt; macro.</p>
<p>Do you want a default value? Then add a default value to the state field:</p>
<pre><code>
class ChangeStatus &lt; ActiveRecord::Migration
  def change
    change_column :users, :status, :integer, default: 1
  end
end
</code></pre>
<p>From now on all instantiated users will have a default state value corresponding to the position we provided in the migration:</p>
<pre><code>
user = User.new
user.state
 # =&gt; "active"
</code></pre>
<p>Be aware there are a few reserved words that cannot be used as enum labels, most notably existing column names, existing class methods and a few more. If you happen to use one of them by mistake then the app will complain raising an error:</p>
<pre><code>
class User
  enum state: [:logger]
end
 # =&gt; ArgumentError: You tried to define an enum named "state" on the model "User", but this will generate a class method "logger", which is already defined by Active Record.</code></pre>
<p>If you are interested in the actual Rails implementation you can peep at the <a href="https://github.com/rails/rails/blob/master/activerecord/lib/active_record/enum.rb">code</a> and  <a href="https://github.com/rails/rails/blob/master/activerecord/test/cases/enum_test.rb">tests</a> on the github repository, of course. Have fun!</p>