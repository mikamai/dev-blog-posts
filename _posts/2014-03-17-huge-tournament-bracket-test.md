---
ID: 271
post_title: Huge Tournament Bracket Test
author: Nicola Racco
post_date: 2014-03-17 10:53:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/17/huge-tournament-bracket-test/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/79859304359/huge-tournament-bracket-test
tumblr_mikamayhem_id:
  - "79859304359"
---
Recently we started working on a tournament web app. The request was to show all tournament matches in one single page, but working with tournaments with over **4000 teams (overfourthousands!)**
<!--more-->

![Over 9000](https://dev.mikamai.com/wp-content/uploads/2014/03/vegeta.gif) {.aligncenter}

It should work like google map, with pinch and zoom and live search, if you get it.

Well, we started figuring out how many tags we had to show on the page.

The bracket is a binary tree where the root node is the final match and the leaves are the bracket first turn matches (2048 leaves when we have 4096 teams).

Each bracket turn will have `match_of_previous_turn / 2` matches. With 4096 teams we have:

```ruby
teams         = 4096
turns         = Math.log2(teams) # =&gt; 12
total_matches = teams - 1 # =&gt; 4095 
```

The tournament web page will show at least 4095 divs. Each div will contain the teams names and icons, and one div for the result (5 additional nodes).

```ruby
match_nodes = 5
nodes = total_matches + (5 * match_nodes) # =&gt; 24570
```

Then we have the connectors between the matches:

![Tree Match](https://dev.mikamai.com/wp-content/uploads/2014/03/tree_match.jpg) {.aligncenter}

This can be seen as three tags used to compose the T. One is going from Match 1 to the center of the T, another from Match 2 to the center of the T, and the third from the center of the T to the next match. This three tags have to be applied to each match except the final one:

```ruby
match_connector_nodes = 3
nodes += match_connector_nodes * (total_matches - 1) # =&gt; 36852
```

The doubt is legit: with a web page completely styled and full of javascript interactions, will a browser be able to render it quickly enough ?

We decided not to risk and to rely on javascript the less we could. So the binary tree should have been created server side.

Me, [Giovanni](https://dev.mikamai.com/author/intinig/) and [Ivan](https://dev.mikamai.com/author/iusedtobeit/), two hours with a pen and a piece of paper, writing down formulas to generate a binary tree using absolute positioning. You can see the demo [here](http://huge-bracket-tree-test.herokuapp.com), the source code [here](https://github.com/mikamai/huge-bracket-tree-test), and the page generation code  with all the formulas [here](https://github.com/mikamai/huge-bracket-tree-test/blob/master/app/views/home/index.html.slim).

**Post Mortem**: We ended up creating an impressive demo, but even if we add pinch and zoom features it's not good from a user interaction point of view. So we decided to switch to a more _classic_ management (mini brackets with 128 teams, and a final bracket to handle all the finals between the 128 winners).