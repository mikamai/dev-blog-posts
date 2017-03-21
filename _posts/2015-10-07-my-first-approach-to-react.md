---
ID: 78
post_title: My first approach to React
author: Elia Morini
post_date: 2015-10-07 10:01:50
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/07/my-first-approach-to-react/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/130675204764/my-first-approach-to-react
tumblr_mikamayhem_id:
  - "130675204764"
---
React is a innovative and modern web framework.
For me, these are the reasons for using React:
<h3>The component</h3>
React.js does not use Sun Shadow polymer as it gives you the opportunity to create your own components that you can later reuse, combine and nest together. React right now is one of the frameworks that are having more success, because components are easy to define and manipulate.
<!--more-->

<h3>Efficiency and virtual DOM</h3>
The React approach to handle rendering is:

1) Render the DOM at server side, the virtual DOM.

2) Compare virtual DOM to the browser/actual DOM and figure out differences.

3) Update only the selective/changed nodes of browser DOM instead of re-rendering the entire DOM.

This allows React to have impressive <a href="http://chrisharrington.github.io/demos/performance/">performance</a>.
<h3>Chrome React.js Tool</h3>
It makes application debugging much easier. After installing the extension, you will have a direct look into the DOM virtual just as if you were browsing in a DOM regular panel elements.
<h3>Easy to write javascript</h3>
React.js is really special. For ease of use the React team has created a special syntax “JSX” which mixes html and js in the same code snippet. This is not a requirement, but it helps us developers and it makes writing component very easy.
<h3>SEO Friendly</h3>
Many of JavaScript frameworks are not exactly search engine friendly.
React.js stands out from the crowd, because you can run React.js on the server, and the Sun Virtual will be created and returned to the browser as a normal web page.
<h3>React Project</h3>
The best project is <a href="https://instagram.com/">Instagram</a> created by React developers after the two companies (Facebook and Instagram), have joined forces.

<a href="https://web.whatsapp.com/">Whatsapp Web</a>

<a href="https://www.netflix.com/it/">Netflix</a>
<h3>And now, this is what it looks like:</h3>
<pre><code>&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;meta charset="UTF-8" /&gt;
    &lt;title&gt;Your First Component&lt;/title&gt;
    &lt;link rel="stylesheet" href="styles.css"/&gt;
    &lt;!-- Import React library --&gt;
    &lt;script src="https://fb.me/react-0.13.3.js"&gt;&lt;/script&gt;
    &lt;!-- &lt;script src="vendor/react/react.js"&gt;&lt;/script&gt; --&gt;
    &lt;script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"&gt;&lt;/script&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;div id="my-component"&gt;&lt;/div&gt;

    &lt;script type="text/babel"&gt;
      //Create Class for React Component
      var MyComponent = React.createClass ({
        render : function () {
          return (
            &lt;div className="list"&gt;
              &lt;h2&gt;List&lt;/h2&gt;
              &lt;ul className="wrap-list" &gt;
                &lt;li className="el" &gt; Link 1 &lt;/li&gt;
                &lt;li className="el" &gt; Link 2 &lt;/li&gt;
                &lt;li className="el" &gt; Link 3 &lt;/li&gt;
              &lt;/ul&gt;
            &lt;/div&gt;
          );
        }
      });

      React.render( &lt;MyComponent/&gt;, document.getElementById('my-component') );
    &lt;/script&gt;

  &lt;/body&gt;
&lt;/html&gt;
</code></pre>
Best regards