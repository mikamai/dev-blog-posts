---
ID: 56
post_title: ECS and KISS dockerization of WordPress
author: Gianluca
post_date: 2016-01-21 13:41:51
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/01/21/ecs-and-kiss-dockerization-of-wordpress/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/137748096409/ecs-and-kiss-dockerization-of-wordpress
tumblr_mikamayhem_id:
  - "137748096409"
---
<p>ECS (EC2 Container Service) is one of the latest Web services released by Amazon and it is among the cool kids around. Why? Well it let you deploy and administer Docker containers by integrating deeply with the other Web services offered by Amazon. To name a few, ELB (Elastic Load Balancer), Launching Configuration and Auto Scaling Groups (ASG).</p>

<p>At the base of ECS reside two fundamental concepts, <strong>tasks</strong> and <strong>services</strong>.</p>

<!--more-->

<p>Each one of these entities require one or more docker containers running inside special EC2 instances called ECS Container Instance. These instances are nothing more than EC2 instances shipped with the Amazon ECS container agent. The agent allows the instances to register themselves to a cluster, i.e. the entity representing the highest level of abstraction in ECS realm. For more info about it <a target="_blank" href="http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html">here</a> you can find the official definition by Amazon. The instances also need to have a proper IAM role that will let them to be handled by the Auto Scaling Group. The first time you create a cluster you get the opportunity to automatically create a proper IAM role, i.e. <code>ecsInstanceRole</code>.</p>

<p>Tasks and services are actually the things that get the work done. The first should be generally used to deliver short term, one shot, functionalities related to the deploying application while the latter are meant to ensure the running of long term tasks. The services are indeed in charge of monitoring the tasks and guarantee that they are running as desired (e.g. in the correct number).</p>

<p>Actually there is a lot (lot!) more to discuss about tasks and services but in this article doesn’t want to be a comprehensive guide to ECS (for that I encourage you to take a deep look at the official Amazon <a target="_blank" href="http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html">doc</a>). </p>

<p>What I want is to share a few choices that I made (with my colleagues) while trying to migrate a Wordpress application on ECS (and consequently on Docker).</p>

<p>The first one was to represent our application as an entire <strong>cluster</strong> in order to have a more fine grained control over the actual architecture of the tasks and the services placed at the base of our application. We still don’t know if it was the right way to go but, until now, we have not encouter any problem with this decision. </p>

<p>To KISS (Keep It Simple and Stupid) I have also decided to deploy the application inside a single docker container with NGINX as application server and the HHVM as PHP “interpreter”.</p>

<p>The database is completely external as it resides on a dedicated RDS instance while the assets are served from an S3 bucket on which are uploaded directly by a Wordpress plugin that I have already mentioned in one of my previous articles, i.e. <a target="_blank" href="https://wordpress.org/plugins/amazon-s3-and-cloudfront/">Wordpress Offload</a>.</p>

<p>Together with a proper Dockerfile, that I will probably describe in a future post, I also restructured the application <code>wp-config.php</code> to rely on the environment variables and the <a target="_blank" href="https://github.com/vlucas/phpdotenv">PHP dotenv</a> library:</p>

<div class="highlight highlight-text-html-php"><pre><span class="pl-pse">&lt;?php</span><span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c">/**</span></span>
<span class="pl-s1"><span class="pl-c"> * The base configurations of a Dockerized WordPress.</span></span>
<span class="pl-s1"><span class="pl-c"> * You can find more information by visiting</span></span>
<span class="pl-s1"><span class="pl-c"> * {<span class="pl-k">@link</span> <a target="_blank" href="http://codex.wordpress.org/Editing_wp-config.php">http://codex.wordpress.org/Editing_wp-config.php</a> Editing wp-config.php}</span></span>
<span class="pl-s1"><span class="pl-c"> */</span></span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Load the .env if present.</span></span>
<span class="pl-s1"><span class="pl-k">if</span> (<span class="pl-c1">file_exists</span>(<span class="pl-c1">dirname</span>(<span class="pl-c1">__FILE__</span>)<span class="pl-k">.</span><span class="pl-c1">DIRECTORY_SEPARATOR</span><span class="pl-k">.</span><span class="pl-s"><span class="pl-pds">'</span>.env<span class="pl-pds">'</span></span>)) {</span>
<span class="pl-s1">  <span class="pl-k">require</span> <span class="pl-s"><span class="pl-pds">'</span>vendor/autoload.php<span class="pl-pds">'</span></span>;</span>
<span class="pl-s1">  <span class="pl-smi">$dotenv</span> <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-c1">Dotenv</span><span class="pl-c1">Dotenv</span>(<span class="pl-c1">__DIR__</span>);</span>
<span class="pl-s1">  <span class="pl-smi">$dotenv</span><span class="pl-k">-&gt;</span>load();</span>
<span class="pl-s1">}</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># The name of the database for WordPress.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DB_NAME<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DB_NAME<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># MySQL database username.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DB_USER<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DB_USER<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># MySQL database password.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DB_PASSWORD<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DB_PASSWORD<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># MySQL hostname.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DB_HOST<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DB_HOST<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Database Charset to use in creating database tables.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DB_CHARSET<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DB_CHARSET<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># The Database Collate type. Don't change this if in doubt.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DB_COLLATE<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DB_COLLATE<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c">/**</span></span>
<span class="pl-s1"><span class="pl-c"> * Authentication Unique Keys and Salts.</span></span>
<span class="pl-s1"><span class="pl-c"> *</span></span>
<span class="pl-s1"><span class="pl-c"> * Change these to different unique phrases!</span></span>
<span class="pl-s1"><span class="pl-c"> * You can generate these using the {<span class="pl-k">@link</span> <a target="_blank" href="https://api.wordpress.org/secret-key/1.1/salt/">https://api.wordpress.org/secret-key/1.1/salt/</a> WordPress.org secret-key service}</span></span>
<span class="pl-s1"><span class="pl-c"> * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.</span></span>
<span class="pl-s1"><span class="pl-c"> *</span></span>
<span class="pl-s1"><span class="pl-c"> * <span class="pl-k">@since</span> 2.6.0</span></span>
<span class="pl-s1"><span class="pl-c"> */</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>AUTH_KEY<span class="pl-pds">'</span></span>,         <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_AUTH_KEY<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>SECURE_AUTH_KEY<span class="pl-pds">'</span></span>,  <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_SECURE_AUTH_KEY<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>LOGGED_IN_KEY<span class="pl-pds">'</span></span>,    <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_LOGGED_IN_KEY<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>NONCE_KEY<span class="pl-pds">'</span></span>,        <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_NONCE_KEY<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>AUTH_SALT<span class="pl-pds">'</span></span>,        <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_AUTH_SALT<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>SECURE_AUTH_SALT<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_SECURE_AUTH_SALT<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>LOGGED_IN_SALT<span class="pl-pds">'</span></span>,   <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_LOGGED_IN_SALT<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>NONCE_SALT<span class="pl-pds">'</span></span>,       <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_NONCE_SALT<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Enable Amazon S3 and Cloudfront.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>AWS_ACCESS_KEY_ID<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_AWS_ACCESS_KEY_ID<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>AWS_SECRET_ACCESS_KEY<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_AWS_SECRET_ACCESS_KEY<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Enable WP Cache.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WP_CACHE<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WP_CACHE<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WPCACHEHOME<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WPCACHEHOME<span class="pl-pds">'</span></span>)); <span class="pl-c">//Added by WP-Cache Manager</span></span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Enable Secure Logins.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>FORCE_SSL_LOGIN<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_FORCE_SSL_LOGIN<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>FORCE_SSL_ADMIN<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_FORCE_SSL_ADMIN<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Use external cron.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DISABLE_WP_CRON<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DISABLE_WP_CRON<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c">/**</span></span>
<span class="pl-s1"><span class="pl-c"> * WordPress Database Table prefix.</span></span>
<span class="pl-s1"><span class="pl-c"> *</span></span>
<span class="pl-s1"><span class="pl-c"> * You can have multiple installations in one database if you give each a unique</span></span>
<span class="pl-s1"><span class="pl-c"> * prefix. Only numbers, letters, and underscores please!</span></span>
<span class="pl-s1"><span class="pl-c"> */</span></span>
<span class="pl-s1"><span class="pl-smi">$table_prefix</span> <span class="pl-k">=</span> <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_TABLE_PREFIX<span class="pl-pds">'</span></span>);</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c">/**</span></span>
<span class="pl-s1"><span class="pl-c"> * WordPress Localized Language, defaults to English.</span></span>
<span class="pl-s1"><span class="pl-c"> *</span></span>
<span class="pl-s1"><span class="pl-c"> * Change this to localize WordPress. A corresponding MO file for the chosen</span></span>
<span class="pl-s1"><span class="pl-c"> * language must be installed to wp-content/languages. For example, install</span></span>
<span class="pl-s1"><span class="pl-c"> * de_DE.mo to wp-content/languages and set WPLANG to 'de_DE' to enable German</span></span>
<span class="pl-s1"><span class="pl-c"> * language support.</span></span>
<span class="pl-s1"><span class="pl-c"> */</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WPLANG<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WPLANG<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c">/**</span></span>
<span class="pl-s1"><span class="pl-c"> * For developers: WordPress debugging mode.</span></span>
<span class="pl-s1"><span class="pl-c"> *</span></span>
<span class="pl-s1"><span class="pl-c"> * Change this to true to enable the display of notices during development.</span></span>
<span class="pl-s1"><span class="pl-c"> * It is strongly recommended that plugin and theme developers use WP_DEBUG</span></span>
<span class="pl-s1"><span class="pl-c"> * in their development environments.</span></span>
<span class="pl-s1"><span class="pl-c"> */</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WP_DEBUG<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WP_DEBUG<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Absolute path to the WordPress directory.</span></span>
<span class="pl-s1"><span class="pl-k">if</span> (<span class="pl-k">!</span><span class="pl-c1">defined</span>(<span class="pl-s"><span class="pl-pds">'</span>ABSPATH<span class="pl-pds">'</span></span>))</span>
<span class="pl-s1">  <span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>ABSPATH<span class="pl-pds">'</span></span>, <span class="pl-c1">dirname</span>(<span class="pl-c1">__FILE__</span>)<span class="pl-k">.</span><span class="pl-c1">DIRECTORY_SEPARATOR</span>);</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># To enable the possibility to update plugins directly from back-end (locally).</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>FS_METHOD<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_FS_METHOD<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Disable updates of plugins and themes together with plugins and themes editor.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>DISALLOW_FILE_MODS<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_DISALLOW_FILE_MODS<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Disable all automatic updates.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>AUTOMATIC_UPDATER_DISABLED<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_AUTOMATIC_UPDATER_DISABLED<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># How to handle all core updates.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WP_AUTO_UPDATE_CORE<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WP_AUTO_UPDATE_CORE<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># WP Siteurl.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WP_SITEURL<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WP_SITEURL<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># WP Home.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WP_HOME<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WP_HOME<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Enable Multisite.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>WP_ALLOW_MULTISITE<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_WP_ALLOW_MULTISITE<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c"># Uploads directory.</span></span>
<span class="pl-s1"><span class="pl-c1">define</span>(<span class="pl-s"><span class="pl-pds">'</span>UPLOADS<span class="pl-pds">'</span></span>, <span class="pl-c1">getenv</span>(<span class="pl-s"><span class="pl-pds">'</span>WORDPRESS_UPLOADS<span class="pl-pds">'</span></span>));</span>
<span class="pl-s1"></span>
<span class="pl-s1"><span class="pl-c">/** Sets up WordPress vars and included files. */</span></span>
<span class="pl-s1"><span class="pl-k">require_once</span>(<span class="pl-c1">ABSPATH</span><span class="pl-k">.</span><span class="pl-s"><span class="pl-pds">'</span>wp-settings.php<span class="pl-pds">'</span></span>);</span></pre></div>

<p>It defines almost all the constants that can be used inside a <code>wp-config.php</code> and valorize them with environment variables.</p>

<p>Given the behavior of the <a target="_blank" href="http://php.net/manual/it/function.getenv.php"><code>getenv()</code></a> function, the constants that are boolean by definition (e.g. <code>WP_DEBUG</code>) should not be defined if they need to be <code>false</code>. The <code>getenv('SOME_ENV_VARIABLE')</code> returns indeed <code>false</code> if <code>'SOME_ENV_VARIABLE'</code> is not defined.</p>

<p>To continue to develop locally it is also needed to:</p>

<ul><li>install <a target="_blank" href="https://getcomposer.org/download/">composer</a>
</li>
<li>install <a target="_blank" href="https://github.com/vlucas/phpdotenv">phpdotenv</a>
</li>
<li>place an <code>.env</code> file in the root of the app repo with all the environment variables that your app need to be running. Here there is an example of a possible <code>.env</code> file:</li>
</ul><pre><code>WORDPRESS_DB_NAME=db_name
WORDPRESS_DB_USER=db_user
WORDPRESS_DB_PASSWORD=db_password
WORDPRESS_DB_HOST=localhost

WORDPRESS_DB_CHARSET=utf8
WORDPRESS_DB_COLLATE=

WORDPRESS_AUTH_KEY=
WORDPRESS_SECURE_AUTH_KEY=
WORDPRESS_LOGGED_IN_KEY=
WORDPRESS_NONCE_KEY=
WORDPRESS_AUTH_SALT=
WORDPRESS_SECURE_AUTH_SALT=
WORDPRESS_LOGGED_IN_SALT=
WORDPRESS_NONCE_SALT=

WORDPRESS_AWS_ACCESS_KEY_ID=MYACCESSKEY
WORDPRESS_AWS_SECRET_ACCESS_KEY=MYACCESSKEY

WORDPRESS_TABLE_PREFIX=wp_
WORDPRESS_WPLANG=en_US
WORDPRESS_WP_DEBUG=
WORDPRESS_FS_METHOD=direct
WORDPRESS_WP_SITEURL=http://my-site.test
WORDPRESS_WP_HOME=http://my-site.test
</code></pre>

<p>The modified version of the <code>wp-config.php</code> take into consideration the possible presence of the <code>.env</code> file and loads the environment variables making them available to you app:</p>

<div class="highlight highlight-text-html-php"><pre><span class="pl-s1"><span class="pl-c"># Load the .env if present.</span></span>
<span class="pl-s1"><span class="pl-k">if</span> (<span class="pl-c1">file_exists</span>(<span class="pl-c1">dirname</span>(<span class="pl-c1">__FILE__</span>)<span class="pl-k">.</span><span class="pl-c1">DIRECTORY_SEPARATOR</span><span class="pl-k">.</span><span class="pl-s"><span class="pl-pds">'</span>.env<span class="pl-pds">'</span></span>)) {</span>
<span class="pl-s1">  <span class="pl-k">require</span> <span class="pl-s"><span class="pl-pds">'</span>vendor/autoload.php<span class="pl-pds">'</span></span>;</span>
<span class="pl-s1">  <span class="pl-smi">$dotenv</span> <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-c1">Dotenv</span><span class="pl-c1">Dotenv</span>(<span class="pl-c1">__DIR__</span>);</span>
<span class="pl-s1">  <span class="pl-smi">$dotenv</span><span class="pl-k">-&gt;</span>load();</span>
<span class="pl-s1">}</span></pre></div>

<p>To deploy the application I relied on a automatic building service, i.e. <a target="_blank" href="https://quay.io">Quay.io</a>, to automatically build the images when changes are pushed to the version control system. ECS can be configured to pull images from a remote repo like the one offered by Quay.io.</p>

<p>Together with the ECS cluster I also created a task definition represeting the application with all the containers needed by it to run. The tasks can be configured through the UI or a <code>JSON</code> file. At this point you can set the required environment variables needed by the <code>wp-config.php</code>. An example of the <code>"environment"</code> section inside the <code>JSON</code> configuration is the following one:</p>

<div class="highlight highlight-source-json"><pre>...
<span class="pl-s"><span class="pl-pds">"</span>environment<span class="pl-pds">"</span></span>: [
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_DB_USER<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>name_of_the_db_user<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_DB_CHARSET<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>utf8<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_DB_HOST<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>the_host_where_the_db_is_placed<span class="pl-pds">"</span></span> <span class="pl-ii">#</span> <span class="pl-ii">possibly</span> <span class="pl-ii">an</span> <span class="pl-ii">RDS</span> <span class="pl-ii">endpoint</span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_NONCE_KEY<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>long_random_key<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_DB_NAME<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>name_of_the_app_db<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_AWS_ACCESS_KEY_ID<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>aws_access_key<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_LOGGED_IN_KEY<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>long_random_key<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_UPLOADS<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>location_of_the_uploads_directory<span class="pl-pds">"</span></span> <span class="pl-ii">#</span> <span class="pl-ii">by</span> <span class="pl-ii">default</span> <span class="pl-ii">`wp-content/uploads`</span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_DISABLE_WP_CRON<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>true<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_AWS_SECRET_ACCESS_KEY<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>aws_secret_access_key<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_DB_PASSWORD<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>db_password_of_the_db_user<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_SECURE_AUTH_KEY<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>long_random_key<span class="pl-pds">"</span></span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_TABLE_PREFIX<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>table_prefix<span class="pl-pds">"</span></span> <span class="pl-ii">#</span> <span class="pl-ii">by</span> <span class="pl-ii">default</span> <span class="pl-ii">`wp_`</span>
  },
  {
    <span class="pl-s"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>WORDPRESS_WPLANG<span class="pl-pds">"</span></span>,
    <span class="pl-s"><span class="pl-pds">"</span>value<span class="pl-pds">"</span></span>: <span class="pl-s"><span class="pl-pds">"</span>en_GB<span class="pl-pds">"</span></span>
  },
  <span class="pl-ii">...</span>
]
...</pre></div>

<p>After creating the task I created a service to handle the delivery of the application and associate with it an ELB to balace the incoming traffic between the instances allocated with the cluster.</p>

<p>The ELB will need to be associated with an Auto Scaling Group which, in turn, will need to be associated with a Launching Configuration. The ELB, the Auto Scaling Group and the Launching Configuration are the entities that control the number of EC2 instances on which the tasks runs on.</p>

<p>The number of EC2 instances that need to be run by a service to handle the deployment of the new versions of the application is always the desired number of EC2 instances plus one. One instance should always be free to hold the new container created by ECS when a new version of the application task is created.</p>

<p>The deploy of a new version of the application a new revision of the related task should be created. This can be accomplished directly from the Amazon Dashboard or by using the aws-cli.</p>

<p>Personally I found particularly usefull this <a target="_blank" href="https://github.com/silinternational/ecs-deploy">script</a>. It still has some problems (e.g. with the region parameter) but if you go with the following command you shouldn’t encounter any problem at all:</p>

<pre><code>ecs-deploy -k AWS_KEY -s AWS_SECRET_KEY -c CLUSTER -n NAME_OF_SERVICE_TO_DEPLOY -i repo.of.image.building.service/ORGANIZATION/REPO_NAME:TAG
</code></pre>

<p>What it does is simply to wrap the <a target="_blank" href="http://docs.aws.amazon.com/cli/latest/reference/ecs/index.html">aws-cli commands</a> needed to update the service that handles the task that delivers your application. This is done, as mentioned before, by creating a new revision of the task.</p>

<p>I realize that I talked about a lot of stuff and that I barely scratched their surface but I still hope that what I shared can be helpfull to everyone approaching the new “ECS world”.</p>

<p>UPDATE: to have a better overview of the serie <a target="_blank" href="http://dev.mikamai.com/post/139422949699/ecs-and-kiss-dockerization-of-wordpress-part-2">here</a> there is a link to the second article related to ECS and WordPress ;)</p>

<p>Cheers!</p>