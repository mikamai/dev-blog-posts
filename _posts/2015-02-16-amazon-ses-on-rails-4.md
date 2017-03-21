---
ID: 155
post_title: Amazon SES on Rails 4
author: Emanuel Carnevale
post_date: 2015-02-16 15:38:47
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/16/amazon-ses-on-rails-4/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/111182251049/amazon-ses-on-rails-4
tumblr_mikamayhem_id:
  - "111182251049"
---
<p>Last week we were launching a new website hosted on Amazon AWS and we had the need to setup its mailers.</p>

<p>After having requested the removal of the limits and be granted Production access (the process can take days, so be farsighted and ask it in advance), we verified the domain and moved to set the mailer up.</p>

<p>A quick google search gave us tons of suggestions, but the smtp configuration we found didn&rsquo;t work and we wanted a simple solution that wouldn&rsquo;t involve the installation of a couple of gems only to use a smtp mailer.</p>

<p>Here is the configuration we found to work flawlessly, note the port we use: the examples we tried with authentication :login and port 465 don&rsquo;t work.</p>