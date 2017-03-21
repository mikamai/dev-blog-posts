---
ID: 312
post_title: >
  How to let your node.js applications
  talk to each other with the fork API
author: Nicola Racco
post_date: 2013-12-02 10:08:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/02/how-to-let-your-nodejs-applications-talk-to-each/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/68769403762/how-to-let-your-nodejs-applications-talk-to-each
tumblr_mikamayhem_id:
  - "68769403762"
---
Recently I had to setup an integration test environment for an express.js app written in coffeescript.
<!--more-->

In this case I could naturally start the server manually on a custom port, and then run the integration tests, but I'm lazy :) So I tried to load the server file in the specs to let them start it and stop it when they need it. [Here's the code](https://gist.github.com/nicolaracco/7747397).

But you cannot always start/stop the server this way. In my case this isn't a good option due to an issue in socket.io that is preventing the server to be stopped gracefully after each spec. After the first spec, I have to find a way to kill it, before proceeding to the next spec.

I can use [ChildProcess.spawn](http://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options) to spawn another instance, but how can I detect that the server has been correctly started?

Looking through the doc, I found [ChildProcess.fork](http://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options). It's not the "classic fork" you learn studying C, but it's a special case of spawn in which you have a builtin communication channel. This means **I can execute any other node application and let the two applications talk to each other**.

[Let me show you the code](https://gist.github.com/nicolaracco/7747356)