---
ID: 236
post_title: 'WordPress and OpsWorks, &#8220;Pride and Prejudice&#8221;'
author: Gianluca
post_date: 2014-06-20 12:04:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/20/wordpress-and-opsworks-pride-and-prejudice-4/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/89353822489/wordpress-and-opsworks-pride-and-prejudice
tumblr_mikamayhem_id:
  - "89353822489"
---
Disclaimer: this is not a comprensive tutorial nor a step by step guide on how to set up a Wordpress site through Amazon OpsWorks.

I’m reporting only my personal experience and a few tips (borrowed form my colleagues) on how to write some recipes and how to use them to manage and deploy a Wordpress website via OpsWorks.

But don’t despair, this may become the first article of a series dedicated to Wordpress and OpsWorks.

Ok, let’s begin.

<!--more-->

Amazon OpsWorks is “simply” and briefly a system (or better a dashboard) that aims to help developers (or better DevOps) to manage web applications in a resilient way, ready to scale out when needed. Obviously there is a lot more: take a look <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/welcome.html" target="_blank">here</a>.

OpsWorks relies on Chef, a tool that allows DevOps to configure and manage machines through Ruby scripts called recipes. These recipes are grouped in cookbooks. Just like for OpsWorks, obviously there is a lot more: take a look <a href="http://docs.opscode.com/" target="_blank">here</a>.<a href="http://docs.opscode.com/" target="_blank">
</a>

Now, in order to set up Wordpress on OpsWorks you need to have access to it. <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-sign.html" target="_blank">Here</a>’s a guide for this.

After that you need to create a Stack, a PHP Layer, and an RDS Layer for the database. You can also set up a Mysql Layer but I used RDS because it was simpler and it seems to be the proper way to do this.

Then you need to create an Instance and a PHP Application.

After you’ve properly configured all the above entities (i.e. Stack, Layers, Instance and Application) you’re ready to set up you recipes.

As far as now I’ve set up a few recipes to properly deploy the Wordpress code and optionally the database.

First tip: to properly write a good recipe you need to know where the information of the deployed app resides in the JSON passed to you from OpsWorks.

Yes, OpsWorks handles all the configuration with a big <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-json.html" target="_blank">JSON</a> that you can access from your recipes.

Besides, you also need to know how to override or add custom information (e.g. salts in wp-config.php). This can be done in <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/customizing.html" target="_blank">some different ways</a>. Among these, <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-json-override.html" target="_blank">custom JSONs</a> are particularly useful.

The first recipe that handles the creation of the wp-config.php file, for example, needs to know some information related to the database. These information are located under the deploy key and are automatically given to you by OpsWorks during the deploy:
<pre><code> 
{
    ...
    "opsworks": {
    ...
    },
    "deploy": {
    "application_name_1": {
      ...,
      "database": {
        "reconnect": true,
        "database": "simplephpapp",
        "host": null,
        "adapter": "mysql",
        "data_source_provider": "stack",
        "password": "d1zethv0sm",
        "port": 3306,
        "username": "root"
      },
      ...
  }
</code></pre>
If you want to override them or add some other info you can write a custom JSON and give it to OpsWorks when you deploy the app or  encapsulate them in a little bigger JSON that should be put inside the Stack configuration.
The bigger JSON will have this strucure:
<pre><code>
{
  "deploy" : {
    "application_name_1" : {
      "adapt_database" :
      {
        "string" : "...",
        "to" : "...",
        "replace" : "..."
      },
      "keys" : {
        "auth_key"         : "...",
        "secure_auth_key"  : "...",
        "logged_in_key"    : "...",
        "nonce_key"        : "...",
        "auth_salt"        : "...",
        "secure_auth_salt" : "...",
        "logged_in_salt"   : "...",
        "nonce_salt"       : "..."
      },
      "w3_total_cache" : false,
      "force_secure_logins" : false,
      "aws_access_key_id" : "...",
      "aws_secret_access_key" : "...",
      "aws_s3_bucket" : "...",
      "cron_url" : "http://your.domain/wp-cron.php?doing_wp_cron"
    },
    "application_name_2" : {
        ...
        ...
    },
    ...
  }
}
</code></pre>
As you can see the info is placed under the specific application configuration (i.e. under the application_name_1 key) that is populated by OpsWorks during the deploy phase (i.e. the deploy key). So you’ll find them right besides the other info handled directly by OpsWorks.

Don’t worry, Opsworks will handle the proper merging or overriding of his and your custom fields.

Obviously you can specify as many configuration fields as you want for any application that you want to deploy on the specific stack that you are customizing.

For the second recipe…well…stay in touch, I’ll write about it in my next article ;)

Cheers!