---
ID: 165
post_title: >
  Arduino Yún + Twitter Streaming API
  blink example
author: Stefano Guglielmetti
post_date: 2015-01-26 14:09:08
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/01/26/arduino-yun-twitter-streaming-api-blink-example/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/109202378934/arduino-yún-twitter-streaming-api-blink-example
tumblr_mikamayhem_id:
  - "109202378934"
---
<img src="https://camo.githubusercontent.com/b7996effbf10e482e3c6e6e262a9af1de7f1900b/687474703a2f2f692e696d6775722e636f6d2f6777574e6765326c2e6a7067" /><p>This is a simple example on how to run some code on the Arduino side of the Arduino Yún on specific events fired through the Twitter Streaming API</p>
<p>i.e. when a tweet containing a specific #hashtag is twitted, Arduino blinks an LED</p>

<p>I love this experiment because it&rsquo;s near real time and it gives a lot of possibilities, and it&rsquo;s very easy to set up</p>

<h2>Requirements</h2>

<ul><li>An Arduino Yún (obviously) with an external SD card (required for hosting the Python libraries you&rsquo;ll need)</li>
<li>A <a href="https://apps.twitter.com/">twitter app</a> you own</li>
<li>The <a href="https://github.com/tweepy/tweepy">Tweepy library</a> for Python</li>
<li>And, finally&hellip; <a href="https://github.com/amicojeko/TwitterBlink">this code</a></li> :)
</ul><h2>Instructions</h2>

<blockquote><p>&ldquo;I love instructions, they give me confidence.&rdquo;
&ndash; <cite>me</cite></p></blockquote>

<ol><li>In case you haven&rsquo;t done it yet, <a href="http://arduino.cc/en/Tutorial/YunSysupgrade">upgrade</a> your Arduino Yún to the latest version and <a href="http://arduino.cc/en/Tutorial/ExpandingYunDiskSpace">expand</a> it&rsquo;s disk space using the SD card</li>
<li>Log to your Arduino using ssh</li>
<li><p>Install a bunch of useful stuff</p>

<pre><code> $ opkg update
 $ opkg install git
 $ opkg install python-expat
 $ opkg install python-openssl
 $ opkg install python-bzip2
</code></pre>

<h4>easy_install</h4>

<pre><code> $ opkg install distribute
</code></pre>

<h4>pip</h4>

<pre><code> $ easy_install pip
</code></pre>

<h4><a href="https://github.com/tweepy/tweepy">tweepy</a></h4>

<pre><code> $ pip install tweepy
</code></pre>

<p>or install it manually cloning it from github</p>

<pre><code>  $ git clone <a href="https://github.com/tweepy/tweepy.git">https://github.com/tweepy/tweepy.git</a>
  $ cd tweepy
  $ python setup.py install
</code></pre></li>
<li><p>If you don&rsquo;t have one, create your <a href="https://apps.twitter.com/">twitter app</a>
and get your consumer key and consumer secret. Then, generate an access token and get your access token and access token secret</p></li>
<li><p>Create a working folder for all the scripts (i.e. /root/python)</p>

<pre><code> $ mkdir /root/python
 $ cd /root/python
</code></pre>

<p>and, while in the python folder, clone the Twitter Blink example</p>

<pre><code>  $ git clone git@github.com:amicojeko/TwitterBlink.git
</code></pre>

<p>don&rsquo;t forget to make the kill_processes.sh script executable!</p>

<pre><code>  $ chmod +x kill_processes.sh
</code></pre></li>
<li><p>Then use vi or nano (I personally installed vim) to put your twitter app configuration data into the streaming.py script, and to configure the string to be searched inside the Twitter Stream Maelstrom</p>

<pre><code> $ nano streaming.py
</code></pre>

<p> now everything should be ready.</p></li>
<li><p>Open the Arduino IDE and load the TwitterBlink.ino script that is in the Arduino folder (you can download it from GitHub )</p></li>
<li><p>Configure the script folder (in case you created a folder different from root/python/ in the previuos steps)</p>

<p> You can customize the script for your needings, in it&rsquo;s initial configuration it will blink the LED attached to pin 13 (and the built in LED as well) so it&rsquo;s perfect for testing</p></li>
<li><p>Upload the script to Arduino, it should start working as soon as it&rsquo;s uploaded</p></li>
<li><p>Have fun!!</p></li>
</ol><p>cheers!</p>

<p>Jeko</p>