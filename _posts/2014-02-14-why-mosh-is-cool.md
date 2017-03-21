---
ID: 285
post_title: Why mosh is cool
author: Andrew Coates
post_date: 2014-02-14 14:07:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/14/why-mosh-is-cool/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/76629419681/why-mosh-is-cool
tumblr_mikamayhem_id:
  - "76629419681"
---
<a href="http://mosh.mit.edu/">Mosh</a> is an SSH replacement I discovered after experiencing SSH input lag on my UK based server while I was located thousands of miles away. 
<br /><br />
According to its website, &ldquo;[Mosh is] more robust and responsive, especially over Wi-Fi, cellular, and <b>long-distance links.</b>&rdquo;. <a href="https://www.youtube.com/watch?v=aT3QzhVZArM">A short video I found on YouTube tests this statement.</a> An explanation of how this works <a href="http://mosh.mit.edu/#techinfo">from the site</a>: &ldquo;This is accomplished using a new protocol called the State Synchronization Protocol, for which Mosh is the first application. SSP runs over UDP, synchronizing the state of any object from one host to another. Datagrams are encrypted and authenticated using AES-128 in OCB mode. While SSP takes care of the networking protocol, it is the implementation of the object being synchronized that defines the ultimate semantics of the protocol.&rdquo;
<br /><br />
Aside from this feature, Mosh also includes a roaming feature that will sustain the connection if your IP address changes, earning its namesake of <b>mo</b>bile <b>sh</b>ell.
<br /><h3>Installation</h3>
Mosh runs natively on all Linux/Unix systems and there are <a href="https://gist.github.com/eerohele/2349067">some handy scripts to get it running on cygwin.</a>
<br /><br />
Installation is simple with root privlages as mosh is included in most package managers. See <a href="http://mosh.mit.edu/">here for more info</a>. I had some trouble with the dependencies on my (non sudo) shared server. <a href="https://gist.github.com/andrewgiessel/4486779">This gist script worked great for me.</a> 
<br /><h3>Usage</h3>
Mosh is used in the same fashion as SSH. ie:
<pre>mosh yourserver.com</pre>
To specify a user:
<br /><pre>mosh user@yourserver.com</pre>
To specify a port:
<br /><pre>mosh -p 2222 yourserver.com</pre>

Give it a try!