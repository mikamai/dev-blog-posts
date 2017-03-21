---
ID: 125
post_title: Spark Core tips and tricks
author: Emanuele Serafini
post_date: 2015-05-13 10:18:34
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/13/spark-core-tips-and-tricks/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/118854354149/spark-core-tips-and-tricks
tumblr_mikamayhem_id:
  - "118854354149"
---
<p><a href="https://www.spark.io/">Spark Core</a>&rsquo;s team  has built an incredible WEB IDE (<a href="https://build.spark.io/">Spark Build</a>) to allow you to develop your application directly from your browser. When your Spark Core is connected to the internet you can deploy your code directly from Spark Build.</p><figure class="tmblr-full"><img src="http://68.media.tumblr.com/bf3e346cae321c778b47afcdc10f5aa2/tumblr_inline_noa7xr8G5Z1qfdn20_540.jpg" /></figure><p>Spark Build includes the Spark Apps section, which displays the name of the current app in your editor, as well as a list of your other applications and community-supported example apps.</p><p>Spark Build is a powerful tool and it&rsquo;s perferct for beginners; but you’d surely prefer to use your beloved text editor, thus being a lot faster.</p><p>Here some tips to improve your experience:</p><p><b>Use Spark CLI</b></p><p>With Spark CLI you can interacts with your cores directly from your terminal. First make sure you have node.js installed. Then open your command prompt or terminal and enter:</p><pre><code>npm install -g spark-cli
</code></pre><p><b>Organize your code</b></p><p>Build a basic structure for your projects. Your first applications will be a large copy/paste of code found on internet, but with the right organization you can reuse library and easily find the code you have written. For example:</p><figure><img src="http://68.media.tumblr.com/966bfaaca0c755718ca58581831e504c/tumblr_inline_noa7xgyqLa1qfdn20_540.jpg" /></figure><p>In this example I&rsquo;ve divided Spark Core dev folder in 3 subfolders <code>apps, firmware, libraries</code>.</p><p><b>Compile and Flash from command line</b></p><p>Spark CLI gives you the ability to compile your project with all the dependent library files. For example if you want to compile <code>thermostat_relay.ino</code> from the above example you can run:</p><pre><code>spark compile thermostat_relay.ino ../libraries/Adafruit_DHT.cpp ../libraries/Adafruit_DHT.h 
../libraries/HttpClient.cpp ../libraries/HttpClient.h 
--saveTo ../firmware/firmware_thermostat_relay.bin 
</code></pre><p>This command takes your library files as arguments and <code>--saveTo</code> defines where the generated firmware will be saved.</p><p>If build succeded you can run command to flash firmware on Spark Core:</p><pre><code>spark flash XXXXXXXXXXXX ../firmware/firmware_thermostat_relay.bin
</code></pre><p>Replace <code>XXXXXXXXXXXX</code> with your core id.</p><p><b>Serial outputs</b></p><p>When your Spark Core is connected to your computer through usb you can debug it using the serial port. In my experience this is a good way to start and understand what is going on under the hood on your core.</p><pre><code>spark serial monitor /dev/cu.usbmodem1421
</code></pre><p>Keep in mind that the serial port path can change. To discover your current connected Spark Core just run:</p><pre><code>spark serial list
</code></pre>