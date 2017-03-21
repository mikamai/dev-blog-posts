---
ID: 296
post_title: rails database management with dbmanager
author: Andrea Longhi
post_date: 2014-01-20 12:05:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/20/rails-database-management-with-dbmanager/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/73940037249/rails-database-management-with-dbmanager
tumblr_mikamayhem_id:
  - "73940037249"
---
<p>Ruby on Rails provides by default different application environments each one with its dedicated database and it is dead simple to add new ones whenever needed.</p>
<p>Sometimes it would be handy to import data from a different environment, for example loading your fresh new development database with the contents of the staging server or vice versa; you can dump the database and import it on your own by hand, or start using the <a href="https://github.com/spaghetticode/dbmanager">db_manager</a> gem.</p>
<p>Need to import the beta database to your local machine? run <code>rake db:import</code>, enter the environment numbers when asked and you&rsquo;re done.</p>
<p>See the gem readme for more features and complete usage recommendations!</p>