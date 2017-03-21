---
ID: 166
post_title: >
  WordPress and OpsWorks, “Pride and
  Prejudice” (Part 3)
author: Gianluca
post_date: 2015-01-21 13:11:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/01/21/wordpress-and-opsworks-pride-and-prejudice-2/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/108731475409/wordpress-and-opsworks-pride-and-prejudice
tumblr_mikamayhem_id:
  - "108731475409"
---
<p><span>I know, this third part of the series regarding Wordpress and OpsWorks comes a little bit late. </span></p>
<p>Between this article and the <a href="http://dev.mikamai.com/post/93782458919/wordpress-and-opsworks-pride-and-prejudice" target="_blank">previous one</a> of the series I wrote a lot of other gems spanning many different topics. <br /> But now I’m here to continue right from where I stopped ;)</p>
<p>Before we begin, here&rsquo;s the <a href="http://dev.mikamai.com/post/89353822489/wordpress-and-opsworks-pride-and-prejudice" target="_blank">first</a> and the <a href="http://dev.mikamai.com/post/93782458919/wordpress-and-opsworks-pride-and-prejudice" target="_blank">second</a> article.</p>
<p>Now, just to wrap up, last time I tried to explain a quirk about the compilation and the convergence phase of a chef node.</p>
<p>To present it I relied on a recipe I wrote to “set up” the database during the deploy of a WP (WordPress) application.</p>
<p>Everyone accustomed with WP development knows that, due the characteristics of WP itself, it is very often difficult to keep everything updated and working as expected.</p>
<p>Many functionalities depend not only on the code but also on the state of the database where most of the configurations are stored. The problem with WP is that there isn’t a recognized and out-of-the-box standard that can be followed to handle the aformentioned database state.</p>
<p>No Rails-like migrations, sorry.</p>
<p>A simple (but bad) way to solve this problem is to push, together with the code, a dump of the database directly inside the repository of the project. In this way it’s state can also be tracked and most of all restored (imported) whenever needed. <br /> Obviously this solution doesn’t fit the case in which there are sensitive information that can’t be simply shared between all the project contributors.</p>
<p>Anyway, assuming this is not the case, if you plan to deploy a WP project through OpsWorks you may end up with the need to automatically import a dump just during a deploy.</p>
<p>This is exactly the purpose of the recipe taken as an example in the last article of this series.</p>
<p>But hey, as L.T. says, “Talk is cheap. Show me the code”. So, here it is:</p>
<pre class="prettyprint"><code class="hljs haml">script 'load_database' do
  only_if { File.file?(db_path) and Chef::Log.info('Load PrestaShop database...') }
  interpreter 'bash'
  user 'root'
  code &lt;&lt;-MIKA
    mysql -h #{<span class="ruby">deploy[<span class="hljs-string">'database'</span>][<span class="hljs-string">'host'</span>]}</span> -u #{<span class="ruby">deploy[<span class="hljs-string">'database'</span>][<span class="hljs-string">'username'</span>]}</span> #{<span class="ruby">deploy[<span class="hljs-string">'database'</span>][<span class="hljs-string">'password'</span>].blank? ? <span class="hljs-string">''</span> <span class="hljs-symbol">:</span> <span class="hljs-string">"-p<span class="hljs-subst">#{deploy[<span class="hljs-string">'database'</span>][<span class="hljs-string">'password'</span>]}</span></span></span>"} #{<span class="ruby">deploy[<span class="hljs-string">'database'</span>][<span class="hljs-string">'database'</span>]}</span> &lt; #{<span class="ruby">db_path}</span>;
  MIKA
end</code></pre>
<p>What I do here is simply to rely on the <a href="https://docs.chef.io/resource_script.html">script</a> resource to invoke the mysql Command Line Interface (CLI) and tell it to import the dump located at the path stored inside the db_path variable inside the proper database. <br /> This is done by relying on the bash interpreter and by using all the info that OpsWorks gives us through a JSON object (one per deployed application) embedded inside the deploy attribute of the JSON associated with the deploy event.</p>
<pre class="prettyprint"><code class="hljs r">{
  <span class="hljs-string">"deploy"</span> : {
    <span class="hljs-string">"app1"</span> : {
      <span class="hljs-string">"database"</span> : {
        <span class="hljs-string">"database"</span> : <span class="hljs-string">"..."</span>,
        <span class="hljs-string">"host"</span> : <span class="hljs-string">"..."</span>,
        <span class="hljs-string">"username"</span> : <span class="hljs-string">"..."</span>,
        <span class="hljs-string">"password"</span> : <span class="hljs-string">"..."</span>,
      }
    },
    <span class="hljs-string">"app2"</span> : {
      <span class="hljs-keyword">...</span>
    },
    <span class="hljs-keyword">...</span>
  }
}</code></pre>
<p>The overmentioned info are picked up by OpsWorks (and underline, Chef) right from the app configuration that can be accessed from the OpsWorks dashboard.</p>
<p>The complete list of the deploy attributes can be found right <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/attributes-json-deploy.html#attributes-json-deploy-app-db">here</a>.</p>
<p>As a side note and as already stated in the previous article, the import of the database gets triggered only if the the dump is actually present and if a proper flag (i.e. “import_database”) is present inside the overmentioned JSON.</p>
<p>Next time I will talk about…well…I don’t know! There are really many things to talk about OpsWorks (and WP) so just stay tuned! ;)</p>
<p>Cheers!</p>