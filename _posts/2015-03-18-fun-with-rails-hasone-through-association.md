---
ID: 142
post_title: >
  Fun with Rails has_one through
  association
author: Andrea Longhi
post_date: 2015-03-18 11:14:20
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/03/18/fun-with-rails-hasone-through-association/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/113951227084/fun-with-rails-hasone-through-association
tumblr_mikamayhem_id:
  - "113951227084"
---
<p>In my Rails developer career I had never felt the need to use the has_one through: association. Lately while working on a cool project I found out how useful and convenient this kind of association can be.</p>

<p>Let&rsquo;s consider a simple use case. We have three ActiveRecord models: <code>Player</code>, <code>Team</code> and the join model <code>Membership</code>. A team has many players and a player can have many teams through its memberships. Of course a team has a captain, but this is something I will show later. Let&rsquo;s start from the basic associations and the migration to create the tables:</p>

<pre><code>
class Player &lt; ActiveRecord::Base
  has_many :memberships
  has_many :teams, through: :memberships
end

class Team &lt; ActiveRecord::Base
  has_many :memberships
  has_many :players, through: :memberships
end

class Membership &lt; ActiveRecord::Base
  belongs_to :team
  belongs_to :player
end

def change
  create_table :players do |t|
    t.string :name
  end

  create_table :teams do |t|
    t.string :name
  end

  create_table :memberships do |t|
    t.references :player, index: true
    t.references :team, index: true
    t.boolean :captain
  end
end
</code></pre>

<p>Alright, nothing really fancy here. When we want to associate a player to a particular team we don&rsquo;t need to explicitly create the team membership record because rails can handle the underlying db details:</p>

<pre><code>
Membership.count
# =&gt; 0
player = Player.create name: 'Racco'
team = Team.create name: 'Mikamai'
team.players &lt;&lt; player
Membership.count
# =&gt; 1
</code></pre>

<p>Let&rsquo;s add another player to the team:</p>

<pre><code>
captain = Player.create name: 'Intini'
team.players &lt;&lt; captain
</code></pre>

<p>That&rsquo;s cool indeed. We how have two players in our team. You can tell by the variable name that we will soon make a captain of the last one that we created. Let&rsquo;s complete the <code>Team</code> model in order to make the captain feature work:</p>

<pre><code>
class Team &lt; ActiveRecord::Base
  has_many :memberships
  has_many :players, through: :memberships

  has_one :captain_membership, -&gt; { where captain: true}, class_name: 'Membership'
  has_one :captain, through: :captain_membership, source: :player
end
</code></pre>

Let&rsquo;s put the new code to good use:

<pre><code>
team.captain = captain
</code></pre>

<p>And that&rsquo;s it! Rails will handle all the association details as before, but this time setting the membership captain boolean value to true as well.</p>

<p>Are we done? No, there&rsquo;s a problem. Let&rsquo;s see how many memberships we have now in the database:</p>

<pre><code>
 team.memberships.count
 # =&gt; 3
</code></pre>


<p>What about players?</p>
<pre><code>
team.players.count
# =&gt; 3
</code></pre>

<p>Ouch. We created only 2 players, so something went wrong.</p>

<pre><code>
team.players.map &amp;:name
# =&gt; ["Racco", "Intini", "Intini"]

team.memberships.map { |membership| membership.player.name }
# =&gt; ["Racco", "Intini", "Intini"]
</code></pre>

<p>When we created the <code>captain</code> relation a new membership record was created as well, this time with the <code>captain</code> attribute set to true. One way to fix the players relationship is to pick players only once, making them unique, and this is quite easy to accomplish, we just need to update the team players relationship:</p>

<pre><code>
class Team &lt; ActiveRecord::Base
  has_many :players, -&gt; { uniq }, through: :memberships
end
</code></pre>

<p>Let&rsquo;s try again with the new code:</p>

<pre><code>
team = Team.find_by_name 'mikamai'
team.players.count
# =&gt; 2
</code></pre>

<p>Success! Now, what happens if we change the captain? The existing captain membership record gets updated with a new player_id:</p>

<pre><code>
team.captain_membership 
# =&gt;Membership id: 7, player_id: 6, team_id: 1, owner: true ...&gt;

racco = Player.find_by_name 'racco'
team.captain = racco

team.captain_membership.reload
# =&gt;Membership id: 7, player_id: 7, team_id: 1, owner: true ...&gt; 
</code></pre>

<p>One last check then we&rsquo;re done: what happens if the captain decides to leave the team? Will both relevant membership records be destroyed?</p>

<pre><code>
captain = team.captain
team.players.delete captain

team.players.count
# =&gt; 1
team.memberships.count
# =&gt; 1
team.captain 
# =&gt; nil
</code></pre>

<p>So everything works smoothly, no need to manually clear out stale association records. Did I mention that I love ActiveRecord?</p>