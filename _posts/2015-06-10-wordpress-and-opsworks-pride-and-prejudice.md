---
ID: 113
post_title: >
  WordPress and OpsWorks, “Pride and
  Prejudice” (Part 4)
author: Gianluca
post_date: 2015-06-10 07:46:33
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/06/10/wordpress-and-opsworks-pride-and-prejudice/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/121174747524/wordpress-and-opsworks-pride-and-prejudice
tumblr_mikamayhem_id:
  - "121174747524"
---
4th part of the serie titled “Wordpress and OpsWorks“ (here you can find the <a href="http://dev.mikamai.com/post/89353822489/wordpress-and-opsworks-pride-and-prejudice">1st,</a> <a href="http://dev.mikamai.com/post/93782458919/wordpress-and-opsworks-pride-and-prejudice">2nd</a> and <a href="http://dev.mikamai.com/post/108731475409/wordpress-and-opsworks-pride-and-prejudice">3rd</a> part).

Despite the title, this time I’m not going to talk about an aspect of OpsWorks specifically related to Wordpress, since I’ll be presenting a brute force way to modify the default attributes merging behavior of OpsWorks.

<!--more-->

Recently I faced a problem with the cookbooks and the recipes I’d written to handle the deploy of specific PHP applications (Wordpress and PrestaShop). I needed to duplicate the information stored inside the default attributes from the coobooks to the custom JSONs (the one related to the stack as well as the one specifiable at deploy time).

My main problem was with the shared assets, i.e. the files and
directories that should be preserved across the deployments such as for example the <code>.htacces</code> or the <code>upload</code>
directory in a Wordpress application.

These assets are generally handled by creating proper symlinks pointing to a location external to the application root. In OpsWorks this is the <code>shared</code> directory, located at the same level as the <code>current</code> directory where the application is deployed.

In my cookbooks I stored the information needed to create the symlinks to the shared assets dir inside the <code>default attributes</code>. The problem was that whenever I specified additional shared assets inside the custom JSONs, those were completly overriding the default ones. In order to fix this I needed to specify all the shared assets inside the JSONs, not only the additional but also the default ones.

<a href="http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-attributes-precedence.html"> I found out</a> that this was due to how OpsWorks handles the merge of all the attributes during execution of the recipes.

Then it occurred to me that the way to solve the problem was to explicitely merge the attributes in the order I needed. To do this I relied on the Chef <code>deep_merge</code> implementation, <a href="https://github.com/chef/chef/blob/master/lib/chef/mixin/deep_merge.rb"><code>Chef::Mixin::DeepMerge.deep_merge()</code></a>.

Here I’ve extraced the code with the set up of the shared assets default attributes and the subsequent merging with the custom ones:
<pre>  ...
  default_deploy = deploy.dup
  custom_deploy = node.fetch('custom_deploy', {}).fetch(application, {}).dup

  default_attributes = {
    ...
    'shared_assets' =&gt; [
      { 'type' =&gt; 'file', 'from' =&gt; '.htaccess', 'to' =&gt; '.htaccess' },
      { 'type' =&gt; 'dir', 'from' =&gt; 'wp-content/uploads', 'to' =&gt; 'uploads' },
      { 'type' =&gt; 'dir', 'from' =&gt; 'wp-content/updraft', 'to' =&gt; 'updraft' },
      { 'type' =&gt; 'dir', 'from' =&gt; 'wp-content/cache', 'to' =&gt; 'cache' },
      { 'type' =&gt; 'dir', 'from' =&gt; 'wp-content/w3tc-config', 'to' =&gt; 'w3tc-config' },
    ],
    ...
  }

  default['deploy'][application] = Chef::Mixin::DeepMerge.deep_merge(
    default_deploy,
    Chef::Mixin::DeepMerge.deep_merge(
      default_attributes,
      custom_deploy
    )
  )
</pre>
The firsts two <a href="http://ruby-doc.org/core-2.2.2/Object.html#method-i-dup"><code>dup</code></a> call shouldn’t really be needed as the Chef <code>deep_merge</code> should already implicitely duplicate the hashes passed to it but, for some reason, it doesn’t work without them. I promise I’ll try to investigate this ;)

As to the last part, what it does is simply to deep merge the default_attributes specified inside the cookbook with the attributes specified inside the custom JSONs under the key <code>custom_deploy</code>. I needed a different key from the classical <code>deploy</code> to bypass the default OpsWorks merging.

The solution I’ve implemented can hardly be considered aligned with Chef’s and OpsWorks’s best practices but for now it does the job: it “dries out” the duplication of configuration and it simplifies the customization of the applications that need to be deployed.

I’m eager to know other opinions and possible solutions so don’t hesitate! Tweet them to me! ;)