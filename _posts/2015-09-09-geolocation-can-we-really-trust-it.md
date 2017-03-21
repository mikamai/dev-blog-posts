---
ID: 91
post_title: Geolocation, can we really trust it?
author: Emanuele Serafini
post_date: 2015-09-09 13:39:12
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/09/09/geolocation-can-we-really-trust-it/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/128706685354/geolocation-can-we-really-trust-it
tumblr_mikamayhem_id:
  - "128706685354"
---
<p>Recently our team has been working on a web app that uses geolocation. We use geolocation to identify real-world geographic location of an user and allow them to check into an event.
A fundamental requirement is the accuracy of the user location. Luckily, HTML5 geolocation API is supported from most of mobile browsers, provides an interface to query device&rsquo;s location and its accuracy is quite high.</p>

<p>The app rewards the user for participating to located events, so we need to be sure that user geolocation is not forged.
The major problem is the high number of possibilities for forgeing geolocation, for example:</p>

<ul><li><a href="http://www.labnol.org/internet/geo-location/27878/">fake location in Chrome from Developers Tool</a></li>
<li><a href="https://addons.mozilla.org/firefox/addon/geolocater/">browser extension that allow one to fake location</a></li>
<li><a href="https://play.google.com/store/apps/details?id=com.incorporateapps.fakegps.fre&amp;hl=en">external apps that allow one to fake location</a></li>
</ul><p>We tried to solve the problem by matching the results of HTML5 geolocation with several geolocation services such as Maxmind GeoIP and Akamai Edgescape, without success. Using Maxmind we&rsquo;ve encountered problems with location accuracy. Maxmind offers an IP location service; when you query their database with IP an API returns a possible geolocation. We have found that some mobile carriers do not change the device IP address for several days, thus Maxmind database answers with the same result even if you keep traveling.</p>

<p>Akamai Edgescape, instead  gives you the location of the pop that served your request. The limit of this method is that the pop can be near the user location or not.</p>

<p>In the end we can say that requiring user&rsquo;s location and integrating geolocation service is quite simple; on the other hand making sure that the location is not forged is pretty hard.</p>