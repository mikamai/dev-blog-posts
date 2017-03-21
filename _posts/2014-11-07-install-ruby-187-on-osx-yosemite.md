---
ID: 189
post_title: Install Ruby 1.8.7 on OSX Yosemite
author: Matteo Zobbi
post_date: 2014-11-07 18:07:45
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/07/install-ruby-187-on-osx-yosemite/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/102021607469/install-ruby-187-on-osx-yosemite
tumblr_mikamayhem_id:
  - "102021607469"
---
With the release of OSX Yosemite, Ruby 1.8.7 is definitely gone from the default system configuration. I decided to make a clean install of Yosemite and since then I faced an issue while trying to install ruby 1.8.7 via RVM. The problem was a dependency on gcc46 which could not be resolved.<!--more-->

Actualy this seems the best workaround for RVM users on Yosemite:

1 Update homebrew/homebrew-versions tap
2 Install gcc48: brew install gcc48
3 Install Ruby 1.8.7.

These steps solve the issue because RVM uses for gcc48 instead of gcc46 when gcc48 is installed.