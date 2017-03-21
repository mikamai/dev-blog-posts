---
ID: 3
post_title: 'Swift: How to send SOAP requests with AlamoFire 4.0'
author: Emanuel Carnevale
post_date: 2016-12-16 16:50:10
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/12/16/swift-how-to-send-soap-requests-with-alamofire/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/154552415259/swift-how-to-send-soap-requests-with-alamofire
tumblr_mikamayhem_id:
  - "154552415259"
---
<p>We do love our JSON APIs, but sometimes we need to work and talk with the Dark Side and perform SOAP requests exchanging XML responses with a server far far away.</p>

<p>In our Objective-C projects we grew fond of AFNetworking, but having switched to Swift (and Swift 3 nonetheless) we had to rely on the new <a href="https://github.com/Alamofire/Alamofire">Alamofire 4.0</a>.</p>

<p>Why new if it is a 4.0 version?</p>

<p>Alamofire 4.0 brings a big rewrite and some breaking changes, needed to address the new Swift 3 syntax and to ease some things  up. The breaking changes involve how to make a request and how to encode our parameters, key aspects if you want to talk with a SOAP server. The changes have been introduced only recently and it seems few people had the same problem of having POST requests on a legacy SOAP server, so the literature on the web is quite scarse and this post wants to fill this gap.</p>

<p>As with everything, the solution when found, is quite simple.</p>

<p>At line #6 we set the headers up, the <code>SOAPAction</code> one is quite important, since without it nothing worked. As always, YMMV, but you should find the correct value inside your WSDL of reference.</p>

<p>At line #16 we setup the request, the <code>encoding</code> parameter is the key of this post: to encode the request we write an extension to <code>String</code>:</p>

<p>In this extension we specify the format for a correct header as requested by our SOAP server and serve it during every request.</p>

<p>And we are ready to go.</p>

<p>Then, if you need to parse the XML response (otherwise why talk with the server to begin with?), we suggest you to use the XMLHash library of <a href="https://github.com/SwiftyJSON/SwiftyJSON">SwiftyJSON</a> inspiration.</p>

<p>We hope this little bit of information will save you some research time.</p>