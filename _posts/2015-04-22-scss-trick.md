---
ID: 133
post_title: Scss trick
author: Elia Morini
post_date: 2015-04-22 13:04:51
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/04/22/scss-trick/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/117081228374/scss-trick
tumblr_mikamayhem_id:
  - "117081228374"
---
<p>Lately I had to develop the front-end part of a mobile app, the mockup was challenging and I had to figure out a few tricks for CSS.</p><p>But before sharing these tricks with you, I&rsquo;ll make a short introduction to some SCSS functionalities.</p>
<!--more-->


<p><b>Variables</b></p>
<p>SCSS allows the use of variables, which can be declared prepending the $ sign.<br />What are exactly variables? They are basically a container where you can put a value to be used in different parts of the project.</p>

<p><b>Example</b></p>

<pre>
<code>$primary-color : #790f26;

$padding-default : 0 15px 10px;

.class {
  color   : $primary-color;
  padding : $padding-default;
}
</code></pre><p><b>Nesting</b></p><p>In SCSS it is possible to nest classes one within the other, just like we do in html.<br />We can&rsquo;t do this with plain CSS, SCSS allows us to follow the html structure closely and thus write our styles more easily.</p><p><b>Example</b></p><pre><code>&lt;div class="wrap-image"&gt;
  &lt;img src="img/home/service-background.jpg" class="img-istructions"&gt;
  &lt;p class="text-image"&gt;
  Example text &lt;span&gt;example&lt;/span&gt;
  &lt;/p&gt;
&lt;/div&gt;

.wrap-image {

  .img-istructions {
    ...
  }

  .text-image {
    ...

    span {
      ...
    }
  }
}
</code></pre><p><b>Mixin</b></p><p>A mixin allows for groups of CSS declarations that you want to reuse throughout the site.</p><p><b>Example</b></p><pre><code>@mixin translate ($x, $y) {
  -webkit-transform : translate($x, $y);
  -ms-transform     : translate($x, $y);
  transform         : translate($x, $y);
}

.class {
  @include translate(-50%, -50%);
}

//transform in css 
.class {
  -webkit-transform : translate(-50%, -50%);
  -ms-transform     : translate(-50%, -50%);
  transform         : translate(-50%, -50%);
}

@mixin transition($property, $duration, $easing) { 
  -webkit-transition: $property $duration $easing; 
  -moz-transition: $property $duration $easing; 
  -ms-transition: $property $duration $easing; 
  -o-transition: $property $duration $easing; 
  transition: $property $duration $easing; 
}

.class {
  @include transition(all, 0.3s, ease-in-out);
}

//transform in css  
.class {
  -webkit-transition: all 0.3s ease-in-out; 
  -moz-transition: all 0.3s ease-in-out; 
  -ms-transition: $all 0.3s ease-in-out; 
  -o-transition: all 0.3s ease-in-out; 
  transition: all 0.3s ease-in-out; 
}
</code></pre><p><b>Import</b></p><p>To keep the code more manageable SCSS has an option to import that serves to divide SCSS in more files, and then concatenate them to run a single file.</p><p><b>Example</b></p><pre><code>//File _menu.scss
nav {
  margin: 0;
  padding: 0;
}

//File application.scss

@import 'menu';

body {
  font             : 10px Arial, sans-serif;
  background-color : #303030;
}
</code></pre><p>Css file compiled</p><pre><code>nav {
  margin: 0;
  padding: 0;
}

body {
  font             : 10px Arial, sans-serif;
  background-color : #303030;
}
</code></pre><p>This was a brief introduction to SCSS.
Now some nifty Tricks.</p><p><b>Footer Sticky</b></p><p>First there is the question of the list footer Sticky</p><p><b>Example</b></p><figure class="tmblr-full"><img src="" alt="image" /></figure><p><b>SCSS</b></p><pre><code>$footer-height : 200px;

// Set container
.container {
  min-height : 100%;
  position   : relative;
  padding    : 0;

  // Set content
  .content {
    left           : 0;
    height         : 100%;
    width          : 100%;
    position       : relative;
    padding-bottom : $footer-height;
  }

  // Set footer
  .footer {
    position : absolute;
    bottom   : 0;
    left     : 0;
    width    : 100%;
    height   : $footer-height;
  }
}
</code></pre><p>The trick is to set the container in position absolute and min-height 100%,
for the container instead height is set to 100% and padding-bottom set height of the footer.</p><p><b>Arrow tab</b></p><p>The second trick  is to add the tab indicator to an arrow  as below</p><p><b>Example</b></p><figure class="tmblr-full"><img src="" alt="image" /></figure><p><b>SCSS</b></p><pre><code>$color-gray-smoke : #303030;

.arrow_box {
  position   : relative;
  background : $color-gray-smoke;

  &amp;:after {
    // position
    top              : 100%;
    left             : 50%;

    border           : solid transparent;
    content          : " ";
    height           : 0;
    width            : 0;
    position         : absolute;

    // Color
    border-color     : rgba(48, 48, 48, 0);
    border-top-color : $color-gray-smoke;

    // size arrow
    border-width     : 15px;
    margin-left      : -15px;
  }
}
</code></pre><p>In this project we used the Bootstrap framework, so the styles mentioned above should have the following classes to make sure that when you click on the arrow tab they get active.</p><pre><code>.nav-tabs {
  li.active {
    ...
    ...
    &amp;:after {
      ...
      ...
    }
  }
}
</code></pre><p><b>Data attribute</b></p><p>The third round is to crop images and add a number to the images after the text with the data attribute</p><p><b>Example</b></p><figure class="tmblr-full"><img src="" alt="image" /></figure><p><b>HTML</b></p><pre><code>&lt;div class="wrap-image"&gt;
  &lt;img src="img/home/service-background.jpg" class="col-xs-12 img-responsive img-istructions text-center"&gt;
  &lt;span class="text-center col-xs-12 text-box text-round-image" data-number=“1"&gt;
  Example text
  &lt;/span&gt;
&lt;/div&gt;
</code></pre><p><b>SCSS</b></p><pre><code>@mixin translate ($x, $y) {
  -webkit-transform : translate($x, $y);
  -ms-transform     : translate($x, $y);
  transform         : translate($x, $y);
}

.wrap-image {
  position   : relative;
  display    : table;
  width      : 300px;
  height     : auto;
  text-align : center;
  color      : #fff;

  .image {
    // Css cropped circle image
    border-radius: 50%;
  }

  .text-round-image {
    // border top and bottom
    border       : 1px solid #fff;
    border-style : solid none;

    display      : table-cell;
    font-size    : 2em;

    // position
    top          : 50%;
    position     : absolute;
    @include translate(-50%, 0);        

    width        : 290px;
  }

  .text-box:after {
    // data attribute
    content       : attr(data-number);

    // position
    position      : absolute;
    top           : 110%;
    text-align    : center;
    display       : block;
    left          : 50%;
    @include translate(-50%, 0);

    // Size
    height        : 38px;
    width         : 38px;

    // border
    border        : 2px solid #fff;
    border-radius : 50%;

    // Fonts
    font-size     : 1.6em;
    font-weight   : normal;
    line-height   : 1.1em;

  }
}
</code></pre><p><b>Vertical Align</b></p><p>The fourth is to center text vertically
</p><figure class="tmblr-full"><img src="" alt="image" /></figure><p><b>HTML</b></p><pre><code>&lt;div class="element"&gt;
  &lt;p&gt;Example text&lt;/p&gt;
&lt;/div&gt;
</code></pre><p><b>SCSS</b></p><pre><code>.element {
  display    : table;
  height     : 250px;
  width      : 250px;
  background : black;
  margin     : 20px;
  text-align : center;

  p {
    display        : table-cell;
    margin         : 0;
    color          : white;
    vertical-align : middle;
  }
}</code></pre><p>Thanks for reading. </p>