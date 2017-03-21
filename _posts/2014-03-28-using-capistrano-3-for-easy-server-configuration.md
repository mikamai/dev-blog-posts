---
ID: 266
post_title: >
  Using capistrano 3 for easy server
  configuration
author: Emanuele Serafini
post_date: 2014-03-28 10:03:25
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/28/using-capistrano-3-for-easy-server-configuration/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/80963920814/using-capistrano-3-for-easy-server-configuration
tumblr_mikamayhem_id:
  - "80963920814"
---
<p>I recently had the need to configure multiple rails apps on a new staging machine. Deploying and configuring applications can be really hard when you don&rsquo;t have complete and working capistrano&rsquo;s recipes.</p>

<p>I figured out that the best solution is having recipes that allow you to configure both the correct deploy of the application and the tuning of the application server, web server, delayed jobs, monitor application.</p>

<p>The advantages of this solutions is the fully control of your configuration which is stored in your repository instead of being scattered around in filesystem, in that way you can also view history of the tuning.</p>

<p>All you need to do is to integrate your recipes with templates generating configuration files customized for each environment.</p>

<p>Here&rsquo;s an example of upstart configuration for puma web server using erb:</p>

<pre><code># lib/capistrano/templates/puma_upstart.erb
description "&lt;%= fetch(:application) %&gt; puma app"

start on filesystem or runlevel [2345]
stop on runlevel [!2345]

setuid &lt;%= fetch(:user) %&gt;
setgid &lt;%= fetch(:user) %&gt;

script
exec /bin/bash &lt;&lt;'EOT'
    source /home/&lt;%= fetch(:user) %&gt;/.rvm/scripts/rvm
    cd &lt;%= current_path %&gt;
    bundle exec puma -e &lt;%= fetch(:rails_env) %&gt; --config &lt;%= fetch(:puma_config)%&gt;
EOT
end script
</code></pre>

<p>Associated recipe to allow the template to run is:</p>

<pre><code># lib/capistrano/tasks/puma.cap
namespace :puma do

  desc "Setup Puma configuration and upstart script"
  task :setup do
    on roles(:app) do
      execute :mkdir, "-p #{shared_path}/config"

      # Upstart script for puma            
      template "puma_upstart.erb", "/tmp/puma_upstart", true
      execute :sudo ,:chmod, "+x /tmp/puma_upstart"
      execute :sudo, :mv, "/tmp/puma_upstart /etc/init/#{fetch(:app_env)}.conf"
    end
  end

end

def template(from, to, as_root = false)
  template_path = File.expand_path("../../templates/#{from}", __FILE__)
  template = ERB.new(File.new(template_path).read).result(binding)
  upload! StringIO.new(template), to

  execute :sudo, :chmod, "644 #{to}"
  execute :sudo, :chown, "root:root #{to}" if as_root == true
end
</code></pre>

<p>Your recipes can be organized in capistrano&rsquo;s folder this way:</p>

<p><img src="http://68.media.tumblr.com/c1dc9275032a0b244dafd69d8f4a635d/tumblr_inline_n339fjTVhL1qfdn20.png" alt="" /></p>

<p>You can find a complete working example <a href="https://github.com/emaserafini/sample_app">here</a>. To make it work you&rsquo;ll need these prerequisites:</p>

<ul><li>Server with Ubuntu 12.04+</li>
<li><a href="https://www.digitalocean.com/community/articles/initial-server-setup-with-ubuntu-12-04">Basic server configuration</a></li>
<li><a href="https://www.digitalocean.com/community/articles/how-to-install-ruby-on-rails-on-ubuntu-12-04-lts-precise-pangolin-with-rvm">Ruby on Rails with RVM</a> (ruby 2.1.1)</li>
<li><a href="https://www.digitalocean.com/community/articles/how-to-install-the-latest-version-of-nginx-on-ubuntu-12-10">Latest version on Nginx</a></li>
<li>Monit</li>
</ul><p>Clone my repository and change <code>config/deploy/staging.rb</code> with your server parameter (remember that deployer user should be a sudoer). If everything is configured correclty you only have to create <code>/var/apps</code> on your server and set deploy user as owner, you&rsquo;ll therefore be able  to run <code>cap staging deploy</code> for your first deploy.</p>

<p>Run <code>cap -vT</code> to check the list of all possible tasks ;)</p>