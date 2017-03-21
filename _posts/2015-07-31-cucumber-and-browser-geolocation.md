---
ID: 92
post_title: Cucumber and Browser Geolocation
author: Emanuele Serafini
post_date: 2015-07-31 13:26:51
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/31/cucumber-and-browser-geolocation/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/125513492799/cucumber-and-browser-geolocation
tumblr_mikamayhem_id:
  - "125513492799"
---
<p>Several days ago I&rsquo;ve started writing an application with browser geolocation support. The main problem encountered has been that of testing user interaction with the geolocation service. It is not possible to interact with browser geolocation so you need a way to mock the geolocation api in test environment. I&rsquo;ve found, this old but still interesting, post wich resolve my problem.</p>
<p><a href="http://lucaguidi.com/2011/02/13/html5-geolocation-testing-with-cucumber.html">http://lucaguidi.com/2011/02/13/html5-geolocation-testing-with-cucumber.html</a></p>