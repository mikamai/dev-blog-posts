---
ID: 284
post_title: 'You Can&#8217;t Touch This! &#8211; An Evil Arduino Based Alarm System'
author: Stefano Guglielmetti
post_date: 2014-02-17 10:48:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/17/you-cant-touch-this-an-evil-arduino-based/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/76945627390/you-cant-touch-this-an-evil-arduino-based
tumblr_mikamayhem_id:
  - "76945627390"
---
<h1>You can&rsquo;t touch this!</h1>

<p><img src="http://i.imgur.com/acHM30xl.jpg" alt="title" /></p>

<p>This was my entry for the <a href="http://makezine.com/connected-home-project-contest/">Connected Home Project Contest</a> by Make Magazine</p>

[youtube https://www.youtube.com/watch?v=a-asHQZwfVU&w=560&h=315]

<p>This project consists in a movement sensor (PIR) plus a camera and a wifi connected device (Arduino Yún was my choice, but it can be easily substitued with a RaspberryPi). Optionally, speakers can be connected to reproduce an alarm sound.</p>

<p>The purpose is clear. You don&rsquo;t want your kids to steal your food from the cupboard, or from the fridge, or someone to open your locket, or you want to take pictures to your pet stealing food, or you are Dwight Schrute and you want to finally unmask the coworker that puts your stuff into jelly&hellip; so you hide the device into the cupboard/fridge/locket, and when the device detects some movement, it will take a picture and post it right to your email! And if you use IFTTT, then you can automatically post the picture of the thief to Facebook, Twitter and show the thief&rsquo;s face to the entire world!</p>

<p>The project is very simple, it doesn&rsquo;t require any soldering/electronics skills and it can be assembled in minutes.</p>

<p>As I said before, my platform choice is Arduino Yún, because it&rsquo;s very easy to use, and it&rsquo;s easy to configure the wifi settings. It&rsquo;s a bit more expensive than the RaspberryPi, but if you use the Yún you can give you project to someone else as a gift, and anyone can configure the wifi settings, while this may be a difficult task with the RaspberryPi, without connecting it to a monitor/mouse/keyboard. But if you like it, I can easily make a Raspberry version.</p>

<p>So, here are the ingredients</p>

<ul><li><a href="http://www.makershed.com/Arduino_Yun_p/mksp24.htm">Arduino Yún</a></li>
<li><a href="http://www.makershed.com/product_p/mkpx6.htm">PIR Sensor</a></li>
<li>UVC compatible USB Camera (don&rsquo;t worry: almost all the USB cameras are UVC compatible)</li>
<li>Micro USB power adapter or USB Battery pack</li>
<li>Recommended: micro SD card </li>
<li>Optional: <a href="http://www.manhattan-products.com/hi-speed-usb-3-d-sound-adapter">USB Sound card</a></li>
<li>Optional: USB HUB (if you want both the USB Camera and the USB Sound card)</li>
</ul><p><img src="http://i.imgur.com/c4Kj6Uil.jpg" alt="totalone" /></p>

<p>Ok, let&rsquo;s start!</p>

<h2>Step 1</h2>

<p><strong>Arduino basic configuration</strong></p>

<p>First of all, you need to configure your Arduino Yún network settings. This should be pretty easy <a href="http://arduino.cc/en/Guide/ArduinoYun#toc13">following this guide</a></p>

<p>Then test the connection to your Arduino: in your browser type <a href="http://arduino.local">http://arduino.local</a>, you will see the Arduino web interface</p>

<p>If it works, open SSH session</p>

<pre><code>$ssh root@arduino.local
</code></pre>

<p>the default password is &ldquo;arduino&rdquo;</p>

<p>Then let&rsquo;s install some useful packages</p>

<pre><code>$opkg update
$opkg install openssh-sftp-server
</code></pre>

<p>Why the openssh-sftp-server? Because it would be easier for you to upload and download files from/to the Arduino: now you can use a generic SFTP client (filezilla, transmit or cyberduck) instead of SCP command</p>

<p>If you have one, I highly recommend to put a micro SD card into the Arduino Yún. It will be automatically mounted in /mnt/sda1</p>

<p>Then, we will install the USB Camera and USB Soundcard on the Yún</p>

<h2>Step 2</h2>

<p><strong>Installing and testing the USB Camera</strong></p>

<p>The UVC package is already available fot Linino via opkg, so installing the Camera is a very easy step, just connect to your Yún via ssh and type</p>

<pre><code>$ opkg update
$ opkg install kmod-video-uvc
</code></pre>

<p>We also need a software for taking pictures, I used fswebcam that is supersmall and very easy to use</p>

<pre><code>$ opkg install fswebcam
</code></pre>

<h2>Step 3</h2>

<p><strong>Take your first picture</strong></p>

<p>First of all, be sure to use and SD card for storage or you will quickly fill the Arduino internal storage, and you don&rsquo;t want to do that. If you are using an SD card, you should see it mounted in /mnt/sda1. If it&rsquo;s not, put a FAT32 formatted micro SD card into the SD Card Slot, and reboot your Arduino</p>

<p>Now plug the camera to the USB host port, and type</p>

<pre><code>$ cd /mnt/sda1
$ fswebcam test.png
</code></pre>

<p>If everything is ok, you took your first picture! Take a look to your SD card contents :)</p>

<p>This means that now we can take pictures from our Arduino sketch, via the Yún&rsquo;s Bridge library, in this way</p>

<pre><code>Process.runShellCommand("fswebcam /mnt/sda1/test.png");
</code></pre>

<h2>Step 4 (optional)</h2>

<p><strong>Install the sound card</strong></p>

<p>Open an ssh session to the yun, and type:</p>

<pre><code>$opkg install kmod-usb-audio
$opkg install madplay
</code></pre>

<p>End of the story. Now you have sound support on the Arduino Yún. I only tried MP3 playback, I didn&rsquo;t try to record some audio yet, but I&rsquo;ll try soon!</p>

<p>To test it just copy an mp3 audio file to the SD card, and type</p>

<pre><code>$cd /mnt/sda1
$madplay yoursound.mp3  
</code></pre>

<p>This means that now we can take play sounds from our Arduino sketch, via the Yún&rsquo;s Bridge library, in this way</p>

<pre><code>Process.runShellCommand("madplay /mnt/sda1/test.mp3");
</code></pre>

<p>We&rsquo;re learning a lot of interesting things!! :)</p>

<h2>Step 5</h2>

<p><strong>The Email script</strong></p>

<p>Now we can take pictures and play sounds&hellip; But we want more from our Arduino! We want to send them via email.</p>

<p>So, how can we do? The answer is very easy&hellip; python script! Even if the Arduino Yún integrates the Temboo library, I wanted this project to be as much portable as possible, so we will use a very simple python script to encode the image file and send it to our email.</p>

<p>I&rsquo;m assuming that you are using GMail, but the script can be easily adapted to any SMTP server</p>

<p>Create a new file, call it &ldquo;sendemail.py&rdquo; and paste this code into it</p>

<pre><code># coding=utf-8

# Copyright (C) 2014  Stefano Guglielmetti

# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.

# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.

# You should have received a copy of the GNU General Public License
# along with this program. If not, see &lt;http://www.gnu.org/licenses/&gt;.

import smtplib, os, sys
from email.MIMEMultipart import MIMEMultipart
from email.MIMEBase import MIMEBase
from email.MIMEText import MIMEText
from email.Utils import COMMASPACE, formatdate
from email import Encoders

#From address, to address, subject and message body
from_address    = 'FROM_ADDRESS@EMAIL.COM'
to_address      = ['YOUR_ADDRESS@EMAIL.COM']
email_subject   = 'Alert!!! Zombies!!! Ahead!!!'
email_body      = 'A non dead intruder has been detected and needs to be eliminated!'

# Credentials (if needed)
username = 'EMAIL_LOGIN'
password = 'EMAIL_PASSWORD'

# The actual mail send
server = 'smtp.gmail.com:587'

def send_mail(send_from, send_to, subject, text, files=[], server="localhost"):
    assert type(send_to)==list
    assert type(files)==list

    msg = MIMEMultipart()
    msg['From'] = send_from
    msg['To'] = COMMASPACE.join(send_to)
    msg['Date'] = formatdate(localtime=True)
    msg['Subject'] = subject

    msg.attach( MIMEText(text) )

    for f in files:
        part = MIMEBase('application', "octet-stream")
        part.set_payload( open(f,"rb").read() )
        Encoders.encode_base64(part)
        part.add_header('Content-Disposition', 'attachment; filename="%s"' % os.path.basename(f))
        msg.attach(part)

    smtp = smtplib.SMTP(server)
    smtp.starttls()
    smtp.login(username,password)
    smtp.sendmail(send_from, send_to, msg.as_string())
    smtp.close()

send_mail(from_address, to_address, email_subject, email_body, [sys.argv[1]], server) #the first command line argument will be used as the image file name
</code></pre>

<p>now you have to customise the email settings</p>

<pre><code>#From address, to address, subject and message body
from_address    = 'FROM_ADDRESS@EMAIL.COM'
to_address      = ['YOUR_ADDRESS@EMAIL.COM']
email_subject   = 'Alert!!! Zombies!!! Ahead!!!'
email_body      = 'A non dead intruder has been detected and needs to be eliminated!'

# Credentials (if needed)
username = 'EMAIL_LOGIN'
password = 'EMAIL_PASSWORD'

# The actual mail send
server = 'smtp.gmail.com:587'
</code></pre>

<p>so change the from address, to address and so on to match your settings</p>

<p>Now you can upload the script to the Arduino Yún SD card via SFTP (or SCP, if you prefer), then open an SSH session to the Yún and test it typing</p>

<pre><code>$cd /mnt/sda1
$python sendemail.py test.png
</code></pre>

<p>And in a few seconds you will receive an email with the picture attachment&hellip; amazing!</p>

<h2>Step 6</h2>

<p><strong>Let&rsquo;s build the circuit!</strong></p>

<p>Don&rsquo;t worry, this is supereasy, there is nothing to solder, just assemble few parts</p>

<p>I used <a href="http://www.makershed.com/product_p/mkpx6.htm">this PIR sensor</a>, because it&rsquo;s reliable and easy to use.</p>

<p>Just follow this scheme</p>

<p><img src="http://i.imgur.com/DyqvJZbl.png" alt="image" /></p>

<p>We&rsquo;ve connected the PIR sensor, and a LED that will turn on when the device detects some movement</p>

<h2>Step 7</h2>

<p><strong>Now it&rsquo;s time to add the Arduino Sketch</strong></p>

<p>Copy this code to the Arduino IDE and upload it to your Yún</p>

<pre><code>/* 
 * Switches a LED, takes a picture and sends it via email
 * according to the state of the sensors output pin.
 * Determines the beginning and end of continuous motion sequences.
 *
 * @author: Stefano Guglielmetti / stefano (at) mikamai (dot) com / <a href="http://jeko.net">http://jeko.net</a>
 * @date:   feb 5, 2014  
 *
 * based on the example by Kristian Gohlke / krigoo (_) gmail (_) com / <a href="http://krx.at">http://krx.at</a>
 * <a href="http://playground.arduino.cc/Code/PIRsense">http://playground.arduino.cc/Code/PIRsense</a>
 *
 * stefano guglielmetti (cleft) 2014 
 *
 * released under a creative commons "Attribution-NonCommercial-ShareAlike 2.0" license
 * <a href="http://creativecommons.org/licenses/by-nc-sa/2.0/de/">http://creativecommons.org/licenses/by-nc-sa/2.0/de/</a>
 *
 *
 * The Parallax PIR Sensor is an easy to use digital infrared motion sensor module. 
 * (<a href="http://www.parallax.com/detail.asp?product_id=555-28027">http://www.parallax.com/detail.asp?product_id=555-28027</a>)
 *
 * The sensor's output pin goes to HIGH if motion is present.
 * However, even if motion is present it goes to LOW from time to time, 
 * which might give the impression no motion is present. 
 * This program deals with this issue by ignoring LOW-phases shorter than a given time, 
 * assuming continuous motion is present during these phases.
 *  
 */

#include &lt;Bridge.h&gt;

/////////////////////////////
//VARS
//the time we give the sensor to calibrate (10-60 secs according to the datasheet)
int calibrationTime = 10;        

//the time when the sensor outputs a low impulse
long unsigned int lowIn;         

//the amount of milliseconds the sensor has to be low 
//before we assume all motion has stopped
long unsigned int pause = 5000;  

boolean lockLow = true;
boolean takeLowTime;  

int pirPin = 6;    //the digital pin connected to the PIR sensor's output
int ledPin = 13;

Process p;
String imageName;

/////////////////////////////
//SETUP
void setup(){
  Bridge.begin();
  Serial.begin(9600);
  pinMode(pirPin, INPUT);
  pinMode(ledPin, OUTPUT);
  digitalWrite(pirPin, LOW);

  //give the sensor some time to calibrate
  Serial.print("calibrating sensor ");
  for(int i = 0; i &lt; calibrationTime; i++){
    Serial.print(".");
    delay(1000);
  }
  Serial.println(" done");
  Serial.println("SENSOR ACTIVE");
  delay(50);
}

////////////////////////////
//LOOP
void loop(){

  if(digitalRead(pirPin) == HIGH){
    digitalWrite(ledPin, HIGH);   //the led visualizes the sensors output pin state
    if(lockLow){  
      //makes sure we wait for a transition to LOW before any further output is made:
      lockLow = false;            
      Serial.println("---");
      Serial.print("motion detected at ");
      Serial.print(millis()/1000);
      Serial.println(" sec"); 

      imageName = uniqueFileName("png"); //generate a new, uniqe file name

      p.runShellCommand("fswebcam /mnt/sda1/" + imageName); //takes the picture
      while(p.running()); //wait till the process ends
      p.runShellCommand("madplay /mnt/sda1/sounds/sirena.mp3"); //play the siren sound
      while(p.running()); //wait till the process ends
      p.runShellCommand("python /mnt/sda1/sendemail.py /mnt/sda1/" + imageName); //sends the picture via email
      while(p.running()); //wait till the process ends

      delay(50);
    }         
    takeLowTime = true;
  }

  if(digitalRead(pirPin) == LOW){       
    digitalWrite(ledPin, LOW);  //the led visualizes the sensors output pin state

    if(takeLowTime){
      lowIn = millis();          //save the time of the transition from high to LOW
      takeLowTime = false;       //make sure this is only done at the start of a LOW phase
    }
    //if the sensor is low for more than the given pause, 
    //we assume that no more motion is going to happen
    if(!lockLow &amp;&amp; millis() - lowIn &gt; pause){  
      //makes sure this block of code is only executed again after 
      //a new motion sequence has been detected
      lockLow = true;                        
      Serial.print("motion ended at ");      //output
      Serial.print((millis() - pause)/1000);
      Serial.println(" sec");
      delay(50);
    }
  }
}


/* A simple function to generate unique timestamp based filenames */

String uniqueFileName(String ext){
  String filename = "";
  p.runShellCommand("date +%s");
  while(p.running()); 

  while (p.available()&gt;0) {
    char c = p.read();
    filename += c;
  } 

  filename.trim();
  filename += "." + ext;

  return filename;
}  
</code></pre>

<p>After 10 seconds (calibration time) it will start working and making pictures!!!</p>

<p>This is the assembled project</p>

<p><img src="http://i.imgur.com/9Vi2xzEl.jpg" alt="assembled" /></p>

<p>All the files of this project are available in <a href="https://github.com/amicojeko/YouCantTouchThis">my GitHub account</a></p>

<p>&hellip;and check the <a href="https://www.youtube.com/watch?v=a-asHQZwfVU">Youtube Video</a> again!</p>

<p>I really hope you enjoyed this project!</p>

<p>Cheers, Stefano</p>