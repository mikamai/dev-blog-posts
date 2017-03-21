---
ID: 251
post_title: 'Writing custom commands for Drush: the Drupal swiss army knife.'
author: Massimo Ronca
post_date: 2014-05-14 12:16:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/14/writing-custom-commands-for-drush-the-drupal/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/85715381049/writing-custom-commands-for-drush-the-drupal
tumblr_mikamayhem_id:
  - "85715381049"
---
Recently I worked on a client project based on the Drupal platform.
The most important part of the job was automating a data import from a remote source,
but instead of writing a script to do the job, I created a command for <a href="https://github.com/drush-ops/drush">Drush</a>.
Quoting from Drush repository site
<blockquote>Drush is a command-line shell and scripting interface for Drupal, a veritable Swiss Army knife designed to make life easier for those who spend their working hours hacking away at the command prompt.</blockquote>
Drush can handle almost every aspect of a Drupal site, from the mundane <a href="http://www.drushcommands.com/drush-7x/cache">cache management</a> to
<a href="http://www.drushcommands.com/drush-6x/user">user management</a>, from <a href="http://www.drushcommands.com/drush-6x/make">packaging a Drupal install into a makefile</a> to
<a href="http://www.drushcommands.com/drush-6x/pm">project management</a> and much more, including a <a href="http://www.drushcommands.com/drush-6x/sql/sql-cli">CLI for running sql queries</a> an <a href="http://www.drushcommands.com/drush-6x/runserver/runserver">http server for development</a> and an <a href="http://www.drushcommands.com/drush-7x/core/core-rsync">rsync wrapper</a>.
Drush commands can also be executed on remote machines, provided Drush is installed, by specifing the server <a href="http://deeson-online.co.uk/labs/drupal-drush-aliases-and-how-use-them">alias</a> (e.g. <code>drush clear-cache @staging</code>).

There are different ways of creating Drush scripts:
<ul>
 	<li>prepending the script with the shebang <code>#!/usr/bin/env drush</code> or <code>#!/full/path/to/drush</code> and using
<a href="http://www.drushcommands.com/">Drush commands</a></li>
 	<li>using Drush php interpreter <code>#!/full/path/to/drush php-script</code> and using the <a href="http://api.drush.org/">Drush
PHP api</a></li>
 	<li>writing custom commands</li>
</ul>
This guide is about the last case.

<!--more-->

Drush commands are much like Rake or Grunt tasks, you give them a name (more like a namespace) and Drush figures out what function must be called.
To create a Drush command, follow these simple steps
<ul>
 	<li>create a <code>namespace.drush.inc</code> in one of the standard import path</li>
 	<li>implement the <code>namespace_drush_command</code> entry point function</li>
 	<li>implement the command functions. By conventions the command functions are called <code>drush_namespace_commandname</code></li>
</ul>
Drush search for commandfiles in the following locations:
<ul>
 	<li><code>/path/to/drush/commands</code> folder</li>
 	<li>system-wide drush commands folder, e.g. <code>/usr/share/drush/commands</code></li>
 	<li>.drush folder in <code>$HOME</code> folder.</li>
 	<li><code>sites/all/drush</code> in the current Drupal installation</li>
 	<li>all enabled modules folders in the current Drupal installation</li>
</ul>
<h2 id="implementing-the-command">Implementing the command</h2>
To implement a Drush command, the script must implement the drush_command hook.
This function must return a data structure containing all the informations that define your custom command.
As an example we will develop a command that rolls a dice and prints the result.
We’ll use <code>diceroller</code> as namespace and <code>roll-dice</code> as command name.
This is the implementation of the main hook function
<div></div>
<div><span class="text html php">function diceroller_drush_command() {</span></div>
<div><span class="text html php"> $items = array();</span></div>
<div></div>
<div><span class="text html php"> $items['roll-dice'] = array(</span></div>
<div><span class="text html php"> 'description' =&gt; "Roll a dice for your pleasure.",</span></div>
<div><span class="text html php"> 'arguments' =&gt; array(</span></div>
<div><span class="text html php"> 'faces' =&gt; 'How many faces the dice has? Default is 6, max is 100.',</span></div>
<div><span class="text html php"> ),</span></div>
<div><span class="text html php"> 'options' =&gt; array(</span></div>
<div><span class="text html php"> 'rolls' =&gt; 'How many times the dice is rolled, default is 1 max is 100',</span></div>
<div><span class="text html php"> ),</span></div>
<div><span class="text html php"> 'examples' =&gt; array(</span></div>
<div><span class="text html php"> 'drush drrd 6 --rolls=2' =&gt; 'Rolls a 6 faced dice 2 times',</span></div>
<div><span class="text html php"> ),</span></div>
<div><span class="text html php"> 'aliases' =&gt; array('drrd'),</span></div>
<div><span class="text html php"> 'bootstrap' =&gt; DRUSH_BOOTSTRAP_DRUSH,</span></div>
<div><span class="text html php"> // see <span class="markup underline link http hyperlink"><a href="http://drush.ws/docs/bootstrap.html">http://drush.ws/docs/bootstrap.html</a></span> for detailed informations</span></div>
<div><span class="text html php"> // about the bootstrap values</span></div>
<div><span class="text html php"> );</span></div>
<div></div>
<div><span class="text html php"> return $items;</span></div>
<div><span class="text html php">}</span></div>
The command is easily implementd this way
<div><span class="text html php">function drush_diceroller_roll_dice($faces=6) {</span></div>
<div><span class="text html php"> $rolls = 1;</span></div>
<div></div>
<div><span class="text html php"> if ($tmp = drush_get_option('rolls')) {</span></div>
<div><span class="text html php"> $rolls = $tmp;</span></div>
<div><span class="text html php"> } </span></div>
<div></div>
<div><span class="text html php"> drush_print(dt('Rolling a !faces faced dice !n time(s)', array(</span></div>
<div><span class="text html php"> '!faces' =&gt; $faces, </span></div>
<div><span class="text html php"> '!n' =&gt; $rolls</span></div>
<div><span class="text html php"> )));</span></div>
<div><span class="text html php"> // for n=0..$rolls</span></div>
<div><span class="text html php"> // roll the nth dice</span></div>
<div><span class="text html php"> // print the result</span></div>
<div><span class="text html php">}</span></div>
In this case we assume that the <code>--rolls</code> option contains a number, but we can guarantee that the function parameters are valid implementing the <code>validate</code> hook (there are others called just before and after the real command function).
<div></div>
<div><span class="text html php">function drush_diceroller_roll_dice_validate($faces=6) {</span></div>
<div></div>
<div><span class="text html php"> if($faces &lt;= 0) {</span></div>
<div><span class="text html php"> return drush_set_error('DICE_WITH_NO_FACES', dt('Cannot roll a dice with no faces!'));</span></div>
<div><span class="text html php"> }</span></div>
<div><span class="text html php"> if($faces &gt; 100) {</span></div>
<div><span class="text html php"> return drush_set_error('DICE_WITH_TOO_MANY_FACES', dt('Cannot roll a sphere!'));</span></div>
<div><span class="text html php"> }</span></div>
<div></div>
<div><span class="text html php"> $rolls = drush_get_option('rolls');</span></div>
<div><span class="text html php"> if(isset($rolls)) {</span></div>
<div><span class="text html php"> if(!is_numeric($rolls))</span></div>
<div><span class="text html php"> return drush_set_error('ROLLS_MUST_BE_INT', dt('rolls value must be a number!'));</span></div>
<div></div>
<div><span class="text html php"> if($rolls &lt;= 0)</span></div>
<div><span class="text html php"> return drush_set_error('NOT_ENOUGH_ROLLS', dt('What you're asking cannot be done!'));</span></div>
<div></div>
<div><span class="text html php"> if($rolls &gt; 100)</span></div>
<div><span class="text html php"> return drush_set_error('TOO_MANY_ROLLS', dt('I'm not your slave, roll it by yourself!'));</span></div>
<div><span class="text html php"> }</span></div>
<div></div>
<div><span class="text html php">}</span></div>
If we did our job diligently, running <code>drush help roll-dice</code> should give us this ouput
<div><span class="source shell">Roll a dice <span class="meta scope for-in-loop shell"><span class="keyword control shell">for</span> <span class="variable other loop shell">your</span> pleasure.</span></span></div>
<div></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell">Examples:</span></span></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell"> drush drrd 6 --rolls=2 Rolls a 6 faced dice 2 <span class="support function builtin shell">times</span></span></span></div>
<div></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell">Arguments:</span></span></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell"> faces How many faces the dice has<span class="keyword operator glob shell">?</span> Default is 6, max is 100.</span></span></div>
<div></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell">Options:</span></span></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell"> --rolls How many <span class="support function builtin shell">times</span> the dice is rolled, default is 1 max is 100</span></span></div>
<div></div>
<div><span class="source shell"><span class="meta scope for-in-loop shell">Aliases: drrd</span></span></div>
Consult the <a href="http://api.drush.org/">Drush api</a> for a complete list of hooks <a href="http://api.drush.org/api/drush/functions/6.x">functions</a> and <a href="http://api.drush.org/api/drush/constants/6.x">constants</a> or launch <code>drush topic docs-api</code> from the command line.
For a complete implementation of a command example, see <code>drush topic docs-examplecommand</code>.