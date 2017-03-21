---
ID: 124
post_title: >
  My misadventures while migrating a rails
  database from Sqlite3 to Mysql
author: Andrea Longhi
post_date: 2015-05-15 12:35:45
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/15/my-misadventures-while-migrating-a-rails-database/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/119022544069/my-misadventures-while-migrating-a-rails-database
tumblr_mikamayhem_id:
  - "119022544069"
---
<p>Last week I needed to migrate a database from sqlite to mysql. This is a legacy rails project that uses 2 databases. One of them  was completely managed by our customer (data, schema and server machine) on the Oracle platform. Recently we migrated this database from Oracle to sqlite, so at the moment the project is using two sqlite3 databases. We now needed to use mysql.</p>

<p>No schema was provided for the second db. My first option was to use the <a href="https://github.com/ricardochimal/taps">taps</a> gem for the migration, I had used this gem on heroku in the past to import remote databases to my local machine and vice versa, but while looking at it I discovered that the migration could be even easier using the <a href="https://github.com/jeremyevans/sequel">sequel</a> gem, which by the way taps relies on. So I created the empty mysql db and entered this command on the terminal:</p>

<pre><code>sequel -C sqlite://db/oracle_dump.sqlite3 mysql://root:@localhost/oracle_dump</code></pre>

<p>Unfortunately, the process failed.</p>
<p>I spent some time trying to figure out what went wrong with no luck. Eventually, while inspecting the db schema, I found out there were a few primary key column names that were empty strings (yes! &ldquo;&rdquo; was the name of the column) and this was for sure an issue when trying to replicate the schema on mysql.</p>
<p>After renaming the empty columns to &ldquo;id&rdquo; I was able to fully recreate the schema on mysql with the command above, but the following step (data import) failed miserably: only a single record was created. The error reported was:</p>
<pre><code>Error: Sequel::UniqueConstraintViolation: Mysql::Error: Duplicate entry '1' for key 'PRIMARY'</code></pre>
<p>How could this be possible? All record ids in the source table were unique, I knew this for sure, it was fine with sqlite and oracle! But wait, the first record had a <code>id</code> value of <code>0</code>! That was the reason for the failure: when mysql received the record data with id zero, it changed the id to &ldquo;1&rdquo; because by default zero is not allowed as primary key value. Later, when it was the turn of the actual record with id &ldquo;1&rdquo; to be inserted, mysql complained that the &ldquo;1&rdquo; key number was already taken. The solution was to temporarily update a mysql global variable to allow the insertion of zero values for primary keys: the variable named <code>sql_mode</code>.</p>
<p>I checked the current value of the variable:</p>
<pre><code>
mysql&gt; SHOW GLOBAL VARIABLES WHERE Variable_name = "sql_mode";
+---------------+--------------------------------------------+
| Variable_name | Value                                      |
+---------------+--------------------------------------------+
| sql_mode      | STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |
+---------------+--------------------------------------------+
1 row in set (0,00 sec)
</code></pre>
<p>And added the <code>NO_AUTO_VALUE_ON_ZERO</code> value to it:</p>
<pre><code>SET GLOBAL SQL_MODE = "NO_AUTO_VALUE_ON_ZERO,STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION";</code></pre>

<p>After this final fix the sequel import process worked perfectly. I later returned the global variable to its original value:</p>

<pre><code>SET GLOBAL SQL_MODE = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION";</code></pre>

<p>So, be warned: empty column names and zero value primary keys can make your database data transfer a bit though ;-)</p>