---
ID: 278
post_title: >
  Let your raspberry PI see this wonderful
  world!
author: Emanuel Carnevale
post_date: 2014-03-03 17:34:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/03/let-your-raspberry-pi-see-this-wonderful-world/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/78453410376/let-your-raspberry-pi-see-this-wonderful-world
tumblr_mikamayhem_id:
  - "78453410376"
---
<p>At <a href="http://mikamai.com">Mikamai</a> we&rsquo;re always testing with new technologies and we&rsquo;ve already expressed our love for the Raspberry PI.</p>

<p>Lately we finally got our hands on a Pi Camera and started using it. Well, the Picamera has been around for a while, but the wait was fruitful since just recently a Python library has been released in the wild!</p>

<p>Raspbian comes with a toolset of useful CLI programs to start grabbing pictures and videos right after unboxing, but a pure Python interface is just perfect for writing Raspberry applications.
Enter <a href="https://github.com/waveform80/picamera">Picamera</a></p>

<p>The setup is really easy:</p>

<p><code>sudo apt-get install python-picamera</code></p>

<p>From grabbing a single picture</p>

<pre><code>#simple grab
import picamera

with picamera.PiCamera() as camera:
    camera.start_preview()
    camera.capture('image.jpg')
    camera.stop_preview()
</code></pre>

<p>to setting up a time lapse, the effort is nil.</p>

<pre><code>#simple time lapse
import time
import picamera

with picamera.PiCamera() as camera:
    camera.start_preview()
    for filename in camera.capture_continuous('img{counter:03d}.jpg'):
        time.sleep(60) # wait 1 minute
</code></pre>

<p>And if we use the GPIO to add an <a href="http://dataissexy.wordpress.com/2013/06/29/raspberry-pi-pir-motion-detection-and-alerting-to-sms-raspberrypi-sms-sensors/">infrared proximity sensor</a> (<a href="http://dev.mikamai.com/post/76945627390/you-cant-touch-this-an-evil-arduino-based-alarm">like we did for Arduino</a>), we can build our cheap alarm camera system!</p>

<p>So procure yourself a camera and then head to the Picamera <a href="http://picamera.readthedocs.org/en/release-1.2/">documentation</a> and show your Raspberry some pictures.</p>