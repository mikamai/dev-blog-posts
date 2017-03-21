---
ID: 277
post_title: >
  How to program an ATTiny85 (or ATTiny45)
  with an Arduino + Arduino ISP + Arduino
  1.5.x
author: Stefano Guglielmetti
post_date: 2014-03-05 15:24:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/05/how-to-program-an-attiny85-or-attiny45-with-an/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/78652180658/how-to-program-an-attiny85-or-attiny45-with-an
tumblr_mikamayhem_id:
  - "78652180658"
---
<p>Yes, this is yet another tutorial on how to program an ATTiny with Arduino, but time changes, things are evolving fast, and tutorials need to be updated.</p>

<p>This is a very short and concise tutorial, so I won&rsquo;t explain what an ATTiny is and why it is useful to program it with an Arduino, as these argument are largely covered. Here you will find instructions on how to program an ATTiny85 (or 45) with an Arduino Leonardo, Arduino Yún, Arduino UNO and Diecimila with the most recent version of the Arduino Software (1.5.x)</p>

<p><img src="http://i.imgur.com/k1ntZ8dl.jpg?1" alt="image" /></p>

<p>You will need:</p>

<ul><li>Arduino Leonardo, Yún, UNO or Diecimila</li>
<li>A Breadboard</li>
<li>Hookup wire</li>
<li>ATtiny85 (or ATtiny45)</li>
<li>10uF 16V electrolytic capacitor (for Arduino UNO or Diecimila)</li>
<li>220ohm &frac14; watt resistor</li>
<li>LED</li>
</ul><h2>Step 1</h2>

<p>First of all, you will need to download the Arduino Tiny plugin from <a href="https://code.google.com/p/arduino-tiny/downloads/detail?name=arduino-tiny-0150-0020.zip&amp;can=2&amp;q=">here</a>, so your Arduino software will be able to program the ATTiny.</p>

<p>If you want to make it work with Arduino 1.0.x software, just download <a href="https://code.google.com/p/arduino-tiny/downloads/detail?name=arduino-tiny-0100-0018.zip&amp;can=2&amp;q=">this file</a> instead!</p>

<p>Then unzip it and copy the &ldquo;Tiny&rdquo; folder into an &ldquo;hardware&rdquo; folder that you have to create into your sketches folder.</p>

<p>The final result sholud be</p>

<p><strong>arduino_sketches_folder/hardware/tiny</strong></p>

<p>Inside the &ldquo;tiny&rdquo; folder, you will find an &ldquo;avr&rdquo; folder. Inside this folder, there is a file named &ldquo;Prospective Boards.txt&rdquo;. Just rename it into &ldquo;boards.txt&rdquo;</p>

<p>When you will start the Arduino Software, you will find new boards under the <strong>Tools &gt; Board</strong> menu, as in this picture.</p>

<h2>Step 2</h2>

<p>Now we have to make the proper connections between the Arduino Board and the ATTiny</p>

<p><img src="http://i.imgur.com/CmYEOwC.png" alt="image" /></p>

<h3>Arduino UNO/Diecimila</h3>

<pre><code>Arduino PIN 10  --&gt; ATTiny Reset PIN 
Arduino PIN 11  --&gt; ATTiny PIN 0
Arduino PIN 12  --&gt; ATTiny PIN 1
Arduino PIN 13  --&gt; ATTiny PIN 2
Arduino GND PIN --&gt; ATTiny PIN GND PIN
Arduino +5V PIN --&gt; ATTiny PIN VCC PIN
</code></pre>

<p>as in the picture below</p>

<p><img src="http://i.imgur.com/VhUXVNgl.jpg" alt="image" /></p>

<h3>or for Arduino Leonardo/Yún</h3>

<pre><code>Arduino PIN 10          --&gt; ATTiny Reset PIN
Arduino ICSP MOSI PIN   --&gt; ATTiny PIN 0
Arduino ICSP MISO PIN   --&gt; ATTiny PIN 1
Arduino ICSP SCK PIN    --&gt; ATTiny PIN 2
Arduino GND PIN         --&gt; ATTiny PIN GND PIN
Arduino +5V PIN         --&gt; ATTiny PIN VCC PIN
</code></pre>

<p><img src="http://i.imgur.com/tG26dSul.jpg" alt="image" /></p>

<h2>Step 3</h2>

<p>Open the <strong>File &gt; Examples &gt; ArduinoISP</strong> Sketch and upload it on your Arduino</p>

<h3>Arduino Leonardo/Yún</h3>

<p>Only fo these platforms, you have to make this changements to the Arduino ISP sketch, BEFORE uploading it</p>

<pre><code>// January 2008 by Randall Bohn
// - Thanks to Amplificar for helping me with the STK500 protocol
// - The AVRISP/STK500 (mk I) protocol is used in the arduino bootloader
// - The SPI functions herein were developed for the AVR910_ARD programmer 
// - More information at <a href="http://code.google.com/p/mega-isp">http://code.google.com/p/mega-isp</a>

#include "pins_arduino.h"
#define RESET     10

#define LED_HB    13
#define LED_ERR   8
#define LED_PMODE 7
#define PROG_FLICKER true
</code></pre>

<p>Now upload it to your Arduino.</p>

<h2>Step 4</h2>

<p>Load the <strong>File &gt; Examples &gt; 01.Basic &gt; Blink</strong> Sketch, and change this line</p>

<pre><code>int led = 13;
</code></pre>

<p>to</p>

<pre><code>int led = 0;
</code></pre>

<p>this is because the ATTiny has only 6 output pins (0,1,2,3,4,5) and so we will use the pin 0 instead of the pin 13</p>

<h2>Step 5</h2>

<p>Choose the proper board settings (i.e. Attiny85 @ 1Mhz), then choose <strong>Tools &gt; Programmer &gt; Arduino as ISP</strong>. Now you can upload the &ldquo;blink&rdquo; example to the ATTiny85</p>

<p>If everything went fine, you will see something like</p>

<pre><code>avrdude: please define PAGEL and BS2 signals in the configuration file for part ATtiny85
avrdude: please define PAGEL and BS2 signals in the configuration file for part ATtiny85
</code></pre>

<p>in the output window of the Arduino Software. Don&rsquo;t care about the warnings, they&rsquo;re normal.</p>

<p>If something went wrong, you will find a message like</p>

<pre><code>avrdude: please define PAGEL and BS2 signals in the configuration file for part ATtiny85
avrdude: Yikes!  Invalid device signature.
         Double check connections and try again, or use -F to override
         this check.
</code></pre>

<p>This means that you have to check your connections again, it happend a lot to me the first time I tried to use an Arduino Leonardo instead of an Arduino UNO, because the ICSP Pins are mapped in a different way on the Leonardo/Yún boards.</p>

<h2>Step 6</h2>

<p>Now we can test it! To build a simple test circuit, for the &ldquo;blink&rdquo; example, just build this simple circuit</p>

<p><img src="http://i.imgur.com/j1TorCzl.png" alt="image" /></p>

<p>When you power up the ATTiny85, the LED sholud start blinking.</p>

<p>I hope this tutorial is useful to you as it would be useful to me if I was able to find one: I got more then an headache trying to put all the pieces together: the proper plugins, the connections and the little software modifications&hellip;</p>

<p>Cheers!</p>

<p>Jeko</p>