---
ID: 247
post_title: Using SVG gradient masks with D3.js
author: Ivan Prignano
post_date: 2014-05-23 09:10:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/23/using-svg-gradient-masks-with-d3js/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/86583300944/using-svg-gradient-masks-with-d3js
tumblr_mikamayhem_id:
  - "86583300944"
---
<p>If you&rsquo;re reading this you&rsquo;ll probably know what<strong> <a href="http://d3js.org/" target="_blank">D3.js</a></strong> is. For anyone that doesn&rsquo;t, D3.js is a great piece of JavaScript software that allows you to do some <a href="https://github.com/mbostock/d3/wiki/Gallery" target="_blank">pretty awesome stuff</a> – generally involving tons of data. It makes great use of HTML, SVG and CSS – nothing more. Neat, right?</p>
<p>The majority of D3.js use cases will obviously involve data representation. But what is data representation without some eyecandy goodness? You <em>need</em> that!</p>
<p>Well, in our last project we extensively used D3.js to show a lot of data, but we also needed some related graphic elements that users could zoom in and out without losing render quality or look weird. So, we decided to implement these <strong>directly in our SVG</strong>, using our beloved D3.js.</p>
<h3>The design</h3>
<p>Our design looked something like that (a picture is worth a thousand words):</p>
<p><img alt="image" src="http://68.media.tumblr.com/c3892c9f1826516364d302950f5d4d8d/tumblr_inline_n60rcmUNhw1riz3e2.png" /></p>

<p><br />It was clear we needed some sweet gradient masking using SVGs. So, how could we do that? It turned out it&rsquo;s very simple.</p>
<h3>The solution</h3>
<p>Starting from a placeholder div for our SVG (let&rsquo;s say with the id &lsquo;svg-container&rsquo;), we appended the <strong>two necessary elements</strong> for our mask in a &lt;defs&gt; tag inside the SVG:<strong> the vertical gradient</strong> (alpha-opaque-alpha) and the <strong>actual mask</strong>.</p>

<pre class="javascript">// Handy vars for our banner dimensions
var bannerWidth  = 1500,
    bannerHeight = 500;

// Let's append the SVG element.
d3.select('#svg-container')
    .append('svg:svg').attr('class', 'my-supah-svg')
    .append('svg:defs')
    .call(function (defs) {
      // Appending the gradient
      defs.append('svg:linearGradient')
        .attr('x1', '0')
        .attr('x2', '0')
        .attr('y1', '0')
        .attr('y2', '1')
        .attr('id', 'supah-gradient')
        .call(function (gradient) {
          gradient.append('svg:stop')
            .attr('offset', '0%')
            .attr('stop-color', 'white')
            .attr('stop-opacity', '0');
          gradient.append('svg:stop')
            .attr('offset', '50%')
            .attr('stop-color', 'white')
            .attr('stop-opacity', '1');
          gradient.append('svg:stop')
            .attr('offset', '100%')
            .attr('stop-color', 'white')
            .attr('stop-opacity', '0');
        });
      // Appending the mask
      defs.append('svg:mask')
        .attr('id', 'gradient-mask')
        .attr('width', bannerWidth)
        .attr('height', bannerHeight)
        .attr('x', 0)
        .attr('y', 0)
        // This is the rect that will actually
        // do the gradient masking job: as you can see it's
        // filled with the linear gradient.
        .call(function(mask) {
          mask.append('svg:rect')
            .attr('width', bannerWidth)
            .attr('height', bannerHeight)
            .attr('fill', 'url(#supah-gradient)')
        });
    })
</pre>
<p>These two elements formed our mask element. We just needed to apply it on our desired element – below there&rsquo;s an example &lt;rect&gt; element with our mask on:</p>
<pre class="javascript">// Masking on a rect filled with a solid color
d3.select('.my-supah-svg').append('svg:rect')
  .attr('class', 'final-rect')
  .attr('width', bannerWidth)
  .attr('height', bannerHeight)
  .attr('x', 0)
  .attr('y', 0)
  .attr('fill', 'salmon')
  .attr('mask', 'url(#gradient-mask)');</pre>
<p>We also needed to apply the masking on <strong>image elements</strong> – here&rsquo;s another example for it:</p>
<pre class="javascript">// Masking on an actual image
d3.select('.my-supah-svg').append('svg:image')
  .attr('class', 'final-bg')
  .attr('xlink:href', 'http://lorempixel.com/output/technics-q-g-1500-500-4.jpg')
  .attr('width', bannerWidth)
  .attr('height', bannerHeight)
  .attr('x', 0)
  .attr('y', 0)
  .attr('mask', 'url(#gradient-mask)');</pre>
<p>Easy peasy!</p>
<p>Just a little heads up: the mask element <strong>must have the same dimensions and position of the element being applied on</strong>. In our examples we set it to a fixed width/height and a 0 X/Y.</p>
<h3>Conclusion</h3>
<p>D3.js is a great, fun tool that can also be used for presentational purposes other than for serious and austere data graphs. Have fun!</p>
<p>Oh, and here you have the <a href="http://codepen.io/iprignano/pen/xKEju" target="_blank">interactive demo on codepen</a>!</p>