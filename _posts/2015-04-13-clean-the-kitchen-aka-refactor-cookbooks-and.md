---
ID: 136
post_title: >
  Clean the kitchen, aka refactor
  cookbooks and recipes (Part 2)
author: Gianluca
post_date: 2015-04-13 12:57:34
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/04/13/clean-the-kitchen-aka-refactor-cookbooks-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/116293060024/clean-the-kitchen-aka-refactor-cookbooks-and
tumblr_mikamayhem_id:
  - "116293060024"
---
<a href="http://dev.mikamai.com/post/111852114524/clean-the-kitchen-aka-refactor-cookbooks-and">Last time</a> I described a general approach I used to decrease the amount of duplication across the recipes I developed in different cookbooks. The goals of these latter is to handle the deploy of specific PHP application via OpsWorks.

The PHP application I’m talking about are based in particular on WordPress (WP) and PrestaShop (PS).

Last time I also cited a “mysterious” recipe upon which I relied for setting the default attributes to specialize its behavior.

<!--more-->

Actually there are two recipes that I use to this purpose but to describe the concept at their base I’m going to present here just one of them, i.e. <code>phpapps::set_config_files</code>.

If you read the last article of mine (if you don’t go for it! ;) ) you can guess that the above mentioned recipe resides in the general cookbook <code>phpapps</code>. Or better its implementation.

As a matter of fact the actual recipe that I select and associate to the deploy phase of a PHP layer in OpsWorks isn’t the one residing in the phpapps cookbook but the one defined inside the application specific cookbooks, i.e. <code>wordpress</code> and <code>prestashop</code>.

Inside these cookbooks there is indeed a <code>set_config_files</code> recipe that does nothing more than include the recipe of the “parent” cookbook <code>phpapps</code>.

The purpose of these recipes is to properly create the config files needed by WordPress and PrestaShop applications. These are respectively the <code>wp_config.php</code> and the <code>defines.inc.php</code> and <code>settings.inc.php</code>.

The creation of these files is handled, as shown below, inside the “parent” recipe <code>phpapps::set_config_files</code>:
<pre><code>
node['deploy'].each do |application, deploy|

  next if deploy['application_type'] != 'php' or deploy['application'] != application

  deploy['config_files'].to_h.each do |config_file, vars|

    template vars['file'] do
      only_if do
        Chef::Log.info "Set config file #{vars['file']}..."
      end
      cookbook deploy['type']
      source vars['template']
      mode 00660
      owner deploy['user']
      group deploy['group']
      variables vars.merge(deploy['environment'])
    end

  end

end
</code></pre>
The recipe does nothing more than loop on the <code>config_files</code> data structure and rely on the <code>template</code> resource provided by the Chef DSL.

There are two peculiarities here to note. The first one is represented by the <code>config_files</code> data structure. As many other data needed by the recipes it is stored as a default attribute inside the specific application cookbook. In particular it is implemented as an hash. Here there is an excerpt of the <code>default.rb</code> file representing the <code>config_files</code> inside the <code>wordpress</code> cookbook:
<pre><code>
node['deploy'].each do |application, deploy|
  ...
  default['deploy'][application]['config_files'] = {
    'wp_config' =&gt; {
      'file'                  =&gt; ::File.join(deploy['absolute_document_root'], 'wp-config.php'),
      'template'              =&gt; 'wp-config.php.erb',
      'db_name'               =&gt; deploy['database']['database'],
      'db_user'               =&gt; deploy['database']['username'],
      'db_password'           =&gt; deploy['database']['password'],
      'db_host'               =&gt; deploy['database']['host'],
      'auth_key'              =&gt; '',
      'secure_auth_key'       =&gt; '',
      'logged_in_key'         =&gt; '',
      'nonce_key'             =&gt; '',
      'auth_salt'             =&gt; '',
      'secure_auth_salt'      =&gt; '',
      'logged_in_salt'        =&gt; '',
      'nonce_salt'            =&gt; '',
      'table_prefix'          =&gt; 'wp_',
      'aws_access_key_id'     =&gt; nil,
      'aws_secret_access_key' =&gt; nil,
      'wplang'                =&gt; 'en-US',
      'wp_cache'              =&gt; false,
      'force_secure_logins'   =&gt; false,
      'disable_wp_cron'       =&gt; false,
      'wp_siteurl'            =&gt; domain,
      'wp_home'               =&gt; domain,
      'wp_allow_multisite'    =&gt; false,
      'wp_uploads'            =&gt; ::File.join('wp-content', 'uploads')
    }
  }  
  ...
end
...
</code></pre>
All the data needed to create the configuration files are handled inside the <code>default.rb</code> file and gets passed to the over mentioned <code>template</code> resource inside the general <code>phpapps::set_config_files</code> recipe.

The second peculiarity of the general <code>set_config_files</code> recipe is <code>cookbook deploy['type']</code>. This attribute is used to tell Chef (and OspWorks) to pickup the correct default attributes and template files, i.e. the ones related to the deployed application type.

The <code>cookbook deploy['type']</code> together with the default attributes allowed us to refactor the specific application <code>set_config_files</code> recipes by moving their common implementation inside a more general cookbook and recipe.

All the specific aspects related to the application configuration files are handled by the default attributes defined inside the specific application cookbook. The implementation of the recipe resides instead inside the general cookbook <code>phpapps</code>.

The key takehome message of this post is that it is possible to avoid code duplication across different cookbooks by leveraging upon the logic of the default attributes, the <code>coobook</code> attribute of the <code>template</code> resource and the inclusion of the recipes (<code>include cookbook_name::recipe</code>).

Next time I’ll discuss another recipe, conceptually similar to the one I briefly presented here, in which I use another feature of Chef to add a little bit of “magic” to the recipes ;)

Cheers!