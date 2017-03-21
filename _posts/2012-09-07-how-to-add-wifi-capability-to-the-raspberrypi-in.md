---
ID: 332
post_title: >
  How to add WiFi capability to the
  RaspberryPi in 10 easy steps
author: Stefano Guglielmetti
post_date: 2012-09-07 16:36:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2012/09/07/how-to-add-wifi-capability-to-the-raspberrypi-in/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/31062030033/how-to-add-wifi-capability-to-the-raspberrypi-in
tumblr_mikamayhem_id:
  - "31062030033"
---
<p>After a bit of struggling, we finally managed to make the <a href="http://www.micronext.co.uk/index.php?option=com_content&amp;task=view&amp;id=45&amp;Itemid=103">Micronext MN-WD552B</a> USB dongle work on the <a href="http://www.raspberrypi.org">RaspberryPi</a>. I choose this dongle beacuse it&rsquo;s tiny, and it was suggested by <a href="http://www.farnell.com">Farnell&rsquo;s</a> website. Unfortunately the driver for Realtek RTL8188CUS chipset provided with wheezy is not working so there&rsquo;s a little bit of hacking to do.</p>
<p>Here&rsquo;s a step-by-step guide to have a perfectly working WiFi RaspberryPi</p>
<ol><li><a href="http://www.amazon.co.uk/MICRONEXT-MN-WD552B-DONGLE-802-11N-Safety/dp/9800343881">Buy your dongle</a> :) </li>
<li>Download a brand new wheezy. New is always better. Download it from <a href="http://www.raspberrypi.org/downloads">here</a>. I installed the 2012-08-16-wheezy-raspbian.zip version.</li>
<li>Install the brand new wheezy image on a brand new SD Card (or a used one, it really doesn&rsquo;t matter because we are going to format it) following <a href="http://elinux.org/RPi_Easy_SD_Card_Setup">these instructions</a>.</li>
<li>Plug your raspberry to a monitor, a keyboard and an ethernet connection, power it on and login locally or via ssh</li>
<li>Complete the initial setup of the RaspberryPi</li>
<li>Download the fantastic python script made by MrEngman from <a href="http://dl.dropbox.com/u/80256631/install-rtl8188cus.sh">here</a>. </li>
<li>Manually put the old driver into the RPi&rsquo;s modprobe blacklist:
<pre class="bash">sudo vi /etc/modprobe.d/raspi-blacklist.conf </pre>
<ul><li>add this line: blacklist rtl8192cu</li>
<li>save the file</li>
<li>reboot the system: sudo reboot</li>
</ul></li>
<li>Exec the MrEngamn&rsquo;s script:
<pre class="bash">sudo bash install-rtl8188cus.sh</pre>
<ul><li>Follow the instructions and reboot</li>
</ul></li>
<li>Exec the script again, as in step 8, this will make the RPi download all the updates
<ul><li>Follow the instructions and reboot again</li>
</ul></li>
<li>THE END, now you finally have a 100% WiFi capable RaspberryPi!!!</li>
</ol><p>I hope it will help! Bye Guys and happy prototyping!</p>