---
ID: 286
post_title: Start running your own DNS
author: Massimo Ronca
post_date: 2014-02-12 10:44:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/12/start-running-your-own-dns/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/76415968842/start-running-your-own-dns
tumblr_mikamayhem_id:
  - "76415968842"
---
As Web developers, one of the first things we do when starting a new project, is creating a development environment as similar as possible to the final destination of our code, AKA <strong>Production</strong>.
One of the most common requirements is running your apps in their own private domain and one of the most common solution is editing the <code class="bash">/etc/hosts</code> file, every time you need to add a new domain.
It’s no secret that this process can quickly become very tedious.
If you work on OS X, you probably have heard of <a href="http://pow.cx/">Pow</a>, but if you only need the domain resolution or don’t work with Ruby, it is probaly overkill to install a full featured application server just to create some dev domain.
<a href="http://www.mamp.info/en/mamp-pro/index.html?utm_medium=twitter&amp;utm_source=twitterfeed">MAMP Pro</a> offers domain resolution too, but it’s not free.

Luckily, there is another solution: running your own instance of a DNS server.
As over-the-top as it may sound, DNS is generally a very lightweight service, you’ll be making just a few thousands queries a day, not a big deal at all, and it can have a beneficial effect on latency, speeding up requests by caching them locally, without hitting the network.
What you need it’s a copy of <a href="http://www.thekelleys.org.uk/dnsmasq/doc.html">dnsmasq</a> and <em>stop worrying and love the shell</em>.

<!--more-->

First of all install <em>dnsmasq</em> and put it in autostart
<pre><code class="bash">brew install dnsmasq  
sudo cp -v $(brew --prefix dnsmasq)/*.plist /Library/LaunchDaemons
</code></pre>
Configure the dns intance
<pre><code class="bash">cat &lt;&lt;- EOF &gt; $(brew --prefix)/etc/dnsmasq.conf
# IPV4
address=/dev/127.0.0.1
# IPV6 or virtual hosts in Maverick won't work
address=/dev/::1
listen-address=127.0.0.1
EOF
</code></pre>
Then configure the resolvers for all domains and create the brand new one for the <code class="bash">.dev</code> suffixes
<pre><code class="bash"># it might already exists
sudo mkdir -p /etc/resolver

# .dev domains
sudo bash -c 'echo "nameserver 127.0.0.1" &gt; /etc/resolver/dev'

# universal catcher
sudo bash -c 'echo "nameserver 127.0.0.1" &gt; /etc/resolver/catchall'
sudo bash -c 'echo "domain ." &gt;&gt; /etc/resolver/catchall'
</code></pre>
You need to set <code class="bash">localhost</code> as the main DNS server, unfortunately you can’t automatically prepend <code class="bash">localhost</code> to the list of DNS your DHCP assigned to you.
You have to change it manually and put it on top.
<pre><code class="bash">networksetup -setdnsservers Ethernet 127.0.0.1
networksetup -setdnsservers Wi-Fi 127.0.0.1
</code></pre>
If everything’s ok, running <code class="bash">scutil --dns</code> should return something like this
<pre><code class="bash">resolver #8
domain   : dev
nameserver[0] : 127.0.0.1
flags    : Request A records, Request AAAA records
reach    : Reachable,Local Address      
</code></pre>
Now you can start dnsmasq
<pre><code class="bash"># start dnsmasq
sudo launchctl -w load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist

# make some tests
$ ping -c 1 anyhostname.dev
PING anyhostname.dev (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.058 ms 

$ dig anyhostname.dev
;; ANSWER SECTION:
anyhostname.dev.    0   IN  A   127.0.0.1
;; Query time: 3 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)


$ dig google-public-dns-a.google.com @127.0.0.1
;; ANSWER SECTION:
google-public-dns-a.google.com. 25451 IN A  8.8.8.8
;; Query time: 30 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)  

# cache will speedup subsequent DNS queries
$ dig google-public-dns-a.google.com @127.0.0.1
;; Query time: 0 msec
</code></pre>
<strong>Bonus</strong>: if you run Apache on your dev machine, you can easily setup the <em>one virtual host to rule them all</em> through <a href="http://httpd.apache.org/docs/2.4/vhosts/mass.html">mass virtual hosting</a>.

Edit <code class="bash">/etc/apache2/extra/httpd-vhosts.conf</code> (your virtual host configuration could be somewhere else) and add these lines below.

Every <code class="bash">.dev</code> domain will point to a folder inside <code class="bash">~/Sites</code>, with the same name, but without the extension.
For example, <code class="bash">myapp.dev</code> will point to <code class="bash">~/Sites/myapp</code>.
You can of course customize your own environment: you could, for example, create a folder to hold all the virtual hosts created with this tecnique, just like <a href="http://pow.cx/manual.html">Pow does</a>, or if you are a Pow user already, reuse <code class="bash">~/.pow</code> folder as a container.
<pre><code class="apacheconf">&lt;VirtualHost *:80&gt;
    ServerAdmin you@localhost
    ServerName anyhostname.dev
    ServerAlias *.dev

    UseCanonicalName Off
    # %1.0 is the domain without the extension, in this case
    # everything before .dev
    # more info: <a href="http://httpd.apache.org/docs/2.4/mod/mod_vhost_alias.html">http://httpd.apache.org/docs/2.4/mod/mod_vhost_alias.html</a>
    VirtualDocumentRoot /Users/&lt;your_login&gt;/Sites/%1.0
    ErrorLog "/var/log/apache2/dev_hosts_error_log"
    CustomLog "/var/log/apache2/dev_hosts_access_log" common

    &lt;Directory /Users/&lt;your_login&gt;/Sites/*&gt;
        AllowOverride All
    &lt;/Directory&gt;    
&lt;/VirtualHost&gt;
</code></pre>
You can finally reload Apache configuration with <code class="bash">sudo apachectl graceful</code>.