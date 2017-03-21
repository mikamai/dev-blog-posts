---
ID: 218
post_title: 'WordPress and OpsWorks, &#8220;Pride and Prejudice&#8221; (Part 2)'
author: Gianluca
post_date: 2014-08-04 15:23:56
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/08/04/wordpress-and-opsworks-pride-and-prejudice-3/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/93782458919/wordpress-and-opsworks-pride-and-prejudice
tumblr_mikamayhem_id:
  - "93782458919"
---
Here we are for Part 2 :)

In my last post about WP and OpsWorks I tried to explain the general setup for the recipes that I’ve developed to automate the deploy of WP applications through OpsWorks. If you missed it <a href="http://dev.mikamai.com/post/89353822489/wordpress-and-opsworks-pride-and-prejudice" target="_blank">here</a>’s the link.

Today I’m gonna talk about a particular problem that I encountered while writing the recipe that currently handles the database importation.

<!--more-->

As anyone with a little bit of experience in WP knows, applications developed on top of it are tightly coupled with the database and so their deploy differs from the one related to applications developed on top of frameworks like Rails or Laravel.

To handle the over mentioned importation I decided to keep it simple and stupid (and dirty!) and put the dump right in the repo. In this way it is automatically available during the deploy after the git fetch performed on the instance by the Amazon infrastructure.

Given the possibility to have or not the dump wherewith seed the database, I encountered the need to check for its presence.

And here’s the first problem:
<pre><code> 
db_path = File.join(deploy[:deploy_to], 'current/dump.sql')

if File.file? db_path 

  # Do stuff like import the database 
  
  ... 
  
end
</code></pre>
If you check for the presence of the dump in this way you’ll end up with a broken deploy when you remove the dump after a proper importation.

This is due the fact that the recipes that get executed by the Amazon infrastructure are actually compiled and so the check will be true even after the first time you remove the dump from the repo.

After a bit of Google, StackOverflow and Chef docs I found that the check should have been wrapped inside a <code><a href="http://docs.opscode.com/resource_ruby_block.html" target="_blank">ruby_block</a></code> Chef resource.

Everything that is done inside this kind of resource is in fact evaluated dynamically right during the execution of the recipe where it is used. Not only what is written inside the block do end but also the proc passed to the <code>not_if</code> <a href="http://docs.opscode.com/resource_common.html#guards" target="_blank">guard</a>.

Here’s the proper check:
<pre><code>
db_path = File.join(deploy[:deploy_to], 'current/dump.sql')

ruby_block 'magic_dynamic_evaluation' do
  not_if { File.file? db_path }
  block do
  
    # Do stuff like import the database
    ...

  end
  notifies :run, 'resource[resource_name]', :immediately
end
</code>
</pre>
With regard to the last element inside the <code>ruby_block</code> (i.e. <code>notifies :run, 'resource[resource_name]', :immediately</code>) it really deserves a proper dissertation because it takes into consideration the notification system implemented by Chef. <a href="http://docs.opscode.com/resource_common.html#notifications" target="_blank">Here</a> you can find a brief doc about it.

Anyway what the last statement inside the <code>ruby_block</code> do is “simply” notify another resource immediately (i.e. <code>:immediately</code>) after and if (i.e. <code>not_if</code>) it is run (i.e. <code>:run</code>).

Next time I’ll try to explain another functionality that I’ve put inside the recipe that I’ve briefly introduced in this post so stay in touch! ;)

<strong>つづく</strong>