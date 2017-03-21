---
ID: 16
post_title: 'A front-end tale: Google Places Library, jQuery promises and Coffeescript'
author: Gianluca
post_date: 2016-07-28 09:26:46
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/07/28/a-front-end-tale-google-places-library-jquery/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/148092225049/a-front-end-tale-google-places-library-jquery
tumblr_mikamayhem_id:
  - "148092225049"
---
When it comes to define my working position I often need to state (and clarify) the differences between design, frontend and backend development.
Given my academic background and my interests I can easily be considered more a backend developer than a frontend. I do not have a great “esthetic sense” and despite my knowledge of HTML5 and CSS (with all its SCSS, SASS, LESS and so on) I do not have the expertise of a “real frontend developer”.
Don’t get me wrong, I studied Javascript basics and I like to keep myself updated with its community news but still, it is not (at least for now ;P) my area of expertise.

<img src="http://www.commitstrip.com/wp-content/uploads/2014/12/Strip-Vengeance-de-codeur-650-finalenglish.jpg" alt="image" />

Nevertheless I like to solve problems independently on their nature and so if there is a “frontend problem” to tackle I’m always ready to dive in.

This is exactly what happened last week. 

<!--more-->

I was working on a backend story while two of my colleagues (a backend and a frontend dev) were handling another one related to the display of the nearest Points Of Interest (i.e. POI) of a given place on a (Google) Map.
I followed the evolution of their work because I never had the opportunity to try the <a href="https://developers.google.com/maps/documentation/javascript/places">Google Places Library</a> and I was thrilled to learn more about it. In particular the requirements requested to use the <a href="https://developers.google.com/maps/documentation/javascript/places#place_search_requests">Nearby Search API</a>.
I was also particularly lucky… Once I closed my story I was able to refactor and complete the POI story. The backend developer that was following it was indeed moved to another project to handle an urgent task.
Anyway, the main features required by the story were already implemented but the underlying solution wasn’t quite optimal.

The principal problem was the tangling of multiple AJAX calls with a big common callback whose aim was to store the results of the calls until a given condition was met and then trigger some calculations and UI adjustments.

When I heard about this kind of problem the first thing that jumped to my mind was: “Promises!”

Exactly like Javascript, I read about them and followed their evolution but, until last week, I never practically used them.
Given the limited scope of our problem and the fact that we already had jQuery included inside the project I decided to take a look at jQuery promises. I remembered reading something about their <a href="https://gist.github.com/domenic/3889970">not being compliant and even “broken”</a> but again, our need was kind of basic and so I gave them a try.

The first thing I end up reading was <a href="https://api.jquery.com/jquery.when/">$.when()</a> and right after that I dove into <a href="https://api.jquery.com/category/deferred-object/">DeferredObject</a>.
After a bit I identified one of the main problems regarding our use case: we didn’t knew in advance how many AJAX calls we had to make. Their number was actually dynamic!
We actually had to pass to the <code>$.when</code> function an arbitrary long array of functions, or better, promises.

Another more trivial problem was you do “transform” a function into a promise.
For this latter the solution was simply to wrap each AJAX call inside a <code>DeferredObject</code>:
<pre><code>...
nearbySearch: (googleTypeName) -&gt;
  deferred = new $.Deferred()
  @placesService.nearbySearch {
    location: @placeCoords
    radius: @nearbySearchRadius
    type: [ googleTypeName ]
  }, @storePoiPlaces(googleTypeName, deferred)
  deferred.promise()
...
</code></pre>
The AJAX call is actually <code>@placesService.nearbySearch</code> while the “promise wrapping” consists of <code>deferred = new $.Deferred()</code> and <code>deferred.promise()</code>.

<code>@placesService</code> is an attribute of a <code>MapDetail</code> object instance that stores an instance of <code>google.maps.places.PlacesService</code>. I didn’t mentioned it but all the original code was implemented in Coffeescript and encapsulated for simplicity inside a Coffeescript class that was named <code>MapDetail</code>.

<code>@placeCoords</code>, <code>@nearbySearchRadius</code> are also instance attributes of an instance of <code>MapDetail</code> while <code>@storePoiPlaces(googleTypeName, deferred)</code> is an instance method that represents a much more lighter and readable version of the initial common callback used by all the AJAX calls:
<pre><code>...
storePoiPlaces: (googleTypeName, deferred) -&gt;
  (places, status) =&gt;
    if status == google.maps.places.PlacesServiceStatus.OK
      @poiPlaces = @poiPlaces.concat places
    else
      # FIXME: remove or replace with something more appropriate to track Google API error statuses
      console.log googleTypeName, status
    deferred.resolve(places, status)
...
</code></pre>
To those unfamiliar with Coffeescript I highly recommend you to go check about the <a href="https://gist.github.com/meandmax/355b7433eb68b47540c5">difference of <code>-&gt;</code> and <code>=&gt;</code></a> ;)

The main concept behind <code>@storePoiPlaces</code> however is that it returns a function that has <code>this</code> (<code>@</code> in Coffeescript) bound to the instance of the <code>MapDetail</code> object. In this way we can access all the instance “functionalities”. Beside the <code>=&gt;</code> there is another peculiarity related to the <code>deferred</code> argument of <code>@storePoiPlaces</code>. It is indeed required to resolve the promises that use the function returned by <code>@storePoiPlaces</code> as callback.

Beware that the call <code>resolve</code> on the <code>deferred</code> requires exactly <code>places</code> and the <code>status</code> as arguments.
Now, here it is the complete part of <code>MapDetail</code> class or at least the part related to the fetch of the nearest places given a specific point on a map. I deliberately left out the part concerning the calculations and UI adjustments as I consider them simple data crunching and transformations functionalities.
<pre><code>class MapDetail
  constructor: (@mapSelector) -&gt;
    @$mapContainer      = $(@mapSelector)
    @poiTypes           = @$mapContainer.data 'map-poi-types'
    @numberOfPoisToShow = @$mapContainer.data('number-of-pois-to-show') - 1
    @googleTypesNames   = Object.keys @poiTypes
    @placeCoords        = new google.maps.LatLng @$mapContainer.data('lat'), @$mapContainer.data('lng')
    @map                = new google.maps.Map @$mapContainer[0], center: @placeCoords, zoom: 17, clickableIcons: false
    @poisBounds         = new google.maps.LatLngBounds()
    @placesService      = new google.maps.places.PlacesService @map
    @nearbySearchRadius = 1000
    @poiPlaces          = []
    @init()

  init: -&gt;
    @addPlaceMarker()
    @fetchPois() if @numberOfPoisToShow &gt;= 0

  addPlaceMarker: -&gt;
    new google.maps.Marker
      position: @placeCoords,
      map: @map,
      icon: "&lt;%= asset_path('selected_pin.svg') %&gt;"
    @poisBounds.extend @placeCoords

  fetchPois: -&gt;
    $.when(@nearbySearches()...).then =&gt; @addPoisMarkers()

  nearbySearches: -&gt;
    @googleTypesNames.map (googleTypeName) =&gt; @nearbySearch(googleTypeName)

  nearbySearch: (googleTypeName) -&gt;
    deferred = new $.Deferred()
    @placesService.nearbySearch {
      location: @placeCoords
      radius: @nearbySearchRadius
      type: [ googleTypeName ]
    }, @storePoiPlaces(googleTypeName, deferred)
    deferred.promise()

  storePoiPlaces: (googleTypeName, deferred) -&gt;
    (places, status) =&gt;
      if status == google.maps.places.PlacesServiceStatus.OK
        @poiPlaces = @poiPlaces.concat places
      else
        # FIXME: remove or replace with something more appropriate to track Google API error statuses
        console.log googleTypeName, status
      deferred.resolve(places, status)
  ...
</code></pre>
I know, there is actually a lot going on here but the principal take aways that I want to stress out in this brief post are:
<ul>
 	<li>how to wrap a function (in particular an instance method of a Coffeescript object) in a promise</li>
 	<li>how to correctly handle the <code>this</code> (<code>@</code> in Coffeescript) while wrapping the instance method in a promise and inside the AJAX callback</li>
 	<li>how to pass an arbitrary number of AJAX call (promises) and wait till their finish to perform some other stuff.
Regarding this last point the main part is definitely the body of the method <code>fetchPois</code> where I use the syntactic sugar of Coffeescript’s splat operator <a href="http://coffeescript.org/"><code>...</code></a> to pass to <code>$.when()</code> function the wanted array of promises.</li>
</ul>
I hope that this brief excursus may be of some help to the devs not so prone towards frontend stuff like myself ;)

To the next time!
Cheers!