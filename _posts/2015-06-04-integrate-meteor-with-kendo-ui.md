---
ID: 116
post_title: Integrate Meteor with Kendo UI
author: Massimo Ronca
post_date: 2015-06-04 10:20:23
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/06/04/integrate-meteor-with-kendo-ui/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/120684737224/integrate-meteor-with-kendo-ui
tumblr_mikamayhem_id:
  - "120684737224"
---
<p>At the end of last year I&rsquo;ve been involved in a testdrive project aimed to explore the possibility of replacing good old desktop applications with modern web applications, a demanding task, but nonetheless a fun one.   </p>

<p>The client gave me complete freedom to chose whatever technology I wanted, except for some key elements, one of them was <a href="http://www.telerik.com/kendo-ui" title="Kendo UI">Kendo UI</a>, a widget library they had been testing for a while with an impressive <a href="http://www.telerik.com/kendo-ui#more-widgets" title="Kend UI widgets">list of features and components</a>.   </p>

<p>On my side I&rsquo;ve decided to put <a href="http://meteor.com" title="Meteor">Meteor</a> on test, I confess, I&rsquo;m not the biggest Javascript fan around, but I&rsquo;ve rarely felt so confident with a new platform like I&rsquo;ve been with Meteor, so I went for it.<br />
The biggest advantage Meteor gave us was speed, we&rsquo;ve been able to put together an incredible (IMHO) amount of work in a very short period of time.   </p>

<p>Long story short: the project was a success, everything was good, we also had the opportunity to experiment on the infrastructure side, and I&rsquo;ve learned a lot in those few months, about things I could never have accessed otherwise.  </p>

<p>There was only one small glitch that constantly bugged me: the integration between Meteor and Kendo, especially in regards to reactivity.  </p>

<p>Meteor might not be the <em>new kid on the block</em> anymore, but it&rsquo;s not the <a href="http://en.wikipedia.org/wiki/Spring_Framework" title="Spring MVC">well established framework</a> either: there was no official Telerik package for Meteor, and I had to roll my own, learning by doing.  </p>

<!--more-->

<p>I don&rsquo;t know wheter Telerik listened to my prayers or not, but on february of this year they released a bunch of <a href="https://atmospherejs.com/telerik">packages on Atmoshpere</a>, making things easier for everyone, including me, so I decided to write this simple guide.   </p>

<p>The Telerik packages differ from each other only in the default theme used, the set of bundled components is the same, you can find the complete list on <a href="https://github.com/telerik/kendo-ui-core" title="Kendo UI Core">their github project page</a>, although they included only the open source version of the components, this guide applies to the paid version as well.   </p>

<p>We are going to build a simple applications that shows a list of tweets for a particular location (in this example the city of Milan).<br />
Let&rsquo;s start by creating a new empty application and configure the packages </p>

<pre class="line-numbers"><code class="language-bash">
$ meteor create telerik-demo
$ cd telerik-demo

# disable autopublish, we don't want to publish all the tweets, but only the
# 30 most recent
# disable insecure too, but it's not required
$ meteor remove autopublish insecure

# add official momentjs
$ meteor add momentjs:moment
# add twitter library
$ meteor add mrt:twit  
# add telerik components 
# choose your favourite theme
# if you install the bootstrap theme, bootstrap CSS framework is included too
$ meteor add telerik:kendo-ui-core-material-theme
</code></pre>

<p>Open <code>telerik-dom.js</code>, the core application code, and add the declaration of our collection of tweets, making it available to both client and server.</p>

<pre class="line-numbers"><code class="language-javascript">Tweets = new Mongo.Collection('tweets');</code></pre>

<p>then, look for the line <code>if (Meteor.isServer) {</code>, this is the entry point of the server side app.<br />
What we need to do is setup the Twitter library, subscribe on a stream filtering by location and language, and then add every tweet we receive to a collection in our MongoDB instance.<br />
You might be tempted to do something like this</p>

<pre class="line-numbers"><code class="language-javascript">Meteor.startup(function () {
  var t = new TwitMaker({
    consumer_key:     '...'
    , consumer_secret:    '...'
    , access_token:     '...'
    , access_token_secret:  '...'
  })

  // I used this tool to get the bounding box coordinates
  // <a href="http://boundingbox.klokantech.com/">http://boundingbox.klokantech.com/</a>
  var milan = ['8.9936308861','45.3026233328','9.5197601318','45.6359571671']
  // subscribe to tweets from Milan in Italian
  var stream = t.stream('statuses/filter', {locations: milan, language: 'it'})

  // listen and wait
  stream.on('tweet', function (tweet) {
    Tweets.insert(tweet);
  );
});</code></pre>

<p>Aaaand you would be wrong…<br />
If you do this, Meteor will exit with the error <code>Meteor code must always run within a Fiber</code>.<br />
The problem is that the code that inserts the new tweets is executed asynchronously, outside the Meteor Fiber, so you need to provide Meteor the right environment in which to run. The esiest way is to wrap the callback inside a <code>Meteor.bindEnvironment</code> call, like this</p>

<pre class="line-numbers"><code class="language-javascript">  stream.on('tweet', Meteor.bindEnvironment(function (tweet) {
    Tweets.insert(tweet);
  }, function () {
    console.log('Failed to bind environment');
  }));</code></pre>

<p>Don&rsquo;t forget to publish your tweets, to be correct, the 30 most recent tweets.</p>

<pre class="line-numbers"><code class="language-javascript">Meteor.publish('latest_tweets', function() {
  return Tweets.find({}, {sort: {timestamp_ms: -1}, limit: 30});
})</code></pre>

<p>Now that we have a working Twitter scraper, let&rsquo;s work on the frontend:
open <code>telerik-demo.html</code> and replace its content with this</p>

<pre class="line-numbers"><code class="language-markup">
&lt;head&gt;
  &lt;title&gt;twitter&lt;/title&gt;
&lt;/head&gt;

&lt;body&gt;
  {{ &gt; tweets }}
&lt;/body&gt;


&lt;template name="tweets"&gt;
  &lt;div id="tweets"&gt;&lt;/div&gt;
  &lt;div id="pager"&gt;&lt;/div&gt;

  &lt;script type="text/x-kendo-tmpl" id="tweet"&gt;
    &lt;div class="tweet" data-twitter-id="${id}"&gt;
      &lt;ul&gt;
        &lt;li&gt;
          &lt;div class="user"&gt;
            &lt;a href="http://twitter.com/${user.screen_name}" aria-label="${user.name} (screen name: ${user.screen_name})" data-scribe="element:user_link"&gt;
              &lt;img alt="" src="${user.profile_image_url}" data-scribe="element:avatar"&gt;
              &lt;span&gt;
                &lt;span data-scribe="element:name"&gt;${user.name}&lt;/span&gt;
              &lt;/span&gt;
              &lt;span data-scribe="element:screen_name"&gt;@${user.screen_name}&lt;/span&gt;
            &lt;/a&gt;
          &lt;/div&gt;
          &lt;p class="tweet"&gt;${text}&lt;/p&gt;
          &lt;p class="timePosted"&gt;Posted &lt;a title="#= moment(created_at).format('dddd, MMMM Do YYYY, h:mm:ss a') #"&gt;#= moment(created_at).fromNow() #&lt;/a&gt;&lt;/p&gt;
          &lt;p class="interact"&gt;${place.full_name}&lt;/p&gt;
        &lt;/li&gt;
      &lt;/ul&gt;
    &lt;/div&gt;
  &lt;/script&gt;


&lt;/template&gt;</code></pre>

<p>The code inside the <code>&lt;script type="text/x-kendo-tmpl"&gt;</code> is the template code that the Kendo ListView is going to use to render every single item in the list.<br />
An explanation of the custom syntax used in Kendo templates is beyond the purpose of this guide, but you can refer to the <a href="http://docs.telerik.com/kendo-ui/framework/templates/overview">Telerik official documentation</a>, it&rsquo;s really easy.</p>

<p>Now back to <code>telerik-demo.js</code>, inside <code>if (Meteor.isClient) { ... }</code> we subscribe to the publications created above</p>

<pre class="line-numbers"><code class="language-javascript">  Meteor.subscribe('latest_tweets', function() {
    console.log("subscribed to latest tweets");
  })</code></pre>

<p>Now taht the channel is open, the server will start to send the data to the client and keep it in sync, automatically.<br />
The last step is to bootstrap the Kendo component and display our tweets on the page</p>

<pre class="line-numbers"><code class="language-javascript">// when the tweets template has done rendering
Template.tweets.rendered = function() {

  // create the datasource for the listview showing 3 items per page
  var dataSource = new kendo.data.DataSource({
    pageSize: 3
  });

  // initialize the listview with the datasource and the template code
  $('#tweets').kendoListView({
    dataSource: dataSource,
    template: kendo.template($("#tweet").html())
  });

  // initialize the pager component
  $('#pager').kendoPager({
    dataSource: dataSource
  });

  // this function is run automatically when dependencies change,
  // in our case when the collection is updated 
  // (items are changed, added or removed)
  // Meteor figured out that if we subscribed to a publication, we wanna
  // be also informed when it changes
  this.autorun(function() {
    // we use fetch becase it returns a Json list of all the items 
    // in the collection, which is one of the formats supported by
    // Kendo datasource 
    // this also triggers the rendering of the list view and the pager
    dataSource.data(Tweets.find({}, {sort: {timestamp_ms: -1}}).fetch()); 
  });
};</code></pre>

<p>And that could be all. If now you launch <code>meteor</code> inside the project folder and navigate to <code>http://localhost:3000</code> you should see a list of tweets automatically updating.<br />
It&rsquo;s not the best looking app ever, but it works.<br /><img src="http://i.imgur.com/lXxC3Gg.png" alt="" /></p>

<p>Let&rsquo;s do something to improve the presentation layer.<br />
I&rsquo;m not a great designer, so I grabbed this <a href="http://codepen.io/nerijusgood/pen/Ggqygo" title="Twitter custom widget CSS">Twitter widget customization</a> from Codepen and copied the relevant bits inside <code>telerik-demo.css</code>.<br />
Next thing I wanted to try is to add a fade-in effect to every new tweets, to make things smoother.<br />
I&rsquo;ve created a class that runs an animation inside the CSS and I wanted to attach this class to every new tweet <code>div</code>.   </p>

<pre class="line-numbers"><code class="language-css">@keyframes fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}

.fade-in {
  animation-duration: 1s;
  animation-name: fade-in;
}

/* I removed the prefixed versions of keyframes and animation-* for clarity */</code></pre>

<p>My first naive approach relied on <a href="http://updates.html5rocks.com/2012/02/Detect-DOM-changes-with-Mutation-Observers" title="Mutation Observers">Mutation Observers</a>.  </p>

<pre class="line-numbers"><code class="language-javascript">  document.addEventListener("DOMNodeInserted", function(event) {
    if($(event.target).hasClass("tweet")) {
      // a new tweet has been added to the DOM, start fading
      event.target.classList.add('fade-in');
      console.warn("Another node has been inserted! ", event, event.target);
    }
  }, false);</code></pre>

<p>Only after a few unsuccesfull attempts, where every tweet was being faded, I realized that when you change the <code>data</code> property of the <code>datasource</code> component, the current view (so only the current page) is re-rendered completely, making every tweet a <em>new</em> tweet to the eyes of the poor browser.<br />
The solution is simple, I made a diff between the actual data and the new data coming from the server and only added the fade to those tweets that really were new, I used the Twitter <code>id</code>   attribute as the key.   </p>

<p>update the template with the new Twitter id attribute</p>

<pre class="line-numbers"><code class="language-markup">&lt;!-- telerik-demo.html --&gt;
&lt;script type="text/x-kendo-tmpl" id="tweet"&gt;
    &lt;div class="tweet" data-twitter-id="${id}"&gt;
</code></pre>

<p>update the autorun function to compute a diff and apply the fade-in effect 
only to the new tweets</p>

<pre class="line-numbers"><code class="language-javascript">this.autorun(function() {
  var current_values = dataSource.data();
  var updated_values = Tweets.find({}, {sort: {timestamp_ms: -1}}).fetch();
  var current_ids = _.map(current_values, function(x) { return x.id; });
  var updated_ids = _.map(updated_values, function(x) { return x.id; });
  var diff = _.difference(updated_ids, current_ids);

  // update dataSource and trigger listview render
  dataSource.data(updated_values);
  // fade in new nodes
  _.each(diff, function(x) {
    $('div[data-twitter-id="'  + x + '"').addClass('fade-in');
  })
});</code></pre>

<p>not strictly required, but advised, remove the fade class when animation ends</p>

<pre class="line-numbers"><code class="language-javascript">var animationListener = function(event){
  if (event.animationName == "fade-in") {
    console.log("remove class fade-in from", event.target);
    event.target.classList.remove('fade-in');
  }
}
document.addEventListener("animationend", animationListener, false); </code></pre>

<p>much better now
<img src="http://i.imgur.com/gr9F232.png" alt="" /></p>

<p>You can see the application in action <a href="http://telerik.meteor.com">here</a> and find the source code on <a href="https://github.com/mikamai/telerik-meteor-integration-demo">Mikamai&rsquo;s Github</a></p>