---
ID: 7
post_title: >
  Rails subdomains with Pow, Capybara and
  Travis CI
author: Andrea Longhi
post_date: 2016-11-14 08:46:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/11/14/rails-subdomains-with-pow-capybara-and-travis-ci/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/153165491919/rails-subdomains-with-pow-capybara-and-travis-ci
tumblr_mikamayhem_id:
  - "153165491919"
---
<p>
  Adding subdomains to a Rails application is an easy task today, but things can get a little trickier when testing is involved.</p>

<!--more--> 

<p>For starters, you may only need to handle subdomains at the routing level, which is quite easy. But after the easy step some troubles may be awaiting for you.
</p>

<p>
  In our example we want to add an “admin” subdomain to a brand new app. We edit the file routes.rb as follows and we’re basically done:
</p>
<pre><code>
  # myappname/config/routes.rb
  
  Rails.application.routes.draw do
    constraints subdomain: 'admin' do
      root to: 'admin#index', as: 'admin_root' # will match <a href="http://admin.myappname.dev/">http://admin.myappname.dev/</a>
    end
  end
</code></pre>
<p>Of course, to make the thing actually work you also need an <code>AdminController</code> with an <code>index</code> template.</p>
<p>Want to see the page in the browser? You can&rsquo;t yet. If you start the server and go to admin.localhost:3000 you will only be able to see the default rails page. You have a few options in order to make this work, let&rsquo;s discuss them one by one</p>

<h2>Lvh.me</h2>
<p>Just visit <code><a href="http://admin.lvh.me:3000/">http://admin.lvh.me:3000/</a></code> and you&rsquo;ll see the admin page. It works because <code>lvh.me</code> is an external website that points to localhost/127.0.0.1.</p>
<p> This solution is ok as long as you have an internet connection and you are willing to pay the time overhead of the necessary DNS search plus redirect.</p>


<h2>Edit /etc/hosts</h2>
<p>In order to avoid hitting an external website the easiest solution is to add to your <code>/etc/hosts</code> file an entry like this:</p>
<pre><code>
127.0.0.1 myappname.dev
127.0.0.1 admin.myappname.dev
</code></pre>

<p>and your app will be available at <code><a href="http://admin.myappname.dev:3000">http://admin.myappname.dev:3000</a></code>. Drawbacks are you need an entry for every single subdomain of each application and you need to manually edit/keep updated the file.</p>


<h2>Pow</h2>
<p>
The most flexible solution is to use <a href="http://pow.cx/">Pow</a> or equivalents (if you&rsquo;re on Linux you can use <a href="http://ysbaddaden.github.io/prax/">Prax</a>). For increased ease of use you can manage pow with the gem <a href="https://github.com/rodreegez/powder">powder</a>. If you hacked the <code>/etc/hosts</code> file you should revert the changes before continuing.</p><p>
Now that pow has taken control of your computer (!) if you visit <code><a href="http://myappname.dev">http://myappname.dev</a></code> (no port required, defaults to 80) you will see Pow page with the instructions. Just follow them (you&rsquo;re just a symlink away) and you&rsquo;re done. </p>

<h2>Testing subdomains with Capybara</h2>
<p>If good coders always test their code, then we&rsquo;re no exception. You may want to write integration tests for your admin page, so the names Rspec and Capybara should be &lsquo;nuff said. I&rsquo;m assuming you already configured Rspec and Capybara in this app.</p>

<h3>RackTest webdriver</h3>

<p>I&rsquo;m going to use a single spec to test the admin page, which will evolve through various steps according to increasing requirements.</p><p>Let&rsquo;s add a very basic spec that uses Capybara&rsquo;s default driver, <code>rack_test</code>, which is the fastest/easiest to work with:</p>
<pre><code>
# myappname/spec/features/admin_page_spec.rb

require 'rails_helper'

describe 'the admin page' do
  it 'can be seen' do
    visit admin_root_path
    expect(page).to have_content 'Admin#index'
  end
end
</code></pre>

<p>The spec won&rsquo;t pass. You need to add as first line of the test the follwing code:</p>
<pre><code>Capybara.default_host = 'http://admin.myappname-test.dev'</code></pre>
<p>where <code>myappname-test</code> can actually be any name you want. Running specs again should result in a green message.</p>

<h3>Selenium webdriver</h3>

<p>I&rsquo;m assuming you have already configured rspec/capybara to use Selenium in the app. To trigger Selenium in the previous spec we only need to add <code>:js</code> to the <code>it</code> description:</p>

<pre><code>
# myappname/spec/features/admin_page_spec.rb

require 'rails_helper'

describe 'the admin page', :js do
  it 'can be seen' do
    Capybara.default_host = 'http://admin.myappname-test.dev'
    visit admin_root_path
    expect(page).to have_content 'Admin#index'
  end
end
</code></pre>

<p>After the change the spec doesn&rsquo;t pass anymore. We need to add some more configuration:</p>

<pre><code>
# myappname/spec/features/admin_page_spec.rb

require 'rails_helper'

describe 'the admin page', :js do
  it 'can be seen' do
    Capybara.server_port = 3030
    Capybara.default_host = 'http://myappname-test.dev'
    Capybara.app_host = 'http://admin.myappname-test.dev'
    visit admin_root_path
    expect(page).to have_content 'Admin#index'
  end
end
</code></pre>

<p>but we&rsquo;re not done yet. We also need to add some more configuration for Pow. Let&rsquo;s create a new file in <code>~/.pow</code> named <code>myappname-test</code> that contains the text <code>3030</code>:</p>

<pre><code> cd ~/.pow &amp;&amp; echo "3030" &gt; myappname-test</code></pre>

<p>At this point <code>rake spec</code> should run without errors. </p>

<h3>Travis integration</h3>
<p>Here at Mikamai we use Travis for continuous integration, so the final step before calling a day it&rsquo;s to push the code and see if the specs pass on the CI server as well. </p>
<p>First, the project must be configured to allow Selenium to work on Travis. This <a href="https://docs.travis-ci.com/user/gui-and-headless-browsers/">link</a> explains the basics. You probably just need to add this to your <code>.travis.yml</code> file:</p>
<pre><code>
# myappname/.travis.yml

before_script:
  - "export DISPLAY=:99.0"
  - "sh -e /etc/init.d/xvfb start"
  - sleep 3 # give xvfb some time to start
</code></pre>

<p>At this point we need to instruct travis to use subdomains by adding the following configuration to the .travis.yml file:</p>
<pre><code>
# myappname/.travis.yml

addons:
  hosts:
    - admin.myappname-test.dev
    - myappname-test.dev
</code></pre>
<p>Again, let&rsquo;s also add some further configuration to our rspec test to make it pass on Travis:</p>

<pre><code>
# myappname/spec/features/admin_page_spec.rb

require 'rails_helper'

describe 'the admin page', :js do
  it 'can be seen' do
    Capybara.server_port = 3030 unless ENV['CI'] # don't use port 3030 on travis
    Capybara.always_include_port = true # for travis  
    Capybara.default_host = 'http://myappname-test.dev'
    Capybara.app_host = 'http://admin.myappname-test.dev' # to change the subdomain
    visit admin_root_path # this route is under under "admin" subdomain
    expect(page).to have_content 'Admin#index'
  end
end
</code></pre>

<p>Now, when you push your code to Travis you should be rewarded with a green outcome&hellip; job done!</p><p>Of course there is one more step that should be done: move the Capybara port/domains initialization into rspec configuration:</p>
<pre><code>
# rspec/rails_helper.rb

Rspec.configure do |config|
  Capybara.server_port = 3030 unless ENV['CI'] # don't use port 3030 on travis
  Capybara.always_include_port = true # for travis 
  Capybara.default_host = 'http://myappname-test.dev'
end
</code></pre>