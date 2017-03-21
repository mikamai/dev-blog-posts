---
ID: 6
post_title: How to create and customize a slider
author: Matteo Revelli
post_date: 2016-11-30 15:42:32
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/11/30/how-to-create-and-customize-a-slider/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/153864723504/how-to-create-and-customize-a-slider
tumblr_mikamayhem_id:
  - "153864723504"
---
A few days ago I had to work as a frontend developer because I needed to create some html static pages for mobile devices. One requirement was to have a scrollable component that mimics a calendar interface.

Building a slider from scratch takes a long time for sure, so I decided to use Swiper, an open-source slider compatible with most browsers and platforms.

I chose Swiper among many other candidates because it can be easily installed via Bower, which is currently one of the most popular package manager tools.<!--more-->

To install Swiper launch:
<pre><code>$ bower install swiper</code></pre>
If you want to save new dependencies to your bower.json, run:
<pre><code>$ bower install swiper -- save</code></pre>
Now we need to include the generated CSS and JS files in our website:
<pre><code>&lt;link rel="stylesheet" href="path/to/swiper.css"&gt;
&lt;script src="path/to/swiper.js"&gt;&lt;/script&gt;</code></pre>
This is the basic html structure of a Swiper slider and we can start from it to build our own:
<pre><code>&lt;!-- Slider main container --&gt;
&lt;div class="swiper-container"&gt;
    
    &lt;!-- Additional required wrapper --&gt;
    &lt;div class="swiper-wrapper"&gt;
        &lt;!-- Slides --&gt;
        &lt;div class="swiper-slide"&gt;Slide 1&lt;/div&gt;
        &lt;div class="swiper-slide"&gt;Slide 2&lt;/div&gt;
        &lt;div class="swiper-slide"&gt;Slide 3&lt;/div&gt;
        ...
    &lt;/div&gt;
    
    &lt;!-- If we need pagination --&gt;
    &lt;div class="swiper-pagination"&gt;&lt;/div&gt;    
    
    &lt;!-- If we need navigation buttons --&gt;
    &lt;div class="swiper-button-prev"&gt;&lt;/div&gt;
    &lt;div class="swiper-button-next"&gt;&lt;/div&gt;    
    
    &lt;!-- If we need scrollbar --&gt;
    &lt;div class="swiper-scrollbar"&gt;&lt;/div&gt;
&lt;/div&gt;</code></pre>
I replicated the structure of the sliders within my html file, adapting it to my needs:
<pre><code>&lt;div class="swiper-container"&gt;
  
  &lt;!-- Slides wrapper --&gt;
  &lt;div class="swiper-wrapper"&gt;    
    
    &lt;!-- Slides --&gt;
    &lt;div class="calendar swiper-slide"&gt;

      &lt;!-- bootstrap list --&gt;
      &lt;div class="list-group"&gt;
      &lt;span class="list-group-item head-list"&gt;9/10&lt;/span&gt;
        &lt;div class="arrow-down"&gt;&lt;/div&gt;
        &lt;a href="#" class="list-group-item"&gt;Book time slot&lt;/a&gt;
        &lt;a href="#" class="list-group-item disabled"&gt;Not available&lt;/a&gt;
        ...
      &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;</code></pre>
The next step is to initialize the slider and its behaviour via javascript:
<pre><code>  var mySwiper = new Swiper ('.swiper-container', {
    pagination: '.swiper-pagination',
    slidesPerView: 2,
    centeredSlides: true,
    paginationClickable: true,
    spaceBetween: 0,
    initialSlide: 0
  })        
</code></pre>
The last step is to add some styles to the slider in order to make it more usable and nicer. I wrote some SASS styles which will be compiled by Gulp in standard CSS. To stylize the main div that contains all the slides, I added the following styles:
<pre><code>// configure slider
.full-page
  padding: 0

.swiper-container
  height: auto
  width: 100%

.swiper-slide
  text-align: center
  background: #fff
  width: 60%
  // Center slide text vertically
  display: -webkit-box
  display: -ms-flexbox
  display: -webkit-flex
  display: flex
  -webkit-box-pack: center
  -ms-flex-pack: center
  -webkit-justify-content: center
  justify-content: center
  -webkit-box-align: center
  -ms-flex-align: center
  -webkit-align-items: center
  align-items: center

.list-group
  width: 100%

.calendar
  margin-top: 20px
  text-align: center</code></pre>
These are the styles for the left fixed sidebar:
<pre><code>// fixed left side-bar
.time-side-bar
  width: 25%
  display: block
  position: absolute
  z-index: 2
  margin-top: 20px
  color: gray</code></pre>
In order to customize the presentation of the active element of the slider I used mostly the “swiper-slider-active” class. This class is added automatically by the library on the selected slide:
<pre><code>// active element
.swiper-slide-active
  .list-group
    .list-group-item
      background: rgba(3, 169, 244, 0.21)

.list-group-item
  border-radius: 0
  border: 0
  border-bottom: 1px solid #d4d4d4
  margin-bottom: 0a

.list-group-item:hover
  color: #03a9f4
  background-color: #fff

.list-group-item.disabled
    background: #fff

.swiper-slide-active
  .list-group
    .list-group-item:first-child
      background: #03a9f4

.swiper-slide-active
  .list-group
    .list-group-item:nth-child(2)
      padding-top: 20px

// list first element (header)
.list-group
  .list-group-item:first-child
    height: 45px
    background: #bfbfbf
    color: #fff
    border-bottom: 0
    font-size: 15px

// arrow for active element
.arrow-down
  position: absolute
  width: 0
  height: 0
  border-left: 93px solid transparent
  border-right: 93px solid transparent
  border-top: 10px solid #03a9f4
</code></pre>
This is the final result:
<figure class="tmblr-full"><img src="http://68.media.tumblr.com/41002717bda7c597f60179a4bd747d33/tumblr_inline_ohgbhmS3Iq1u6zrof_540.gif" alt="image" /></figure>
In my opinion Swiper has proved to be an extremely versatile and powerful tool. It provides a long list of available parameters that make easier to achieve different goals. Moreover, there are many available demos from which you can get some ideas and realize that the customization possibilities are endless.

Just go to this <a href="http://idangero.us/swiper/#.WD6xS6LhCRs">link</a> and take what you need to start building your slider.