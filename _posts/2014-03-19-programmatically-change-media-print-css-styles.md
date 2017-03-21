---
ID: 270
post_title: >
  Programmatically change @media print css
  styles via javascript
author: Andrea Longhi
post_date: 2014-03-19 14:03:36
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/19/programmatically-change-media-print-css-styles/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/80066279513/programmatically-change-media-print-css-styles
tumblr_mikamayhem_id:
  - "80066279513"
---
<p>Sometimes our customers ask us to be able to print the webpages we build for them with specific styles, and most of the times you can get away with it by adding a separate css file that targets the print media. Rather straightforward.</p>
<p>What if the customer asks you to change print styles at runtime?</p>
<p>Well, it seems that this cannot be easily done, at least there is no official way of doing it: javascript doesn&rsquo;t allow you to change media specific styles, but only the global one.</p>
<p>This won&rsquo;t stop us: after banging my head against the wall for a while I came up with a simple yet effective trick: what if we programmatically change the text of a style tag with media attribute set to print? Will the browser see the change and apply it? Well, it seems so.</p>
<p>So, let&rsquo;s review a simple example that will allow us to choose the color of the printed page text according to the button we press.</p>
<p>Let&rsquo;s start from the page head tag:</p>
<pre><code>
&lt;head&gt;
  &lt;script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-2.1.0.js"&gt;&lt;/script&gt;
  &lt;style type="text/css"&gt;
    @media print {
      button {
        display: none;
      }
    }
    @media screen {
      .red {
        color: red;
      }
      .green {
        color: green;
      }
    }
  &lt;/style&gt;
&lt;/head&gt;
</code></pre>
<p>Nothing fancy here, we&rsquo;re just loading jquery and adding some styles that hide buttons on the printed page while coloring them on the screen.</p>
<p>Let&rsquo;s see the page body content:</p>
<pre><code>
&lt;body&gt;
  &lt;p&gt;hello world!&lt;/p&gt;
  &lt;button class="red"&gt;print this page in red&lt;/button&gt;
  &lt;button class="green"&gt;print this page in green&lt;/button&gt;
  &lt;style type="text/css" media="print" id="print"&gt;&lt;/style&gt;
&lt;/body&gt;
</code></pre>
<p>Here we have the page content, the buttons and an empty style tag that specifically targets print media, which is left empty and will be populated with our custom styles at runtime.</p>
<p>Finally, the javascript:</p>
<pre><code>
$(function() {
  var buttons     = $('button');
  var printStyle  = $('#print');
  var style;

  buttons.click(function() {
    style = 'p { color: ' + $(this).attr('class') + '};';
    printStyle.text(style);
    window.print();
  });
});
</code></pre>
<p>What happens here? When we click one of the buttons the code finds the color we want, it fills the empty style tag with the custom styles and then will open the browser integrated print interface. Simple yet effective.</p>
<p>The complete code can be seen in action in <a href="http://jsfiddle.net/spaghetticode/ZUUQz/">this fiddle</a>.</p>