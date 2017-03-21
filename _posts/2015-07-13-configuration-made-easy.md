---
ID: 99
post_title: Configuration made easy
author: Gianluca
post_date: 2015-07-13 09:37:54
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/13/configuration-made-easy/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/123968449159/configuration-made-easy
tumblr_mikamayhem_id:
  - "123968449159"
---
Recently I was writing a simple program (more a script than a full-fledged program) to report the number of commits for each dev on all repos owned by a given organization.

Following the will to develop something extensible I set up a CLI tool (actually a gem) based on the famous gem <a href="http://whatisthor.com/">Thor</a>.

In order to have full power to configure my gem I choose to rely on another gem called <a href="https://github.com/beatrichartz/configurations"><b>configuration</b></a>.

<!--more-->

As stated inside its README it
<blockquote>provides a unified approach to do configurations using the <code>MyGem.configure do ... end</code> idiom…</blockquote>
Many implementation of this idiom have been extensively described through the years. Two examples are <a href="https://robots.thoughtbot.com/mygem-configure-block">Dan Crock’s MyGem.configure Block</a> and <a href="http://brandonhilkert.com/blog/ruby-gem-configuration-patterns/">Brandon Hilkert’s Ruby Gem Configuration Patterns</a>.

Likewise the latter two implementations, the <b>configuration</b> gem also grants the possibility to configure your gem by passing an hash. This is accomplished by using its <code><a href="http://www.rubydoc.info/gems/configurations/Configurations/Configuration#from_h-instance_method">from_h</a></code> method.

This is particularly useful when you need to create a configuration composed by a set of <i>a priori</i> unknown keys.

There is however something to take into consideration if you need to use this feature. Before version <code>2.1.0</code> the <code><a href="http://www.rubydoc.info/gems/configurations/Configurations/Configuration#from_h-instance_method">from_h</a></code> method doesn’t work with hashes composed by string keys. It only works with symbol keys.

I discovered this issue while trying to set up a dynamic configuration and thanks to the creator of the <b>configuration</b> gem the “problem” was successfully solved as stated by this <a href="https://github.com/beatrichartz/configurations/issues/10">issue</a>.

So now, happy configuration to everyone! :P

Cheers!