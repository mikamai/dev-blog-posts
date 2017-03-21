---
ID: 49
post_title: Jenkins and Docker on OpsWorks
author: Diomede Tripicchio
post_date: 2016-02-18 15:26:13
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/02/18/jenkins-and-docker-on-opsworks/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/139545098284/jenkins-and-docker-on-opsworks
tumblr_mikamayhem_id:
  - "139545098284"
---
<p>Hi guys! Today I want to talk about a WIP task. So keep that in mind as you read this article, thanks :D</p>

<p>I have to set up a system with <a href="https://jenkins-ci.org/">Jenkins</a> that runs its builds inside separate Docker containers.</p>
<!--more-->

<p>First of all, I have to set up this environment on OpsWorks, so to keep it simple, for now, I do everything on a single instance that takes care of all the tasks.</p>

<h2>Gathering some recipes</h2>

<p>To archive this objective I need a couple of recipes from different cookbooks, in detail:</p>

<ul><li><a href="https://github.com/intinig/opsworks-docker/">Docker Cookbook</a> from this I use <code>docker::install</code> obviously to install docker, and <code>docker::registries</code> to login into a remote docker register such as Quay.io.</li>
<li><a href="https://github.com/chef-cookbooks/jenkins">Jenkins Cookbook</a> from this I use only <code>jenkins::master</code> to have a working jenkins installation for a master node.</li>
</ul><p>In order to use these recipes I have created a custom cookbook that you can find here: <a href="https://github.com/oeN/jenkins-cookbooks">My Cookbook</a></p>

<p>Inside the repo you can find also an override of a configuration template file. I need this ovveride only for defining the Jenkins user from the stack settings.</p>

<h2>Setup</h2>

<p>First of all I’ve created a Stack on OpsWorks and added a custom layer.
In the “Custom Chef Recipes” section I have used <a href="https://github.com/oeN/jenkins-cookbooks">My Cookbook</a> as repository URL an then in the setup section I’ve added:</p>

<p><code>docker::install docker::registries jenkins::master</code></p>

<p>Before adding an instance to this layer I have created an EBS volume mounted on <code>/vol/jenkins</code> in this way I can keep my Jenkins data across machines creation and destruction cycle.</p>

<p>In order to do this I have to tell Jenkins where to save its data and I do this through Stack setting “Custom JSON”:</p>

<pre><code>{
  “jenkins”: {
    “master”: {
      “home”: “/vol/jenkins”,
      “user”: “root”,
      “port”: “80”
    }
  }
}
</code></pre>

<p>I have also defined which user runs Jenkins and on what port. For now, to keep it simple, the user is <code>root</code> and the port is <code>80</code>. I know, I should have created a <code>jenkins</code> user and set the right permissions. I could also run Jenkins on a different port and expose it through NGINX and proxy; but for now this works.</p>

<p>Now I add an instance to the layer and wait for the setup.</p>

<h2>Config Jenkins</h2>

<p>Once the instance setup is finished I have a machine with Docker and Jenkins up and running. Now it’s the time to configure Jenkins and allow it to use Docker.</p>

<p>To do this I just follow the “<a href="https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin">Jenkins - Docker Plugin</a>” documentation, with a little customization for the Docker slave starting from <code>evarga/jenkins-slave</code>. I allowed the <code>root</code> user permission to log in via ssh.</p>

<p>However at the time of this article the plugin documentation misses a little step which, unfortunately is essential for my configuration. Let me show you with an example.</p>

<p>I create a basic Jenkins job that runs a simple <code>uname -a</code>. After the setup of the Docker plugin I check the fields to run this job inside a Docker container:</p>

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/c401371802faa22c569c6c63bcbacd46/tumblr_inline_o2r1hzcPFn1qzktze_540.png" alt="image" /></figure><p>But building this job, I receive:</p>

<p><code>Linux nemesis . . . GNU/Linux</code></p>

<p>Where <i>nemesis</i> is the name of my instance on OpsWorks, so the command is executed on the master node and not in a Docker slave container.</p>

<p>I have found out I can solve this problem by defining a “Label Expression” to restrict where my job can run. Now if I insert the label of my docker container <code>docker-slave-test</code>.</p>

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/ee631f5f25d2e9b97d55d9c47a4c8ffb/tumblr_inline_o2r1hodxU01qzktze_540.png" alt="image" /></figure><p>Building it again, now I receive:</p>

<p><code>Linux 55310b7d848c . . . GNU/Linux</code></p>

<p>Where <i>55310b7d848c</i> is the ID of the Docker container that Jenkins has started and uses as slave.</p>

<p>So at last I have a machine with Jenkins that can run its builds on an on-the-fly created slave inside a Docker container.</p>

<p>The are some further steps to do before it can be considered complete, but for now it works and helps us with our CI process!</p>

<p>Bye and see you soon!</p>