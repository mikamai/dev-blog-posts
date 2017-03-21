---
ID: 303
post_title: >
  Arduino Yún with SOUND the supereasy
  way
author: Stefano Guglielmetti
post_date: 2013-12-12 09:47:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/12/arduino-yun-with-sound-the-supereasy-way/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69775973742/arduino-yún-with-sound-the-supereasy-way
tumblr_mikamayhem_id:
  - "69775973742"
enclosure:
  - |
    http://s3.amazonaws.com/Jeko/ItCouldWork.mp3
    257913
    audio/mpeg
    
---
<p><span>At the <a href="http://milano.codemotionworld.com/">Codemotion in Milano</a>, I had a chat with Federico Vanzati from Officine Arduino, and he gave me the fantastic idea to try to use a supercheap USB Audio card with Arduino Yún to give to it full audio capabilities with zero effort.</span></p>

<p>And suddenly I was like&hellip;</p>

<p><img alt="image" src="http://68.media.tumblr.com/e1bdc6a2c09bb56003a18a459daea9e4/tumblr_inline_mxow0j61z61r8xdak.gif" /></p>



<p>I mean, it&rsquo;s awesome! It could give me (and hopefully to you) infinite new possibilities! Sound with Arduino with no external libraries or crappy MP3 shields? How great could it be? Now that we have a great Wi-Fi support thanks to the Yún, this was the real missing feature!</p>

<p>I started with buying this sound card (<a href="http://www.manhattan-products.com/hi-speed-usb-3-d-sound-adapter">http://www.manhattan-products.com/hi-speed-usb-3-d-sound-adapter</a>)</p>

<p>Then, all I had to do was plug the sound card in, open an ssh session to the yun, and type:</p>

<pre>opkg update
opkg install kmod-usb-audio
opkg install madplay</pre>

<p>End of the story. Now you have sound support on the Arduino Yún. I only tried MP3 playback, I didn&rsquo;t try to record some audio yet, but I&rsquo;ll try soon!</p>

<p>To test it just copy an mp3 audio file (I named it <a href="http://s3.amazonaws.com/Jeko/ItCouldWork.mp3">ItCouldWork.mp3</a>) in the root folder, or, even better, in the SD card, then upload this sketch</p>

<pre class="prettyprint">#include &lt;Process.h&gt;

const int buttonPin = 2;
const int ledPin =  13;

int buttonState = 0;
Process p;

void setup() {
  Bridge.begin();
  pinMode(ledPin, OUTPUT);      
  pinMode(buttonPin, INPUT);     
  Serial.begin(57600);
}

void loop(){
  buttonState = digitalRead(buttonPin);
  if (buttonState == HIGH) {     
    digitalWrite(ledPin, HIGH);  
    p.runShellCommand("madplay /root/ItCouldWork.mp3");
    while(p.running());  
    Serial.println("it works!");
  } 
  else {
    digitalWrite(ledPin, LOW); 
  }
}

</pre>

<p><br />It&rsquo;s based on the button example, so you have to connect a pushbutton to the digital Pin 2.</p>

<p><img alt="image" src="http://68.media.tumblr.com/5c5b911e082cd45b7cdf94b79947b574/tumblr_inline_mxm8xe7nds1r8xdak.png" /></p>



<p>Now, connect the sound card to the speakers and&hellip; push the button!</p>

<p>Magic, isn&rsquo;t it?</p>

<p>It doesn&rsquo;t sound too bad, actually the sound quality is good! And the delay is very short! I mean, it&rsquo;s just the first experiment, we can work on it!</p>

<p>I really hope this will amaze you as it amazed me&hellip; I&rsquo;m superhappy!</p>

<p>Many thanks to my friend and coworker <a href="https://github.com/wstucco">Massimo</a>, that knows Linux better than I know myself.</p>

<p>Bye!</p>

<p>Jeko</p>