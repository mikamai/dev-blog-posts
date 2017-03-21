---
ID: 330
post_title: 'Arduino Yún Gmail Lamp &#8211; Part one'
author: Stefano Guglielmetti
post_date: 2013-09-13 10:06:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/09/13/arduino-yun-gmail-lamp-part-one/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/61101360559/arduino-yún-gmail-lamp-part-one
tumblr_mikamayhem_id:
  - "61101360559"
---
<p>Finally!!! Finally I put my hands on a brand new Arduino Yún. I&rsquo;ve been waiting for this a long, loooong time. I&rsquo;ve been playing with Arduino since the &ldquo;diecimila&rdquo; model came out and I, as a lot of people, always suffered the lack of connectivity and of real computing power. I tried to solve some of these problems using RaspberryPi and/or Electric Imp, but I always missed the Arduino approach&hellip; easy, lots of shields and Arduino ready parts, a lot of documentation, a strong community and the freedom of Open Source.</p>
<div align="center"><img alt="image" src="http://i.imgur.com/X2k2CI4l.jpg" /></div>
<p>Now one of my dreams came true, and every time I go deeper into the discovery of the Yún&rsquo;s capabilities, I find something amazing, very smart and very well done.</p>
<p>I won&rsquo;t describe the platform itself, as many articles talking about that are already published and there will be many more to come. I&rsquo;ll start directly with a real life example, in just a few hours I finally built something really, really useful to me, something I already built several times in various ways but none of which really satisfied me.</p>
<p>The task is pretty simple, and I believe it will be very useful to many people: I need to be alerted in real time when I receive some important emails. Not all the emails: we provide customer care for many clients, with different SLAs, and I need to be alerted only for the most important ones. Moreover, sometimes I look forward to receiving a precise email&hellip; a shipment confirmation, a mail from a special someone&hellip; I need something flexible, eye catching, that doesn&rsquo;t depend on my computer or my cellphone (that always has only 1% battery)</p>
<p>So I decided to build a GMail Lamp and Arduino Yún was the perfect choice (and now that I built it, I can confirm that. It is perfect)</p>
<p>The working principle is very straightforward: On GMail, I defined a new label, so I can quickly change the rules for the messages that will go under it, then I tell to Arduino Yún which label to watch for (via REST APIs&hellip; amazing) and that&rsquo;s it! The lamp (actually only just a led, the lamp will come in the future) turns on every time I get new messages under that label. It&rsquo;s the bat-signal principle! :)</p>
<div align="center"><img alt="image" src="http://i.imgur.com/ZJb97bol.jpg" /></div>
<p>Okay, now let&rsquo;s get a bit more technical&hellip; how did I do it?</p>
<p>The hardware:</p>
<p>- An Arduino Yún, and it must be connected to the internet.<br />- A LED (or a relay if you want to turn on a real lamp, as I will do in the future)<br />- Optional: a MAX7219 based 7 segment LED Display, if you also want to know how many unread messages you got. I used this one,http://dx.com/p/8-segment-led-display-board-module-for-arduino-147814 that&rsquo;s supercheap and works perfectly.</p>
<p>This is the connection scheme (supersimple)</p>
<p>The LED goes on Digital Pin 13<br />The LED Display uses pins 10,11,12 and, obviously, GND and +5V</p>
<div align="center"><img alt="image" src="http://i.imgur.com/eiaMH2Gl.png" /></div>
<p>Ok, the hardware is ready. When you will program the Yún for the first time, even if you can program it over the wifi network, I suggest you use the serial port via USB because it&rsquo;s faster and I still use the serial port to debug (even if you have a brand new Console object :). But it&rsquo;s just a personal choice.</p>
<p>Now, the Code.</p>
<p>Even if it&rsquo;s short, I think it&rsquo;s very interesting because I used many new features of the Yún. I&rsquo;m not going to describe all the code, that you can freely download or fork from GitHub (<a href="https://github.com/amicojeko/Arduino-Yun-Gmail-Check">https://github.com/amicojeko/Arduino-Yun-Gmail-Check</a>). I&rsquo;ll try to describe only the parts that involve brand new code and that are peculiar of the Yún</p>
<p>Let&rsquo;s start from the beginning</p>
<pre>#include &lt;Process.h&gt;</pre>
<p>With the Process library, you can run some code on the Linux side of the Yún and catch the stdout on Arduino. It&rsquo;s amazing because you can delegate to Linux all the dirty jobs and the heavy computing. In this case, I use the Linux &ldquo;curl&rdquo; command to get the ATOM feed of my label from GMail.</p>
<p>The Process library also includes the Bridge library, that allows you to pass information between the two sides of the Yún (Linux and Arduino) using a key/value pairing. And it gives you the power of REST APIs, I use it to configure the label to observe.</p>
<pre>#include &lt;FileIO.h&gt;</pre>
<p>With this library, you can use the inernal memory or a micro SD card/USB key for storage. All these features are native on the Yún!</p>
<pre>#include "LedControl.h" /* Downloaded From <a href="http://playground.arduino.cc/Main/LedControl">http://playground.arduino.cc/Main/LedControl</a> */</pre>
<p>I use this library to control the 7 segment LED Display</p>
<pre>const int ledPin = 13;</pre>
<p>I&rsquo;ve used the pin 13 for the led. As I told you before, you can replace the LED with a relay in order to turn on and off a real lamp!</p>
<pre>const char* settings_file = "/root/gmail_settings"; /* This is the settings file */</pre>
<p>I&rsquo;m saving under &ldquo;/root&rdquo; cause /tmp and /var will be erased at every reboot.</p>
<pre>Bridge.get("label", labelbuffer, 256);</pre>
<p>This is a supercool line of code that uses an übercool Yún&rsquo;s feature. I&rsquo;m telling Arduino to listen for a REST call on the URL <a href="http://arduino.local/data/put/label/LABEL">http://arduino.local/data/put/label/LABEL</a></p>
<p>When I get some data, it will put the value of LABEL in the localbuffer. The localbuffer was initialized like that</p>
<pre>char labelbuffer[256];</pre>
<p>That means that you can actually talk with your Arduino while it runs projects! You can get or put variables, you can finally make dynamic projects! I used it to tell Arduino which label to observe, but I can, and I will go further, I promise.</p>
<pre>label = String(labelbuffer);
File settings = FileSystem.open(settings_file, FILE_WRITE);
settings.print(label);
settings.close();
</pre>
<p>This is cool too. Using the FileIO object, I save the label in a local file on the Linux side of Arduino, so when I will turn it off and on again, It will remember my settings.</p>
<pre>File settings = FileSystem.open(settings_file, FILE_READ);
while (settings.available() &gt; 0){
  char c = settings.read();
  label += c;
}
settings.close();</pre>
<p>This is how I read a file from the filesystem.</p>
<pre>Process p;

p.runShellCommand("curl -u " + username + ":" + password + " "https://mail.google.com/mail/feed/atom/" + label + "" -k --silent |grep -o "&lt;fullcount&gt;[0-9]*&lt;/fullcount&gt;" |grep -o "[0-9]*"");

while(p.running()); // do nothing until the process finishes, so you get the whole output
int result = p.parseInt();
</pre>
<p>This is another bit of Yún&rsquo;s magic. I run the curl command to get the ATOM feed of a specific label, and then I parse it with the grep command, and finally I get the number of unread messages for that label. Even if on the Yún&rsquo;s Linux stack there are both Python and Lua, I thought that this solution was the most simple and stupid, and I love to KISS.</p>
<p>That&rsquo;s it, now i just have to turn the LED on and to display the number of unread messages on the LED Display&hellip;</p>
<p>In a single day I learned how to use the Bridge library to get data from REST webservices, how to save and load data from the Linux filesystem, and how to run processes on the Linux side and get the STDOUT results. I already knew how to use the LED Display but I hope that someone learned something new even about that :)</p>
<p>Now I will build the actual lamp, improving both the Hardware and the Software sides, I will make it gorgeous and fully configurable, and I will keep you informed about that!</p>
<p><br />Cheers to everybody and happy hacking!</p>
<p>Jeko</p>