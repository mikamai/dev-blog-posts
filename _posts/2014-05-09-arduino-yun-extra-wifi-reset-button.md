---
ID: 253
post_title: Arduino Yún Extra WiFi Reset Button
author: Stefano Guglielmetti
post_date: 2014-05-09 13:44:40
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/09/arduino-yun-extra-wifi-reset-button/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/85216465159/arduino-yún-extra-wifi-reset-button
tumblr_mikamayhem_id:
  - "85216465159"
---
<p>One of the biggest problems with Arduino Yún is that the WiFi reset button is embedded on the board, and it&rsquo;s very difficult to reach. Once you have packed you project in an enclosure, there&rsquo;s no way to reset the wifi settings, unless the wifi reset button is reachable through a pinhole.</p>

<p>But how does the integrated wifi reset button work? When it&rsquo;s pressed for more than 5 seconds, Arduino calls a little script: /usr/bin/wifi-reset-and-reboot</p>

<p>So I&rsquo;ve made a very simple sketch that does exactly the same: it waits for our extra reset button to be pressed for more than 5 seconds, and when it happens it calls the same exact script using the Yún&rsquo;s Bridge library.</p>

<p>So you can add your extra wifi reset button, and put it where you want!</p>

<p>Here&rsquo;s the code</p>

<pre><code>#include &lt;Process.h&gt;

const int  buttonPin = 7;  // the pin that the pushbutton is attached to
const int ledPin = 13;     // the pin that the LED is attached to

int buttonPushTimer = 0;   // timer for the reset button
int buttonState = 0;       // current state of the button
int lastButtonState = 0;   // previous state of the button

void setup() {
  Bridge.begin();
  pinMode(buttonPin, INPUT);
  pinMode(ledPin, OUTPUT);
}

void loop() {
  // read the pushbutton input pin:
  buttonState = digitalRead(buttonPin);
  // compare the buttonState to its previous state
  if (buttonState != lastButtonState) {
    // if the state has changed, start the timer
    if (buttonState == LOW) {
      buttonPushTimer = millis();
    }
  }
  else {
    // if the state has changed, and the button is still pressed, check the timer
    if (buttonState == LOW) {
      if (millis() &gt; buttonPushTimer + 5000) {
        // if the button has been pressed for 5 seconds, reset arduino wifi
        wifi_reset();
      }
    }
    // Delay a little bit to avoid bouncing
    delay(50);
  }
  lastButtonState = buttonState;
}



void wifi_reset() {
  //blink led 10 times
  for (int i = 0; i &lt; 10; i++) {
    digitalWrite(ledPin, HIGH);
    delay(50);
    digitalWrite(ledPin, LOW);
    delay(50);
  }
  // call the wifi reset script
  Process p;
  p.runShellCommand("wifi-reset-and-reboot");
}
</code></pre>

<p>I Hope it was useful!</p>

<p>Happy resetting!</p>

<p>Jeko</p>