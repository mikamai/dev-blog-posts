---
ID: 317
post_title: >
  You should always specify the
  :inverse_of option on AR associations
author: Elia Schito
post_date: 2013-11-15 13:43:37
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/15/you-should-always-specify-the-inverseof-option/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/67055545219/you-should-always-specify-the-inverseof-option
tumblr_mikamayhem_id:
  - "67055545219"
---
<p>Directly from <a href="http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#label-Bi-directional+associations">ActiveRecord documentation on bi-directional associations</a>:</p>

<blockquote>
  <h2>Bi-directional associations</h2>
  
  <p>When you specify an association there is usually an association on the associated model that specifies the same relationship in reverse. For example, with the following models:</p>

<pre><code>class Dungeon &lt; ActiveRecord::Base
  has_many :traps
  has_one :evil_wizard
end

class Trap &lt; ActiveRecord::Base
  belongs_to :dungeon
end

class EvilWizard &lt; ActiveRecord::Base
  belongs_to :dungeon
end
</code></pre>
  
  <p>The traps association on Dungeon and the dungeon association on Trap are the inverse of each other and the inverse of the dungeon association on EvilWizard is the evil_wizard association on Dungeon (and vice-versa). By default, Active Record doesn’t know anything about these inverse relationships and so no object loading optimization is possible. For example:</p>

<pre><code>d = Dungeon.first
t = d.traps.first
d.level == t.dungeon.level # =&gt; true
d.level = 10
d.level == t.dungeon.level # =&gt; false
</code></pre>
  
  <p>The Dungeon instances <code>d</code> and <code>t.dungeon</code> in the above example refer to the same object data from the database, but are actually different in-memory copies of that data. **Specifying the <code>:inverse_of</code> option on associations lets you tell Active Record about inverse relationships and it will optimise object loading. For example, if we changed our model definitions to:</p>

<pre><code>class Dungeon &lt; ActiveRecord::Base
  has_many :traps, inverse_of: :dungeon
  has_one :evil_wizard, inverse_of: :dungeon
end

class Trap &lt; ActiveRecord::Base
  belongs_to :dungeon, inverse_of: :traps
end

class EvilWizard &lt; ActiveRecord::Base
  belongs_to :dungeon, inverse_of: :evil_wizard
end
</code></pre>
  
  <p>Then, from our code snippet above, <code>d</code> and <code>t.dungeon</code> are actually the same in-memory instance and our final <code>d.level == t.dungeon.level</code> will return true.</p>
  
  <p>There are limitations to <code>:inverse_of</code> support:</p>
  
  <ul><li>does not work with <code>:through</code> associations.</li>
  <li>does not work with <code>:polymorphic</code> associations.</li>
  <li>for <code>belongs_to</code> associations <code>has_many</code> inverse associations are ignored.</li>
  </ul></blockquote>