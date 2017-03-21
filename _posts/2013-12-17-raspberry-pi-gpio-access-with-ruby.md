---
ID: 302
post_title: Raspberry PI GPIO access with ruby
author: Andrea Longhi
post_date: 2013-12-17 09:33:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/17/raspberry-pi-gpio-access-with-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/70279085241/raspberry-pi-gpio-access-with-ruby
tumblr_mikamayhem_id:
  - "70279085241"
---
<p>One of the main features of the raspberry pi is the GPIO port, a 21 pins connector where you can attach external hardware and have fun.</p>
<p>You can access the GPIO via the C <a href="http://wiringpi.com/">wiringpi library</a>, which happens to have nice <a href="https://github.com/WiringPi/WiringPi-Ruby">bindings for ruby</a> as well , so let&rsquo;s install it on the raspberry with</p>
<p><code>gem install wiringpi</code></p>
<p>Today we&rsquo;re going to use ruby to monitor a push button that, when pressed, will activate a LED diode.</p>
<p>On the GPIO port we can use pins 0 to 7 either for input or output (see then complete reference for GPIO pins <a href="https://projects.drogon.net/raspberry-pi/wiringpi/pins/">here</a>). We&rsquo;re going to use pin 1 for the button and pin 4 for the LED.</p>
<p>So, the connections from the GPIO port to the breadboard with electric parts are as follows:</p>
<p><img alt="image" src="http://68.media.tumblr.com/e874a74a0c96bf4e54049a25b10568e0/tumblr_inline_mxws7b3UQT1s337n9.jpg" /></p>

<p></p>
<p>Now that the hardware is ready, let&rsquo;s write some code:</p>
<pre>require 'wiringpi'

# initialize the GPIO port:
gpio = WiringPi::GPIO.new
# name the pins, for easier reference:
button = 1
led = 4

# initialize the pin functions:
gpio.mode button, INPUT
gpio.mode led, OUTPUT

# turn off the led, just in case:
gpio.write led, 0

# loop on all the pins, when the button pin has 
# 0 value (button is pressed) the led will turn
# on.
loop do
  state = gpio.readAll
  if state[button] == 0
    gpio.write led, 1
  end    
  sleep 0.3
end
  <br /><br /><br /></pre>

<p><a href="https://www.youtube.com/watch?v=dQw4w9WgXcQ">That&rsquo;s all folks!</a></p>