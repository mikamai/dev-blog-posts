---
ID: 176
post_title: >
  Geolocalization galore for your rails
  app
author: Emanuel Carnevale
post_date: 2014-12-10 16:50:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/10/geolocalization-galore-for-your-rails-app/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/104850301399/geolocalization-galore-for-your-rails-app
tumblr_mikamayhem_id:
  - "104850301399"
---
<p>Having to geolocalize our users and not requiring a high level of accuracy, we decided to explore the IP geolocalization of the request instead of using the native API of the browser.</p>

<p>We found and tried a couple of gems:</p>

<ul><li><a href="https://github.com/jeroenj/geo_ip">geo_ip</a></li>
<li><a href="https://github.com/geokit/geokit-rails">geokit-rails</a></li>
</ul><h2>geo_ip</h2>

<p>Geo_ip retrieves a geolocation of an IP address using the ipinfodb.com api. You need to register for an API key, but after having obtained it and set into your app, you are set.</p>

<p>You just need to add in your controller</p>

<p><code>location = GeoIp.geolocation(request.remote_ip)</code></p>

<p>and in <code>location</code> you&rsquo;ll have a hash with the latitude and longitude, the Country, the City, and the zip code.</p>

<h2>geokit-rails</h2>

<p>Geokit-rails uses the geokit gem to offer a set of location-based features for your app, not only IP geolocalization. The list includes ActiveRecord distance-based queries, distance calculations and geocoding.</p>

<p>As for the IP geocoding, the code to use is the following:</p>

<p><code>location = Geocoders::MultiGeocoder.geocode(request.remote_ip)</code></p>

<p>the interesting part is that geokit is IP geocoder-agnostic since it provides a failover aong multiple IP geocoders and it queries them using the order you provide. Please, check the <a href="https://github.com/geokit/geokit/blob/master/README.markdown">Geokit documentation</a> to see the configuration options.</p>

<p>You can also write your own geocoder!</p>

<p>A special mention goes to the <a href="http://www.rubygeocoder.com">rubygeocoder</a> gem, which we didn&rsquo;t use for our app (it seems it doesn&rsquo;t behave well with Heroku, where our app was being served from) but it does look interesting nonetheless.</p>