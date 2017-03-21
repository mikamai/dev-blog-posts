---
ID: 308
post_title: 'Intel Galileo: Getting Started with Mac OS X'
author: Stefano Guglielmetti
post_date: 2013-12-06 12:09:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/06/intel-galileo-getting-started-with-mac-os-x/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69163914657/intel-galileo-getting-started-with-mac-os-x
tumblr_mikamayhem_id:
  - "69163914657"
---
<p>A step-by-step tutorial by jeko</p>
<p><strong>IMPORTANT: Connect your intel Galileo to the 5V power supply before any other connection or you will damage the board.</strong></p>
<p>During the Rome Maker Faire, I was lucky to get an Intel Galileo. But when I had to use it, it was suddenly clear that it’s not as straightforward as Arduino. I had to resort to desperate measures and do something that really only a very restricted class of noble people do, I had to read the manual.</p>
<p><img alt="image" src="http://68.media.tumblr.com/fab2cfc694531fba0b35ba4ac4f4fb47/tumblr_inline_mwx6t8gm0J1r8xdak.jpg" /></p>
<p>The getting started guide by Intel really helped, but the problem is that it’s clearly written by engineers, while the Galileo board wants to attract makers, designers, artists and many people that are used to the “tutorial” or “step by step” approach and are not really into manual reading.</p>
<p>I used to be an <em>RTFM</em> fanatic, but I’m getting old, and I have learned to appreciate when people help me, so I’m very happy to contribute and give some help to those in need.</p>
<p>So. You have an Intel Galileo board, a Mac (I assume you have Mac OS X Mavericks installed), and you want to run the very first basic example: the “Blink” sketch from the Arduino examples. Right?</p>
<p><img alt="image" src="http://68.media.tumblr.com/b6af97697fa3330ba15717f3170e2d42/tumblr_inline_mxcf5wdeZ91r8xdak.jpg" /></p>
<p>This is what you have to do, straight and easy.</p>
<p>So, let’s start:</p>
<ol><li>Download the <a href="https://communities.intel.com/community/makers/software/drivers">Galileo software</a> from the Intel website. If you already have the Arduino IDE, don’t overwrite it. Even if the Intel software is based on the Arduino IDE, it is not 100% identical, and it will only manage the Galileo board, so if you want to use them both, simply unzip the Intel software, rename it into “Galileo”, and move it into your Application folder. Don’t rename it into something with spaces, and do not put it into folders with a space in their name, because it won’t work.</li>
<li>Plug the 5V adapter, wait 10 seconds and then connect the USB client (the little usb port next to the ethernet plug) to your Mac’s USB port</li>
<li>You have to wait a minute: it takes a little while for the USB stack to start on the Galileo board, then you can start the Galileo IDE</li>
<li>Go to <strong>Tools &gt; Port</strong> menu and select the <strong>/dev/cuXXX</strong> serial port. If you don’t see the port, close the IDE, wait a minute, and open it again.</li>
<li>Now, you have to update the Galileo’s firmware, so, go on <strong>Help &gt; Firmware Upgrade</strong>. If everything is ok, you should see a message asking for confirmation if the 5V power cable is plugged in. Since you have followed these instructions, it should be, so click yes, do the same on the next dialog and the process should begin. It will take 5-6 minutes. During the process, avoid touching the cables or the board or the IDE. Just sit down and relax.</li>
<li>When the process is complete, the <strong>“Target Firmware upgraded successfully.”</strong> message is displayed. Click <strong>OK</strong> to close the message.</li>
<li>Now choose the blink example from <strong>File &gt; Examples &gt; 01.Basics &gt; Blink</strong></li>
</ol><p>Upload the blink sketch and you’re done! It’s time to celebrate! :)</p>
<p>Jeko</p>
<p>P.S. The Quark processor can become very hot, don’t worry it’s normal.</p>