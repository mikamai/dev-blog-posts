---
ID: 111
post_title: Remember to tune spring before giving up
author: Nicola Racco
post_date: 2015-06-15 13:15:13
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/06/15/remember-to-tune-spring-before-giving-up/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/121587062354/remember-to-tune-spring-before-giving-up
tumblr_mikamayhem_id:
  - "121587062354"
---
Spring is a greatest addition to the Ruby on Rails standard distribution. but if you haven’t read its readme carefully you could find yourself with weird issues. It’s time to know how it works and how you can prevent it to ruining your day.
<!--more-->

Spring is a gem required by default in development (don’t try to use it in production). Its aim is to speed up development preloading the rails environment and reusing it every time you start a rails server, a rails console, or a rake task, and so on.

It comes with some CLI commands:

- `spring status` tells you what (and when) processes spring have been spawn
- `spring stop` stops all spawned processes

You don't need any start/restart command: the first time you’ll start a `rails server`, spring will load the rails environment in memory.

While a process is spawn, spring will look for changes in models, controllers and all others rails related files. When a change is detected, the process is restarted.

The weirdest issues are due precisely to this feature: spring doesn’t watch every file in your rails app, so a change to `config/application.rb` will cause a restart, but a change to `config/application.yml` will not cause anything.

Say you're using [figaro](https://github.com/laserlemon/figaro) to handle the app configuration; you're continuing to touch and change the `config/application.yml`, and the app continues to show you the old value. In these cases you should take care of modifying `config/spring.rb` (create it if it doesn’t exist) adding something like:

```ruby
Spring.watch &quot;config/application.yml&quot;
```

Then, you can restart all spawn processes with

```bash
touch config/application.rb
```

Spring will notice that the file has been changed and will restart all processes, and the weird issues will be gone.