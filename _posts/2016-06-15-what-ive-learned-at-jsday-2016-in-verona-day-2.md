---
ID: 24
post_title: 'What I&#8217;ve learned at jsday 2016 in Verona | Day 2'
author: Linkme s.r.l.
post_date: 2016-06-15 15:54:39
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/06/15/what-ive-learned-at-jsday-2016-in-verona-day-2/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/145964907404/what-ive-learned-at-jsday-2016-in-verona-day-2
tumblr_mikamayhem_id:
  - "145964907404"
---
<p>This is a follow-up from Day 1, if you are interested check it out at this <a href="http://dev.mikamai.com/post/145258317709/what-ive-learned-at-jsday-2016-in-verona">link</a></p>

<h2>Keynote: Shipping one of the largest Microsoft JavaScript applications (Visual Studio Code&rsquo;s story)</h2>

<p>Alexandru Dima is a Microsoft developer in the VS Code team and he did a very great talk about the history of their product.</p>

<p>It was really helpful to understand at least why Microsoft didn&rsquo;t just fork (and contribute on) <em>Atom editor</em> but they choose to create another editor with the same technology.</p>

<p>Basically they started to work on it in 2014 using <code>node-webkit</code> as a platform switching to <code>electron</code> later on.
He talked about the roadmap they had, early preview in April, beta in November 2015 and 1.0 in April 2016.
Nice insights on the product, sure something to play with when you get bored of <em>Atom</em>.</p>

<h2>Building Reactive Architectures</h2>

<p>The talk by <a href="https://twitter.com/mattpodwysocki">@mattpodwysocki</a> was great, I was skeptical at first, again <em>Functional reactive programming</em>, but in the end was a really nice talk, detailing all you need to know about RxJs inside.</p>

<p>According to @mattpodwysocki async is awful, we are living in a <code>Callback Hell</code> era and <code>Events and state are a mess</code>.
<strong>Promises are only part of solution</strong>.</p>

<p>You can solve the Callback hell in the reactive way!</p>

<p>Keys:
- React to load
- React to failure
- React to users</p>

<p>This man looks great on stage with his sorcerer hat on his head.</p>

<blockquote>
  <p>You must flatMap it</p>
</blockquote>

<p>Nice talk to follow in my opinion.</p>

<p>Conclusion: Make Reactive Great Again! Along with a new hat.</p>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Make Reactive Great Again! <a href="https://twitter.com/hashtag/MRGA?src=hash">#MRGA</a> <a href="https://twitter.com/hashtag/makereactivegreatagain?src=hash">#makereactivegreatagain</a> <a href="https://t.co/YMO2JEHdhd">pic.twitter.com/YMO2JEHdhd</a></p><div>— λ Calrissian (@mattpodwysocki) </div><a href="https://twitter.com/mattpodwysocki/status/718469012259217409">8 aprile 2016</a></blockquote>

<h2>Discover the information within your data with d3.js</h2>

<p>Not much to say about it, it was a really entry level talk by Daniela Mogini.</p>

<p>Some insights on how to manipulate DOM, handling Array of data natively in D3js.</p>

<p>The entry slogan was good: <code>this has nothing to do with jQuery</code> but it seems a lot like it to me in general.
Don&rsquo;t get me wrong, I know that D3.js is really powerful but at the end of this talk I haven&rsquo;t felt much of its awesomeness.</p>

<h2>The Evolution of Asynchronous JavaScript</h2>

<p><a href="https://twitter.com/cirpo">@cirpo</a> gave us another really good talk about async in javascript.</p>

<p>He started with history, how cool callbacks can be and how, if you aren&rsquo;t a lazy developer, you can avoid the feared callback hell.</p>

<p>He Talked about <em>Promises</em>, of course, and how they become the official way to provide async in js today.
But <em>Promises</em> aren&rsquo;t always the best choice, you don&rsquo;t use them in flow control. You can end up easily with a <strong>Promise Hell</strong> if you don&rsquo;t pay attention.</p>

<blockquote>
  <p><strong>Generators + Promises</strong> for the win.</p>
</blockquote>

<p><em>Generators</em>, a new type of function that doesn&rsquo;t have the <em>run-to-completion</em> behaviour of the other functions.
They have a <em>yield</em> keyword that can pause/restart the flow and you have full control on it.</p>

<p>Unfortunately you have to import libraries to use them but seems like the next javascript, ES7 (ehm ES2016), will ship async / await natively.
You can start playing with it using babel (or typescript).
&ldquo;`
npm install -g babel-cli
npm install babel-plugin-transform-async-to-generator</p>

<p>//add it either to .babelrc or package.json
{
  &quot;plugins&rdquo;: [&ldquo;transform-async-to-generator&rdquo;]
}</p>

<p>babel a.es6 -o a.js
node a.js
&ldquo;`</p>

<blockquote>
  <p>CHOOSE YOUR CONCURRENCY MODEL
   - callbacks
   - promises
   - async/await
   - highland (RxJs, streams)</p>
</blockquote>

<h2>(Web?) Components in production</h2>

<p><a href="https://twitter.com/olamad313">@olamad313</a> is a UX Designer at Amazon Web Services and gave us a great view about how they handle components in their teams across the world. She didn&rsquo;t speak about libraries or polyfill (<em>Polimer</em> for example), they don&rsquo;t actually use one apparently because they don’t want to depend on it.</p>

<p>They build their components using plain javascript embracing web components philosophy and then they also build the bridge to connect them with frameworks or libraries their teams choose to use (GWT, Angular, and React).</p>

<p>They keep principles like custom elements and style encapsulation from web components and they reached some benefits:
 - Components are easy to grab for new teams.
 - No longer need to review the components final implementation in production.
 - Little effort after a component redesign
 - A sparked community engagement.</p>

<h2>FFTT: A new concept of build tool</h2>

<p><a href="https://twitter.com/m_a_s_s_i">@m_a_s_s_i</a> talked about his side project, and invited everyone (who can understand what is it about) to contribute on it. Basically he&rsquo;s building his own build tool to replace <code>grunt</code>, <code>gulp</code> or <code>webpack</code> (I&rsquo;m writing only the most famous names).
Unsatisfied with the job those tools are doing @m_a_s_s_i develops his own to accomplish what he really needs: a fast, deterministic, reliable and language agnostic build tool.</p>

<p>The build system meets functional programming, immutable data structures, memoization of functions results, each step is a pure function.
You never mess with the environment, it&rsquo;s just data transformation.
The build graph is declarative and functional. Each build step is imperative but inside a <em>container</em>, so you don&rsquo;t mess up with the system.</p>

<p>It&rsquo;s built with not many lines of javascript but it&rsquo;s totally programming language independent.</p>

<p>It&rsquo;s not ready for production yet but feel free to contribute if you are interested in it!</p>

<p>Here the github <a href="https://github.com/massimiliano-mantione/fftt">link</a></p>

<h2>Higher Order Components in React</h2>

<p>Finally React :-D. Thanks to <a href="https://twitter.com/cef62">@cef62</a>.</p>

<p>After an excursus on React in general, and the component concept in particular we learn how to extend a component using an high order one.</p>

<p>He spoke about mixins (deprecated with ES6 syntax) as well and he gave us a good <code>HoC vs Mixins</code> preview:
- Declarative vs Imperative
- Customizable vs Favors inheritance
- Easy to read vs Hard to read
- Enforce encapsulation</p>

<p>He explains the <em>high order function</em> concept:<br />
a function that takes one or more functions as arguments and returns a new one as its result.</p>

<p>So HOC are high order functions that, instead, return a component.</p>

<h2>A Modern debugging experience using DevTools by <a href="https://twitter.com/umaar">@umaar</a>, modern techniques for debugging JavaScript code.</h2>

<p>Umar covered different ways to debug our code using the Chrome DevTools.
For example he explained how to:
  - Use Quick Source Pane for faster CSS editing
  - Trigger pseudo classes in DevTools
  - Use the animation inspector to change and modify running animations
  - Try out the official DevTools dark theme
  - Copy the response of a network resource to your clipboard
and much more!</p>

<p>You can find all this techniques explained in his <a href="https://umaar.github.io/devtools-animated-2016">slides</a></p>

<h2>A Class Action</h2>

<p>Last talk of the conference by <a href="https://twitter.com/unlucio">@unlucio</a>, a dispute on probably the most controversial feature in ES2016: the <code>class</code> keyword.</p>

<p>He called few witnesses for the <em>dispute</em>:
  - <code>miss function</code>: do we really need a shorthand for manually defining a constructor Function?</p>

<ul><li><code>the prototype</code>: If you want to write once and use your code in multiple situations, prototypal inheritance makes it easy.</li>
<li><code>class</code>: does it really add something new to the language and will it simplify everybody&rsquo;s life?</li>
</ul><p><em>OOP is good for you&hellip; at least that’s what they say</em></p>

<p><img src="http://thinknsmile.com/wp-content/uploads/2014/05/butter_is_good_for_you.jpg" alt="OOP is good for you... at least that’s what they say" /></p>

<p>For code segregation choose <code>modules</code> over <code>classes</code></p>

<p>I liked the reference to the coffee and sugar:</p>

<blockquote>
  <p>When coffee is good you don&rsquo;t need sugar, and <code>class</code> seems more like aspartame than sugar.
  The difference is that aspartame can kill you!</p>
</blockquote>

<p>To sum up, it was a really nice conclusion of these wonderful two days conference in Verona.
I encourage you to have a look at the <a href="http://www.slideshare.net/unlucio/a-class-action">slides</a> that are self explanatory.</p>

<hr><blockquote class="twitter-tweet"><p lang="en" dir="ltr">jsDay Faces - Day 2 - 2016 <a href="https://t.co/XfczO1ZKL8">https://t.co/XfczO1ZKL8</a></p><div>— JS Italian Conf (@jsconfit) </div><a href="https://twitter.com/jsconfit/status/730832745107050496">12 maggio 2016</a></blockquote>