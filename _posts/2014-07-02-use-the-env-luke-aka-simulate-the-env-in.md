---
ID: 231
post_title: 'Use the ENV Luke! aka: simulate the ENV in OpsWorks using Chef and Dotenv'
author: Massimo Ronca
post_date: 2014-07-02 16:39:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/02/use-the-env-luke-aka-simulate-the-env-in/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/90567083464/use-the-env-luke-aka-simulate-the-env-in
tumblr_mikamayhem_id:
  - "90567083464"
---
<p>OpsWorks is an impressive piece of software, but sometimes it lacks the comfort zone we developers love so much.<br />
One feature that I really miss, is the ability to configure my application using ENV variables.<br />
I&rsquo;m not aware of any easy way (ie: Heroku like) to create environment variables in OpsWorks that the application can consume.     </p>

<p>Fortunately OpsWorks is based on Chef and can be customized in any way you want.<br />
Be warned, it&rsquo;s not always an easy path, basic Chef knowledge is required, the interface is quite convoluted, but in the end it gets the job done.<br /><br />
So, we were saying <strong>environment</strong>! <br />
We know environment is not supported in OpsWorks, so what we really need is to simulate it in some way. <br />
A common solution among Rails developers, is the <a href="https://github.com/bkeepers/dotenv">Dotenv gem</a> which load the file <code>.env</code> in the root of you app and create the correspondent  keys in the <code>ENV</code> object.    </p>

<p>I will assume you already have created a Stack in OpsWorks with a Rails application layer.   </p>

<!--more-->

<h2>Rails side</h2>

<p>Add the Dotenv gem to your <code>Gemfile</code>    </p>
    <pre><code class="ruby">gem 'dotenv-rails'
</code></pre>

<p>run bundle</p>
    <pre><code class="bash">$ bundle
</code></pre>

<p>as soon as possible, load the environment through Dotenv   </p>
    <pre><code class="ruby"># application.rb
require File.expand_path('../boot', __FILE__)
require 'dotenv'
Dotenv.load

</code></pre>

<p>push the new code to Github  </p>

<h2>OpsWorks side</h2>

<p>From the Stack dashboard click on <code>stack settings</code>, select yes for <code>Use custom Chef cookbooks</code>, chose Git as repository type and insert <code><a href="https://github.com/mikamai/opsworks-dotenv">https://github.com/mikamai/opsworks-dotenv</a></code> as the repository URL.<br />
If you wanna use a private repository, you also need to enter the SSH private key.<br />
Choose your branch (<code>master</code> in this case) and add the following JSON in the <code>Custom JSON</code> box  </p>
    <pre><code class="javascript">{
  "deploy":{
    "your_app_name":{
      "symlink_before_migrate":{
        ".env" : ".env"
      },
      "app_env": {
        "YOUR_ENV_KEY": "KEY_VALUE",
        "ANOTHER_ENV_KEY": "SECOND_VALUE"
      }
    }
  }
}

</code></pre>

<p><strong>Do not forget</strong> the <code>symlink_before_migrate</code> key, it tells Chef to link the <code>.env</code> file created in the shared deploy folder into<br />
the current deploy folder, so that the Rails app can pick it up.<br /><br />
To retrieve the <code>your_app_name</code> value go to the Apps page in AWS console, click on the app name you want to configure and from there, copy the <code>short name</code> app property.   </p>

<p>The last step is to instruct Chef to run your recipe on every deploy. <br />
Click on the recipes link in the Rails layer section<br /><br /><img src="http://i.imgur.com/Oc2f8Ja.png" alt="OpsWorks layers" /></p>

<p>Add the custom recipe and click the plus icon (the recipe name is exaclty <code>rails::dotenv</code>)<br /><br /><img src="http://i.imgur.com/eC1E8sI.png" alt="Custom recipes" /></p>

<p>It should look like this<br /><br /><img src="http://i.imgur.com/t4XIA5i.png" alt="Custom recipes added" /></p>

<p>Click save in the lower right corner and update you custom cookbooks by clicking on <code>Stack</code> and then <code>Run command</code><br /><br /><img src="http://i.imgur.com/J1AYWgE.png" alt="Update Custom Cookbooks" /><br /><br />
This step must be performed every time a recipe in the custom cookbook is added or updated.<br /><br />
You can now deploy your app and enjoy your shiny new ENV.   </p>

<p><strong>TL;DR</strong>: add <code>Dotenv</code> gem, clone <code><a href="https://github.com/mikamai/opsworks-dotenv">https://github.com/mikamai/opsworks-dotenv</a></code>, add it as Custom Chef Cookbook, <br />
run <code>rails::dotenv</code>  recipe on every deploy, be happy!  </p>