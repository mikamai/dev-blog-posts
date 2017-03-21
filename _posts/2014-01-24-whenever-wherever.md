---
ID: 294
post_title: Whenever, wherever
author: Emanuele Serafini
post_date: 2014-01-24 13:40:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/24/whenever-wherever/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/74380955196/whenever-wherever
tumblr_mikamayhem_id:
  - "74380955196"
---
<p>Cron jobs are really useful for recurring background jobs ( mailing, indexing, rake task etc.). In Ruby on Rails world there are plenty of options for background proccessing but most of them are not that simple and easy to being with. However <a href="https://github.com/javan/whenever/">Whenever</a> from <a href="https://github.com/javan">Javan Makhmali</a> is pretty simple and easy to use DSL.</p>
<p>Whenever helps you keep your cron jobs with your code so that there is no separation of logic. Since Whenever is a wrapper for cron, however, it&rsquo;s really focused on on UNIX and UNIX-like machines.</p>
<p>Here an example of Whenever configuration for multiple environments:</p>
<pre><code>set :output, { :error =&gt; 'log/cron.error.log', :standard =&gt; 'log/cron.log' }

job_type :rake,    "cd :path &amp;&amp; RAILS_ENV=:environment bundle exec rake :task --silent :output"
job_type :script,  "cd :path &amp;&amp; RAILS_ENV=:environment bundle exec script/:task"

case @environment
when 'production'

  every 12.hours do
    rake 'load_prices:run'
  end

  every '15,45 * * * *' do
    rake 'load_availabilities:run'
  end

  every :hour do
    script 'move_transaction'
  end

when 'alpha'

  every :day, :at =&gt; '6:30 am' { rake 'load_prices:run' }

  every :day, :at =&gt; '6:00 am'  { rake 'load_availabilities:run' }

  every 8.hours { script 'copy_production_files' }

when '...'
  ...
  ...
end
</code></pre>
<p>To test and see the result of Whenever, execute <code>whenever --set environment=production</code> in terminal.</p>
<pre><code>0 * * * * /bin/bash -l -c 'cd /app/path &amp;&amp; RAILS_ENV=production bundle exec script/move_transaction'

0 0,12 * * * /bin/bash -l -c 'cd /app/path &amp;&amp; RAILS_ENV=production bundle exec rake load_prices:run --silent &gt;&gt; log/cron.log 2&gt;&gt; log/cron.error.log'

15,45 * * * * /bin/bash -l -c 'cd /app/path &amp;&amp; RAILS_ENV=production bundle exec rake load_availabilities:run --silent &gt;&gt; log/cron.log 2&gt;&gt; log/cron.error.log'
</code></pre>