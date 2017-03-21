---
ID: 1004
post_title: 'There is trouble brewing, or: Openssl issues on Ruby with Homebrew'
author: Cecilia Nardini
post_date: 2017-02-13 15:01:42
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/02/13/there-is-trouble-brewing-or-openssl-issues-on-ruby-with-homebrew/
published: true
---
This post is to describe an issue I've run into using &lt; 2.3 versions of Ruby on OSX with rvm and brew.

Don't even get me started on working with older versions of Ruby -- we have some maintenance projects running in <em>ruby-1.8.3</em>! Even modern Rubies may give you a hard time, however, and this is one of the times. Actually, I am writing this post as a reminder <em>for me</em> because I ran into the same problem <em>twice</em> now, and what is worse, the second time it took me nearly the same amount of fruitless googling to solve it as it took the first. And the solution was the same! So, let us proceed with order.

<!--more-->

After system-wide update of openssl using Homebrew on OSX Yosemite I encountered this problem bundling a Ruby 2.2 project:
<pre><code>
$bundle
Fetching source index from https://rubygems.org/
Retrying fetcher due to error (2/4): Bundler::Fetcher::CertificateFailureError Could not verify the SSL certificate for https://rubygems.org/.
There is a chance you are experiencing a man-in-the-middle attack, but most likely your system doesn't have the CA certificates needed for verification. For information about OpenSSL certificates, see http://bit.ly/ruby-ssl. To connect without using SSL, edit your Gemfile sources and change 'https' to 'http'.
</code></pre>
This is somehow disconcerting if, as it happened with me, everything else (including other versions of Ruby) were working happily and not complaining about CA certificates in any way. So what I did was <code>rvm remove</code>ing and <code>rvm install</code>ing the offending Ruby. But it did not work! This time, as soon as I tried to <code>gem install bundle</code>...
<pre><code>(Gem::RemoteFetcher::FetchError)
SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)
</code></pre>
A word of warning: Googling this will immediately point you to <a href="http://guides.rubygems.org/ssl-certificate-update/">this Rubygems troubleshooting page</a>, <strong>don't go there</strong> as it won't solve your problem. It is actually misleading as the strategy proposed for manually updating the certificate tells you to look inside the <code>/{ruby-version}/rubygems/ssl-cert</code> directory -- as we will see, this is not the right place to look.

The situation is all the more strange since if you try to <em>directly</em> connect to rubygems:
<pre><code>openssl s_client -showcerts -connect rubygems.org:https
</code></pre>
it works flawlessly. This, however, is actually a step towards the light, because it points you to the fact that it is <em>ruby</em> who is fetching the wrong certificate.

You can make sure the diagnosis is right by running:
<pre><code>ruby -ropenssl -e 'puts OpenSSL::X509::DEFAULT_CERT_FILE'
</code></pre>
(even more useful if you can compare it with a different and working Ruby version).
The right path should be <code>/usr/local/etc/openssl/cert.pem</code> (the path to the Homebrew version of openssl) so if you instead find that the output of the previous command is <code>/etc/openssl/cert.pem</code> (path to the system version), that's a sure sign that the problem lies with the Ruby install.

The key here is that the rvm install of some Rubies (in my case the black sheep was Ruby 2.2.1) come with pre-compiled binaries that reference the system version of openssl. So if you have openssl installed with Homebrew, you need to compile the Ruby onto your system on install in order to make sure it will reference the correct version.

So, the solution is simple enough: if the problem persists after a plain <code>rvm install</code>, try installing from source using the <code>--disable-binary<code> flag:</code></code>
<pre><code>rvm install 2.2.1 --disable-binary
</code></pre>
Now you can verify that the path to certificates is the correct one:
<pre><code>$ ruby -ropenssl -e 'puts OpenSSL::X509::DEFAULT_CERT_FILE'
/usr/local/etc/openssl/cert.pem
</code></pre>
And your bundle should now work smoothly (after you <code>gem install bundler</code>, obviously!).
<h3>TL;DR</h3>
If you have certificate problems with Ruby after updating Openssl with Homebrew you should <code>rvm uninstall</code> Ruby and then
<pre><code>rvm install ruby-version-you-need --disable-binary
</code></pre>
to make sure the Homebrew version of Openssl is linked inside the Ruby install