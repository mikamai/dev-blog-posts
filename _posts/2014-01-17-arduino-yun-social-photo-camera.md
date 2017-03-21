---
ID: 297
post_title: Arduino Yún social photo camera
author: Stefano Guglielmetti
post_date: 2014-01-17 14:21:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/17/arduino-yun-social-photo-camera/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/73613525813/arduino-yún-social-photo-camera
tumblr_mikamayhem_id:
  - "73613525813"
---
<p>After making an <a href="http://dev.mikamai.com/post/69775973742/arduino-yun-with-sound-the-supereasy-way">USB Audio card work with Arduino Yún</a>, the logical second step was try to use an USB camera instead, and it worked! I was so amazed by this new huge set of possibilitities to create, that I started a little weekend project: to build an Arduino based Facebook photo camera</p>

<p>I was so into this project that I didn&rsquo;t realize that, even if the single steps are very simple, they&rsquo;re quite a lot! So I decided to write this step-by-step tutorial and to split it into &hellip; articles.</p>

<p>Just to be clear:</p>

<p><strong>The goal</strong></p>

<p><em>To build photocamera that shoots pictures and publishes them directly to a specific Facebook Album.</em></p>

<h2>Ingredients</h2>

<ul><li>Arduino Yún</li>
<li>USB camera (this guide is for UVC compatible cameras)</li>
<li>Pushbutton circuit (<a href="http://arduino.cc/en/tutorial/button">http://arduino.cc/en/tutorial/button</a>)</li>
<li>SD Card</li>
<li>Facebook developers account (<a href="https://developers.facebook.com/">https://developers.facebook.com/</a>)</li>
<li>Temboo account (<a href="https://temboo.com/">https://temboo.com/</a>)</li>
<li>Working, brand new Facebook Application (we&rsquo;ll create one during the tutorial)</li>
<li><a href="https://github.com/amicojeko/ArduinoUSBCamera/blob/master/ArduinoUSBCamera.ino">This Arduino Sketch</a></li>
<li><a href="https://github.com/amicojeko/ArduinoUSBCamera/blob/master/sendphoto.py">This Python script</a></li>
</ul><h2>The Steps</h2>

<ol><li>Install the USB Camera on Arduino Yún</li>
<li>Take your first picture</li>
<li>Create the Facebook Application</li>
<li>Post your first picture with the Temboo API</li>
<li>Install the Temboo SDK on Arduino Yún</li>
<li>Make the python script and publish your first photo to Facebook from Arduino</li>
<li>Create the Arduino sketch</li>
<li>Put everything together</li>
</ol><h3>1. Install the USB Camera on Arduino Yún</h3>

<p>I assume that your Arduino Yún is already configured for wi-fi (or connected via the Ethernet port), and that you have SSH access to it, if not, follow <a href="http://arduino.cc/en/Guide/ArduinoYun#toc13">this link</a> to configure the wifi network and <a href="http://arduino.cc/en/Tutorial/LinuxCLI">this link</a> to connect via SSH.</p>

<p>The UVC package is already available fot Linino via opkg, so installing the Camera is a very easy step, just connect to your Yún via ssh and type</p>

<pre><code>$ opkg update
$ opkg install kmod-video-uvc
</code></pre>

<p>We also need a software for taking pictures, I used fswebcam that is supersmall and very easy to use</p>

<pre><code>$ opkg install fswebcam
</code></pre>

<p>Now we can test it!</p>

<h3>2. Take your first picture</h3>

<p>First of all, be sure to use and SD card for storage or you will quickly fill the Arduino internal storage, and you don&rsquo;t want to do that. If you are using an SD card, you should see it mounted in /mnt/sda1. If it&rsquo;s not, put a FAT32 formatted micro SD card into the SD Card Slot, and reboot your Arduino</p>

<p>Now plug the camera to the USB host port, and type</p>

<pre><code>$ cd /mnt/sda1
$ fswebcam test.png
</code></pre>

<p>If everything is ok, you took your first picture!</p>

<p>This means that now we can take pictures from our Arduino sketch, via the Yún&rsquo;s Bridge library, in this way</p>

<pre><code>Process.runShellCommand("fswebcam /mnt/sda1/test.png");
</code></pre>

<p>Now we will learn how to share them on Facebook using Temboo.</p>

<h3>3. Create the Facebook Application</h3>

<p>This is easy, but you must be a Facebook developer to do that. That simply means that you have to accomplish <a href="https://developers.facebook.com/docs/create-developer-account/">these easy steps</a> before starting. Done? ok! So now let&rsquo;s create a blank Facebook Application</p>

<p><img src="http://i.imgur.com/z3SG4sq.png" alt="image creation" /></p>

<p>and configure it in this way:</p>

<p><img src="http://i.imgur.com/QCTJR6T.png" alt="configuration" /></p>

<h3> Create a Temboo Application</h3>

<p>First of all, register on Temboo. When you&rsquo;re in, create a new application, then go to <a href="https://temboo.com/library/Library/Facebook/Publishing/UploadPhoto/">https://temboo.com/library/Library/Facebook/Publishing/UploadPhoto/</a></p>

<p>add new credentials for the new Facebook app.</p>

<p><img src="http://i.imgur.com/mxrJtlU.png" alt="temboo config" /></p>

<p>Choose oAuth wizard, go directly to the third step and copy and paste the app id &amp; app secret from your Facebook application.</p>

<p>Go to the next step, click on launch authorisation and grant the authorisation to the Facebook application.</p>

<p>Now you have the access token</p>

<p>Click on use these values and save, copy and paste the access token in the access token form field if needed.</p>

<p>Now you need the target album id. to get an album id, just go on Facebook, open an album (not a single photo) and the URL will look like this.</p>

<p><img src="http://imgur.com/RElG4Zg.png" alt="albumid" /></p>

<p>The album id is the highlighted part. copy and paste it into the AlbumID field on Temboo</p>

<p>I also choose to open the Optional Input section and to put a custom message :)</p>

<p><img src="http://i.imgur.com/HKNU5sX.png" alt="message" /></p>

<p>If you take a look to the sample code, Temboo has made pretty much all the job for you, but we need to make some changes, we will make them later. Now we need some gratification, so let&rsquo;s post our first picture!</p>

<h3>4. Post your first picture with the Temboo API</h3>

<p>You already put your Access Token and the desired AlbumID. The only thing that&rsquo;s missing is the Base64 Encoded image. So go to <a href="http://www.freeformatter.com/base64-encoder.html">http://www.freeformatter.com/base64-encoder.html</a> and paste the URI of the image you want to encode, then copy the encoded string and paste it into the Temboo interface.</p>

<p><img src="http://i.imgur.com/DT1Ij8U.png" alt="http://i.imgur.com/DT1Ij8U.png" /></p>

<p>Click TRY IT OUT and voilà, the image it&rsquo;s magically posted on your album!</p>

<h3>5. Install the Temboo SDK on Arduino Yún</h3>

<p>We will use the Temboo API from a Python script, so we need to download the SDK. The download page is here <a href="https://temboo.com/download">https://temboo.com/download</a>, all we have to do is download the Python SDK and unzip it into the root folder of your Micro SD card (it will unzip into a /temboo folder).</p>

<p>A very important step is installing OpenSSL support for Python, as the Temboo libraries will try to use SSL to publish the photos.</p>

<pre><code>$ opkg update
$ opkg install python-openssl
</code></pre>

<h3>6. Make the python script and publish your first photo to Facebook from Arduino</h3>

<p>As I wrote before, the generated code from Temboo is a good starting point, but we need to make some modifications in order to make it work.</p>

<pre><code># Instantiate the Choreo, using a previously instantiated TembooSession object, eg:
# session = TembooSession('jeko', 'APP_KEY_NAME', 'APP_KEY_VALUE')
uploadPhotoChoreo = UploadPhoto(session)

# Get an InputSet object for the choreo
uploadPhotoInputs = uploadPhotoChoreo.new_input_set()

# Set credential to use for execution
uploadPhotoInputs.set_credential('ArduinoCamera')

# Set inputs
uploadPhotoInputs.set_Message("This photo has been shot and published with an Arduino Yún")
uploadPhotoInputs.set_AlbumID("")

# Execute choreo
uploadPhotoResults = uploadPhotoChoreo.execute_with_results(uploadPhotoInputs)
</code></pre>

<p>Download <a href="https://github.com/amicojeko/ArduinoUSBCamera/blob/master/sendphoto.py">my version of the script</a> and copy it into the root folder of the SD card. Here&rsquo;s what I did.</p>

<p>Step number one, the encoding. I&rsquo;m italian and we use a lot of accented characters, so I&rsquo;ll use UTF-8 as the default script encoding, so I added</p>

<pre><code># coding=utf-8
</code></pre>

<p>at the top of my script. Now we need to read the photo from the disk, and to base64 encode it, so we will need the base64 and the sys libraries.</p>

<pre><code>import base64
import sys
</code></pre>

<p>The Temboo libraries are not included in the generated script by Temboo, so we will have to add them</p>

<pre><code>from temboo.core.session import TembooSession
from temboo.Library.Facebook.Publishing import UploadPhoto
</code></pre>

<p>The script will read the first argument, and use it as the name of the image file to encode</p>

<pre><code>with open(str(sys.argv[1]), "rb") as image_file:
    encoded_string = base64.b64encode(image_file.read())
</code></pre>

<p>In this line you have to put your Temboo&rsquo;s account name, and the app name and key (from <a href="https://temboo.com/account/applications/">this page</a>)</p>

<pre><code>session = TembooSession('ACCOUNT_NAME', 'APP_KEY_NAME', 'APP_KEY_VALUE')
</code></pre>

<p>You&rsquo;ll also need to put yout access token, message (if you want one) and Album ID into the script</p>

<pre><code>uploadPhotoInputs.set_AccessToken("ACCESS_TOKEN")
uploadPhotoInputs.set_Message("MESSAGE")
uploadPhotoInputs.set_AlbumID("ALBUM_ID")   
</code></pre>

<p>The rest of the script is basically identical to the Temboo&rsquo;s example.</p>

<p>We can now try to publish our first photo with the script!</p>

<pre><code>$fswebcam testpic.jpg
$python sendphoto.py testpic.jpg 
</code></pre>

<p>At this point I strongly hope that it&rsquo;s working for you as it&rsquo;s working for me (otherwise contact me and we will try to solve the problem together) because you have just published your first picture to Facebook with Arduino! The only missing step is to make an Arduino sketch to take and send the picture without using SSH</p>

<h3>7. Create the Arduino sketch</h3>

<p>I started from the basic &ldquo;Button&rdquo; example, as I wanted the camera to be triggered by a simple Pushbutton. The circuit is the same of the <a href="http://arduino.cc/en/tutorial/button">button example</a>. Just download <a href="https://github.com/amicojeko/ArduinoUSBCamera/blob/master/ArduinoUSBCamera.ino%20and%20open%20it%20with%20the%20Arduino%20IDE">this arduino sketch</a></p>

<p>you can customise the LED and Button pin if you want</p>

<pre><code>const int buttonPin = 2; // pin for the pushbutton 
const int ledPin =  13; // pin for the status led
</code></pre>

<p>as well as the path for the photos and the Python script (important! this folder must contain the Temboo SDK)</p>

<pre><code>String path = "/mnt/sda1/"; // path of the images and script file 
</code></pre>

<p>The setup is pretty simple, we will use the Bridge library to invoke the Python script, and the Serial library for a very basic debugging</p>

<pre><code>void setup() {
    Bridge.begin(); // The bridge library is used to run processes on the linux side
    pinMode(ledPin, OUTPUT);      
    pinMode(buttonPin, INPUT); 
    Serial.begin(57600); // Serial port for debugging
    filename = "test";
}
</code></pre>

<p>Then, when the pushbutton is pressed, the script generates a new timestamp based filename running the &ldquo;date +%s&rdquo;    command</p>

<pre><code>if (buttonState == LOW) {
    filename = "";
    p.runShellCommand("date +%s"); //generate a timestamp
    while(p.running()); // wait until the command has finished running

    while (p.available()&gt;0) {
        char c = p.read(); //reads the first available character
        filename += c; //and adds it to the filename string
    } 

    filename.trim(); //used to avoid trailing spaces or newline characters
    filename += ".png"; //finally I add the png extension
</code></pre>

<p>This part of the code will take the picture and send it to Facebook using the Bridge library</p>

<pre><code>p.runShellCommand("fswebcam " + path + filename); //let's take the picture
while(p.running());  //waits till the picture is saved
Serial.println("picture taken"); 

p.runShellCommand("python " + path + "sendphoto.py " + path + filename); //sends the picture to facebook 
while(p.running()); 
Serial.println("picture sent"); //last debug message    
</code></pre>

<h3>8. Put everything together</h3>

<p>Ok! Everything is ready! Just upload the code to Arduino, push the button and&hellip; Smile!! You&rsquo;re on Facebook! :)</p>

<p>I really hope that you enjoyed this tutorial, it opens a lot of possibilities! Trap cameras, cameras triggered by doors, by sounds or any kind of sensor&hellip; I&rsquo;d really like to see your projects and share them!</p>

<p>Cheers and happy hacking!</p>

<p><strong>Stefano</strong></p>