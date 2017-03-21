---
ID: 139
post_title: A containerized approach to Rails apps
author: Giovanni Intini
post_date: 2015-03-30 15:14:38
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/03/30/a-containerized-approach-to-rails-apps/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/115033718734/a-containerized-approach-to-rails-apps
tumblr_mikamayhem_id:
  - "115033718734"
---
<p>In the <a href="http://dev.mikamai.com/post/110066104864/zero-downtime-deployments-with-docker-on-opsworks">first article</a> of this series we talked about how to deploy Opsworks. In the <a href="http://dev.mikamai.com/post/113340319939/docker-and-opsworks-configuration-basics">second one</a> we talked about how to configure the deployment. In this third installment we’re going to look at a sample <a href="http://rubyonrails.org">Rails</a> Application, and how to Dockerize it in its composing parts.</p>

<p>We’re going to look at several Dockerfiles, and we’re going to discuss the hows and whys of certain choices. So, without further ado, let’s begin. Let’s say we have a normal real world Ruby on Rails application. Most people nowadays serve RoR application using a frontend server (usually <a href="http://nginx.org">nginx</a>, but <a href="http://projects.apache.org/projects/http_server.html">Apache</a> is still strong), an application server (<a href="http://puma.io">Puma</a>, <a href="http://unicorn.bogomips.org">Unicorn</a>, <br /><a href="https://www.phusionpassenger.com">Passenger</a>, …)  and might use a SQL Database (your choice of <a href="http://mysql.com">MySQL</a> or <a href="http://www.postgresql.org">Postgres</a>) and maybe a key-value store, like <a href="http://redis.io">Redis</a>.</p>

<p>Many applications make good use of an asynchronous queue, like <a href="http://sidekiq.org">Sidekiq</a>. That’s a lot of components, isn’t it? :)</p>

<p>Today we’re not going to look at containers that run MySQL, Posgtres or Redis. We’re going to focus on the app container, that will our app server of choice, and how this app container talks to its serving brothers, nginx and <a href="http://www.haproxy.org">haproxy</a>.</p>

<p>The first think we do is creating an empty <code>Dockerfile</code> in your app source code repo. As you should know by now, <code>Dockerfile</code> will tell docker how to build our container.</p>

<p>Let’s start to fill it with instructions:</p>



<pre class="prettyprint"><code class="hljs tex">FROM ubuntu:14.04 <span class="hljs-special">#</span> my favorite way of starting containers
MAINTAINER Giovanni Intini &lt;giovanni@mikamai.com&gt; <span class="hljs-special">#</span> yep that's me folks

ENV REFRESHED_AT 2015-01-28 <span class="hljs-special">#</span> this is a trick I read somewhere
                            <span class="hljs-special">#</span> useful when you want to retrigger a build

<span class="hljs-special">#</span> we install all prerequisites for ruby and our app. Your app might need
<span class="hljs-special">#</span> less or more packages
RUN apt-get -yqq update
RUN apt-get install -yqq autoconf <span class="hljs-command">
</span>                         build-essential <span class="hljs-command">
</span>                         libreadline-dev <span class="hljs-command">
</span>                         libpq-dev <span class="hljs-command">
</span>                         libssl-dev <span class="hljs-command">
</span>                         libxml2-dev <span class="hljs-command">
</span>                         libyaml-dev <span class="hljs-command">
</span>                         libffi-dev <span class="hljs-command">
</span>                         zlib1g-dev <span class="hljs-command">
</span>                         git-core <span class="hljs-command">
</span>                         curl <span class="hljs-command">
</span>                         node <span class="hljs-command">
</span>                         libmagickcore-dev <span class="hljs-command">
</span>                         libmagickwand-dev <span class="hljs-command">
</span>                         libcurl4-openssl-dev <span class="hljs-command">
</span>                         imagemagick <span class="hljs-command">
</span>                         bison <span class="hljs-command">
</span>                         ruby

<span class="hljs-special">#</span> here we start installing ruby from sources. If you're curious about why we
<span class="hljs-special">#</span> also installed ruby from apt-get, the short story is that we need it, but
<span class="hljs-special">#</span> we're gonna remove it later in this file :)

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.1

RUN mkdir -p /usr/src/ruby

RUN curl -SL "http://cache.ruby-lang.org/pub/ruby/<span class="hljs-formula">$RUBY_MAJOR/ruby-$</span>RUBY_VERSION.tar.bz2" <span class="hljs-command">
</span>        | tar -xjC /usr/src/ruby --strip-components=1

RUN cd /usr/src/ruby <span class="hljs-special">&amp;</span><span class="hljs-special">&amp;</span> autoconf <span class="hljs-special">&amp;</span><span class="hljs-special">&amp;</span> ./configure --disable-install-doc <span class="hljs-special">&amp;</span><span class="hljs-special">&amp;</span> make -j"<span class="hljs-formula">$(nproc)"

RUN apt-get purge -y --auto-remove bison <span class="hljs-command">
</span>                                   ruby

RUN cd /usr/src/ruby <span class="hljs-special">&amp;</span><span class="hljs-special">&amp;</span> make install <span class="hljs-special">&amp;</span><span class="hljs-special">&amp;</span> rm -rf /usr/src/ruby

RUN rm -rf /var/lib/apt/lists/*

<span class="hljs-special">#</span> let's stop here for now</span></code></pre>

<p>Everything up to here is simple. When possible we chain statements so we have fewer images built by docker as it layers each step on top of each other.</p>

<p>Now the interesting part. We want to be able to cache gems in between image builds, and we also want to be able to override them with our local gems when developing locally using this image. We do it in two steps: first we configure bundler and add only Gemfile and Gemfile.lock to the image, then we add the rest of the application.</p>



<pre class="prettyprint"><code class="hljs brainfuck"><span class="hljs-comment">RUN</span> <span class="hljs-comment">echo</span> <span class="hljs-comment">'gem:</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">no</span><span class="hljs-literal">-</span><span class="hljs-comment">rdoc</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">no</span><span class="hljs-literal">-</span><span class="hljs-comment">ri'</span> &gt;&gt; <span class="hljs-comment">$HOME/</span><span class="hljs-string">.</span><span class="hljs-comment">gemrc</span>
<span class="hljs-comment">RUN</span> <span class="hljs-comment">gem</span> <span class="hljs-comment">install</span> <span class="hljs-comment">bundler</span>
<span class="hljs-comment">RUN</span> <span class="hljs-comment">bundle</span> <span class="hljs-comment">config</span> <span class="hljs-comment">path</span> <span class="hljs-comment">/remote_gems</span>

<span class="hljs-comment">COPY</span> <span class="hljs-comment">Gemfile</span> <span class="hljs-comment">/app/Gemfile</span>
<span class="hljs-comment">COPY</span> <span class="hljs-comment">Gemfile</span><span class="hljs-string">.</span><span class="hljs-comment">lock</span> <span class="hljs-comment">/app/Gemfile</span><span class="hljs-string">.</span><span class="hljs-comment">lock</span>
<span class="hljs-comment">WORKDIR</span> <span class="hljs-comment">/app</span>

<span class="hljs-comment">RUN</span> <span class="hljs-comment">bundle</span> <span class="hljs-comment">install</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">deployment</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">without</span> <span class="hljs-comment">development</span> <span class="hljs-comment">test</span></code></pre>

<p>The order in which we do the rest is important, because we want to be sure to <br />
minimize the number of discarded images when we change something in the application.</p>



<pre class="prettyprint"><code class="hljs vala"><span class="hljs-preprocessor"># here we just add all the environment variables we want</span>
<span class="hljs-preprocessor"># more on that soon</span>
ENV<span class="hljs-constant"> RAILS_ENV </span>production
ENV<span class="hljs-constant"> RACK_ENV </span>production
ENV<span class="hljs-constant"> SIDEKIQ_WORKERS </span><span class="hljs-number">10</span>

<span class="hljs-preprocessor"># what we do if we run a container from this image without telling it explicitly</span>
<span class="hljs-preprocessor"># what command to use</span>
CMD [<span class="hljs-string">"bundle"</span>, <span class="hljs-string">"exec"</span>, <span class="hljs-string">"unicorn"</span>, <span class="hljs-string">"--help"</span>]

<span class="hljs-preprocessor"># now we add in the code</span>
COPY . /app

<span class="hljs-preprocessor"># assets precompilation as late as possible</span>
RUN bundle exec rake assets:precompile
ENV<span class="hljs-constant"> GIT_REVISION </span>ef4312a
ENV<span class="hljs-constant"> SECRET_KEY_BASE </span>af9cbc68d3ad9fe71669e791c59878ffd1fc</code></pre>

<p>The first thing we have to be careful to do when we write an application that is supposed to run in containers and to scale indefinitely, is to configure it via environment variables. This is the most flexible way of working and allows you to deploy it to Heroku or Opsworks, or Opsworks+Docker.</p>

<p>Now, if you recall, we used Opsworks to pass in lots environment variables to our app, but a lot of those can be safely have default values set directly in here. Usually I put in <code>Dockerfile</code> all the variables that are required by the deployment process or that won’t change often.</p>

<p>The <code>GIT_REVISION</code> variable and <code>SECRET_KEY_BASE</code> are generated automatically by a <a href="https://gist.github.com/intinig/06dba9f914cfb94e8c8d">rake task</a> I wrote (warning: very rough, but it does its job). I usually expose <code>GIT_REVISION</code> in the backend of my applications, so I always know which version is deployed at a glance.</p>

<p>After writing our nice <code>Dockerfile</code> we launch <code>rake docker:build</code> or just build it with <code>docker build -t whatever/myself/app_name .</code>
and we have our cool image ready to be launched.</p>

<p>Let’s try to launch it with <code>docker run whatever/myself/app_name</code>:</p>



<pre class="prettyprint"><code class="hljs haml">Usage: unicorn [ruby options] [unicorn options] [rackup config file]
Ruby options:

  -<span class="ruby">e, --eval <span class="hljs-constant">LINE</span>          evaluate a <span class="hljs-constant">LINE</span> of code
</span>  -<span class="ruby">d, --debug              set debugging flags (set <span class="hljs-variable">$DEBUG</span> to <span class="hljs-keyword">true</span>)
</span>  -<span class="ruby">w, --warn               turn warnings on <span class="hljs-keyword">for</span> your script
</span>  -<span class="ruby"><span class="hljs-constant">I</span>, --<span class="hljs-keyword">include</span> <span class="hljs-constant">PATH</span>       specify <span class="hljs-variable">$LOAD_PATH</span> (may be used more than once)
</span>  -<span class="ruby">r, --<span class="hljs-keyword">require</span> <span class="hljs-constant">LIBRARY</span>    <span class="hljs-keyword">require</span> the library, before executing your script
</span>unicorn options:
  -<span class="ruby">o, --host <span class="hljs-constant">HOST</span>          listen on <span class="hljs-constant">HOST</span> (<span class="hljs-function">default: </span><span class="hljs-number">0</span>.<span class="hljs-number">0</span>.<span class="hljs-number">0</span>.<span class="hljs-number">0</span>)
</span>  -<span class="ruby">p, --port <span class="hljs-constant">PORT</span>          use <span class="hljs-constant">PORT</span> (<span class="hljs-function">default: </span><span class="hljs-number">8080</span>)
</span>  -<span class="ruby"><span class="hljs-constant">E</span>, --env <span class="hljs-constant">RACK_ENV</span>       use <span class="hljs-constant">RACK_ENV</span> <span class="hljs-keyword">for</span> defaults (<span class="hljs-function">default: </span>development)
</span>  -<span class="ruby"><span class="hljs-constant">N</span>                       <span class="hljs-keyword">do</span> <span class="hljs-keyword">not</span> load middleware implied by <span class="hljs-constant">RACK_ENV</span>
</span>      -<span class="ruby">-no-default-middleware
</span>  -<span class="ruby"><span class="hljs-constant">D</span>, --daemonize          run daemonized <span class="hljs-keyword">in</span> the background
</span>
  -<span class="ruby">s, --server <span class="hljs-constant">SERVER</span>      this flag only exists <span class="hljs-keyword">for</span> compatibility
</span>  -<span class="ruby">l {<span class="hljs-constant">HOST</span><span class="hljs-symbol">:PORT|PATH</span>},     listen on <span class="hljs-constant">HOST</span><span class="hljs-symbol">:PORT</span> <span class="hljs-keyword">or</span> <span class="hljs-constant">PATH</span>
</span>      -<span class="ruby">-listen             this may be specified multiple times
</span>                           (default: 0.0.0.0:8080)
  -<span class="ruby">c, --config-file <span class="hljs-constant">FILE</span>   <span class="hljs-constant">Unicorn</span>-specific config file
</span>Common options:
  -<span class="ruby">h, --help               <span class="hljs-constant">Show</span> this message
</span>  -<span class="ruby">v, --version            <span class="hljs-constant">Show</span> version</span></code></pre>

<p>For now everything seems to be working, but obviously we need to tell unicorn what to do. For now something like this should do the trick:</p>



<pre class="prettyprint"><code class="hljs mel">$ docker run -p <span class="hljs-number">80</span>:<span class="hljs-number">9292</span> --<span class="hljs-keyword">env</span> REQUIRED_VAR=foo whatever/myself/app_name bundle <span class="hljs-keyword">exec</span> unicorn</code></pre>

<p>If your app is simple it should already be serving requests (probably with no static assets) on port 80 of your host machine.</p>

<p>We’re not done yet, it’s now time to work on our haproxy and nginx configurations. I have mine in app_root/container/Dockerfile, but probably another repository is a better place.</p>

<p>In the first article I explained how I prefer to have haproxy serving requests to nginx to have zero downtime deployments.</p>

<p>Here’s my haproxy <code>Dockerfile</code>.</p>



<pre class="prettyprint"><code class="hljs avrasm">FROM dockerfile/haproxy
MAINTAINER Giovanni Intini &lt;giovanni@mikamai<span class="hljs-preprocessor">.com</span>&gt;
ENV REFRESHED_AT <span class="hljs-number">2014</span>-<span class="hljs-number">11</span>-<span class="hljs-number">22</span>

<span class="hljs-keyword">ADD</span> haproxy<span class="hljs-preprocessor">.cfg</span> /etc/haproxy/haproxy<span class="hljs-preprocessor">.cfg</span></code></pre>

<p>Very simple, isn’t it? Everything we need (not a lot actually) is done in <code>haproxy.cfg</code> (of which I present you an abridged version, just with the directives we need for our rails app).</p>



<pre class="prettyprint"><code class="hljs lasso"><span class="hljs-built_in">global</span>
  <span class="hljs-keyword">log</span> ${REMOTE_SYSLOG} local0
  <span class="hljs-keyword">log</span><span class="hljs-attribute">-send</span><span class="hljs-attribute">-hostname</span>
  user root
  <span class="hljs-keyword">group</span> root

frontend main
  bind /<span class="hljs-built_in">var</span>/run/app<span class="hljs-built_in">.</span>sock mode <span class="hljs-number">777</span>
  default_backend app

backend app :<span class="hljs-number">80</span>
  option httpchk GET /ping
  option redispatch
  errorloc <span class="hljs-number">400</span> /<span class="hljs-number">400</span>
  errorloc <span class="hljs-number">403</span> /<span class="hljs-number">403</span>
  errorloc <span class="hljs-number">500</span> /<span class="hljs-number">500</span>
  errorloc <span class="hljs-number">403</span> /<span class="hljs-number">503</span>

  server app0 unix@/<span class="hljs-built_in">var</span>/lib/haproxy/socks/app0<span class="hljs-built_in">.</span>sock check
  server app1 unix@/<span class="hljs-built_in">var</span>/lib/haproxy/socks/app1<span class="hljs-built_in">.</span>sock check</code></pre>

<p>Haproxy is a very powerful but not very accessible, I know. What this configuration does is easy: it defines two backend servers that will be accessed via unix sockets, and exposes a unix socket itself, that will be used by nginx.</p>

<p>The most attentive readers will already know we’re going to tell unicorn to serve requests via unix sockets, and the ones with a superhuman memory will know we already showed how to to that in the first article :)</p>

<p>I promise we’re almost there, let’s look at nginx.</p>



<pre class="prettyprint"><code class="hljs avrasm">FROM nginx
MAINTAINER Giovanni Intini &lt;giovanni@mikamai<span class="hljs-preprocessor">.com</span>&gt;
ENV REFRESHED_AT <span class="hljs-number">2015</span>-<span class="hljs-number">01</span>-<span class="hljs-number">15</span>

RUN rm /etc/nginx/conf<span class="hljs-preprocessor">.d</span>/default<span class="hljs-preprocessor">.conf</span>
RUN rm /etc/nginx/conf<span class="hljs-preprocessor">.d</span>/example_ssl<span class="hljs-preprocessor">.conf</span>

COPY proxy<span class="hljs-preprocessor">.conf</span> /etc/nginx/sites-templates/proxy<span class="hljs-preprocessor">.conf</span>
COPY nginx<span class="hljs-preprocessor">.conf</span> /etc/nginx/nginx<span class="hljs-preprocessor">.conf</span>
COPY mime<span class="hljs-preprocessor">.types</span> /etc/nginx/mime<span class="hljs-preprocessor">.types</span>
COPY headers<span class="hljs-preprocessor">.conf</span> /etc/nginx/common-headers<span class="hljs-preprocessor">.conf</span>
COPY common-proxy<span class="hljs-preprocessor">.conf</span> /etc/nginx/common-proxy<span class="hljs-preprocessor">.conf</span>

WORKDIR /etc/nginx

CMD [<span class="hljs-string">"nginx"</span>]</code></pre>

<p>As for haproxy we’re starting with the default image and just adding in our own configuration. I can’t share everything I use because it’s either taken from somewhere else or private, but I’ll give you the important details (from <code>proxy.conf</code>):</p>



<pre class="prettyprint"><code class="hljs nginx"><span class="hljs-title">upstream</span> rgts {
    <span class="hljs-title">server</span> <span class="hljs-url">unix:/var/run/rgts.sock</span> max_fails=<span class="hljs-number">0</span>;
}

<span class="hljs-title">server</span> {
    <span class="hljs-title">listen</span> <span class="hljs-number">80</span> deferred;
    <span class="hljs-title">listen</span> [::]:<span class="hljs-number">80</span> deferred;

    <span class="hljs-title">server_name</span> awesome-app.com;

    <span class="hljs-comment"># IMPORTANT!!! We rsync this with the Rails assets to ensure that you can</span>
    <span class="hljs-comment"># server up-to-date assets.</span>
    <span class="hljs-title">root</span> /var/static;
    <span class="hljs-title">client_max_body_size</span> <span class="hljs-number">4G</span>;
    <span class="hljs-title">keepalive_timeout</span> <span class="hljs-number">10</span>;

    <span class="hljs-title">open_file_cache</span>          max=<span class="hljs-number">1000</span> inactive=<span class="hljs-number">20s</span>;
    <span class="hljs-title">open_file_cache_valid</span>    <span class="hljs-number">30s</span>;
    <span class="hljs-title">open_file_cache_min_uses</span> <span class="hljs-number">2</span>;
    <span class="hljs-title">open_file_cache_errors</span>   <span class="hljs-built_in">on</span>;

    <span class="hljs-title">spdy_keepalive_timeout</span> <span class="hljs-number">300</span>;
    <span class="hljs-title">spdy_headers_comp</span> <span class="hljs-number">6</span>;

    <span class="hljs-comment"># This is for files in /public, assets included</span>
    <span class="hljs-title">location</span> / {
        <span class="hljs-title">try_files</span> <span class="hljs-variable">$uri</span>/index.html <span class="hljs-variable">$uri</span> <span class="hljs-variable">@app</span>;

        <span class="hljs-title">expires</span> max;
        <span class="hljs-title">add_header</span> Cache-Control public;

        <span class="hljs-title">etag</span> <span class="hljs-built_in">on</span>;
    }

    <span class="hljs-comment"># Dynamic pages</span>
    <span class="hljs-title">location</span> <span class="hljs-variable">@app</span> {
        <span class="hljs-title">add_header</span> Pragma <span class="hljs-string">"no-cache"</span>;
        <span class="hljs-title">add_header</span> Cache-control <span class="hljs-string">"no-store"</span>;

        <span class="hljs-title">proxy_redirect</span> <span class="hljs-built_in">off</span>;
        <span class="hljs-title">proxy_set_header</span> Host <span class="hljs-variable">$http_host</span>;
        <span class="hljs-title">proxy_set_header</span> X-Real-IP <span class="hljs-variable">$remote_addr</span>;
        <span class="hljs-title">proxy_set_header</span> X-Forwarded-For <span class="hljs-variable">$proxy_add_x_forwarded_for</span>;
        <span class="hljs-title">proxy_set_header</span> X-Forwarded-Proto <span class="hljs-variable">$http_x_forwarded_proto</span>;

        <span class="hljs-title">proxy_pass</span> <span class="hljs-url">http://app</span>;
    }
}</code></pre>

<p>Simple, and it does the job. These three images will become the key containers in your app stack.</p>

<p>You can add containers with databases and other services, or use AWS for that, but the main building blocks are there.</p>

<p>Orchestrate them locally with docker compose or in the cloud with Opsworks and friends, and you shall be a happy camper.</p>

<p>Thanks for reading up to here, feel free to comment via mail, reddit, snail mail or pigeons, and most of all, have fun with Docker.</p>