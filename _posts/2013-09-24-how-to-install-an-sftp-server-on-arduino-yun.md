---
ID: 329
post_title: >
  How to install an SFTP Server on Arduino
  Yùn
author: Stefano Guglielmetti
post_date: 2013-09-24 11:35:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/09/24/how-to-install-an-sftp-server-on-arduino-yun/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/62145565365/how-to-install-an-sftp-server-on-arduino-yùn
tumblr_mikamayhem_id:
  - "62145565365"
---
<p>By default, Arduino Yùn&rsquo;s Linino has Dropbear, an SSH server that procides basic SCP but has no SFTP capabilities, so you cannot connect to the Yún with client software such as Cyberduck.</p>
<p>You can solve this basic issue by installing OpenSSH&rsquo;s SFTP Server running this command in the Arduino&rsquo;s SSH console.</p>
<pre>opkg update
    opkg install openssh-sftp-server
</pre>
<p>The server will be installed in <em>/usr/libexec/sftp-server, </em>exactly where Dropbear will look for it.</p>