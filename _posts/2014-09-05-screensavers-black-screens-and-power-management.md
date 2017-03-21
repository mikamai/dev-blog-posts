---
ID: 213
post_title: >
  Screensavers, black screens and power
  management
author: Gianluca
post_date: 2014-09-05 14:08:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/05/screensavers-black-screens-and-power-management/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/96702545659/screensavers-black-screens-and-power-management
tumblr_mikamayhem_id:
  - "96702545659"
---
“Oh nice, for today I’m done! Now lets watch a movie before going to sleep!”

This could be considered a justified desire right? After a day of work why not enjoy a movie?

Unfortunately, if you are running a Linux distro you may encouter some problems depending on the Desktop Environment (DE) and the power management system you are relying on.

Those problems can all be summarized in more or less random screensavers activations and/or consequent screens blanking.

<img src="http://68.media.tumblr.com/e48269889f3b8a16bc9a100d28415749/tumblr_inline_nb7rvjEFad1ss63kf.png" alt="image" />

&nbsp;

<!--more-->

Some DE comes with proper shortcuts to disable the system that handle the functionality under discussion.

However, it may happens that the functionality gets handled not entirely by a single application (i.e. system) but by more than one (e.g. screensaver, DE locker application, and eventual power management system)

In this case you may not find a quick solution that works right out of the box but nothing stops you from implementing your personal custom solution!

This is exactly what I did and here’s what I end up for my system (i.e. XFCE 4.10 with LightLocker and the default xfce4-power-manager).

This isn’t really nothing special, just a switch in a few lines of bash! ;)
<pre><code> 
#!/bin/sh 

enabled_flag='DPMS is Enabled' 

if [ "`xset -q | grep "${enabled_flag}"`" ]; then 
  notify-send "Screensaver disabled" 
  xset s off -dpms 
else 
  notify-send "Screensaver enabled" 
  xset s on +dpms 
fi 
    
</code></pre>
As you can see I just check the state of <a href="https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling" target="_blank">DPMS</a> by grepping the result of querying the <a href="http://linux.die.net/man/1/xset" target="_blank">xset</a> utility and then adjust the settings (still through xset) accordingly.

The <code>notify-send "..."</code> is only used to get some visual feedback.

Once saved and made it executable you can simply bind the tiny script to a custom shortcut. Depending on the DE you are using there may be some differences in this last step but it shouldn’t be particular difficult to get it done.

For a more comprehensive explanation of what is going on under the hood take a look at the great Arch doc linked above (<a href="https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling">https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling</a>).

Now you only need to fire up your favorite player and invoke the over mentioned script through the chosen shortcut to watch what you want without being bothered!

Enjoy it! :)