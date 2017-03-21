---
ID: 144
post_title: 'Docker and Opsworks: configuration basics'
author: Giovanni Intini
post_date: 2015-03-11 13:52:22
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/03/11/docker-and-opsworks-configuration-basics/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/113340319939/docker-and-opsworks-configuration-basics
tumblr_mikamayhem_id:
  - "113340319939"
---
<p><a href="http://dev.mikamai.com/post/110066104864/zero-downtime-deployments-with-docker-on-opsworks">Last time we talked about Docker and Opsworks</a> I left you with a huge JSON stack configuration, without too much info about how and why it worked.</p>

<p>Today we&rsquo;re gonna review it key by key and explore the capabilities offered by <a href="https://github.com/intinig/opsworks-docker">opsworks-docker</a>.</p>

<p>You can find the whole JSON in the <a href="http://dev.mikamai.com/post/110066104864/zero-downtime-deployments-with-docker-on-opsworks">original article</a>, let&rsquo;s break it down.</p>

<p>The top level keys are:</p>

<pre><code>
{
    "logrotate": {...},
    "docker": {...}
    "deploy": {...},
}
</code></pre>

<p><code>logrotate</code> is used to declare which container will have to be restarted when we truncate the docker logfiles. I forward all my logs to <a href="http://papertrailapp.com">papertrail</a> using <a href="https://github.com/gliderlabs/logspout">logspout</a>, so I declare it like this:</p>

<pre><code>
"logrotate": {
    "forwarder": "logspout0"
},
</code></pre>

<p>It&rsquo;s important to note the <code>0</code> at the end of the container name, we&rsquo;ll soon talk about it.</p>

<p><code>docker</code> contains information about custom docker configuration. Just private registries for now. It&rsquo;s tested only with <a href="http://quay.io">quay.io</a>, the one we use, but it should work out of the box (or with minimal modifications) for any other private registry.</p>

<pre><code>
"docker": {
    "registries": {
      "quay.io": {
        "password": "",
        "username": ""
    }
  }
}
</code></pre>

<p>It&rsquo;s pretty simple, you add in the <code>registries</code> key an hash for each registry you want to login to, and then you specify your username and your password.</p>

<p>We got rid of the simple configuration, let&rsquo;s now look at the big <code>deploy</code> hash. In this hash we describe all the applications we want to support, all their containers, and the relationships among those.</p>

<pre><code>
"deploy": {
    "app_name": {
        "data_volumes": {...info about my data only containers...},
        "containers": {...all my running containers...}
    }
}
</code></pre>

<p>In <code>data_volumes</code> we declare all the <a href="https://docs.docker.com/userguide/dockervolumes/#creating-and-mounting-a-data-volume-container">data volume containers</a>.</p>

<p>It&rsquo;s pretty self-explanatory:</p>

<pre><code>
"data_volumes": {
    "volume_container_one": ["/all", "/my/volumes"],
    "volume_container_two": ["/other/volumes"]
}
</code></pre>

<p>The example in my previous article was taken from a real world use case, where we use shared volumes on data only containers to share unix socket files between front-end webservers and application servers.</p>

<p><code>containers</code> holds information about the container graph we&rsquo;ll have running on our host machine.</p>

<pre><code>
"containers": []
</code></pre>

<p>The first difference you&rsquo;ll notice between <code>containers</code> and the rest of the configuration is that while everything else is an hash, <code>containers</code> is an array. Having it as an array was needed because we want to be sure in which order the containers will be deployed.</p>

<pre><code>
"containers": [
    {
        "an_example": {
            "deploy": "auto",
            "image": "quay.io/mikamai/an_example",
            "database": "true",
            "containers": "3",
            "startup_time": "60",
            "ports": ["80:80", "443:443"],
            "volumes_from": ["volume_container_one", "volume_container_two"],
            "entrypoint": "entrypoint.sh",
            "command": "bundle exec unicorn -c config/unicorn.rb -l /var/lib/haproxy/socks/%{app_name}.sock",
            "env": {
                "RANDOM_VAR": "foo"
            },
            "env_from": "another_container_name",
            "hostname": "%{opsworks}",
            "migration": "bundle exec rake db:migrate",
            "notifications": {
                "rollbar" : {
                    "access_token": "",
                    "env_var": "RACK_ENV",
                    "rev_var": "GIT_REVISION"
                }
            }
        }
    }
]
</code></pre>

<p><code>deploy</code> can be either <code>auto</code> or <code>manual</code> or <code>cron</code>. <code>auto</code> is the basic one. It means you want this container to be deployed every time you deploy on opsworks.</p>

<p><code>manual</code> is a little bit different. When deploying your application if the container is not running, opsworks will spin it up. Otherwise it will be left untouched. It&rsquo;s useful for frontend servers, proxies, and anything that must not be restarted everytime you want to deploy.</p>

<p><code>cron</code> is for containers that need to be run as cron jobs. The deploy recipe will set up a cron job that will docker run at specified intervals. <code>cron</code> containers need an extra key:</p>

<pre><code>
    "a_cron_example": {
        "deploy": "cron",
        ...
        "cron": {"minute": 59, "hour": "3", "weekday": "1"}
    }
</code></pre>

<p>Omitting one of <code>minute</code> <code>hour</code> or <code>weekday</code> has the same effect of specifying <code>*</code> for that value in crontab.</p>

<p><code>image</code> is self explanatory, it&rsquo;s the image name that docker will pull when deploying this container.</p>

<p><code>database</code> tells opsworks whether it should supply DB info to the container via environment variables:</p>

<pre><code>
DB_ADAPTER
DB_DATABASE
DB_HOST
DB_PASSWORD
DB_PORT
DB_RECONNECT
DB_USERNAME
</code></pre>

<p><code>containers</code> tells opsworks how many copies of the same container it should run. It&rsquo;s mainly used for application servers, and to achieve zero downtime deployments. This is the reason why at the beginning we declared our forwarder to be <code>logspout0</code>. An integer is appended at the end of every running container. For containers running only in one copy that would be <code>0</code>, but for multiple copies of the same containers we would have something like <code>app_server0</code>, <code>app_server1</code>, &hellip;, <code>app_serverX</code>.</p>

<p><code>startup_time</code> is used when <code>containers</code> is greater than <code>0</code>. Most application servers take a while to boot and load all of your application code. Timing how much time you app requires to boot, and specifying it in <code>startup_time</code> allows you to achieve zero downtime deployments, because at least one application server will always be up and answering requests.</p>

<p><code>ports</code>, <code>volumes_from</code>, <code>entrypoint</code>, <code>command</code> and <code>env</code> work exactely like their fig/commandline counterparts, with one exception, that is valid throughout the configuration JSON: every string is filtered via a lightweight templating system (it&rsquo;s actually built-in in the Ruby String class) that supports the following substitutions:</p>

<pre><code>
"%{public_ip}" # maps to the host instance public ip
"%{app_name}"  # name of the container + container_id, when containers &gt; 0
"%{opsworks}"  # hostname of the host instance
</code></pre>

<p><code>env_from</code> is a very useful helper. It allows you to copy the environemnt from another container definition. Immensely helpful when you have several containers that run off the same image, require the same environment, but are assigned different responsibilities.</p>

<pre><code>
"big_env_container": {
    ...
    "env": {...lots of keys...}
    ...
},
"leeching_container": {
    ...
    "env_from": "big_env_container"
    ...
}
</code></pre>

<p><code>hostname</code> allows you to override the default container hostname and force it to something else. I use it with the <code>%{opsworks}</code> substitution for containers like logspout that must represent the whole stack. Feel free to come up with other uses :)</p>

<p>Finally <code>migration</code> and <code>notifications</code> were added to help our Ruby on Rails development, but can be adapted to other uses.</p>

<p>If <code>migration</code> is present opsworks will try to run its value to migrate the database before launching a new container. The smartest among you will have noticed that you can easily use this hook to launch any other <code>docker run</code> command you need before spinning up your containers. Notice that when <code>containers</code> is greater than <code>0</code>, opsworks will only try to migrate the first one.</p>

<p><code>notifications</code> is used during deployment to notify an external service that a deploy just happened. We only support <a href="http://rollbar.com">rollbar</a> for now, via the <code>docker::notify_rollbar</code> recipe, but will gladly accept pull requests for other services (gitter, campfire, irc, you name it).</p>

<p><em>CONGRATULATIONS</em> if you read all of this. In the next article we&rsquo;ll talk about dockerizing your application and breaking it in small containers that you can juggle on your favorite docker host.</p>