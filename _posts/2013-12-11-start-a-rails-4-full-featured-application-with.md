---
ID: 304
post_title: >
  Start a Rails 4 full-featured
  application with uWSGI
author: Giovanni Cimmino
post_date: 2013-12-11 10:28:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/11/start-a-rails-4-full-featured-application-with/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69680190127/start-a-rails-4-full-featured-application-with
tumblr_mikamayhem_id:
  - "69680190127"
---
<div><span><a href="http://projects.unbit.it/uwsgi/">uWSGI</a> is a full stack application server that can act as a proxy service, process monitor and much more. In this post we will setup </span><a href="https://github.com/feedbin/feedbin">Feedbin</a><span>, a Rails 4 application that uses </span><a href="http://sidekiq.org/">Sidekiq</a><span> as background jobs engine, </span><a href="http://www.postgresql.org/">PostgreSQL</a><span> as rdbms and </span><a href="http://www.elasticsearch.org/">Elasticsearch</a><span> as full-text search engine.</span> Configuring uWSGI is quite easy, we just need to write a file!</div>
<h2>Server configuration</h2>
<p>Configuring a local virtual machine with <a href="https://www.virtualbox.org/">Virtualbox</a> and <a href="http://www.vagrantup.com/">Vagrant</a> is easy and out of the scope of this post, but it could help taking a look at the following links:</p>
<ul><li><a href="http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box">Ubuntu server 14.04 LTS image for virtual box</a></li>
<li><a href="https://help.ubuntu.com/community/PostgreSQL">How to install PostgreSQL</a></li>
<li><a href="http://codecuriosity.com/blog/2013/10/29/install-redis-on-ubuntu/">How to install Redis</a></li>
</ul><p>After you ensured ruby 2.0.0 is installed, you can install uWSGI via  rubygems:</p>
<pre><code>gem install uwsgi
</code></pre>
<p>Clone the Feedbin repo into your machine, I cloned the repo into ~/apps:</p>
<pre><code>git clone <a href="https://github.com/feedbin/feedbin.git">https://github.com/feedbin/feedbin.git</a>
</code></pre>
<p>and comment out unicorn, capistrano-unicorn, therubyracer and foreman in the Gemfile:</p>
<pre><code># gem 'capistrano-unicorn', github: 'sosedoff/capistrano-unicorn', ref: '52376ad', require: false
# gem 'unicorn'
# gem "therubyracer", require: 'v8'
# gem 'foreman'
</code></pre>
<p>Feedbin takes advantage of env variables, so we add <em><a href="https://github.com/bkeepers/dotenv">dotenv-rails</a></em> to the Gemfile, this way we don&rsquo;t have to export the env settings each time we run a command in console:</p>
<pre><code>gem 'dotenv-rails'
</code></pre>
<p><em>dotnev-rails</em> reads settings from the <em>.env</em> file:</p>
<pre><code>BUNDLE_GEMFILE=/home/vagrant/apps/feedbin/Gemfile
RAILS_ENV=production
MEMCACHED_HOSTS=127.0.0.1:11121
POSTGRES_USERNAME=vagrant
DATABASE_URL=postgres://vagrant:secret@127.0.0.1:5432/feedbin_production
SECRET_KEY_BASE=yoursecretkey</code></pre>
<p>And now, bundle:</p>
<pre><code>bundle install --without development test
</code></pre>
<p>Let&rsquo;s create a new database:</p>
<pre><code>createdb -T template0 feedbin_production
</code></pre>
<p>go ahead loading schema and running the migrations (we need to load the schema because at the moment there is an error in one of the old migrations preventing the rake task to be completed correctly)</p>
<pre><code>rake db:schema:load
rake db:migrate
</code></pre>
<p>Application setup is completed. Now we can configure uWSGI.</p>

<h2>uWSGI</h2>
<p>Create the uwsgi config file in config/uwsgi.ini:</p>
<pre><code>[uwsgi]<br /># this is the application user/uid to use in our uWSGI process.
uid = vagrant<br /># Setting master = true I'm telling uWSGI to enable the master process.<br /># This way uWSGI can act as an application instance monitor
master = true<br /># Number of application instances to spawn
processes = 2<br /># Full path of the unix socket that apache/nginx will use to speak to uWSGI
socket = /home/vagrant/apps/feedbin/tmp/uwsgi.sock<br /># Setting this modifier to 7 will enable the ruby/rack plugin
socket-modifier1 = 7<br /># Application root directory
chdir = /home/vagrant/apps/feedbin<br /># Rackup file location
rack = /home/vagrant/apps/feedbin/config.ru<br /># post-buffering and buffer-size are required by the Rack specification
post-buffering = 4096<br /></code><code>buffer-size = 25000<br /># load the bundle subsystem<br /></code><code>rbrequire = rubygems<br />rbrequire = bundler/setup<br /># disable logging<br />disable-logging = true<br /># uWSGI log file location<br />daemonize = /home/vagrant/apps/feedbin/log/uwsgi.log<br /># uWSGI pid file location<br />pidfile = /home/vagrant/apps/feedbin/tmp/uwsgi.pid<br /># Start sidekiq when you start the application.<br /># Using smart-attach-daemon uWSGI can be used to start external services,<br /># and monitor them (for example it automatically restarts the daemon when you restart uWSGI)<br />smart-attach-daemon = %(chdir)/tmp/pids/sidekiq.pid bundle exec sidekiq -P %(chdir)/tmp/pids/sidekiq.pid -e production </code></pre>
<p>As you can see we configured only one file to start both the application itself and Sidekiq.</p>
<p>Now we can start the application with the follow command:</p>
<pre><code>uwsgi ~/apps/feedbin/config/uwsgi.ini
</code></pre>
<p>Our Feedbin instance is now up and running, but in order to let it work properly we need to configure nginx/apache in front of it.  If you need to install nginx you can follow the instructions at this <a href="https://help.ubuntu.com/community/Nginx">Ubuntu community</a> link.</p>
<p>Let&rsquo;s now create the application configuration; below there is my nginx config file (we also need to create https <a href="http://www.akadia.com/services/ssh_test_certificate.html">self-signed certs</a> for Feedbin):</p>
<pre><code>server {
  listen 80;
  listen 443 default ssl;

  ssl_certificate /etc/nginx/certs/myfeedbin.crt;
  ssl_certificate_key /etc/nginx/certs/myfeedbin.key;


  location / {

    root /home/vagrant/apps/feedbin/public;
    include uwsgi_params;
    if (!-f $request_filename) {
      uwsgi_pass unix:///home/vagrant/apps/feedbin/tmp/uwsgi.sock;
    }
    uwsgi_param     UWSGI_SCHEME $scheme;
    uwsgi_param     SERVER_SOFTWARE    nginx/$nginx_version;
    uwsgi_param     HTTPS           on;
    uwsgi_modifier1 7;

    location ~ ^/assets/ {
      expires 1y;
      add_header Cache-Control public;

      add_header ETag "";
      break;
    }
  }

}
</code></pre>
<p>Restart nginx.</p>
<p><strong>DONE!</strong></p>
<p>Open your browser and enjoy: the application is up and running!</p>

<h2>Appendix: Other useful uWSGI settings</h2>
<p><strong><span>RVM</span></strong></p>
<p><span>If your app uses rvm you can add the following to the uwsgi.ini in order to load the rvm environment:</span></p>
<pre><code># load rvm<br />rvm-path = /home/vagrant/.rvm </code></pre>

<p><strong>Environment Variables</strong></p>
<p>We used dotenv to set environment vars inside a config file, but you may prefer to set these variables directly inside uWSGI. You can do that (for our example) by adding the following to uWSGI.ini</p>
<pre><code>env = BUNDLE_GEMFILE=/home/vagrant/apps/feedbin/Gemfile<br />env = RAILS_ENV=production <br />env = MEMCACHED_HOSTS=127.0.0.1:11121 <br />env = POSTGRES_USERNAME=vagrant <br />env = DATABASE_URL=postgres://vagrant:secret@127.0.0.1:5432/feedbin_production <br />env = SECRET_KEY_BASE=yoursecretkey<br /></code></pre>
<p>The pattern is <strong>env = &lt;name&gt;=&lt;value&gt;</strong></p>

<h2>Final considerations</h2>
<p>uWSGI is a powerful piece of software with a lot of interesting features like fast-routing, auto scaling, support for new plugins, alarms, mules, crontab, application caching and multiple applications management mode&hellip; oddly enough it&rsquo;s so much more famous in the python world than in the ruby community.</p>