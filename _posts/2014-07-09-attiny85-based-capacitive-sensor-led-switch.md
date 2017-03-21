---
ID: 228
post_title: >
  AtTiny85 based capacitive sensor LED
  switch
author: Stefano Guglielmetti
post_date: 2014-07-09 17:32:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/09/attiny85-based-capacitive-sensor-led-switch/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/91267125014/attiny85-based-capacitive-sensor-led-switch
tumblr_mikamayhem_id:
  - "91267125014"
---
<p>Not so much time ago, I met a girl, and she designs ceramic lamps. </p>
<p>
<img src="http://68.media.tumblr.com/3acf464daf4e66c71ce88d221f2263a7/tumblr_inline_n8ghitzo3E1r8xdak.jpg" /></p>
<p>
<a href="http://www.viondesign.com">viondesign.com</a>
</p>

<p>I mean, the lamps are really, really gorgeous, and they are illuminated by a 12V LED module.
</p>
<p>
As soon as I saw one of these lamps, I decided to make some experiments with them, because they are so beautiful that the classic ON/OFF switch did not do justice to them.
</p>
<p>
Recently, I studied the Arduino <a href="http://playground.arduino.cc/Main/CapacitiveSensor?from=Main.CapSense">capsense library</a> for other purposes, and I thought that a capacitive sensor would have been a very nice way to turn the lamp on and off without the use of a regular switch.
</p>
<p>
The basic idea was to paint the inside of the lamp with some <a href="http://www.bareconductive.com/shop/electric-paint-50ml/">conductive paint</a> and use the lamp itself as a switch.
</p>
<p>
So, here&rsquo;s what I did.
</p>
<p>
First of all, the prototyping. The best thing about AtTinys is that you can use Arduino for prototyping, and then, with very small code adjustments, burn the code into the AtTiny, make the circuit supersmall, and go to production.
</p>
<p>
I started from the basic <a href="http://playground.arduino.cc/Main/CapacitiveSensor">Capacitive Sensor</a> library example. 
</p>
<p>
It worked nice, but the reading was not very stable, and since I linked the LED intensity directly to the sensor reading, the LED was flickering a lot (as you can see in this video)
</p>
<p>
[youtube https://www.youtube.com/watch?v=iJm6xpb27X4&w=420&h=315]
</p>
<p>
So I followed the instructions, and put a very small capacitor (20pF) between the sensor (an aluminium foil for prototyping) and ground. That helped a lot, but it wasn&rsquo;t enough.
</p>
<p>
Then I decided to add some <a href="http://arduino.cc/en/Tutorial/Smoothing">smoothing</a> code to my sketch.. Open source code is soooo cool :) You always find someone that can really help you :)
</p>
<p>
The original code takes the average reading between 100 readings, but since the AtTiny is much slower than the Arduino (1Mhz vs 16 Mhz) I decided to make an average reading between 10 values, and it worked great for me.
</p>
<p>
Ok, now the LED turns on and off when my hand is close to the sensor. But I wanted it to stay on or off, like a regular switch does. So I put this code, that works in this way: if the sensor reaches the maximum value (when I touch it), then I switch the lamp on/off and I stop the reading for a second.
</p>
<p>
So this is the final code for Arduino.
</p>

<pre><code>#include &lt;CapacitiveSensor.h&gt;

int cap_pin_out = 4;
int cap_pin_in = 2;
int lowcap = 300;  // just above reading when noting is near
int highcap = 1800; // cap reading when almost touching
CapacitiveSensor   capsense = CapacitiveSensor(cap_pin_out, cap_pin_in);  // 10M resistor between pins 1 &amp; 2, pin 2 is sensor pin, add a wire and or foil if desired
int ledPin = 13;
int dur = 10; //duration is 10 loops
int brightness;
bool isOn = 0;

/* smoothing */
const int numReadings = 10;
int readings[numReadings];      // the readings from the analog input
int index = 0;                  // the index of the current reading
long total = 0;                  // the running total
int average = 0;                // the average

void setup()
{
  pinMode(ledPin, OUTPUT); // output pin
  pinMode(cap_pin_in, INPUT); // output pin
  pinMode(cap_pin_out, OUTPUT); // output pin

  // initialize all the readings to 0:
  for (int thisReading = 0; thisReading = numReadings)
    // ...wrap around to the beginning:
    index = 0;

  // calculate the average:
  average = total / numReadings;
  // send it to the computer as ASCII digits

  long start = millis();

  brightness = map(average, lowcap, highcap, 0, 255);
  brightness = constrain(brightness, 0, 255);

  if (isOn == true &amp;&amp; brightness == 255) {
    isOn = false;
    delay(500);
  } else if (isOn == false &amp;&amp; brightness == 255) {
    isOn = true;
    delay(500);
  }

  if (isOn == true) {
    analogWrite(ledPin, 255);
  } else {
    analogWrite(ledPin, brightness);
  }

  delay(10);

}</code></pre>
<p>
At this point I had a couple of things left to do: first of all, to control an hi-power 12V led, instead of a 5V one, and, second, try to power the circuit with the 12V LED power supply.
</p>
<p>
To control an 12V LED, I used a TIP102, that is a really heavy duty affordable NPN darlington transistor. It&rsquo;s perfect to control LED strips, hi power LEDs and LED modules. The connection is really easy, it just needs an 1K resistor and this simple scheme
</p>
<p>
<img src="http://68.media.tumblr.com/6a335600a11435f1ad145c82a4e2d224/tumblr_inline_n8gf3gjwxs1r8xdak.png" /></p>
<p>
Since the ATTiny can be powered at 5V, and I did not want to use two power supplies, I decided to use <a href="http://www.tme.eu/html/EN/dcdc-converters-in-sil-4-pin-case-dc12s-series/ramka_1102_EN_pelny.html">this 12v to 5v voltage converter</a>. It&rsquo;s very simple to use: just connect the IN Pins to the 12V and you get 5V from the output pins (yes, I know, I should have used a capacitor for stability, but the AtTiny is very forgiving and works perfectly even without it)
</p>
<p>
Ok now that everything was perfect, I burned the code to the AtTiny. If you need some instructions on how to do that, here&rsquo;s a step by step tutorial I wrote some time ago.
</p>
<p>
I had to make some minor adjustments to the code: I changed the sensor pins form 4,2 to 1,2 and the LED pin from 13 to 0&hellip; and boom! we&rsquo;re ready to change platform! :)
</p>
<p>
So here&rsquo;s the schematics for the AtTiny85
</p>
<p>
<img src="http://68.media.tumblr.com/8b1b6955818c8a905b504dbc7b829587/tumblr_inline_n8gfpapUKp1r8xdak.png" /></p>
<p>
The circuit is pretty simple and can be easily assembled on a protoboard.
</p>
<p>
I painted the inside of the lamp with this paint, so now the inside of the skull is conductive and can be used as a capacitive sensor.
</p>
<p>
To connect the sensor wire to <a href="http://www.tme.eu/en/details/33_200/protection-screen-coatings/kontakt-chemie/207606091202/#">this paint</a>, I used some <a href="http://www.bareconductive.com/">BareConductive electric paint</a>
</p>
<p>
The assembled circuit was so tiny that fitted perfectly inside the skull
</p>
<p>
A little hint: when using capacitive sensor, is very important to ground the circuit, so be sure to have a ground connection between the circuit, the power supply, and the mains ground.
</p>
<p>
Here is the final result&hellip; isn&rsquo;t it nice? :)
</p>
<p>
[youtube https://www.youtube.com/watch?v=Edw7Iva594Y&w=420&h=315]
</p>
<p>
Ciao!
</p>
<p>
Jeko
</p>



Another important thing with capacitive sensors is grounding.