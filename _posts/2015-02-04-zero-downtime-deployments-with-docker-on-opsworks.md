---
ID: 160
post_title: >
  Zero Downtime deployments with Docker on
  Opsworks
author: Giovanni Intini
post_date: 2015-02-04 13:23:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/04/zero-downtime-deployments-with-docker-on-opsworks/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/110066104864/zero-downtime-deployments-with-docker-on-opsworks
tumblr_mikamayhem_id:
  - "110066104864"
---
<p><em>Updated on 2015-02-19, changed the stack JSON with new interpolation syntax</em></p>

<p>When I was younger I always wanted to be a cool kid. Growing up as computer enthusiast that meant using the <a href="http://enlightenment.org">latest window manager</a>, and the most cutting edge versions of all software. Who hasn’t gone through a recompile-kernel-every-day-several-times-a-day phase in their lives?</p>

<p>Luckily I was able to survive that phase (except a 120GB Raid0 setup lost, but that’s another story), and maturity gave me the usual old guy-esque fear of change.</p>

<p>That’s the main reason I waited a bit before approaching <a href="http://docker.io">Docker</a>. All the cool kids where raving about it, and I am a cool kid no more, just a calm, collected, young-ish man.</p>

<p>You might have guessed it from the title of this article, but Docker blew my mind. It solves in a simple way lots of common development and deployment problems.</p>

<p>After learning to rebuild my application to work from a Docker container, more about it in the next weeks, my focus went to deploying Docker. We run our application on the AWS cloud, and our Amazon service of choice for that is <a href="http://aws.amazon.com/opsworks/">Opsworks</a>. Easy to setup, easy to maintain, easy to scale.</p>

<p>Unfortunately Opsworks does not support Docker, and ECS, Amazon’s own container running solution, is not production ready. I needed to migrate my complicated application to Docker, and I wanted to do it now. The only solution was convincing Opsworks to support Docker.</p>

<p>Googling around I found a <a href="http://blogs.aws.amazon.com/application-management/post/Tx2FPK7NJS5AQC5/Running-Docker-on-AWS-OpsWorks">tutorial</a> and <a href="https://github.com/jweiss/docker-opsworks">some</a> <a href="https://github.com/classmethod-aws/opsworks-docker-sample">github</a> <a href="https://github.com/coryflucas/opsworks-docker">repos</a>, but none were up to the task for me.</p>

<p>My application needs several containers to work, and I was aiming for something that allowed me to orchestrate the following:</p>

<ul><li>1 load balancer (but it might be ELB!)</li>
<li>1 frontend webserver, like nginx</li>
<li>1 haproxy to juggle between app servers, to allow for zero downtime deployments</li>
<li>2 application containers, running their own app server</li>
<li>1 Redis for caching (but it might be Elasticache!)</li>
<li>1 PostgreSQL as DB (but it might be Amazon RDS)</li>
<li>1 or more cron tasks</li>
<li>1 or more sidekiq instances for async processing</li>
</ul><p>In addition to that I wanted everything to scale horizontally based on load. It seemed a daunting task at first, but as I started writing my chef recipes everything started to make sense. My inspiration was <a href="http://www.fig.sh">fig</a>. I like the way fig allows you to specify relations between containers, and I wanted to do something like that on Opsworks.</p>

<p>The result of my work can be found <a href="https://github.com/intinig/opsworks-docker">here</a>. At the time of this writing the README.md file is still blank, but I promise some documentation should appear there sooner or later… for now use this article as a reference :)</p>

<p>The first thing we’ll do to deploy our dockerized app is login on AWS, access Opsworks and click on create a stack. Fill in the form like this:</p>

<ul><li>Name: whatever you like</li>
<li>Region: whatever you prefer</li>
<li>Default root device type: EBS backend</li>
<li>Default SSH key: choose your key if you want to access your instances</li>
<li>[set the rest to default or your liking if you know what you’re doing]</li>
<li>Advanced <br /><ul><li>Use custom Chef cookbooks: YES</li>
<li>Repository URL: <a href="https://github.com/intinig/opsworks-docker.git">https://github.com/intinig/opsworks-docker.git</a></li>
<li>Manage Berkshelf: YES</li></ul></li>
</ul><p>Once you have your stack you have to add a layer. Click on add a layer and fill in the form:</p>

<ul><li>Layer type: Custom</li>
<li>Name: Docker</li>
<li>Short name: docker</li>
</ul><p>After you create the layer go Edit its configuration and click the EBS volumes tab. We’ll need to add ad 120GB volume for each instance we add to the stack. Why you ask? Unfortunately on Amazon Linux/EC2 docker will use devicemapper to manage your containers, and devicemapper creates a file that will grow with normal use to up to 100GB. The extra 20GB are used for your images. You can go with less than that, or even no EBS volume, but know that sooner or later you’ll hit that limit.</p>

<ul><li>Mount point: /var/lib/docker</li>
<li>Size total: 120GB</li>
<li>Volume type: General Purpose (SSD)</li>
</ul><p>After that let’s edit our layer to add our custom recipes:</p>

<ul><li>Setup <br /><ul><li>docker::install, docker::registries, logrotate::default, docker::logrotate</li></ul></li>
<li>Deploy <br /><ul><li>docker::data_volumes, docker::deploy</li></ul></li>
</ul><p>What do our custom recipes do?</p>

<ul><li>docker::install is easy, it just installs docker on our opsworks instances</li>
<li>docker::registries is used to login in private docker registries. It should work with several type of registries, but I have personally tested it only with <a href="http://quay.io">quay.io</a></li>
<li>logrotate::default and docker::logrotate manage the setup of logrotate to avoid ever growing docker logs. This setup assumes you’re actually sending logs to a remote syslog, we use <a href="http://papertrailapp.com">papertrail</a> for that</li>
</ul><p>Now let’s add an application. From the Opsworks menu on the left click Apps and add a new one.</p>

<ul><li>Name: amazing application</li>
<li>Type: Other</li>
<li>Data Source Type: here I choose RDS, but you can feel free to use OpsWorks, or no DB at all and pass the data to your app via docker containers or other sources</li>
<li>Repository type: Other</li>
</ul><p>Now add just one Env variable to the app:</p>

<ul><li>APP_TYPE: docker</li>
</ul><p>Everything else will be configured via the (enormous) Stack JSON. Go to your stack settings and edit them. You will need to compile a stack json for your containers. Here’s an example one:</p>



<pre class="prettyprint"><code class="hljs json">{
  "<span class="hljs-attribute">logrotate</span>": <span class="hljs-value">{
    "<span class="hljs-attribute">forwarder</span>": <span class="hljs-value"><span class="hljs-string">"logspout0"</span>
  </span>}</span>,
  "<span class="hljs-attribute">deploy</span>": <span class="hljs-value">{
    "<span class="hljs-attribute">amazing application</span>": <span class="hljs-value">{
      "<span class="hljs-attribute">data_volumes</span>": <span class="hljs-value">[
      {
        "<span class="hljs-attribute">socks</span>": <span class="hljs-value">{
          "<span class="hljs-attribute">volumes</span>": <span class="hljs-value">[<span class="hljs-string">"/var/run"</span>, <span class="hljs-string">"/var/lib/haproxy/socks"</span>]
        </span>}</span>,
        "<span class="hljs-attribute">static</span>": <span class="hljs-value">{
          "<span class="hljs-attribute">volumes</span>": <span class="hljs-value">[<span class="hljs-string">"/var/static"</span>]
        </span>}
      </span>}
      ]</span>,
      "<span class="hljs-attribute">containers</span>": <span class="hljs-value">[
        {
          "<span class="hljs-attribute">app</span>": <span class="hljs-value">{
            "<span class="hljs-attribute">deploy</span>": <span class="hljs-value"><span class="hljs-string">"auto"</span></span>,
            "<span class="hljs-attribute">image</span>": <span class="hljs-value"><span class="hljs-string">"quay.io/mikamai/awesome-app"</span></span>,
            "<span class="hljs-attribute">database</span>": <span class="hljs-value"><span class="hljs-literal">true</span></span>,
            "<span class="hljs-attribute">containers</span>": <span class="hljs-value"><span class="hljs-number">2</span></span>,
            "<span class="hljs-attribute">volumes_from</span>": <span class="hljs-value">[<span class="hljs-string">"socks"</span>, <span class="hljs-string">"static"</span>]</span>,
            "<span class="hljs-attribute">entrypoint</span>": <span class="hljs-value"><span class="hljs-string">"/app/bin/entrypoint.sh"</span></span>,
            "<span class="hljs-attribute">command</span>": <span class="hljs-value"><span class="hljs-string">"bundle exec unicorn -c config/unicorn.rb -l /var/lib/haproxy/socks/%{app_name}.sock"</span></span>,
            "<span class="hljs-attribute">migration</span>": <span class="hljs-value"><span class="hljs-string">"bundle exec rake db:migrate"</span></span>,
            "<span class="hljs-attribute">startup_time</span>": <span class="hljs-value"><span class="hljs-number">60</span></span>,
            "<span class="hljs-attribute">env</span>": <span class="hljs-value">{
              "<span class="hljs-attribute">RANDOM_VAR</span>": <span class="hljs-value"><span class="hljs-string">"foo"</span>
            </span>}</span>,
            "<span class="hljs-attribute">notifications</span>": <span class="hljs-value">{
              "<span class="hljs-attribute">rollbar</span>" : <span class="hljs-value">{
                "<span class="hljs-attribute">access_token</span>": <span class="hljs-value"><span class="hljs-string">""</span></span>,
                "<span class="hljs-attribute">env_var</span>": <span class="hljs-value"><span class="hljs-string">"RACK_ENV"</span></span>,
                "<span class="hljs-attribute">rev_var</span>": <span class="hljs-value"><span class="hljs-string">"GIT_REVISION"</span>
              </span>}
            </span>}
          </span>}
        </span>},
        {
          "<span class="hljs-attribute">cron</span>": <span class="hljs-value">{
            "<span class="hljs-attribute">deploy</span>": <span class="hljs-value"><span class="hljs-string">"cron"</span></span>,
            "<span class="hljs-attribute">image</span>": <span class="hljs-value"><span class="hljs-string">"quay.io/mikamai/awesome-app"</span></span>,
            "<span class="hljs-attribute">database</span>": <span class="hljs-value"><span class="hljs-literal">true</span></span>,
            "<span class="hljs-attribute">containers</span>": <span class="hljs-value"><span class="hljs-number">1</span></span>,
            "<span class="hljs-attribute">env_from</span>": <span class="hljs-value"><span class="hljs-string">"app"</span></span>,
            "<span class="hljs-attribute">command</span>": <span class="hljs-value"><span class="hljs-string">"bundle exec rake cron:run"</span></span>,
            "<span class="hljs-attribute">cron</span>": <span class="hljs-value">{"<span class="hljs-attribute">minute</span>": <span class="hljs-value"><span class="hljs-number">59</span></span>}
          </span>}
        </span>},

        {
          "<span class="hljs-attribute">sidekiq</span>": <span class="hljs-value">{
            "<span class="hljs-attribute">deploy</span>": <span class="hljs-value"><span class="hljs-string">"auto"</span></span>,
            "<span class="hljs-attribute">image</span>": <span class="hljs-value"><span class="hljs-string">"quay.io/mikamai/awesome-app"</span></span>,
            "<span class="hljs-attribute">database</span>": <span class="hljs-value"><span class="hljs-literal">true</span></span>,
            "<span class="hljs-attribute">containers</span>": <span class="hljs-value"><span class="hljs-number">1</span></span>,
            "<span class="hljs-attribute">env_from</span>": <span class="hljs-value"><span class="hljs-string">"app"</span></span>,
            "<span class="hljs-attribute">command</span>": <span class="hljs-value"><span class="hljs-string">"bundle exec sidekiq"</span>
          </span>}
        </span>},
        {
          "<span class="hljs-attribute">haproxy</span>": <span class="hljs-value">{
            "<span class="hljs-attribute">deploy</span>": <span class="hljs-value"><span class="hljs-string">"manual"</span></span>,
            "<span class="hljs-attribute">hostname</span>": <span class="hljs-value"><span class="hljs-string">"opsworks"</span></span>,
            "<span class="hljs-attribute">image</span>": <span class="hljs-value"><span class="hljs-string">"quay.io/mikamai/configured-haproxy"</span></span>,
            "<span class="hljs-attribute">volumes_from</span>": <span class="hljs-value">[<span class="hljs-string">"socks"</span>]</span>,
            "<span class="hljs-attribute">env</span>": <span class="hljs-value">{
              "<span class="hljs-attribute">REMOTE_SYSLOG</span>": <span class="hljs-value"><span class="hljs-string">"logs.papertrailapp.com:1234"</span>
            </span>}
          </span>}
        </span>},
        {
          "<span class="hljs-attribute">nginx</span>": <span class="hljs-value">{
            "<span class="hljs-attribute">deploy</span>": <span class="hljs-value"><span class="hljs-string">"manual"</span></span>,
            "<span class="hljs-attribute">image</span>": <span class="hljs-value"><span class="hljs-string">"quay.io/mikamai/configured-nginx"</span></span>,
            "<span class="hljs-attribute">ports</span>": <span class="hljs-value">[
              <span class="hljs-string">"80:80"</span>,
              <span class="hljs-string">"443:443"</span>
            ]</span>,
            "<span class="hljs-attribute">volumes_from</span>": <span class="hljs-value">[
              <span class="hljs-string">"socks"</span>,
              <span class="hljs-string">"static"</span>
            ]
          </span>}
        </span>},
        {
          "<span class="hljs-attribute">logspout</span>": <span class="hljs-value">{
            "<span class="hljs-attribute">deploy</span>": <span class="hljs-value"><span class="hljs-string">"manual"</span></span>,
            "<span class="hljs-attribute">hostname</span>": <span class="hljs-value"><span class="hljs-string">"%{opsworks}"</span></span>,
            "<span class="hljs-attribute">image</span>": <span class="hljs-value"><span class="hljs-string">"progrium/logspout"</span></span>,
            "<span class="hljs-attribute">volumes</span>": <span class="hljs-value">[
              <span class="hljs-string">"/var/run/docker.sock:/tmp/docker.sock"</span>
            ]</span>,
            "<span class="hljs-attribute">command</span>": <span class="hljs-value"><span class="hljs-string">"syslog://logs.papertrailapp.com:1234"</span>
          </span>}
        </span>}
      ]
    </span>}
  </span>}</span>,
  "<span class="hljs-attribute">docker</span>": <span class="hljs-value">{
    "<span class="hljs-attribute">registries</span>": <span class="hljs-value">{
      "<span class="hljs-attribute">quay.io</span>": <span class="hljs-value">{
        "<span class="hljs-attribute">password</span>": <span class="hljs-value"><span class="hljs-string">""</span></span>,
        "<span class="hljs-attribute">username</span>": <span class="hljs-value"><span class="hljs-string">""</span>
      </span>}
    </span>}
  </span>}
</span>}</code></pre>

<p>WOW! That’s a lot to digest, isn’t it? In the next article we’ll go through the Stack JSON and see what each of the keys mean and what they enable you to do.</p>

<p>Thanks for reading through this, see you soon!</p>