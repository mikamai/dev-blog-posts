---
ID: 328
post_title: 'The &#8220;Facebook&#8217;s Like&#8221; Alarm'
author: Stefano Guglielmetti
post_date: 2013-10-24 12:08:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/10/24/the-facebooks-like-alarm/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/64949879226/the-facebooks-like-alarm
tumblr_mikamayhem_id:
  - "64949879226"
---
<p>Hi there! Ever wanted to know in real time if someone likes your facebook page?</p>
<p><img src="http://68.media.tumblr.com/246c265275402103ea312edf48d9d482/tumblr_inline_mvfo6qgwdB1r8xdak.jpg" /></p>

<p></p>
<p>We’ve got the solution for you! Here, in Mikamai, as most companies do, we have a Facebook page, and we are very proud of it. We wanted to have some kind of physical feedback in our office, to let us know in real time when we get a new fan.</p>
<p>So, that&rsquo;s how we did it.</p>
<p>First of all, the device. We needed to have it on 24/7, it should be very cheap, low powered, and we wanted to have the possibility to add new features (audio, video, lights, physical devices ecc&hellip;) so the perfect choice was the RaspberryPI</p>
<p>Second, the programming language. The obvious choice for us is Ruby :)</p>
<p>Third&hellip; what will happen? We wanted something simple, effective and not too annoying, so we choose a simple sound, the Super Mario coin sample was perfect for us.</p>
<p>The set up is very very simple.</p>
<p>We started froma a brand new RaspberryPI, and we installed Ruby with RVM, following these easy steps</p>
<p><a href="https://rvm.io/rvm/install">https://rvm.io/rvm/install</a></p>
<p>everything worked like a charm.</p>
<p>Then, we put this simple script in /home/pi/like_it/</p>
<pre>    
require 'rubygems'
require 'net/http'
require 'json'

LIKES_FILE = "likes"
PLAYER = (RUBY_PLATFORM == "armv6l-linux-eabihf") ? "omxplayer" : "mpg123"

while true do
  begin
    response = Net::HTTP.get_response("graph.facebook.com","/mikamai")
    json = JSON.parse(response.body)

    old_likes =  File.open(LIKES_FILE, 'r').gets.to_i
    likes = json["likes"].to_i

    if (likes &gt; old_likes)
      #fixme: with mpg123 the -o local option (obviously) returns an error, it's non blocking but could be fixed
      system(PLAYER + " coin.mp3 -o local")
      p "like"
    end

    File.open(LIKES_FILE, 'w') { |file| file.write(likes) }

    sleep (10)
  rescue
    p "ERROR!!! YOU SHOULD INVESTIGATE"
  end
end
</pre>
<p><br />If you are using GIT (You have to install GIT on your RaspberryPI as well) simply go in the home directory via ssh and checkout the project from <br /><br /><strong><a href="https://github.com/amicojeko/Like-It">https://github.com/amicojeko/Like-It</a></strong></p>
<p>In the Github project we also added a simple start script</p>
<pre>#! /bin/sh

DIR=/home/pi/like_it
cd $DIR

# stdout goes in trash, stderr gets logged 
rvm-auto-ruby like_it.rb 1&gt; /dev/null 2&gt; $DIR/logs/like_it_error &amp;
</pre>
<p>the coin sample file, the (empty) log files for errors and stdout, and the counter file (using a database for this project is a bit overkill)</p>
<p>and&hellip; that&rsquo;s it!</p>
<p>The script is supersimple, it runs an infinite loop and checks for new likes every 10 seconds (since the real time API for likes is not available yet). It just calls the Facebook Open Graph Public API, it searches for the &ldquo;likes&rdquo; counter in the JSON file, and confronts it with the likes previously stored in the &ldquo;likes&rdquo; file. If the likes are increasing, it plays the audio file.</p>
<p>Simple, stupid and sooo effective :)</p>
<p>To make it start automatically everytime the RaspberryPI reboots, jus add this in the /etc/rc.local file</p>
<pre># starts like-it daemon as pi user
su pi -c '/home/pi/like_it/start'
</pre>