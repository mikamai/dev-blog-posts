---
ID: 17
post_title: Rails update tips and tricks
author: Emanuele Serafini
post_date: 2016-07-21 15:45:32
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/07/21/rails-update-tips-and-tricks/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/147751986369/rails-update-tips-and-tricks
tumblr_mikamayhem_id:
  - "147751986369"
---
<p>In this post, I&rsquo;m going to give you some advice on how to upgrade a Rails application. Rails 5 has been officially released recently, and I&rsquo;m pretty sure that many developers are now trying to update their application to the latest version of the framework. Upgrading complex applications requires more time and a good test coverage can be really helpful in order to quickly discover broken features.</p>

<p>Running your test suite should also reveal unsopported gems and code  incompatibilities with the upgraded version. Not every gem is ready to work with Rails 5 yet. If a gem does not work, you should check if there’s already a branch with Rails 5 support, or if the latest commit just works. Otherwise, you must resort to another solution or adapt the gem to Rails 5.</p>

<p>After creating a new branch for your app you should follow these simple two steps:</p>

<h2 id="toc_0">Upgrade using rails:update</h2>

<pre><code>⦿ ema:rails_app (rails_upgrade) $ rake rails:update
   identical  config/boot.rb
       exist  config
    conflict  config/routes.rb
Overwrite /Users/ema/dev/project/rails_app/config/routes.rb? (enter "h" for help) [Ynaqdh] a</code></pre>

<p>In this step you should allow <code>rake rails:update</code> to overwrite your files (writing <code>a</code> in the console). Many of your configuration files files will be changed. You’ll need to check each changed file and restore your custom configuration, if you did change something in the past. This operation can be really painful if you don&rsquo;t know what changes are included in the newer version. If you&rsquo;re not sure what to discard and what to keep you should look at the <a href="http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html">Rails upgrade guide</a>.</p>

<p>During this delicate operation I suggest you to use a GUI for git, my favorite one is <a href="https://rowanj.github.io/gitx/">GitX</a>, to analize each changed file and stage only the relavant lines.</p>

<h2 id="toc_1">Re-initialize your app</h2>

<p>I know it seems strange but if you want to include all the new features you should run <code>rails new your_project</code> in the parent folder of your app:</p>

<pre><code>⦿ ema:rails_app (rails_upgrade) $ cd ..
⦿ ema:project $ rails new rails_app -T -d postgresql
       exist
Overwrite /Users/ema/dev/project/rails_app/README.md? (enter "h" for help) [Ynaqdh] a</code></pre>

<p>Just like with the <code>rails:update</code> command, you should allow to overwrite all the files and then check again each one of them.
After running this command you&rsquo;ll notice a lot differences compared to the running of the Rails update command.</p>

<p>For example if you upgrade from Rails 4.x to Rails 5 you&rsquo;ll find the new class <code>ApplicationRecord</code> inside <code>app/models/application_record.rb</code>, you will find Action Cable code inside the files <code>app/assets/javascripts/cable.js</code> and <code>app/channels/application_cable/connection.rb</code> and some other files related to ActiveJob. All these changes are not included if you upgrade using only <code>rails:update</code>.</p>

<h3 id="toc_2">Gemfile</h3>

<p>At this point your <code>Gemfile</code> should be clean, all your gems are gone, only Rails 5 gems are there. Now, with patience, you should add all your gems dependecies and if necessary upgrade them to the latest version.</p>

<h3 id="toc_3">Other files</h3>

<p>Most of the modified files are part of the Rails configuration. Verify all these files and commit only the required changes.</p>

<h3 id="toc_4">Launch test suite</h3>

<p>After incorporating all the changes, you should run your test suite. It&rsquo;s normal that a major upgrade will break some tests. Fix them and then continue to test the application in the browser, in order to be sure that every part of the application still works as expected.</p>

<p>This workflow seems a bit complicated but remember that you&rsquo;re working on a different branch so you can take all time you need to test and fix all the problems.</p>