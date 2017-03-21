---
ID: 18
post_title: Create a Round Robin Tournament in Ruby
author: Diomede Tripicchio
post_date: 2016-07-20 08:57:09
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/07/20/create-a-round-robin-tournament-in-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/147689292839/create-a-round-robin-tournament-in-ruby
tumblr_mikamayhem_id:
  - "147689292839"
---
<p>Today I want to share with you a funny side of a project I&rsquo;m working on.</p>
<p>In the past weeks I&rsquo;ve implemented a system that seeds the rounds of a <a href="https://en.wikipedia.org/wiki/Round-robin_tournament">Round Robin Tournament</a> aka all-play-all tournament.</p>
<!--more-->
<p>The interesting part is how we can seed the matches to let each team play against all the others. For example in a tournament of <strong>6 teams</strong> we have <strong>5 rounds</strong> with <strong>3 matches</strong> per round.<br />Let&rsquo;s see how we can schedule all the matches:</p>
<p>Teams: <code>A, B, C, D, E, F</code></p>
<p>Round 1:</p>
<pre><code>A    B    C
|    |    |
F    E    D
</code></pre>
<p>Round 2:</p>
<pre><code>A    F    B
|    |    |
E    D    C
</code></pre>
<p>&hellip;</p>
<p>Round 5:</p>
<pre><code>A    C    D
|    |    |
B    F    E
</code></pre>
<p>The important thing we can see is that the first team <strong>A</strong> keeps its position across all the rounds while all the other teams change their position in each round. This is the trick we&rsquo;re going to use in order to create the all-play-all matches.</p>
<p>Let&rsquo;s see how we can implement this behavior in Ruby.</p>
<p>First of all we need a class for the tournament.</p>
<pre><code class="language-ruby">class Tournament
  attr_reader :rounds

  def initialize teams
    @rounds = {}
    seed teams
  end

  private

  def seed teams
    # logic for seeding all the rounds
  end
end
</code></pre>
<p>Now we have to implement the seed method. We need an array of teams with an even number of participants, otherwise we have to add a <strong>nil</strong> element to the teams array, which will eventually deliver a <a href="https://en.wikipedia.org/wiki/Bye_(sports)">bye</a> match for each team.</p>
<pre><code class="language-ruby">def seed teams
  teams.push(nil) if teams.size.odd?
  # ...
end
</code></pre>
<p>At this point we have to create all the rounds. In a Round Robin Tournament the number of rounds equals the number of participants, less 1.</p>
<pre><code class="language-ruby">def seed teams
  teams.push(nil) if teams.size.odd?
  rounds = teams.size - 1
  rounds.times do |index|
    @rounds[index] = []
    # ...
  end
end
</code></pre>
<p>For each round we want to seed the matches. The number of matches for each round are equal to the half of the number of participants. To implement the behavior showed above we need to take the team in a certain position in the <em>teams</em> array, reverse it and take the opponent in the same position.</p>
<pre><code class="language-ruby">def seed teams
  teams.push(nil) if teams.size.odd?
  rounds = teams.size - 1
  matches_per_round = teams.size / 2
  rounds.times do |index|
    @rounds[index] = []
    matches_per_round.times do |match_index|
      @rounds[index] &lt;&lt; [teams[match_index], teams.reverse[match_index]]
    end
    # ...
  end
end
</code></pre>
<p>The last and most important part of this method is to cycle the teams array in order to create different matches in all rounds. We said that the cycle process does not concern the first element which have to preserve its position.</p>
<pre><code class="language-ruby">def seed teams
  teams.push(nil) if teams.size.odd?
  rounds = teams.size - 1
  matches_per_round = teams.size / 2
  rounds.times do |index|
    @rounds[index+1] = []
    matches_per_round.times do |match_index|
      @rounds[index+1] &lt;&lt; [teams[match_index], teams.reverse[match_index]]
    end
    teams = [teams[0]] + teams[1..-1].rotate(-1)
  end
end
</code></pre>
<p>Thanks to the <strong>rotate(-1)</strong> method applied to the whole array of participants except for the first element we achieve exactly what we want.
Creating a tournament with our example array results in:</p>
```ruby
t = Tournament.new [&quot;A&quot;, &quot;B&quot;, &quot;C&quot;, &quot;D&quot;, &quot;E&quot;, &quot;F&quot;]
# 5
t.rounds
# {
#   1=&gt;[[&quot;A&quot;, &quot;F&quot;], [&quot;B&quot;, &quot;E&quot;], [&quot;C&quot;, &quot;D&quot;]],
#   2=&gt;[[&quot;A&quot;, &quot;E&quot;], [&quot;F&quot;, &quot;D&quot;], [&quot;B&quot;, &quot;C&quot;]],
#   3=&gt;[[&quot;A&quot;, &quot;D&quot;], [&quot;E&quot;, &quot;C&quot;], [&quot;F&quot;, &quot;B&quot;]],
#   4=&gt;[[&quot;A&quot;, &quot;C&quot;], [&quot;D&quot;, &quot;B&quot;], [&quot;E&quot;, &quot;F&quot;]],
#   5=&gt;[[&quot;A&quot;, &quot;B&quot;], [&quot;C&quot;, &quot;F&quot;], [&quot;D&quot;, &quot;E&quot;]]
# }
```
<p>And this is the code for the complete class:</p>
<pre><code class="language-ruby">class Tournament
  attr_reader :rounds

  def initialize teams
    @rounds = {}
    seed teams
  end

  private

  def seed teams
    teams.push(nil) if teams.size.odd?
    rounds = teams.size - 1
    matches_per_round = teams.size / 2
    rounds.times do |index|
      @rounds[index+1] = []
      matches_per_round.times do |match_index|
        @rounds[index+1] &lt;&lt; [teams[match_index], teams.reverse[match_index]]
      end
      teams = [teams[0]] + teams[1..-1].rotate(-1)
    end
  end
end
</code></pre>
<p>I hope you enjoyed this post!<br />
See you soon!</p>