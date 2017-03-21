---
ID: 27
post_title: 'What I&#8217;ve learned at jsday 2016 in Verona'
author: Linkme s.r.l.
post_date: 2016-06-01 15:00:59
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/06/01/what-ive-learned-at-jsday-2016-in-verona/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/145258317709/what-ive-learned-at-jsday-2016-in-verona
tumblr_mikamayhem_id:
  - "145258317709"
---
<hr><p>I had the chance to participate to <a href="http://2016.jsday.it/">jsday</a> in Verona early this year and I wanted to make a small summary to all of you that couldn’t be there. I really liked <a href="https://medium.com/@kitze/lessons-learned-at-react-amsterdam-51f2006c4a59#.leg4e0mjn">Kitze</a>’s idea to write about React Amsterdam conference he attended.</p>

<p>Unfortunately this summary can’t be complete since there were 2 distinct tracks and I couldn’t follow both of them at the same time but still, I hope this can be interesting for you anyway.</p>

<p>I’ll add slides and videos links as soon as they are available.</p>

<h2>TL;DR</h2>

<p><img src="https://avatars3.githubusercontent.com/u/984368?v=3&amp;s=200" alt="RxJs" /></p>

<p>The big topic was <em>Functional reactive programming</em>, or RxJs if you prefer.
That seems like the big thing in javascript 2016.</p>

<p>2 talks, a workshop and mentions in many talks, all about <em>Functional reactive programming</em>.</p>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Recap from <a href="https://twitter.com/hashtag/jsday?src=hash">#jsday</a> 2016 <a href="https://twitter.com/hashtag/functional?src=hash">#functional</a> <a href="https://twitter.com/hashtag/reactive?src=hash">#reactive</a> <a href="https://twitter.com/hashtag/programming?src=hash">#programming</a> is the new black <a href="https://twitter.com/hashtag/javascript?src=hash">#javascript</a></p><div>— Daniele Bertella (@_denb) </div><a href="https://twitter.com/_denb/status/730801587493437440">12 maggio 2016</a></blockquote>

<h2>Day 1</h2>

<h2>First Keynote: What we need from the Web, and what it needs from us.</h2>

<p><a href="https://twitter.com/shwetank">@shwetank</a>, Opera developer, talked about <em>Progressive web apps</em>, probably the second big thing in javascript 2016. It was fun because I just attended another great conference at google Italia few days before the jsday on the exact same topic. I was thinking <em>PWA</em> was mostly (only) a Google thing but I was happy to realize other browsers are thinking about it as a way to save <code>the web</code> against <code>mobile native</code> development.</p>

<p>How can <em>PWA</em> beat native?
Basically the native approach forces you to install the application in order to try it out. PWAs, instead, give you a <code>try before install</code> model. 
Pretty much the same capabilities:</p>

<ul><li>Add to home screen</li>
<li>Work offline</li>
<li>Push notifications</li>
<li>Plus the web’s power: different urls, SEO, accessibility…</li>
</ul><p>without the need for installing.<br />
With <em>PWA</em> you let the user choose and that for @shwetank can be the key.</p>

<h2>Functional Programming and Async Programming Workshop</h2>

<p>The topic was tempting but then after a short and interesting introduction from <a href="https://twitter.com/mattpodwysocki">@mattpodwysocki</a> people lost interest.</p>

<p>Everybody was following the exercise from the reactivex <a href="http://reactivex.io/learnrx/">site</a>, that I really encourage you to do, but I’d rather have done it at home than spending my time there during the conference.</p>

<p>This is not a critique towards him or the organization, just a reminder for me that next time I&rsquo;d rather focus on talks.</p>

<p>Anyway the presentation slides can be found <a href="https://github.com/mattpodwysocki/jsday-workshop-2016">here</a> along with all the other assets and I encourage you to check them out.</p>

<h2>Forgotten funky functions</h2>

<p><a href="https://twitter.com/jakobmattsson">@jakobmattsson</a> gave a quick talk about javascript in general and the simple things you can easily accomplish with function but you may have forgotten. In javascript everything seems really simple, but still you can do great things.</p>

<p>He talked about 3 main topics:</p>

<ul><li>functional programming</li>
<li>meta programming</li>
<li>there is no class</li>
</ul><p>@jakobmattsson discussed about <code>apply</code>, <code>call</code>, <code>clojures</code>, <code>eval</code>. Topics that (almost) every js developer has doubt about.</p>

<p>He concluded with a useful explanation on ES6 or coffeescript classes syntactic sugar that you may not need if you already have a solid understanding of <strong>Object.create</strong>. Here the <a href="https://speakerdeck.com/jakobmattsson/forgotten-funky-functions-1">slides</a></p>

<h2>Functional Reactive programming with React.js</h2>

<p><strong>Spoiler alert</strong>, <a href="https://twitter.com/lucamezzalira">@lucamezzalira</a> didn’t talk about react at all! In the end what he really talked about was cycle.js; nonetheless it was a good <em>excursus</em> about RxJs.</p>

<p>You have to start learning a few concepts to understand reactive programming, like <em>streams</em>, <a href="http://reactivex.io/documentation/observable.html">cold and hot observables</a> and <a href="http://reactivex.io/documentation/operators.html">operators</a>.</p>

<p>If you are using a <em>Flux</em> implementation for your front end architecture or even worst MVC / MVVM / MV* pattern, you are living in the past! Say hello to <strong>MVI</strong> aka <a href="http://thenewstack.io/developers-need-know-mvi-model-view-intent/">Model View Intent</a>.</p>

<p>I’m personally quite happy with <em>Redux</em> for now but I think you need at least to know what’s going on around you.</p>

<p>The conclusion was that <em>Functional Reactive programming</em> isn’t always the best thing to do but for sure it’s great when you need to fetch lots of data and react to them.
You can find slides <a href="http://www.slideshare.net/flashplatform/reactive-programming-with-cyclejs">here</a>.</p>

<h2>Out of the browser and onto the streets</h2>

<p><a href="https://twitter.com/Rumyra">@Rumyra</a> did the most pleasant talk of the day, with music and visual, it was party time already!</p>

<p>Apparently she’s <a href="https://github.com/Rumyra/VJing">VJing</a> around projecting visual animations on buildings along with music. I’d love to see her live. Unfortunately she hasn’t had the chance to do it during the party in Verona, but maybe next year 😀.</p>

<p><img src="http://68.media.tumblr.com/7a1d1e8c4b7aefaec59eddd5337b9fe8/tumblr_inline_o7u69h3sWB1un6tid_540.jpg" alt="image" /></p>

<p>She talked about the newest web APIs such as <code>animation</code>, <code>audio</code>, <code>stream</code> and <code>midi</code>. I encourage you to check her <a href="https://github.com/Rumyra">github</a> account. It’s not that easy to find samples but you can have an idea about her projects.</p>

<p>She showed us a demo with voice control using a microphone and then another mixing music, video and visual animation.</p>

<p>Last she ended up showing her handmade bag with a mac and her “mini” MIDI controller inside that she’s using for VJing.</p>

<hr><p><strong>Did you like this review? Here you can find <a href="http://dev.mikamai.com/post/145964907404/what-ive-learned-at-jsday-2016-in-verona-day-2">Day 2</a></strong></p>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">jsDay Faces - Day 1 - 2016 <a href="https://t.co/oT5klm1epR">https://t.co/oT5klm1epR</a></p><div>— JS Italian Conf (@jsconfit) </div><a href="https://twitter.com/jsconfit/status/730457541453484033">11 maggio 2016</a></blockquote>

<p>Stay tuned</p>