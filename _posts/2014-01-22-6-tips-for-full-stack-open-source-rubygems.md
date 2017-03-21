---
ID: 295
post_title: >
  6 Tips for Full Stack Open Source
  RubyGems Development
author: Giovanni Intini
post_date: 2014-01-22 10:26:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/22/6-tips-for-full-stack-open-source-rubygems/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/74159739828/6-tips-for-full-stack-open-source-rubygems
tumblr_mikamayhem_id:
  - "74159739828"
---
<p>Call me old fashioned but up to really recently I used to write my rubygems the old way: manually. </p>
<p>You know, the .gemspec, the various directories, everything.</p>
<p>It was kinda boring, and that&rsquo;s why my contributions to the gems ecosystem became scarcer than ever.</p>
<p>Around last December though I wanted to start working on a new project, <a href="https://github.com/mikamai/ruby-lol" title="ruby-lol" target="_blank">ruby-lol</a> (more on it in a new article), and I wanted to have everything people expect on a modern gem:</p>
<ul><li><span>continous integration badge</span></li>
<li><span>coverage badge</span></li>
<li><span>gem version badge</span></li>
<li><span>code quality badge</span></li>
<li><span>gem dependency badge</span></li>
</ul><p><span>The most important thing though was minimizing the effort I had to put in to obtain all of that.</span></p>
<p><span>So, without further ado, here&rsquo;s a list of tips to streamline your gems development:</span></p>
<p><strong><span>1. bundle gem your-gem-name</span></strong></p>
<p><span>This won&rsquo;t be big news for anyone, but the best way to start developing a gem is calling</span></p>
<pre>$ bundle gem your-gem-name</pre>
<p>from the terminal. It will create a directory with everything you need to start coding in there. Edit the license, the readme, and few fields on the gemspec and you can start happily tapping on your keyboard.</p>
<p><strong>2. use a gem version badge</strong></p>
<p>Once you release your gem (hint: <code>gem build your-gem-name.gemspec &amp;&amp; gem push your-gem-name-version.gem</code>) you can add a gem version badge to your readme. It&rsquo;s pretty simple, open the gem page on rubygems.org (i.e. <a href="http://rubygems.org/gems/ruby-lol">ruby-lol</a>) and click on the <a href="http://badge.fury.io/for/rb/ruby-lol" title="ruby-lol gem version badge.">badge</a> link. </p>
<p>You&rsquo;ll be redirected to <a href="http://badge.fury.io">badge.fury.io</a>, where you can pick your favorite choice of markup and paste the badge in your readme file. Commit, push and you&rsquo;re done :)</p>
<p><strong>3. continous integration with travis-ci.org</strong></p>
<p>I am officially in love with <a href="https://travis-ci.org" title="Travis CI">Travis</a>. No one can not be in love with a working and easy to use hosted continous integration service that is <strong>FREE </strong>for open source development. </p>
<p>So how do we set it up? Easy. Sign in travis-ci.org with your github account and give it access to your public projects. Follow the three simple steps and you&rsquo;re done.</p>
<p>After you have your build up and running remember to copy and add the badge markup to your readme.</p>
<p><strong>4. easy code metrics with code climate</strong></p>
<p>Code quality is something that usually makes people argue. I know I do a lot. Code quality metrics are even worse. I know people that swear by them, and people that think they&rsquo;re worthless.</p>
<p>I don&rsquo;t know where you are with this, but I like showing off the quality of my code, and easily finding weak spots I overlooked.</p>
<p>Code climate is a (pricey) service that is very good at analyzing your code and allowing you to navigate around its issues. I am a paid subscriber, but like many companies they also have a free way to check the code quality of gems.</p>
<p>Their <a href="https://codeclimate.com/github/signup">add github repository</a> page is what you&rsquo;re looking for. </p>
<p>As for the other services remember, when you&rsquo;re done, to add the badge to your readme.</p>
<p><strong>5. test coverage, some love it, some hate it</strong></p>
<p>As with code metrics, test coverage is something that at least makes people talk.</p>
<p>I like keeping track of the coverage of my tests/specs, because knowing that everything I have written is covered by tests makes me work with a more relaxed state of mind.</p>
<p><a href="https://coveralls.io" target="_blank">Coveralls.io</a> is another great service, free code coverage for opensource projects, easy to setup, it integrates perfectly with Travis.</p>
<p>Once again, setting it up is easy. Signup with github, give access to public projects, get the badge and put it in your readme.</p>
<p><strong>6. keep up with your dependencies</strong></p>
<p>Last but not least, <a href="http://gemnasium.com" target="_blank">Gemnasium</a> is a service that allows you to keep track of your gem dependencies and notifies you when you become outdated.</p>
<p>Free for open source projects, you might be familiar with the set up process. Sign in with github, copy the badge when you&rsquo;re done :)</p>
<p>Gemnasium ends the list of our list of tips. If you want, let me know what you think of it and whether there are more easy-to-set-up and useful services. Have fun!</p>