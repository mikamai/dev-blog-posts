---
ID: 310
post_title: Add a tree to your rails application
author: Emanuel Carnevale
post_date: 2013-12-05 11:25:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/05/add-a-tree-to-your-rails-application/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69067094129/add-a-tree-to-your-rails-application
tumblr_mikamayhem_id:
  - "69067094129"
---
<p>I was looking for a tree structure for a tagging application and I decided to check out wether a solution different from <code>acts_as_tree</code> was available.</p>

<p>I stumbled upon <a href="https://github.com/mceachen/closure_tree">Closure_tree</a> and I was sold: it’s really fast and supports lots of customisations and features.</p>

<p>The installation is straightforward:</p>

<ol><li>add the gem (<code>closure_tree</code>)</li>
<li>add a <code>parent_id</code> column to the model of yours you want to behave like a tree</li>
<li>create a migration to generate a TagHierarchy table to store the tree structure.</li>
</ol><p>For all the nitty-gritty details, check out the <a href="https://github.com/mceachen/closure_tree#installation">installation doc</a>.</p>

<p>The basics commands to create a node and its relatives are what you expect from ActiveRecord relationships:</p>

<p><code>parent = Tag.create(:name =&gt; 'Parent')</code>
and a child:</p>

<p><code>child = parent.children.create(:name =&gt; 'Child')</code></p>

<p>But what really interested me, was the ability to create a hierarchy quickly and with only one call:</p>

<p><code>child = Tag.find_or_create_by_path(["grandparent", "parent", "child"])</code></p>

<p>Every tag of the chain will be created <em>only if not already present.</em></p>

<p>And as a bonus, it supports <a href="http://www.graphviz.org/">Graphviz</a> for rendering the trees.</p>

<p>The only feature it doesn’t have, is the support for multiple parents: it’d have been interesting to have it, but the author says pull requests are welcomed, so if the need arises, I may contribute to it. ;)</p>

<p>ps.</p>

<p>As for the speed, I didn’t need huge performances for my pet project, so it wasn’t a requirement, but it’s nice to know that it requires only one SELECT to get almost all the informations:</p>

<ul><li>Fetch your whole ancestor lineage in 1 SELECT.</li>
<li>Grab all your descendants in 1 SELECT.</li>
<li>Get all your siblings in 1 SELECT.</li>
<li>Find a node by ancestry path in 1 SELECT.</li>
</ul>