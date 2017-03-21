---
ID: 72
post_title: Metaprogramming in OpsWorks recipes
author: Gianluca
post_date: 2015-10-19 11:57:38
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/19/metaprogramming-in-opsworks-recipes/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/131484281754/metaprogramming-in-opsworks-recipes
tumblr_mikamayhem_id:
  - "131484281754"
---
<p>The last time I worked on OpsWorks I needed to implement a recipe to handle the setup and configuration of cron jobs.</p>

<p>Actually I had already written some code to handle this functionality in the past. However, the recipe I ended up with wasn’t particularly flexible.</p>

<!--more-->

<p>The main problem was that it didn’t allow the use of all the properties of the actual Chef <a href="https://docs.chef.io/resource_cron.html">cron</a> resource. It allowed only the setup of a command and its execution time (i.e. minute, the hour and the weekday). These properties were simply <strong>hardcoded</strong> inside the resource block.</p>

<p>With a little more experience by my side I decided to improve the recipe by letting it handle all the possible configurations offered by the cron Chef resource.</p>

<p>To do that I tried to rely on a little bit of metaprogramming and it turns out it works pretty well.</p>

<p>Here is the complete code of the recipe:</p>

<pre><code>#
# Cookbook Name:: utilities
# Recipe:: set_cron_jobs
#

node['deploy'].each do |application, deploy|

  next if deploy['application'] != application or !deploy['cron_jobs']

  deploy['cron_jobs'].each do |cron_job, settings|

    cron "#{application}_#{cron_job}" do
      only_if do
        Chef::Log.info "Set #{cron_job} cron..."
      end
      settings.each do |cron_job_setting, value|
        send cron_job_setting, value
      end
    end

  end

end
</code></pre>

<p>The fun part is right inside the block passed to the Chef <a href="https://docs.chef.io/resource_cron.html">cron</a> resource:</p>

<pre><code>...
settings.each do |cron_job_setting, value|
  send cron_job_setting, value
end
...
</code></pre>

<p>The <code>settings</code> is an Hash in which each key corresponds to a resource property specified inside the custom or the stack JSON. The values are the actual values passed to each property.</p>

<p>Here there is an example of the JSON structure expected by the recipe:</p>

<pre><code>...
'app_name' : {
  ...
  'cron_jobs' : {
    'cron_job_name' : {
      'command' : 'echo "hi there! :)"',
      'minute' : '0',
      'hour' : '0',
      'weekday' : '1',
      ...
    }
    ...
  },
  ...
}
...
</code></pre>

<p>What happens inside the block passed to <code>settings.each</code> method is a series of calls (or better messages sending) to the method identified by the value of the variable <code>cron_job_setting</code> with <code>value</code> as an argument. The messages are actually the properties of the cron resource.</p>

<p>With this little snippet of code I can handle all the possible properties of the resource by specifying them right inside the aforementioned JSONs.</p>

<p>The moral of the story is pretty straightforward: Chef resources are written in Ruby and you can leverage quite everything the language offers to you.</p>

<p>To conclude let me leave you with <a href="http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-extend-cron.html">this</a> just to remind you an important thing to consider when dealing with cron jobs on OpsWorks. This situation happens indeed pretty often:</p>

<blockquote>
<p>If your stack has multiple application servers, assigning cronjob.rb to the PHP App Server layer’s lifecycle events might not be an ideal approach. For example, the recipe runs on all of the layer’s instances, so you will receive multiple reports. A better approach is to use a custom layer to ensure that only one server sends a report.</p>
</blockquote>

<p>To the next time! ;)</p>

<p>Cheers.</p>