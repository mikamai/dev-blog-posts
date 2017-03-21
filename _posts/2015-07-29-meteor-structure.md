---
ID: 93
post_title: Meteor structure
author: Elia Morini
post_date: 2015-07-29 13:19:36
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/29/meteor-structure/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/125342343514/meteor-structure
tumblr_mikamayhem_id:
  - "125342343514"
---
<p>Coming from structured framework like Rails and Symphony, Meteor does not enforce a fixed structure, if fact you can write your files anywhere, still there are conventions like the following one:</p>
<!--more-->


<pre><code>/client
Files in this folder are only served to the client. This is a good place to keep your HTML, CSS, and UI-related JavaScript code.

/server
Files in this directory are only used by the server, they are never  sent to the client. Use /server to store source files with sensitive logic or data that should not be visible to the client.

/public
Files in /public are served to the client on an as-is basis. Use this directory  to store assets such as images. For example, if you have an image located at /public/ background.png, you can include it in your HTML with img src='/background.png'/ or in your CSS with background-image:
url(/background.png). Note that /public is not part of the image URL.

/private
These files can only be accessed by server code through Assets API and are not accessible to the client.

"Meteor official documentation"
</code></pre>

<p>An alternative is to rely on something like <a href="https://github.com/iron-meteor/iron-cli">Iron Generator</a>. This CLI (npm package) creates your desired application structures (i.e. folders and files), by providing at the same time proper generators to create scaffolds for your models, controllers and views.</p>

<p><a href="https://scotch.io/tutorials/how-to-speed-up-meteor-development-with-scaffolding-and-automatic-form-generation">Here</a> there is an example of usage of the above mentioned Iron Generator.</p>