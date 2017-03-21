---
ID: 240
post_title: The Creepy Doll
author: Stefano Guglielmetti
post_date: 2014-06-12 06:58:15
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/12/the-creepy-doll/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/88552602099/the-creepy-doll
tumblr_mikamayhem_id:
  - "88552602099"
---
<p>[youtube https://www.youtube.com/watch?v=rW-KUg9zjrE&w=560&h=315]</p>

<p>Here at <a href="http://www.mikamai.com">Mikamai</a>, we often organise events and hackathons. After the last hackathon, someone left an old doll, and it was kinda creepy… so why not make it even creepier?</p>

<p>I decided to put two red LEDs instead of the eyes, and a vibration sensor to turn on the LEDs when you shake the doll. Everything is powered by an AtTiny85 and a single CR2032 battery.</p>

<h2>Materials and tools</h2>

<p><img src="http://i.imgur.com/7H81cOul.jpg" alt="image" /></p>

<p><strong>Materials:</strong></p>

<ul><li>a doll</li>
<li>an <a href="http://store.arduino.cc/index.php?main_page=index&amp;cPath=11">Arduino</a> (Diecimila, Uno, Leonardo or Yún are ok)</li>
<li><a href="https://www.sparkfun.com/products/9378">AtTiny85</a></li>
<li><a href="https://www.sparkfun.com/products/7937">8 Pin DIP Socket</a></li>
<li>two <a href="https://www.sparkfun.com/products/533">3mm red LEDs</a></li>
<li>a stripboard</li>
<li>a <a href="https://www.sparkfun.com/products/10289">tilt sensor</a></li>
<li><a href="https://www.sparkfun.com/products/783">CR2032 battery holder</a></li>
<li>CR2032 battery</li>
</ul><p><img src="http://i.imgur.com/QhcrdvRl.jpg" alt="image" /></p>

<p><strong>Tools:</strong></p>

<ul><li>A sharp cutter</li>
<li>Hot glue gun</li>
<li>A soldering iron and solder</li>
<li>Needle and thread, to sew it back</li>
<li>optional - a Dremel to drill the eyes</li>
</ul><h2>Prototyping and testing the circuit</h2>

<p><img src="http://i.imgur.com/rbxcB0Xl.png" alt="image" /></p>

<p>Using the AtTiny 85 is a great choice for those who are familiar with Arduino, because you have the possibility to use the Arduino IDE to program it. This means that you can prototype the circuit with Arduino before deploying it to the AtTiny.</p>

<p>So I wrote this code, and I built a simple testing circuit with Arduino.</p>

<pre><code>#import &lt;Arduino.h&gt;


int led = 0; // LEDs pin
int button = 2; // Tilt sensor pin
int brightness = 0;    // how bright the LED is
int fadeAmount = 5;    // how many points to fade the LED by
int storedVal = 0;     // used to save the tilt sensor state

void setup() {
  pinMode(button, INPUT_PULLUP); // initialize the button pin a pullup input, so I don't have to use an external pullup resistor.
  pinMode(led, OUTPUT); // initialize the digital pin as an output.
}


void loop() {

  int sensorVal = digitalRead(2); // Read the sensor state

  if (sensorVal != storedVal) { //if the sensor value has changed, blink the eyes
    storedVal = sensorVal; // store the sensor state
    fadeEyes(); // call the eyes led fade function
  } else {
    digitalWrite(led, LOW); // otherwise, turn the led off
  }

  delay(10); // a small delay for debouncing the sensor
}

void fadeEyes() {

  for (int i = 0; i &lt; 768; i++) { //cycle 3 times

    analogWrite(led, brightness); // set the brightness of led pin:
    if (brightness == 255) { // at maximum brightness, wait 5 seconds
      delay(5000); 
    }
    // change the brightness for next time through the loop:
    brightness = brightness + fadeAmount;

    // reverse the direction of the fading at the ends of the fade:
    if (brightness == 0 || brightness == 255) {
      fadeAmount = -fadeAmount;
    }

    // wait for 30 milliseconds to see the dimming effect
    delay(100);

  }

  digitalWrite(led, LOW);
}   
</code></pre>

<p>The code is pretty simple: it waits for a changement in the tilt sensor state, and when it happens, it starts a little loop fading the leds brightness</p>

<p>When the code works on Arduino, you are ready to deploy it on your AtTiny85</p>

<h2>Moving to AtTiny85</h2>

<p>Programming an AtTiny with an Arduno can be tricky, but fear not! I made <a href="http://dev.mikamai.com/post/78652180658/how-to-program-an-attiny85-or-attiny45-with-an">a very simple tutorial</a> on how to do that with the latest Arduino IDE and Arduino Uno/diecimila or Arduino Leonardo/Yun. Just follow these steps, and you can easily use this sketch on the AtTiny85</p>

<h2>Testing the AtTiny85 based circuit</h2>

<p><img src="http://i.imgur.com/h7sqBJpl.png" alt="image" /></p>

<p><img src="http://i.imgur.com/GSVkxSfl.jpg" alt="image" /></p>

<p>Now we can move the programmed AtTiny on a new breadboard, following this scheme</p>

<p>There is no resistor on the LED because I’m using a 3.3V coin battery. The circuit should start working as soon as you plug the battry: when the tilt sensor is shook, the LED fades :)</p>

<h2>Making the circuit</h2>

<p><img src="http://i.imgur.com/73tFazll.jpg" alt="image" /></p>

<p>Now that everything is tested, we can make the circuit. I decided to use a stripboard because even if it’s a little big, the result will be more stable and short circuit proof. Start placing the components on the stripboard and soldering them, the circuit is supersimple so this should be a very easy task.</p>

<p>I’ve put the LEDs in parallel, I know that I should have used a small resistor, but I was more focused on the simplicity of the project than on its life expectation, plus, it will be powered by a 3.3V battery, so I didn’t really care about it :)</p>

<p>The only precaution was to put some tape on the led wires to avoid short circuits.</p>

<p>Now you can test the circuit alone. If everything is fine, you can move to the next step</p>

<h2>The doll surgery</h2>

<p><img src="http://i.imgur.com/KcpQRlrl.jpg" alt="image" /></p>

<p>With the cutter, remove some stitches from the doll and remove the head’s stuffing. Than drill the holes on the eyes. I used a Dremel for this task, but you can even use the cutter. Now insert the circuit, starting from the eyes. When the led pupils are in the holes, pour a lot of hot glue from the inside of the head, so you will be sure that the LEDs will stay at their place. Then put some more stuffing, the circuit, some more stuffing, the tilt sensor and the rest of the stuffing. In this way you are sure that everything won’t accidentally move and you will reduce the risk of short circuits to the minimum.</p>

<p>I suggest to use some velcro or a couple of buttons to close back the doll, so you can change the battery whanever you want</p>

<h2>You’re done!</h2>

<p><img src="http://i.imgur.com/et1i8wpl.jpg" alt="image" /></p>